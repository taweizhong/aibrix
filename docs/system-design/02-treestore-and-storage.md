# 02. 统一 TreeStore 与存储层计划

## 1. 数据模型

```go
// 核心实体（建议）
type Entry struct {
    Path      string
    Value     []byte
    Version   int64
    Metadata  map[string]any
    UpdatedAt time.Time
    ExpiredAt *time.Time
}
```

## 2. 路径命名空间

- `/config/*`: 配置文件（文件粒度）
- `/kv/*`: KV 节点（path 粒度）
- `/svc/*`: 服务发现节点

## 3. 分层存储职责

### 3.1 L1（进程内存）

- 最低延迟读取
- 缺点：重启丢失
- 目标：优先承接高频读与 watch 比对

### 3.2 L2（Redis）

- 集群共享数据
- 提供 TTL、INCR、Pub/Sub
- 作为 Agent 多副本间一致性的主要媒介

### 3.3 L3（本地 DB）

- 最终持久化
- 字段要求：path(unique), value(blob), version, metadata(json), expired_at
- 用于冷启动恢复与 Redis 故障降级

## 4. 一致性模型

- 配置中心：以内容 MD5 变化感知，不依赖全局版本
- KV：按 path 维护单调递增版本（INCR），watch 基于 fromVersion
- 写入策略：L1/L2/L3 顺序写入 + 失败补偿队列（建议）

## 5. Long Poll 统一引擎

### 5.1 能力

- 支持注册 watcher（按 path/prefix）
- 支持超时唤醒
- 支持外部事件唤醒（Pub/Sub）
- 支持批量 notify

### 5.2 配置 watch 与 KV watch 分离策略

- 配置 watch：提交“文件名 -> MD5”映射，后端进行比对
- KV watch：提交 path/prefix + fromVersion，后端按版本判断

## 6. 任务拆解

1. 定义 TreeStore interface 与内存实现
2. 实现 RedisStore（Set/Get/Delete/ListPrefix/INCR/TTL/PubSub）
3. 实现 DBStore（upsert + list + gc expired）
4. 实现 MultiLayerStore 编排与回写策略
5. 实现 LongPollManager（watch lifecycle）
6. 覆盖核心单测（含并发写、TTL 到期、前缀 watch）

## 7. 验收标准

- 1w QPS 读场景下，P95 读取延迟在目标范围内（由压测定义）
- KV watch 在秒级内收到变更
- Redis 故障时，读能力可降级到 L1/L3
- 重启后可从 L3 恢复基础数据
