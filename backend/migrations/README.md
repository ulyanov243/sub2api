# 数据库迁移

## 概览

此目录包含用于变更数据库 Schema 的 SQL 迁移文件。迁移系统使用 SHA256 校验和，以保证迁移不可变且在不同环境中保持一致。

## 迁移文件命名

格式：`NNN_description.sql`

- `NNN`：顺序编号（如 001、002、003）
- `description`：使用 snake_case 的简短说明

示例：`017_add_gemini_tier_id.sql`

### `_notx.sql` 命名与执行语义（并发索引专用）

当迁移包含 `CREATE INDEX CONCURRENTLY` 或 `DROP INDEX CONCURRENTLY` 时，必须使用 `_notx.sql` 后缀，例如：

- `062_add_accounts_priority_indexes_notx.sql`
- `063_drop_legacy_indexes_notx.sql`

运行规则：

1. `*.sql`（不带 `_notx`）按事务执行。
2. `*_notx.sql` 按非事务执行，不会包裹在 `BEGIN/COMMIT` 中。
3. `*_notx.sql` 仅允许并发索引语句，不允许混入事务控制语句或其他 DDL/DML。

幂等要求（必须）：

- 创建索引：`CREATE INDEX CONCURRENTLY IF NOT EXISTS ...`
- 删除索引：`DROP INDEX CONCURRENTLY IF EXISTS ...`

这样可以保证灾备重放、重复执行时不会因对象已存在/不存在而失败。

## 迁移文件结构

项目使用自定义迁移执行器（`internal/repository/migrations_runner.go`），按原样执行完整 SQL 文件内容。

- 常规迁移（`*.sql`）：在事务中执行。
- 非事务迁移（`*_notx.sql`）：按语句拆分，并在事务外执行（用于 `CONCURRENTLY`）。

```sql
-- 仅向前迁移（推荐）
ALTER TABLE usage_logs ADD COLUMN IF NOT EXISTS example_column VARCHAR(100);
```

> ⚠️ 请勿在同一文件中放置可执行的“Down” SQL。执行器不会解析 goose 的 Up/Down 分段，而会执行文件中的全部 SQL 语句。

## 重要规则

### ⚠️ 不可变原则

**迁移一旦应用于任何环境（开发、预发布或生产），就绝不可修改。**

原因：

- 每个迁移都有存储在 `schema_migrations` 表中的 SHA256 校验和。
- 修改已应用迁移会导致校验和不匹配错误。
- 各环境会出现不一致的数据库状态。
- 会破坏审计链路和可复现性。

### ✅ 正确工作流

1. **创建新迁移**
   ```bash
   # 使用下一个顺序编号创建新文件
   touch migrations/018_your_change.sql
   ```

2. **编写仅向前的迁移 SQL**
   - 文件中只放入计划的 Schema 变更。
   - 如需回滚，创建新的迁移文件来撤销。

3. **本地测试**
   ```bash
   # 应用迁移
   make migrate-up

   # 测试回滚
   make migrate-down
   ```

4. **提交并部署**
   ```bash
   git add migrations/018_your_change.sql
   git commit -m "feat(db): add your change"
   ```

### ❌ 禁止事项

- ❌ 修改已应用的迁移文件
- ❌ 删除迁移文件
- ❌ 修改迁移文件名
- ❌ 调整迁移编号顺序

### 🔧 意外修改已应用的迁移时

**错误信息：**
```
migration 017_add_gemini_tier_id.sql checksum mismatch (db=abc123... file=def456...)
```

**解决方法：**
```bash
# 1. 查找原始版本
git log --oneline -- migrations/017_add_gemini_tier_id.sql

# 2. 还原到其首次应用时的提交
git checkout <commit-hash> -- migrations/017_add_gemini_tier_id.sql

# 3. 为变更创建一个新的迁移
touch migrations/018_your_new_change.sql
```

## 迁移系统详情

- **校验和算法**：去除首尾空白后的文件内容的 SHA256。
- **跟踪表**：`schema_migrations`（filename、checksum、applied_at）。
- **执行器**：`internal/repository/migrations_runner.go`。
- **自动执行**：服务启动时自动执行迁移。

## 最佳实践

1. **让迁移保持小且聚焦**
   - 每个迁移只包含一个逻辑变更。
   - 更易于评审和回滚。

2. **编写可逆迁移**
   - 始终提供可运行的 Down 迁移。
   - 提交前测试回滚。

3. **使用事务**
   - 尽可能将 DDL 语句包在事务中。
   - 确保原子性。

4. **添加注释**
   - 说明为何需要此变更。
   - 记录特殊注意事项。

5. **先在开发环境测试**
   - 在本地应用迁移。
   - 验证数据完整性。
   - 测试回滚。

## 迁移示例

```sql
-- 为 Gemini OAuth 账户添加 tier_id 字段以跟踪配额
UPDATE accounts
SET credentials = jsonb_set(
    credentials,
    '{tier_id}',
    '"LEGACY"',
    true
)
WHERE platform = 'gemini'
  AND type = 'oauth'
  AND credentials->>'tier_id' IS NULL;
```

## 故障排除

### 校验和不匹配

请参阅上文“意外修改已应用的迁移时”。

### 迁移失败
```bash
# 查看迁移状态
psql -d sub2api -c "SELECT * FROM schema_migrations ORDER BY applied_at DESC;"

# 如有必要可手动回滚（谨慎使用）
# 更好的做法是修复迁移并新建一个迁移
```

### 需要跳过迁移（仅限紧急情况）
```sql
-- 危险：仅在开发环境使用，或务必极度谨慎
INSERT INTO schema_migrations (filename, checksum, applied_at)
VALUES ('NNN_migration.sql', 'calculated_checksum', NOW());
```

## 参考资料

- 迁移执行器：`internal/repository/migrations_runner.go`
- PostgreSQL 文档：https://www.postgresql.org/docs/
