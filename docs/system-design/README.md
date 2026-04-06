# 基础设施服务项目计划总览（Go）

## 1. 文档目的

本目录用于规划一个基于 Go 的基础设施服务，统一承载以下三大能力：

1. 配置中心（Hub 管控、审批后发布、SDK 多级合并与热更新）
2. 服务注册/发现（集群内可选启用，独立部署环境必须启用）
3. 分布式 KV（支持长轮询 watch、TTL、前缀订阅）

> 目标：通过“Hub + Agent + SDK”架构，建设一个可扩展、可运维、支持故障降级的控制平面与数据平面系统。

---

## 2. 模块文档索引

- [`01-overall-architecture.md`](./01-overall-architecture.md): 系统边界、部署拓扑、关键流程、技术选型建议
- [`02-treestore-and-storage.md`](./02-treestore-and-storage.md): 统一树模型、L1/L2/L3 存储、数据一致性、Watch 引擎
- [`03-hub-module-plan.md`](./03-hub-module-plan.md): Hub 的配置管理、审批流程、发布状态机、WebSocket 推送
- [`04-agent-module-plan.md`](./04-agent-module-plan.md): Agent 集群职责、Leader/Follower、配置/KV/发现 API
- [`05-sdk-module-plan.md`](./05-sdk-module-plan.md): Go SDK 的 Config/KV/Discovery 模块与本地缓存
- [`06-api-and-contracts.md`](./06-api-and-contracts.md): Hub/Agent API 细化、请求响应模型、错误码规范
- [`07-delivery-milestones-and-tasks.md`](./07-delivery-milestones-and-tasks.md): 分阶段实施里程碑、任务拆解、验收标准

---

## 3. 实施原则

- **先打底能力，后做高级特性**：先完成 TreeStore、基础 API、SDK 初始化路径，再补审批流程、灰度发布、高级观测。
- **模块可独立演进**：Hub、Agent、SDK 之间通过稳定协议解耦，便于并行开发。
- **高可用优先于强一致**：在配置中心和 KV 的需求下，优先保证可用性、延迟与可恢复能力。
- **明确“待定项”边界**：Leader 选举、Agent 间数据共享等保留可插拔策略，避免早期锁死设计。

---

## 4. 建议目录结构（新项目）

```text
infra-platform/
├── cmd/
│   ├── hub/
│   ├── agent/
│   └── migrate/
├── internal/
│   ├── hub/
│   ├── agent/
│   ├── treestore/
│   ├── storage/
│   ├── longpoll/
│   ├── release/
│   ├── approval/
│   ├── security/
│   ├── api/
│   └── observability/
├── sdk/go/
│   ├── config/
│   ├── kv/
│   ├── discovery/
│   └── transport/
├── api/openapi/
├── deploy/
│   ├── hub/
│   ├── agent/
│   └── redis/
└── docs/
```

---

## 5. 交付定义（Definition of Done）

- Hub/Agent/SDK 三端均有可执行样例（含最小集成 demo）
- 配置中心支持：多级拉取、文件级 metadata、MD5 热更新
- KV 支持：Put/Get/Delete/List、Watch/WatchPrefix、TTL
- 服务发现支持：注册、续约心跳、下线、查询
- 全链路具备基础观测：日志、指标、健康检查、错误码
- 关键故障场景具备自动降级或可操作恢复路径
