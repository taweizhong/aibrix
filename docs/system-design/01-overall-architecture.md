# 01. 整体架构设计计划

## 1. 系统定位

该系统是一个“控制 + 数据”融合的基础设施平台：

- **控制面（Hub）**：配置管理、审批、发布编排、变更推送
- **边缘数据面（Agent）**：对业务 Pod 提供低延迟配置读取、KV 读写与 watch、可选服务发现
- **客户端（Go SDK）**：业务应用内的统一访问入口，负责配置合并、热更新回调、解密调用

## 2. 核心角色与职责

### 2.1 Hub

- 管理配置文件生命周期（CRUD、历史、diff）
- 接入审批流并驱动发布状态机
- 对 Agent Leader 建立 WebSocket 推送链路
- 加密写入敏感配置（调用外部加密服务）

### 2.2 Agent

- 维护本地 TreeStore（L1 内存 + L2 Redis + L3 DB）
- 提供 SDK 访问 API（配置读取、配置 watch、KV API、服务发现 API）
- 通过 Leader 与 Hub 同步配置，Follower 与 Leader 保持一致

### 2.3 SDK

- 启动全量拉取并初始化内存态配置
- 对同名配置文件执行多级深度合并
- 热配置长轮询、变更回调、按需解密
- 提供 KV 与服务发现编程接口

## 3. 关键数据流

### 3.1 配置中心（单向）

`Hub -> Agent Leader -> Agent 集群同步 -> SDK 长轮询感知 -> App`

### 3.2 KV（双向）

`App SDK -> Agent -> TreeStore(L1/L2/L3) -> Pub/Sub -> Watchers -> App SDK`

### 3.3 服务注册发现（双向）

`App SDK <-> Agent 服务发现模块`

## 4. 推荐技术方案（可落地优先）

- **Web 框架**：Gin/Fiber（二选一，建议 Gin）
- **持久化抽象**：Repository + DAO（DB 具体选型被代理层屏蔽）
- **缓存与事件**：Redis（String/Hash + PubSub + TTL + INCR）
- **本地 DB**：建议 SQLite/PostgreSQL（依部署形态可切换）
- **序列化**：JSON（API）、TOML（配置内容）
- **观测**：OpenTelemetry + Prometheus + structured logging

## 5. 架构任务清单

1. 定义 Hub/Agent/SDK 的边界与职责文档
2. 明确模块级接口（Go interface）与依赖方向
3. 确定通信协议（HTTP + WS + 长轮询）
4. 输出部署拓扑（K8s 与独立环境）
5. 输出故障降级与恢复流程图

## 6. 验收标准

- 能画出完整 C4 Level-2 组件图并与代码目录一一映射
- 核心流程（配置发布、KV watch）有时序图
- 架构文档中所有外部依赖均有“失败时策略”
