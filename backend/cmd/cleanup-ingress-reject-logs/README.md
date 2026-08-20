# Ingress 拒绝日志清理

此维护命令从 `ops_error_logs` 中删除历史准入拒绝记录，不会匹配无关的身份验证错误或上游错误。除非指定 `--execute`，否则只会试运行；并且始终要求提供明确的 RFC3339 截止时间。

```sh
go run ./cmd/cleanup-ingress-reject-logs --before 2026-07-17T00:00:00Z
go run ./cmd/cleanup-ingress-reject-logs --before 2026-07-17T00:00:00Z --execute
```

仅在所有应用实例完成升级后才可执行带 `--execute` 的命令，以避免旧实例在所选截止时间之前新增 Ingress 拒绝记录。分类器会保留 `USER_NOT_FOUND`、数据库错误、配额/计费错误和上游失败等不变量错误。

确认发布和清理结果后，请在维护窗口执行 `backend/scripts/finalize-ingress-reject-cleanup.sql`，以删除已废弃的明文密钥审计表和归因字段。
