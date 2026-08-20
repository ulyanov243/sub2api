# Channel Monitor V2 安全默认值与平缓回填

**日期：** 2026-08-08
**状态：** 已批准实施
**分支：** `fix/channel-monitor-v2-ops-ui-blockers`（基于 channel-monitor-v2）

## 问题

1. **H2** — 迁移 195 与代码默认值将 `channel_monitor_mode=v2`，因此所有没有现有键的升级都会自动启用被动 V2、停止 V1 探测，并可能让用户看到空的 V2 视图。
2. **M1** — 首次启用的引导会强制使用 5 秒 tick 和约 24 小时的 `RecomputeRange` 分块，直到 `now-30d`，从而重度占用主数据库。
3. 错误去重 SQL 匹配 `request_id` 时未限制 `created_at`，会通过 `candidate_ids` 扫描 `ops_error_logs` 历史记录。

## 决策

| 主题 | 决策 |
|-------|----------|
| 默认模式 | **v1**（保留主动探测）。V2 必须显式启用。 |
| 已有的 `channel_monitor_mode=v2` 行 | **不变**（`ON CONFLICT DO NOTHING` / 不强制改写）。 |
| 切换至 v2 | V1 runner 的 `fire()` 与 `RunCheck` 要求 `ActiveProbesAllowed()` → 探测停止。 |
| 回填 | 统一硬门控 + 自适应软门控（规则相同，观测负载不同）。 |
| 可选 profile 配置项 | **不在本次变更中**（YAGNI）。 |

## 模式默认值

- 迁移 `195_channel_monitor_mode.sql`：键不存在时插入 `'v1'`。
- 若 195 已以旧校验和执行，添加迁移校验和兼容性（不强制将已应用的数据库改为 v1）。
- 代码：`defaultChannelMonitorMode = v1`；空值/无效值规范化为 v1。
- 前端 Settings 表单默认值及公开功能标志回退：缺失/无效时均为 v1。
- V1 `RunCheck` 路径上 Nil settings 仍保持**故障关闭**（不探测）以确保测试安全，这与产品默认值无关。

## 平缓回填（低资源阶段）

### 硬门控（所有服务器）

- Tick 间隔 = `refresh_interval`（60 或 300 秒）。**不使用 5 秒引导覆盖。**
- 每个 tick：（1）近期重叠（约 10 分钟），然后（2）在产品引导未完成/保留期遍历进行时，**至多一个**历史分块。
- 单 leader 锁；单次短事务预算（约 55 秒）。
- 分块上限随深度变化（产品阶段 90 分钟 → 1 天 → 7 天 → 30 天 → 静默 90 天）。

### 软门控（自适应）

- 初始历史分块：**1 小时**（90 分钟 UI 的首次 seed 仍为 **2 小时**）。
- 最小分块：**15 分钟**；按深度的最大值：≤1 天 → **2 小时**，≤7 天 → **4 小时**，更早 → **6 小时**。
- 成功且快速：缓慢扩大（×1.5），直到该深度上限。
- 失败/超时：分块减半；等待采用指数退避（上限 10 分钟）。
- 渐进式 UI：展示已有覆盖范围；显示引导横幅或 30 天产品窗口。

### 错误去重

- 为 `request_id IN candidate_ids` 分支添加以下边界：
  `created_at >= $1 - INTERVAL '90 minutes' AND created_at < $2`.

## 不在范围内

- 强制将已有 v2 → v1。
- 独立回填 worker / 只读副本。
- 管理端 `backfill_profile` 设置（可能后续添加）。
