---
name: sub2api-admin
description: 管理 Sub2API 的 admin API，包括账户、兑换码、分组、代理、错误透传规则、TLS 指纹 profile、导入、导出、批量更新和原始管理员 API 调用。用户提到 Sub2API、admin API Key、账户管理、兑换码管理、充值码、邀请码、批量账户导入/导出、保留或删除账户、刷新账户、清除错误、CRS 同步，或通过 admin API 管理 Sub2API Backend 设置时使用此 skill。
---

# Sub2API Admin

请使用随附 CLI，而不是临时编写 `curl`。示例应在此 skill 目录中运行。

```bash
export SUB2API_BASE_URL='https://your-sub2api-host'
export SUB2API_ADMIN_API_KEY='<admin api key>'
# Or, when the deployment uses admin JWT login instead of an admin API key:
# export SUB2API_JWT='<admin access_token>'
node scripts/sub2api-admin.js accounts list
```

所有命令和 payload 示例请参阅 [references/admin-cli.md](references/admin-cli.md)。

## 工作流

1. 复用环境中的 `SUB2API_BASE_URL` 以及 `SUB2API_ADMIN_API_KEY` 或 `SUB2API_JWT`。
2. 先运行只读命令：`accounts list`、`accounts get <id>`、`groups all` 或 `proxies all`。
3. 在破坏性或批量写入前，打印目标账户名称和 ID。
4. 仅在目标集合明确后执行写入命令。
5. 执行后续只读命令验证结果。

## 常用命令

```bash
node scripts/sub2api-admin.js accounts list --page-size 20
node scripts/sub2api-admin.js accounts get 40
node scripts/sub2api-admin.js accounts usage 40
node scripts/sub2api-admin.js accounts set-schedulable 40 true
node scripts/sub2api-admin.js accounts bulk-update --ids 40,39 --json '{"concurrency":10}'
node scripts/sub2api-admin.js redeem-codes list --page-size 20
node scripts/sub2api-admin.js redeem-codes generate --json '{"count":1,"type":"balance","value":10}' --idempotency-key redeem-$(date +%s)
node scripts/sub2api-admin.js redeem-codes create-and-redeem --json '{"code":"order_123","type":"balance","value":10,"user_id":123}' --idempotency-key order-123
node scripts/sub2api-admin.js error-rules list
node scripts/sub2api-admin.js tls-profiles list
```

## 安全说明

- 认证优先使用 `SUB2API_ADMIN_API_KEY` 中的 `x-api-key`，其次回退到 `SUB2API_JWT` 中的 `Authorization: Bearer <jwt>`。
- API 返回 `INVALID_ADMIN_KEY` 时，请让用户重新生成 admin API Key。使用 JWT 时，以管理员用户登录并从 `POST /api/v1/auth/login` 响应中复制 `access_token`。
- `accounts export` 包含凭据和 token。优先使用 `--file`，避免在聊天中打印导出内容。
- 兑换码创建/兑换命令应在支付或充值工作流中使用 `--idempotency-key`。
- 对于不确定或新增加的 Backend API，先做只读检查，再使用 `api <METHOD> <admin-path>`。
