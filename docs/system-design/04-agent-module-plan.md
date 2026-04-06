# 04. Agent 模块实现计划

## 1. 模块职责

- 承接 SDK 所有运行期请求
- 维护本地多层 TreeStore
- 同步 Hub 下发配置
- 提供配置长轮询、KV 长轮询、服务发现 API

## 2. 子模块设计

### 2.1 Leader 管理

候选方案（待定）：

- Redis 分布式锁
- K8s Lease
- 内置 Raft

任务：

- 抽象 LeaderElector interface
- 先落地一种默认策略（建议 K8s Lease 或 Redis 锁）
- 保留策略可替换能力

### 2.2 Config Handler

功能：

- 读取 scope 文件并返回给 SDK
- 支持 watch 请求的 MD5 比对
- 仅返回变更文件

任务：

- 构建 scope 数据聚合器
- 热文件筛选（metadata.hot=true）
- 接入长轮询唤醒机制

### 2.3 KV Handler

功能：

- PUT/GET/DELETE/LIST
- Watch(path)/WatchPrefix(prefix)
- TTL 到期处理

任务：

- path 规范校验与命名空间隔离
- 写入 version 递增
- Pub/Sub 通知 watchers
- 到期 GC 与删除事件策略（待定项可配置）

### 2.4 Discovery Handler（可选）

功能：

- 服务注册、心跳续约、下线
- 按 name 查询实例列表

任务：

- 定义服务实例结构（id/name/addr/meta/ttl）
- 心跳续约机制
- 失效实例清理

## 3. Agent 模块任务清单

- [ ] Leader 选举模块（可插拔）
- [ ] Hub WS 客户端与配置同步器
- [ ] Config Pull + Watch API
- [ ] KV CRUD + Watch + TTL
- [ ] Discovery API（按部署模式开关）
- [ ] Agent 间缓存一致性（Pub/Sub）
- [ ] 健康检查与就绪探针

## 4. 验收标准

- Follower 在 Leader 切换后仍可继续服务读请求
- 配置更新能触发长轮询即时返回
- KV 同 path 高频写入保持最终一致（Last Write Wins）
- TTL 到期行为符合策略配置
