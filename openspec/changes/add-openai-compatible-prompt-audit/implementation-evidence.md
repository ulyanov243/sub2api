# Prompt Audit 实现证据

此文件记录可复现的实施期证据。不包含提示词正文、Guard 凭据、Authorization 值或 Redis payload。

## 2026-07-16 — 源码冻结与目标基线

### 冻结源码恢复

- 基础提交：`7a50378851a80650cb0c086260b23abeb3469e6b`
- 冻结清单：`source-freeze/MANIFEST.md`
- 清单 SHA-256：`badab312bf6af4d2c77857a9400381f4da4fbf45722d9f4a6df23bc7005273b6`
- 恢复结果：已跟踪补丁和未跟踪归档已恢复到分离 worktree；`git diff --check` 通过。
- `go test ./internal/service/promptaudit -count=1`：通过。
- `go test ./internal/router ./internal/relay ./internal/gatewayadapter/transport -run 'PromptGuard|PromptAudit|ConcurrencyOrder' -count=1`：通过。

### 目标变更前基线

- `cd backend && go test ./internal/service -run ContentModeration -count=1`：通过（`1.138s`）。
- `pnpm --dir frontend exec vitest run src/views/admin/__tests__/RiskControlView.spec.ts src/router/__tests__/feature-access.spec.ts`：通过（2 个文件，9 个测试）。

### 评审切片

实现被划分为可独立评审的切片，且不改变最终范围：

1. 数据和核心契约。
2. 异步审计引擎。
3. 管理 API 和控制台。
4. 协调器和同步 Guard。
5. 可观测性、验证、发布和部署证据。

该功能在整个实施过程中保持默认关闭。只有满足 `verification.md` 中已签署的发布门槛后，才允许在生产环境启用阻断。
