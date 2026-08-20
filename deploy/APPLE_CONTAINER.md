# Apple container 部署

Sub2API 可通过 Apple 的 `container` CLI 以原生三服务栈运行。此工作流无需 Docker Desktop 或兼容 Docker 的 daemon，即可运行已发布的 Sub2API、PostgreSQL 和 Redis OCI 镜像。

## 支持范围

Apple `container` 支持面向本地开发和在 Mac 上由运维人员管理的部署。生产环境仍推荐使用 Docker Compose。

Apple `container` 1.1 不提供重启策略、自动启动、工作负载健康调度、Docker API socket 或完整的 Compose 编排。调用 `apple-container.sh` 时，它会提供有序启动和就绪检查；但它不是持续运行的 supervisor。

## 要求

- 搭载 Apple silicon 的 Mac
- macOS 26 或更高版本
- Apple `container` 1.1.0 或更高版本
- 用于生成初始密钥的 `openssl`
- 首次启动已发布容器时，macOS 提示后为 `container-runtime-linux` 授予 Local Network 访问权限

从其[官方发布页](https://github.com/apple/container/releases)安装 Apple `container`，然后验证：

```bash
container --version
```

## 快速开始

```bash
git clone https://github.com/Wei-Shaw/sub2api.git
cd sub2api/deploy

# Creates .env with random PostgreSQL, JWT, and TOTP secrets.
./apple-container.sh init

# Review optional settings before startup.
nano .env

# Creates volumes/network/containers, waits for dependencies, and starts Sub2API.
./apple-container.sh up

# Verifies PostgreSQL, Redis, and the application endpoint.
./apple-container.sh status
```

打开 `http://localhost:8080`。如果 `ADMIN_PASSWORD` 为空，请使用以下命令获取生成的密码：

```bash
./apple-container.sh logs app
```

环境文件使用字面量 `KEY=value` 语法。请勿使用 `${VALUE:-default}` 这类 Compose 表达式；除非引号本身是值的一部分，否则不要给值加引号。`BIND_HOST` 必须是 IPv4 地址，`SERVER_PORT` 必须在 1025 至 65535 之间。

## 命令

```bash
# Start dependencies and recreate the lightweight app container with current IPs.
./apple-container.sh up

# Also recreate PostgreSQL and Redis containers, preserving their volumes.
./apple-container.sh up --recreate

# Stop containers while preserving all resources and data.
./apple-container.sh down

# Restart PostgreSQL, Redis, and Sub2API in dependency order.
./apple-container.sh restart

# Show resource state and run live health probes.
./apple-container.sh status

# Follow one service's logs.
./apple-container.sh logs app -f
./apple-container.sh logs postgres -f
./apple-container.sh logs redis -f

# Pull all configured images for linux/arm64, then recreate containers.
./apple-container.sh pull
./apple-container.sh up --recreate

# Delete containers and the network, preserving named volumes.
./apple-container.sh destroy --yes

# Permanently delete the stack and all application/database/cache data.
./apple-container.sh destroy --volumes --yes
```

`destroy --volumes` 不会删除 `.env`、备份文件或已拉取镜像。下线部署时，请分别删除凭据和备份。仅在确认没有其他 Apple 容器使用镜像后，才可使用 `container image delete <image>`。

主机重启或执行 `container system stop` 后，请再次运行 `./apple-container.sh up`。Apple `container` 不会自动重启持久化容器。

## 配置

脚本使用 `deploy/.env`，这与 Docker Compose 使用的是同一源文件。在当前 shell 中导出 `SUB2API_ENV_FILE`，即可让所有命令使用其他文件：

```bash
export SUB2API_ENV_FILE=/absolute/path/to/sub2api.env
./apple-container.sh init
./apple-container.sh up
```

可使用 Apple 专用的镜像覆盖：

```dotenv
APPLE_CONTAINER_SUB2API_IMAGE=weishaw/sub2api:latest
APPLE_CONTAINER_POSTGRES_IMAGE=postgres:18-alpine
APPLE_CONTAINER_REDIS_IMAGE=redis:8-alpine
```

常规 `up` 命令会重建应用容器，因此应用环境变更会立即生效。变更 PostgreSQL 或 Redis 容器镜像、或 Redis 运行时配置时，请使用 `up --recreate`。持久数据会保留在命名卷中。

`POSTGRES_USER`、`POSTGRES_PASSWORD` 和 `POSTGRES_DB` 仅在 PostgreSQL 初始化空数据卷时生效。在 `.env` 中修改它们再重建容器，并不会修改现有数据库。请使用 `ALTER ROLE` 轮换密码，并为用户或数据库变更规划明确迁移。若要有意初始化新的空数据库，请先备份旧数据库，然后使用 `destroy --volumes`。

共享设置在 Apple 工作流中的处理方式：

| 设置 | Apple 工作流行为 |
|---|---|
| 应用和网关变量 | 从 `.env` 传给 Sub2API |
| `BIND_HOST`、`SERVER_PORT` | 用于 macOS 发布端口 |
| `POSTGRES_USER`、`POSTGRES_PASSWORD`、`POSTGRES_DB` | 仅 PostgreSQL 首次初始化时使用 |
| `REDIS_PASSWORD` | 应用于 Redis 和 Sub2API |
| `DATABASE_PORT`、`REDIS_PORT` | 内部端口固定为 5432 和 6379 |
| `POSTGRES_MAX_*`、`REDIS_MAXCLIENTS` | 当前不应用到数据库/缓存服务器 |

## 受管资源

脚本只会创建带有 `org.sub2api.stack=apple-container` 标签的资源：

| 类型 | 名称 |
|---|---|
| 容器 | `sub2api-apple`、`sub2api-apple-postgres`、`sub2api-apple-redis` |
| 网络 | `sub2api-apple` |
| 卷 | `sub2api-apple-data`、`sub2api-apple-postgres-data`、`sub2api-apple-redis-data` |

PostgreSQL 卷挂载在 `/var/lib/postgresql`，保留 PostgreSQL 18 默认的子级数据目录。Sub2API 和 Redis 同样将数据存放在其 Apple 卷挂载点下的子目录中。这是必要的，因为 Apple 命名卷不具备 Docker 的 copy-up 和挂载点所有权行为。

## 网络

Apple `container` 1.1 不提供 Compose 风格的网络范围服务别名。PostgreSQL 和 Redis 启动后，脚本从 `container inspect` 读取它们当前的私有网络 IPv4 地址，将这些地址注入新建的应用容器，然后启动 Sub2API。脚本不会修改 `~/.config/container/config.toml` 或 macOS 主机解析器。

三个服务仅加入私有 `sub2api-apple` 网络。仅应用发布主机端口；数据库和 Redis 端口不对外发布。

每次 `up` 和 `restart` 操作都会有意重建应用容器，因为依赖的 VM 地址在停止后可能发生变化。应用数据会保留在 `sub2api-apple-data` 中。

脚本在报告成功前会从 macOS 检查已发布的 `/health` 端点。首次启动时请允许 Local Network 提示。若内部探测成功但主机端口探测因连接重置失败，请为 `container-runtime-linux` 启用 Local Network 访问，依次运行 `container system stop`、`container system start`，然后再次执行 `up`。运行时升级可能会再次请求权限。

## 备份与升级

为持久数据使用此工作流前，请在 `.env` 中固定镜像发布标签或 digest。升级应用或数据库镜像前，请在栈健康时创建备份：

```bash
umask 077
mkdir -p backups

# Logical PostgreSQL backup.
container exec sub2api-apple sh -c \
  'PGPASSWORD="$DATABASE_PASSWORD" pg_dump -h "$DATABASE_HOST" -U "$DATABASE_USER" "$DATABASE_DBNAME"' \
  > backups/sub2api.sql

# Application configuration and local files.
container exec sub2api-apple sh -c 'tar -C "$DATA_DIR" -czf - .' \
  > backups/sub2api-data.tar.gz

./apple-container.sh pull
./apple-container.sh up --recreate
./apple-container.sh status
```

数据库迁移仅向前执行。在确认升级后的栈有效之前，请保留之前的镜像引用和两份备份；仅回滚镜像无法逆转已迁移的数据库。在将此工作流用于重要数据前，请测试恢复流程。

要将这些备份恢复到现有栈，请先确保镜像版本与备份兼容，再停止写入方并替换两组数据：

```bash
# Ensure empty/current resources exist, then stop the stack.
./apple-container.sh up
./apple-container.sh down

# Remove only the app container so a helper can mount its named volume.
container delete sub2api-apple
SUB2API_IMAGE=weishaw/sub2api:latest # Match APPLE_CONTAINER_SUB2API_IMAGE in .env.
container run --rm --name sub2api-apple-data-restore \
  --entrypoint /bin/sh \
  --volume sub2api-apple-data:/restore \
  --volume "$PWD/backups:/backup:ro" \
  "$SUB2API_IMAGE" \
  -c 'rm -rf /restore/data && mkdir -p /restore/data && tar -xzf /backup/sub2api-data.tar.gz -C /restore/data'

# Restore the logical database while the application is absent.
container start sub2api-apple-postgres
until container exec sub2api-apple-postgres sh -c 'pg_isready -U "$POSTGRES_USER" -d "$POSTGRES_DB"'; do sleep 1; done
container copy backups/sub2api.sql sub2api-apple-postgres:/tmp/sub2api.sql
container exec sub2api-apple-postgres sh -c '
  export PGPASSWORD="$POSTGRES_PASSWORD"
  dropdb -h 127.0.0.1 -U "$POSTGRES_USER" --if-exists --force "$POSTGRES_DB"
  createdb -h 127.0.0.1 -U "$POSTGRES_USER" "$POSTGRES_DB"
  psql -h 127.0.0.1 -U "$POSTGRES_USER" -d "$POSTGRES_DB" -v ON_ERROR_STOP=1 -f /tmp/sub2api.sql
  rm /tmp/sub2api.sql
'

./apple-container.sh up
./apple-container.sh status
```

删除命名卷后的灾难恢复中，请先运行一次 `up` 创建新栈，再执行恢复流程。请先使用非生产数据进行恢复演练。

升级 Apple runtime：

```bash
./apple-container.sh down
container system stop
# Install/update Apple container 1.1.0 or newer.
container system start
./apple-container.sh up
```

## 运行限制

- 没有等价于 `restart: unless-stopped` 的功能。重启后请运行 `up`，或自行增加 launchd supervisor。
- 健康探测在 `up`、`restart` 和 `status` 期间运行；Apple `container` 不会持续调度它们。
- Docker Compose、Testcontainers、Buildx 及需要 `/var/run/docker.sock` 的工具不能直接使用此运行时。
- 将此工作流用于重要数据前，必须测试命名卷备份和恢复。
- 脚本面向原生 `linux/arm64` 镜像。常规 Sub2API 发布包含 arm64 变体。
- 包括凭据在内的运行时环境值会保留在 Apple container 配置中，能够检查本地运行时的用户可见这些值。
