# 组合分组

组合分组是 API Key 的管理端路由层：它会根据请求的模型选择具体 provider，而不是将 Key 绑定到单一 provider 分组。它同时支持内置模型检测，以及供管理员配置公共模型别名的模型路由注册表。

## 支持的 Provider

组合分组可以路由到以下具体账户平台：

- Anthropic
- Gemini
- OpenAI
- Antigravity
- Grok

选定的具体平台用于账户选择、用户平台配额检查、用量后计费、运维错误的平台归因、通道映射/定价查询以及平台用量报表。

## 路由注册表

管理员可通过分组列表的 `Routes` 操作或 admin API 为组合分组配置路由：

- `GET /api/v1/admin/groups/:id/composite-routes`
- `POST /api/v1/admin/groups/:id/composite-routes`
- `PUT /api/v1/admin/groups/:id/composite-routes/:route_id`
- `DELETE /api/v1/admin/groups/:id/composite-routes/:route_id`
- `POST /api/v1/admin/groups/:id/composite-routes/preview`

每条路由属于一个组合分组，并包含：

- `public_model`：client 发送的模型标识符。
- `match_type`：`exact` 或 `prefix`。
- `target_platform`：具体 provider 平台。
- `upstream_model`：发送到上游的模型标识符；省略时复用公共模型。
- `endpoint`：`any`、`messages`、`count_tokens`、`responses`、`chat_completions`、`embeddings`、`images` 或 `gemini`。
- `priority`：匹配特异度相同时，数值更低者优先。
- `enabled`：禁用的路由会被运行时解析忽略，但管理员仍可查看。

解析顺序是先显式路由，再内置检测。多条显式路由匹配时，exact 匹配优先于 prefix 匹配，端点专用路由优先于 `any`，较长前缀优先于较短前缀，之后比较较低的 `priority`，最后比较较低的路由 ID。

对于 JSON body 端点，网关会在派发前将请求的 `model` 字段重写为路由的 `upstream_model`。对于 `/v1beta/models/{model}:generateContent` 等 Gemini 原生路径，网关解析 `{model}`，handler 转发解析后的上游模型。

Codex Alpha Search 和 Live 请求使用 `responses` 路由域。Live 请求从 `session.model` 解析模型（包括 multipart `session` payload），并在派发前应用配置的 `upstream_model`。Codex 模型 manifest 请求会在组合分组内复用现有 OpenAI 账户选择与故障转移路径。

## 内置检测

组合路由会检测常见公共模型 ID 及带 provider 前缀的 ID：

- `claude-*` 和 `anthropic/claude-*` 路由到 Anthropic。
- `gemini-*` 和 `google/gemini-*` 路由到 Gemini。
- `gpt-*`、`o*`、`codex-*`、`text-embedding-*`、`dall-e-*` 和 `openai/*` 路由到 OpenAI。
- `grok-*` 和 `xai/grok-*` 路由到 Grok。

未知或存在歧义的模型名称会故障关闭并返回 client 错误，不会猜测 provider。

## 管理工作流

- 管理员可以创建 platform 为 `composite` 的分组。
- 管理员可以新增、编辑、删除和预览组合模型路由。
- 组合分组可以从具体 provider 分组复制账户。
- 可在创建/编辑账户及批量账户工作流中，将具体 provider 账户直接分配给组合分组。
- 当组合分组的 `subscription_type` 为 `subscription` 时，订阅支付计划可以绑定该组合分组。计划授予访问组合分组的权限；每个请求仍会按解析出的具体 provider 平台计费和检查配额。
- 通道配置会在具体 provider 分区中展示组合分组。通道 `group_ids` payload 仍为扁平结构；provider 专用的模型映射和定价仍以具体平台为键。

## Bucket 2 配置：OpenAI + Claude + Gemini + Grok

当一个面向客户的计划需要暴露跨 OpenAI、Claude、Gemini 和 Grok 的模型别名，且无需为每个 provider 分发单独 Key 时，请使用一个组合订阅分组。

1. 为上游账户池创建具体 provider 分组，例如 `OpenAI Paid`、`Claude Paid`、`Gemini Paid` 和 `Grok Paid`。
2. 创建 `composite` 分组，并将 `subscription_type` 设为 `subscription`。
3. 将 provider 账户直接分配给组合分组，或在创建分组时从具体 provider 分组复制账户。
4. 为不应依赖内置模型检测的公共别名添加显式路由：

   | 公共模型 | 端点 | 目标平台 | 上游模型 |
   | --- | --- | --- | --- |
   | `all/gpt-5` | `responses` | `openai` | `gpt-5` |
   | `all/claude-sonnet` | `messages` | `anthropic` | `claude-sonnet-4-6` |
   | `all/gemini-pro` | `gemini` | `gemini` | `gemini-2.5-pro` |
   | `all/grok` | `responses` | `grok` | `grok-4.3` |

5. 在每条路由指定的具体平台下配置通道定价和模型映射。组合路由不会创建定价记录。
6. 为组合分组创建订阅支付计划。

同一组合分组也可以依赖内置检测处理 `gpt-*`、`claude-*`、`gemini-*` 和 `grok-*` 等标准模型名称。建议为捆绑计划别名配置显式路由，因为管理员可在 UI 中审核端点、provider 和上游模型归因。

## 限制

组合路由选择具体 provider 和上游模型；它们本身不会创建合成的模型元数据、定价或上游能力记录。请持续为路由目标的具体 provider 平台配置通道定价/模型映射。

此 PR 有意不实现以下功能：

- 在多个 provider 之间为同一抽象任务进行 AUTO 智能路由。
- 不使用组合分组而将 API Key 直接绑定到多个现有分组。
- 与协议无关的 provider 解耦或 LiteLLM 风格的适配器重写。
