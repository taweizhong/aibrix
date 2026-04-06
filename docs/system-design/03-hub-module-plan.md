# 03. Hub 模块实现计划

## 1. 模块分解

1. 配置管理模块（Config Service）
2. 审批集成模块（Approval Adapter）
3. 发布编排模块（Release Orchestrator）
4. 推送通道模块（Agent WS Broker）
5. 历史版本模块（Versioning & Diff）

## 2. 配置管理模块

### 2.1 功能

- 按 scope 管理配置文件（global/env/cluster/app/instance）
- 维护 metadata（hot/encrypted/format）
- 支持历史版本与 diff

### 2.2 任务

- 定义 ConfigFile、ConfigVersion、ConfigScope 数据模型
- 实现 CRUD API + 参数校验
- 接入敏感字段扫描器与加密服务
- 存储密文并标记 metadata.encrypted=true

## 3. 审批与发布状态机

### 3.1 状态建模

- 普通环境：`draft -> pending -> approved -> published`
- 生产环境：`draft -> pending -> approved -> canary -> prod -> stable -> dr`

### 3.2 任务

- 定义状态机转移表（含非法状态拦截）
- 集成飞书审批轮询器（任务调度 + 重试 + 幂等）
- 每阶段支持审批/发布/回滚操作
- 发布操作沉淀审计日志

## 4. Agent 推送通道

### 4.1 功能

- 仅 Agent Leader 与 Hub 建立 WS 连接
- 支持多集群并发连接
- 支持心跳与断线重连

### 4.2 任务

- 实现 Agent 鉴权（初期可基于内网与静态 token）
- 实现集群维度推送路由
- 失败重试与消息确认（ACK）机制
- 推送结果指标上报

## 5. Hub 任务列表（可执行）

- [ ] Hub API OpenAPI 文档与 mock
- [ ] Config CRUD + 历史版本 + diff
- [ ] 审批适配器（飞书轮询）
- [ ] 发布状态机与回滚
- [ ] WS Broker 与 Agent Leader 长连接
- [ ] Hub 侧观测（日志/指标/trace）

## 6. 验收标准

- 配置发布能在目标 Agent 集群秒级可见
- 审批结果变化可正确驱动状态推进
- 生产流程每阶段可手工审批与回滚
- Hub 多 Pod 扩缩容不影响 API 正常性
