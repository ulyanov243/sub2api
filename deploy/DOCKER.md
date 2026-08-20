# Sub2API Docker 镜像

Sub2API 是用于分发和管理 AI 产品订阅 API 配额的 AI API 网关平台。

## 快速开始

```bash
docker run -d \
  --name sub2api \
  -p 8080:8080 \
  -e DATABASE_URL="postgres://user:pass@host:5432/sub2api" \
  -e REDIS_URL="redis://host:6379" \
  weishaw/sub2api:latest
```

## Docker Compose

```yaml
version: '3.8'

services:
  sub2api:
    image: weishaw/sub2api:latest
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/sub2api?sslmode=disable
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=sub2api
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## 环境变量

| 变量 | 说明 | 必填 | 默认值 |
|----------|-------------|----------|---------|
| `DATABASE_URL` | PostgreSQL 连接字符串 | 是 | - |
| `REDIS_URL` | Redis 连接字符串 | 是 | - |
| `PORT` | 服务器端口 | 否 | `8080` |
| `GIN_MODE` | Gin 框架模式（`debug`/`release`） | 否 | `release` |

## 支持的架构

- `linux/amd64`
- `linux/arm64`

## 标签

- `latest` - 最新稳定版本
- `x.y.z` - 指定版本
- `x.y` - 指定次版本的最新补丁
- `x` - 指定主版本的最新次版本

## 链接

- [GitHub 仓库](https://github.com/weishaw/sub2api)
- [文档](https://github.com/weishaw/sub2api#readme)
