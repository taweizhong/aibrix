# 05. Go SDK 模块实现计划

## 1. SDK 目标

- 业务接入成本低：全局单例 + 少量参数初始化
- 配置读取零拷贝/低延迟：内存态 Get/Unmarshal
- 具备健壮性：初始化失败可感知，运行期具备重试与缓存兜底

## 2. 模块划分

1. `config`：配置拉取、分层合并、热更新、解密
2. `kv`：KV 读写删查、watch/prefix watch
3. `discovery`：注册、发现、watch（可选）
4. `transport`：HTTP、长轮询、重试、熔断
5. `cache`：L0 内存 + L0.5 文件缓存

## 3. Config 详细计划

### 3.1 初始化流程

1. 拉取 Agent 全量配置
2. 解析并按文件名分组
3. 按 `Global -> Env -> Cluster -> App -> Instance` 合并
4. 反序列化到业务结构体
5. 启动热文件 watch 循环

### 3.2 任务

- TOML 深度合并器（map/table/array 策略需定义）
- 热文件 MD5 计算与 watch payload 组装
- 文件级与全局级回调机制
- encrypted 文件的解密器接口

## 4. KV 详细计划

任务：

- 实现 Put/PutWithTTL/Get/Delete/List
- 实现 Watch 与 WatchPrefix（自动重连）
- 封装版本推进与重试逻辑

## 5. Discovery 详细计划

任务：

- 注册对象生命周期管理（Deregister on close）
- 周期心跳与失败告警钩子
- 实例列表缓存与刷新

## 6. SDK 任务清单

- [ ] `configsdk.New(...)` 初始化骨架
- [ ] 配置全量拉取与多级合并
- [ ] Config 热更新长轮询
- [ ] KV API + Watch API
- [ ] Discovery API（可选）
- [ ] 本地文件缓存与启动恢复
- [ ] 接入示例与 README

## 7. 验收标准

- 初始化失败时可返回明确错误并由业务选择退出
- 热配置更新后回调触发准确且无重复风暴
- KV watch 在网络抖动后能自动恢复
- SDK 无 goroutine 泄漏（压测与单测验证）
