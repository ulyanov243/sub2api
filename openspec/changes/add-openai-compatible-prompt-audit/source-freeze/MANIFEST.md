# AICodex Prompt Audit 源码冻结清单

- 冻结时间：`2026-07-16 20:21:19 CST (+0800)`
- 源仓库：`/Users/mt/code/mt-ai/aicodex/aicodex-api`
- 采集时的源分支：`yjb`
- 基础提交：`7a50378851a80650cb0c086260b23abeb3469e6b`
- 冻结方式：不可变的已跟踪 patch 加未跟踪 tar archive
- 已恢复的验证 worktree：从基础提交分离，再仅使用下方两项 artifact 填充

## Artifact

| Artifact | 大小 | SHA-256 |
| --- | ---: | --- |
| `aicodex-prompt-audit-tracked.patch` | 124674 bytes | `f751a13cce3f3a73cd60cae3aececcef6e1e76dcec8c551a7a4747f032234d2b` |
| `aicodex-prompt-audit-untracked.tar.gz` | 39342 bytes | `1536e2781703b7620e26f2d08b249431fa5846ad9e32b2e8b0d547c3fa3b3632` |

已跟踪 patch 包含 38 个文件、1306 处插入和 227 处删除。它使用 `git apply` 应用于上述基础提交。

## 未跟踪 archive 条目

- `ai-gateway/internal/gatewaycore/prompt_guard.go`
- `ai-gateway/internal/relay/ws_responses_prompt_guard_order_test.go`
- `ai-gateway/internal/router/prompt_guard_order_test.go`
- `ai-gateway/internal/service/promptaudit/outbound_security.go`
- `ai-gateway/internal/service/promptaudit/synchronous_guard.go`
- `ai-gateway/internal/service/promptaudit/synchronous_guard_test.go`
- `openspec/changes/add-prompt-audit-synchronous-blocking/.openspec.yaml`
- `openspec/changes/add-prompt-audit-synchronous-blocking/design.md`
- `openspec/changes/add-prompt-audit-synchronous-blocking/proposal.md`
- `openspec/changes/add-prompt-audit-synchronous-blocking/specs/prompt-input-audit/spec.md`
- `openspec/changes/add-prompt-audit-synchronous-blocking/specs/prompt-input-guard/spec.md`
- `openspec/changes/add-prompt-audit-synchronous-blocking/tasks.md`

## 恢复与验证结果

Artifact 已恢复到 `/tmp/aicodex-prompt-audit-freeze-7a503788`，即位于基础提交的分离 worktree。`git diff --check` 通过。

以下命令在恢复后的副本中通过：

```text
cd ai-gateway
go test ./internal/service/promptaudit -count=1
ok github.com/mt21625457/aicodex/internal/service/promptaudit 2.081s

go test ./internal/router ./internal/relay ./internal/gatewayadapter/transport \
  -run 'PromptGuard|PromptAudit|ConcurrencyOrder' -count=1
ok github.com/mt21625457/aicodex/internal/router 1.184s
ok github.com/mt21625457/aicodex/internal/relay 2.201s
ok github.com/mt21625457/aicodex/internal/gatewayadapter/transport 3.233s
```

源 worktree 保持未改动。若该冻结实现与目标架构不同，则目标 OpenSpec spec 仍具有权威性。
