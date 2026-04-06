# 06. API 与协议契约计划

## 1. 设计原则

- 向后兼容优先：新增字段不破坏旧客户端
- 错误可机器处理：统一错误码 + message + request_id
- 长轮询接口明确 timeout 行为与重试建议

## 2. Hub API 计划

### 2.1 管理实体 API

- environments
- clusters
- apps

任务：

- 定义分页、过滤、排序参数规范
- 定义资源唯一性约束（name + env 等）

### 2.2 配置 API

- `GET /api/v1/configs?scope=...`
- `POST /api/v1/configs`
- `GET|PUT|DELETE /api/v1/configs/{id}`
- `GET /api/v1/configs/preview`
- `GET /api/v1/configs/{id}/history`
- `GET /api/v1/configs/{id}/diff`

任务：

- schema 定义（文件名、scope、content、metadata）
- 版本历史查询与差异输出格式

### 2.3 发布 API

- release 创建、审批、发布、回滚、状态查询

任务：

- 阶段动作幂等性定义
- 非法状态错误码定义

## 3. Agent API 计划

### 3.1 Config API

- 全量拉取接口
- watch 接口（MD5 map）

任务：

- 明确 changed=false 的超时返回结构
- 返回体包含 scope 与 metadata，便于 SDK 合并

### 3.2 KV API

- CRUD + List + Watch

任务：

- path 编码与 URL 安全规范
- 前缀查询 recursive 参数定义
- watch 返回事件格式（PUT/DELETE/EXPIRE）

### 3.3 Discovery API

- register/deregister/discover/heartbeat

任务：

- 注册租约 TTL 默认值
- 心跳失败重试与摘除策略

## 4. 协议治理任务

- [ ] OpenAPI 3.1 文档生成
- [ ] 错误码手册（Hub/Agent 共用）
- [ ] API 兼容性测试（契约测试）
- [ ] SDK 对齐测试（golden cases）

## 5. 验收标准

- OpenAPI 可用于生成 SDK Stub
- 关键接口具备请求/响应示例
- 回归测试覆盖主要错误码路径
