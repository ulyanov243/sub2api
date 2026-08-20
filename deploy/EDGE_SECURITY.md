# 边缘与 HTTP Ingress 安全

Sub2API 支持长连接 SSE 和 WebSocket 请求。请保护请求 Ingress，但不要设置响应 `WriteTimeout`：写入截止时间会中断正常的长生成和流式响应。

## 应用默认值

- `server.max_header_bytes: 65536` 将 HTTP/1 请求头限制为 64 KiB；Go 会将其映射为对应的 HTTP/2 header-list 限制。
- `server.read_header_timeout: 10` 限制慢请求头攻击；它不限制请求处理或响应流式传输。
- `server.max_request_body_size: 268435456` 是绝对的 256 MiB 安全网。
- `gateway.max_body_size: 268435456` 仍可用于多模态、Gemini、图像、视频和批量图像端点。
- `gateway.text_max_body_size: 33554432` 将已知的纯文本 `/embeddings` 和 `/alpha/search` 端点限制为 32 MiB。
- H2C 默认每个连接 50 个并发流、2 MiB 连接上传窗口以及 512 KiB 流上传窗口。
- 无效凭据滥用按受信任客户端 IP（IPv6 `/64`）在进程内限流：60 秒内失败 120 次后封禁 60 秒。这是单实例安全网；多实例限流仍应由负载均衡器、CDN 或 WAF 完成。

不要添加全应用统一的请求 semaphore：一个 SSE 请求可以合理地占用它数分钟。请在边缘层施加连接和未认证请求控制；已认证用户/API Key 的并发量仍由应用负责。

## 受信任的客户端 IP

为升级兼容性，`security.trust_forwarded_ip_for_api_key_acl` 默认开启。启用期间，原始转发请求头会接管日志和安全敏感路径的客户端 IP 解析。`security.forwarded_client_ip_headers` 中的自定义请求头按配置顺序检查，优先于内置的 `CF-Connecting-IP`、`X-Real-IP` 和 `X-Forwarded-For` 回退项。请求头名称不区分大小写，在加载时规范化、去重，并限制为 16 个唯一且有效的 HTTP 字段名。请求头值必须包含 IP 字面量；支持逗号分隔值，会跳过无效项，并优先使用公网地址而非私网回退地址。

该列表可通过 YAML 或逗号分隔的环境变量 `SECURITY_FORWARDED_CLIENT_IP_HEADERS` 提供；明确设置为空的环境变量值会清除 YAML 值。也可在管理端安全设置中编辑，并会在无需重启的情况下更新运行时设置。一个请求会同时快照开关和请求头列表，因此不会混用新旧设置。关闭开关时会完全忽略自定义请求头。此时 Gin 的 `server.trusted_proxies` 链具有权威性：只配置直接连接到 Sub2API 的精确 CIDR/IP 地址。明确的空列表表示不信任任何转发客户端 IP。

首次升级到此模式时，仅在未明确配置 `server.trusted_proxies` 的情况下，才将旧版 `false` 值改为 `true`；明确的代理策略会保持安全模式。新安装会在数据库初始化时持久化配置的自定义请求头列表。现有安装会从 YAML 配置回填缺失的数据库值。隐藏的迁移标记可防止之后的管理员改动被覆盖。若无法读取设置或持久化的自定义请求头列表格式错误，进程将故障关闭为不带自定义请求头的受信任代理模式。若迁移写入失败，计算出的模式仍会在当前进程中生效，且启动时记录警告。

兼容性接管会接受转发请求头而不验证直接对端，包括任何配置的自定义请求头。启用时请保护源站免受直接访问。CDN 部署必须通过防火墙限制源站，使其仅能由 CDN 或负载均衡器访问；该代理必须覆盖每个受信任客户端 IP 请求头，而不是附加不受信任的客户端值。

同主机代理示例：

```yaml
server:
  trusted_proxies:
    - 127.0.0.1/32
    - ::1/128
```

## Nginx 基线配置

在 `http` 块中定义共享 zone。应根据测得的合法流量调整速率；以下值是保守的起点，而非通用容量目标。

```nginx
limit_conn_zone $binary_remote_addr zone=sub2api_conn:20m;
limit_req_zone  $binary_remote_addr zone=sub2api_auth:20m rate=5r/s;
limit_req_zone  $binary_remote_addr zone=sub2api_api:40m rate=30r/s;
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    client_header_timeout 10s;
    client_max_body_size 256m;
    large_client_header_buffers 4 16k;
    limit_conn sub2api_conn 40;

    location ~ ^/(auth|api/auth)/ {
        limit_req zone=sub2api_auth burst=10 nodelay;
        proxy_pass http://127.0.0.1:8080;
    }

    location ~ ^/(v1/)?(embeddings|alpha/search)$ {
        client_max_body_size 32m;
        limit_req zone=sub2api_api burst=60 nodelay;
        proxy_pass http://127.0.0.1:8080;
    }

    location / {
        limit_req zone=sub2api_api burst=60 nodelay;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_read_timeout 1800s;
        proxy_send_timeout 1800s;
        proxy_pass http://127.0.0.1:8080;
    }
}
```

若在 `http` 块中启用了 Nginx gzip，请将 `text/event-stream` 排除在 `gzip_types` 外，并且不要为 Sub2API 使用 `gzip_types *`。上面的 `proxy_buffering off` 会禁止代理缓冲，但不会关闭 gzip 响应过滤器。请为普通响应使用显式列表：

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript application/xml image/svg+xml;
```

如果共享的全局配置无法按内容类型排除 SSE，请在服务流式 API 路由的 location 中设置 `gzip off;`。这样 Web UI 和静态资源仍可使用 gzip。

除非 Nginx real-IP 处理已限制为明确的受信任代理 CIDR，否则不要使用传入的 `$http_x_forwarded_for` 值。

## Caddy 与 CDN

随附的 `deploy/Caddyfile` 设置了 64 KiB 请求头限制、10 秒请求头超时、256 MiB 绝对请求体限制，并会使用 TCP 对端覆盖转发地址。因此它是直连 Caddy 的基线配置。不要在 CDN 后原样使用其 `{remote_host}` 转发行：所有客户端都会被归因为 CDN 出口地址，使拒绝聚合和无效认证限流合并到无关用户。

随附的 Caddy 配置未设置 `flush_interval`，以便 Caddy 自动 flush `text/event-stream` 响应，同时仍将客户端取消传递给上游。不要全局设置它：正值会增加流式延迟，而 Caddy 2.6.2 的特殊 `-1` 模式还会导致客户端断开后反向代理请求继续运行。该配置为压缩使用显式响应内容类型列表。不要将其替换为 `text/*` 或简写 `encode gzip zstd`：二者都会匹配 `text/event-stream`，并可能将 SSE 缓冲到响应结束。应保持流式响应未压缩，同时保留 Web UI、JSON 和静态资源的压缩。

CDN 部署时，首先通过防火墙限制源站，使其仅允许当前 CDN 出口 CIDR 连接。然后将这些精确范围配置为 Caddy 受信任代理，并从 Caddy 解析出的 `{client_ip}` 派生上游请求头。例如：

```caddyfile
{
	servers {
		trusted_proxies static 192.0.2.0/24 2001:db8:1234::/48
		trusted_proxies_strict
		client_ip_headers CF-Connecting-IP X-Forwarded-For
	}
}

api.example.com {
	reverse_proxy 127.0.0.1:8080 {
		header_up X-Real-IP {client_ip}
		header_up X-Forwarded-For {client_ip}
	}
}
```

请将文档中的范围替换为 CDN 公开的、自动维护的出口范围。仅因直接源站访问被阻止且 Caddy 仅信任这些 TCP 对端，`CF-Connecting-IP` 在此才是安全的。请将 Sub2API `server.trusted_proxies` 配置为 Caddy 地址/私有子网，以使应用仅接受 Caddy 重写的请求头。

Caddy core 不提供通用请求限流器；请使用受信任的 CDN/WAF、受支持的限流模块或主机防火墙控制。

在 CDN/WAF 上，应在流量到达源站前配置连接限制、请求头/请求体限制、Bot 挑战和按 IP/ASN 的速率限制。仅允许 CDN 出口 CIDR 或私有负载均衡器访问源站 Ingress。请勿将应用端口暴露在公共互联网。

## DDoS 边界

应用检查可在连接到达 Go 后降低放大效应，但无法吸收大流量攻击、TLS 洪泛、带宽饱和或大量分布式来源。这些威胁需要上游网络容量、CDN/WAF 过滤、云服务商防火墙规则和源站隔离。在拒绝风暴期间，应避免高基数指标或按请求写入的数据库安全日志。
