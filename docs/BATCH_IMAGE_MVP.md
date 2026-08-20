# Batch Image MVP

Sub2API Batch Image MVP 通过统一 API 提供异步 Gemini 图像批量生成，底层由 Redis worker、PostgreSQL 状态和 provider 专用批处理后端支持。

支持的 provider：

- `gemini_api`
- `vertex`

API 用户不会看到 Gemini 文件名、Vertex job 名称、GCS 路径、signed URL、API Key 或 service account 材料。本 MVP 中的下载经由 Sub2API 代理。

## API 路由

```text
POST   /v1/images/batches
GET    /v1/images/batches/{id}
GET    /v1/images/batches/{id}/items
GET    /v1/images/batches/{id}/items/{custom_id}/content
GET    /v1/images/batches/{id}/download
POST   /v1/images/batches/{id}/cancel
DELETE /v1/images/batches/{id}/outputs
```

提交请求：

```json
{
  "model": "gemini-2.5-flash-image",
  "provider": "gemini_api",
  "items": [
    {
      "custom_id": "cover_001",
      "prompt": "A clean product hero image...",
      "output_count": 1,
      "reference_images": [
        {
          "id": "product-front",
          "type": "subject",
          "mime_type": "image/png",
          "data": "<base64 image bytes without a data URL prefix>"
        },
        {
          "id": "style",
          "type": "style",
          "mime_type": "image/jpeg",
          "file_uri": "gs://internal-managed-bucket/batch-image/refs/style.jpg"
        }
      ]
    }
  ],
  "image_size": "1K",
  "response_mime_type": "image/png"
}
```

每个 item 的 `reference_images` 均为可选项。内联 `data` 是由后端解码的 base64 字符串；`file_uri` 仅预留给内部 Google Cloud Storage 引用，必须为 `gs://` URI。每张参考图像必须使用 `image/png`、`image/jpeg` 或 `image/webp` 之一。当前模型限制：

- `gemini-2.5-flash-image` 及其他 Flash Image 别名：每个 item 最多 3 张参考图像。
- `gemini-3-pro-image` 及其他 Pro Image 别名：每个 item 最多 14 张参考图像。
- 每个批处理 job：在所有 item 按 `output_count` 展开后，最多 1,000 个参考图像附件。这是 Sub2API 用于请求大小和成本控制的内部护栏，不是生成图像上限，也不是 Pro Image 的单 item 能力。每个 job 的生成图像上限为 200 张。
- 每个批处理 job：解码后的内联参考图像数据合计最多 128 MB。对于大型批次或重复参考图像，优先使用 `gs://` `file_uri` 引用，或将请求拆分为多个 job。

每个 item 的 `output_count` 为可选，默认 `1`。它表示“将此 prompt 和参考图像集合重复 N 次”，而非依赖 Gemini 在单次上游请求中返回多张图像。后端会将每次重复展开为独立 provider JSONL 行，并添加如 `cover_001_01`、`cover_001_02` 的 custom id 后缀。当前限制：

- 每个 prompt item 最多 4 张输出图像。
- 每个批处理 job 展开后最多 200 张预期输出图像。这是单 job 的硬性生成输出上限；client 和 Codex skill 必须在提交前拆分更大的工作负载。
- 输出图像限制有意与默认 ZIP item 限制一致，以保证按 item 数计算时新提交的 job 总可下载为一个 ZIP。ZIP 字节大小仍单独受 `max_download_bytes_per_request` 限制。

公开批处理响应：

```json
{
  "id": "imgbatch_0123456789abcdef0123456789abcdef",
  "object": "image.batch",
  "status": "queued",
  "model": "gemini-2.5-flash-image",
  "provider": "gemini_api",
  "item_count": 1,
  "success_count": 0,
  "fail_count": 0,
  "estimated_cost": 0.25,
  "actual_cost": null,
  "created_at": 1783123200,
  "submitted_at": 1783123201,
  "settled_at": null
}
```

公开 item 响应：

```json
{
  "object": "list",
  "data": [
    {
      "custom_id": "cover_001",
      "status": "succeeded",
      "mime_type": "image/png",
      "file_extension": "png",
      "image_count": 1,
      "error": null
    }
  ],
  "has_more": false
}
```

## 生命周期

内部生命周期：

```text
created -> uploading -> submitted -> running -> indexing -> settling -> completed
```

终态与清理状态：

```text
failed
cancelled
completed -> output_deleted
```

公开状态映射：

```text
created/uploading/submitted -> queued
running                    -> running
indexing                   -> processing_results
settling                   -> settling
completed                  -> completed
failed                     -> failed
cancelled                  -> cancelled
output_deleted             -> output_deleted
```

`completed -> output_deleted` 发生于手动删除输出或 TTL 清理之后。

## Redis

Redis 用于唤醒、重试、worker 协调、每个 job 的锁和下载限流。PostgreSQL 仍是事实来源。

`batch_image.queue_enabled` 默认 `false`。设置为 `true` 时，应用启动会为 Redis 就绪队列、延迟队列移动器和陈旧 active 恢复启动 `BatchImageWorker` runtime loop。worker 从 Redis 就绪队列预留 job；没有可用 job 时会在该队列阻塞。

Redis 结构：

- 就绪队列：`batch_image.queue_ready_key`
- 延迟队列：`batch_image.queue_delayed_key`
- Active 集合：`batch_image.queue_active_key`
- Inflight 键：`batch_image.inflight_key_prefix`
- 每 job 锁键：`batch_image.lock_key_prefix`
- 队列幂等键：`batch_image.idempotency_key_prefix`
- 由下载限流器管理的下载限流键

worker 应从 Redis 预留任务，而不应作为数据库扫描循环运行。

worker 不执行 DB 扫描轮询。仅在 Redis 队列预留得到具体 batch id 后才读取数据库。

## 计费

MVP 计费规则：

- 提交时可估算成本。
- 结算在结果索引后执行。
- 仅成功图像收费。
- 失败 item 不收费。
- 参考图像作为输入发送至 Gemini，可能产生少量上游 input-token 与临时存储成本。当 `output_count > 1` 时，它们按每个展开输出请求计一次，但公开 MVP 计费模型不增加单独的参考图像附加费。面向用户的预估、冻结和结算金额仍基于输出图像数量和配置的 batch image 单价。
- 结算 request id 为 `batch_image_settlement:{batch_id}`。
- 结算是幂等的；重复执行结算不得重复收费。
- 结算计费失败会在有限次数内重试。超过重试上限后，job 标记失败，并通过幂等释放路径释放剩余冻结金额。

精确的生产定价通过模型定价配置解析，本文不作定义。

## 清理

默认值：

- 终态后的输入保留时间：24 小时。
- 终态后的输出保留时间：72 小时。
- 最大输出保留时间：7 天。
- 清理间隔：30 分钟。
- 清理批大小：100。

手动删除输出：

```text
DELETE /v1/images/batches/{id}/outputs
```

输出清理后，下载会返回带有 `BATCH_IMAGE_OUTPUT_DELETED` 的 `410 Gone`。

清理绝不接受用户提供的 provider 路径。provider 清理必须使用服务端生成的引用和 prefix-safe 删除。

对于托管的 Vertex/GCS batch bucket，请禁用 Cloud Storage soft delete，或谨慎配置生命周期，以避免隐藏的保留存储成本。

## Provider 说明

`gemini_api`：

- 使用 JSONL 文件模式的 Gemini Batch API。
- 支持已配置 API Key 的 Gemini `apikey` 上游账户。
- 结果文件引用仅供内部使用。
- 绝不返回 API Key。
- 管理员配置 Gemini API-key 上游账户后，可通过 Sub2API 选择并提交该 provider。在 2026-07-07 PR 验证中，此路径已验证可选择/调用；但因测试 API Key 没有预付款，未继续完成成功图像生成。

`vertex`：

- 使用带托管 GCS JSONL 的 Vertex `BatchPredictionJob`。
- 支持具有有效 service account JSON 的 Gemini `service_account` 上游账户。
- GCS bucket 和 prefix 由服务端管理。
- Vertex job 名称和 GCS 路径仅供内部使用。
- MVP 中 batch image 输出只能视为 `1K`/默认值。
- 不要承诺 `2K` 或 `4K`。

除非其他 Gemini 账户/登录类型通过同一 provider 流暴露等效的 API-key 或 service-account 凭据，否则当前 batch image provider 不会选择它们。它们不在 2026-07-07 PR 验证范围内。

## Google 官方启用

运营人员必须先在 Google 官方控制台启用 Gemini/Vertex 能力，之后才能为任意分组开启 Sub2API batch image。Sub2API feature flag 和分组开关本身不会创建 Google 端的访问权限。

推荐生产路径：

- 使用已启用 billing 的 Google Cloud 项目。
- 为项目启用相关 Gemini API / Vertex AI APIs。
- 为 Sub2API runtime 使用 service account 或 Application Default Credentials。
- 为 batch image 输入和输出创建一个固定 Cloud Storage bucket，并向 runtime 和 Vertex service agent 授予最小必需的 bucket 权限。
- 使用 project id、location、托管 bucket、provider 账户、模型白名单和定价配置 Sub2API。
- 全局启用 `BATCH_IMAGE_ENABLED`，在目标 Gemini 分组启用图像生成，然后为该分组启用 `allow_batch_image_generation`。非 Gemini 分组不能使用 batch image generation；只有为 Gemini 分组启用图像生成后，管理 UI 才显示 batch image 分组开关。

API-key 路径：

- Google API Key 适用于 Gemini API 开发及受支持的 Gemini 方法。
- Sub2API `x-goog-api-key` 兼容请求头仍要求 Sub2API Key，而不是普通 Google Key。
- 不应将普通 Google API Key 记录为 Vertex service-account batch job 的默认生产凭据。
- 如果管理员配置 Gemini API-key 上游账户，请在 Google 账户具备所需 billing/预付款状态后，用一个低成本 batch image 验证它。没有预付款时，只记录该 provider 可选择/调用以及失败提交会释放冻结金额。

官方参考资料：

- Gemini API key guide: https://ai.google.dev/gemini-api/docs/api-key
- Gemini API Batch API: https://ai.google.dev/gemini-api/docs/batch-api
- Gemini API image generation and batch image notes: https://ai.google.dev/gemini-api/docs/image-generation
- Vertex/Gemini batch inference: https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/capabilities/batch-inference
- Vertex batch predictions API: https://docs.cloud.google.com/gemini-enterprise-agent-platform/reference/models/batch-prediction-api

## 配置

以下键位于 `backend/internal/config/config.go`：

```yaml
batch_image:
  enabled: false
  max_items_per_job_default: 200
  max_items_per_job_trial: 50
  max_output_images_per_job: 200
  max_output_images_per_item: 4
  max_prompt_chars_per_item: 8000
  max_reference_images_per_job: 1000
  max_reference_inline_bytes_per_job: 134217728
  default_response_mime_type: "image/png"
  default_image_size: "1K"

  max_download_items_zip: 200
  max_download_bytes_per_request: 536870912
  max_download_duration_seconds: 600
  max_download_concurrency_per_user: 1

  input_retention_after_terminal_hours: 24
  output_retention_after_terminal_hours: 72
  output_retention_max_days: 7
  cleanup_interval_minutes: 30
  cleanup_batch_size: 100

  queue_enabled: false
  queue_ready_key: "batch_image:queue:ready"
  queue_delayed_key: "batch_image:queue:delayed"
  queue_active_key: "batch_image:queue:active"
  inflight_key_prefix: "batch_image:queue:inflight:"
  lock_key_prefix: "batch_image:queue:lock:"
  idempotency_key_prefix: "batch_image:queue:idem:"
  inflight_ttl_seconds: 604800
  job_lock_ttl_seconds: 300
  default_requeue_delay_seconds: 30
  error_retry_delay_seconds: 60
  lock_conflict_delay_seconds: 5
  stale_active_after_seconds: 600
  delayed_mover_interval_seconds: 5
  recovery_interval_seconds: 300
  delayed_move_limit: 100
  recover_limit: 100

  vertex_enabled: false
  vertex_project_id: ""
  vertex_location: "global"
  vertex_managed_gcs_bucket: ""
  vertex_managed_gcs_prefix: "batch-image/{env}/{batch_id}"
  vertex_input_retention_hours: 24
  vertex_output_retention_hours: 72
  vertex_batch_prediction_base_url: ""
  vertex_gcs_base_url: ""
```

Feature flag 默认关闭。

## 运维检查清单

- 启用 `batch_image.enabled`。
- 配置 Redis。
- 需要 worker 消费队列 job 时，启用 `batch_image.queue_enabled`。
- 配置 provider 账户。
- 使用 Vertex 时配置 Vertex 托管 GCS bucket。
- 确认 bucket 权限正确。
- 禁用或管理 GCS soft delete。
- 配置清理 worker 设置。
- 配置每 job 最大 item 数。
- 配置下载并发。
- 确认计费定价。
- 启用前运行 smoke test。

## 后续优化

- 可选的对象存储下载卸载：将已完成的图像输出持久化到运营人员配置的 GCS、S3 或 R2 等对象存储，再向用户签发短期 signed download link。这样可避免大型图像/ZIP 下载经由 Sub2API 服务器，适合带宽较小的部署。应保持可选，因为它需要额外存储凭据、生命周期清理、signed-URL 过期策略、访问审计以及与输出删除的兼容性。

## 安全检查清单

- 公开响应中不含 provider 引用。
- 不暴露 GCS URI。
- 不暴露 signed URL。
- 不暴露 service account。
- 不暴露 API Key。
- PostgreSQL 中不存储图像字节/base64。
- 日志中不写 base64。
- 状态、item、下载、取消和删除路由都限定 owner。
- 输出删除限定 owner。
- 清理路径仅由服务端生成。

## 测试命令

核心 smoke 与编译命令：

```bash
go test -tags=unit ./internal/service -run 'BatchImage' -count=1
go test -tags=unit ./internal/config ./internal/service ./internal/repository -count=1
go test ./internal/config ./internal/service ./internal/repository ./internal/handler ./internal/server/routes -run '^$'
go test ./... -run '^$'
```

这些命令不应依赖 Docker、testcontainers、Redis、GCP、Gemini、Vertex 或 GCS。

## PR 卫生检查清单

- 除非 maintainer 明确需要，否则不要误提交 `rfcs/batch-image-issue-draft.md`。
- 保持迁移顺序：`159_batch_image_foundation.sql`，然后 `160_batch_image_provider_refs.sql`，之后才是后续迁移。
- 若仓库提交生成代码，请包含生成的 Ent 代码。
- 保持生成的 server 与 wire 文件更新。
- 除非 maintainer 另有要求，否则 feature flag 默认保持关闭。
- 不要提交真实密钥、API Key、service account JSON 或本机路径。
- fixture 应小且为虚构数据；不得使用真实 cloud 引用或凭据。
- 此稳定化 PR 不要新增公开路由、provider、dashboard、队列或计费行为。
