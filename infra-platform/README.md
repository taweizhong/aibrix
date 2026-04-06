# Infra Platform (Hub / Agent / Go SDK)

该目录是基于 `docs/system-design/` 规划文档创建的项目脚手架，目标是实现：

- 配置中心（Hub）
- 边缘 Agent（配置读取、KV、服务发现）
- Go SDK（Config/KV/Discovery）

## 目录说明

- `cmd/`: 可执行入口（hub/agent/migrate）
- `internal/`: 服务端核心实现
- `sdk/go/`: SDK 代码
- `api/openapi/`: OpenAPI 契约
- `deploy/`: 部署清单
- `docs/`: 项目内补充文档

## 当前状态

当前仅完成目录骨架和最小可运行入口，后续按 `docs/system-design/07-delivery-milestones-and-tasks.md` 分阶段实现。
