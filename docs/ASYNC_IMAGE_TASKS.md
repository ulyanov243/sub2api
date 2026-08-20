# 异步图像任务

异步图像任务允许客户端提交长时间运行的 OpenAI 兼容图像请求，而无需保持一个 HTTP 连接打开。它可避免 Cloudflare 524 等代理/CDN 响应超时，同时保留既有的图像路由、计费、审核、并发和故障转移行为。

## 端点

已认证网关同时提供 `/v1` 路径及其既有的无前缀别名：

```text
POST /v1/images/generations/async
POST /v1/images/edits/async
GET  /v1/images/tasks/{task_id}
```

别名为 `/images/generations/async`、`/images/edits/async` 和 `/images/tasks/{task_id}`。

仅支持 OpenAI 和 Grok 分组。请求使用与对应同步端点相同的 JSON 或 multipart payload。流式图像请求会被拒绝，因为轮询任务只会返回一个最终 JSON 结果。

## 启用功能（对象存储）

异步图像任务**默认关闭**，并受对象存储配置控制。开关关闭或 S3 凭据不完整时，异步端点返回 `404`，且不会创建任务或向 Redis 写入数据。这是有意的：若不卸载结果，大型 `b64_json` 结果（每项数 MB，例如 `gpt-image-1`）会累积在 Redis 中并耗尽内存。

### 通过管理 UI（推荐）

**Admin → Backup → Async image object storage。** 保存表单后立即生效：下一个请求会重建对象存储 client，无需重启容器。

异步图像存储和数据库备份共用一个 S3 client，因此表单默认**复用备份 S3 配置**：它借用上方已配置的 endpoint、region 和凭据，只保留自己的 bucket 和 prefix，这样备份仍存于 `backups/`，图像存于 `images/`。留空 bucket 则同样使用备份 bucket。取消勾选可将图像指向完全独立的账户。

启用该门槛时，保存操作要求 step-up 2FA，原因与备份 S3 表单相同：更改目标会将生成内容重定向到其他账户。

关闭开关会停止新提交，但已接受的任务仍可轮询，因此不会遗留进行中的任务。

### 通过配置文件

管理端设置优先。若从未在管理端保存过，系统将改用 `config.yaml` 中的 `image_storage` 块，因此在管理 UI 出现前已启用该功能的部署可继续正常工作。

在 `config.yaml` 中配置 S3 兼容对象存储（AWS S3、Cloudflare R2、Aliyun OSS、MinIO 等；所有键都接受 `IMAGE_STORAGE_*` 环境变量覆盖）：

```yaml
image_storage:
  enabled: true
  endpoint: "https://<account_id>.r2.cloudflarestorage.com"  # AWS 官方可留空
  region: "auto"
  bucket: "my-images"
  access_key_id: "..."
  secret_access_key: "..."
  prefix: "images/"
  force_path_style: false          # MinIO/path-style buckets set true
  public_base_url: ""              # set to return public_base_url/key直链; empty → presigned URL
  presign_expiry_hours: 24         # presigned link TTL when public_base_url is empty
  max_download_bytes: 33554432     # cap when re-hosting an upstream image URL (32MB)
```

任务完成后，每个生成的图像会上传到 bucket，结果会重写为紧凑形式：`data[].url` 指向已存储对象（永久的 `public_base_url/key` 链接或有时限的 presigned URL），并移除 `b64_json`。Redis 中只存储这份较小的 JSON。上传失败时，任务会标记为 `failed`，而不会持久化原始 base64。

如需支持 S3 兼容 client 以外的厂商，请实现 `service.ImageStorage` 接口（`Save(ctx, key, contentType, data) (url, error)`），并以该实现替代 S3 实现。

### 故障排除：启用后端点返回 404

`404 async image tasks are not enabled` 表示 `image_storage` 未解析为完整配置，因此功能保持关闭。无论是否启用路由都会存在；404 来自 handler，而非未注册路径，所以很容易被误判为构建缺失。

请在启动日志中查找：

```text
WARN image_storage.enabled is true but object storage is not fully configured; async image tasks are disabled  missing_keys=[...]
```

`missing_keys` 会准确列出加载配置时为空的凭据。

注意，**v0.1.161 之前**的版本在仅通过环境变量提供 `IMAGE_STORAGE_ENDPOINT`、`_BUCKET`、`_ACCESS_KEY_ID`、`_SECRET_ACCESS_KEY` 和 `_PUBLIC_BASE_URL` 时会静默丢弃这些值：这些键没有注册默认值，而 viper 无法读取它原本不知道的键的环境变量。因此，纯粹由 `environment:` 驱动的部署（`deploy/docker-compose.yml` 默认就是如此）会报告 `enabled: true`，但凭据为空，并使每个异步调用返回 404。受影响版本的解决方法是同时在 `/app/data/config.yaml` 中放入 `image_storage` 块（从 `deploy/config.example.yaml` 复制）；一旦这些键存在于文件中，环境变量覆盖就会正常生效。

另外两种与存储无关的 404 原因是：API Key 所在分组必须为 **OpenAI 或 Grok** 平台（其他平台，或没有分组的 Key，都会返回 `Images API is not supported for this platform`）；一个任务只能用**提交它的同一 API Key**轮询——同一用户的另一把 Key 轮询时，按设计会返回 `image task not found`。

## 提交任务

```bash
curl -i https://api.example.com/v1/images/generations/async \
  -H 'Authorization: Bearer sk-...' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "gpt-image-1",
    "prompt": "A lighthouse during a winter storm",
    "size": "1536x1024"
  }'
```

服务端会将初始任务存入 Redis，并返回 `202 Accepted`：

```json
{
  "id": "imgtask_0123456789abcdef",
  "task_id": "imgtask_0123456789abcdef",
  "object": "image.generation.task",
  "status": "processing",
  "created_at": 1784092800,
  "expires_at": 1784179200,
  "poll_url": "/v1/images/tasks/imgtask_0123456789abcdef"
}
```

`Location` 包含轮询路径，`Retry-After: 3` 给出建议的轮询间隔。

## 轮询任务

请使用提交该任务的同一 API Key：

```bash
curl https://api.example.com/v1/images/tasks/imgtask_0123456789abcdef \
  -H 'Authorization: Bearer sk-...'
```

任务进行中时：

```json
{
  "id": "imgtask_0123456789abcdef",
  "task_id": "imgtask_0123456789abcdef",
  "object": "image.generation.task",
  "status": "processing",
  "created_at": 1784092800,
  "expires_at": 1784179200
}
```

成功时，`result` 与同步图像 API 响应体一致，但每张图像都已卸载到对象存储：`data[].url` 指向已存储对象，且 `b64_json` 被移除（因此无论上游返回 URL 还是 base64，最终都会成为紧凑的存储链接）：

```json
{
  "id": "imgtask_0123456789abcdef",
  "task_id": "imgtask_0123456789abcdef",
  "object": "image.generation.task",
  "status": "completed",
  "http_status": 200,
  "image_url": "https://...",
  "result": {
    "created": 1784092923,
    "data": [{"url": "https://..."}]
  },
  "created_at": 1784092800,
  "completed_at": 1784092923,
  "expires_at": 1784179323
}
```

对于 URL 响应，`image_url` 会镜像第一项 `data[].url`，便于简单 client 使用。失败时，任务进入 `failed` 状态，并在可用时暴露原始 OpenAI 兼容错误对象：

```json
{
  "id": "imgtask_0123456789abcdef",
  "task_id": "imgtask_0123456789abcdef",
  "object": "image.generation.task",
  "status": "failed",
  "http_status": 502,
  "error": {
    "type": "api_error",
    "message": "Upstream request failed"
  },
  "created_at": 1784092800,
  "completed_at": 1784092923,
  "expires_at": 1784179323
}
```

所有提交和轮询响应都包含 `Cache-Control: no-store`，以防 CDN 缓存 `processing` 状态。任务和结果在其最后一次状态更新 24 小时后过期。一个任务最长执行 30 分钟。

任务所有权同时受用户和 API Key 限定。未知任务 ID 和归属于另一把 Key 的 ID 都返回 `404`，以避免泄露任务是否存在。即使已完成的生成消耗了该 Key 的剩余额度，仍可轮询；常规认证、禁用 Key、用户、IP 和分组检查依然适用。
