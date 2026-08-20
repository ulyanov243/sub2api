# 模型定价数据

此目录保存镜像模型定价数据的本地副本，作为回退机制。

## 来源

原始文件由 LiteLLM 项目维护，并通过 GitHub Actions 镜像到本仓库的 `price-mirror` 分支：

- 镜像分支（可通过 `PRICE_MIRROR_REPO` 配置）：https://raw.githubusercontent.com/<your-repo>/price-mirror/model_prices_and_context_window.json
- 上游来源：https://raw.githubusercontent.com/BerriAI/litellm/main/model_prices_and_context_window.json

## 用途

远程文件因以下原因无法下载时，使用此本地副本回退：

- 网络限制
- 防火墙规则
- DNS 解析问题
- 某些地区无法访问 GitHub
- Docker 容器的网络限制

## 更新流程

`pricingService` 会：

1. 先从 GitHub 下载最新版。
2. 下载失败时使用本地副本回退。
3. 使用回退文件时记录警告日志。

## 手动更新

若自动化不可用，可按以下方式将此文件更新为最新定价数据：
```bash
curl -s https://raw.githubusercontent.com/BerriAI/litellm/main/model_prices_and_context_window.json -o model_prices_and_context_window.json
```

## 文件格式

文件包含模型定价信息的 JSON 数据，包括：

- 模型名称和标识符
- 输入/输出 token 成本
- 上下文窗口大小
- 模型能力

最后更新：2025-08-10
