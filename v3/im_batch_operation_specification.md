# IM 批量操作规范

## 1. 设计理念

**核心原则**：统一、原子、高效
**适用场景**：批量创建、更新、删除等操作
**技术标准**：RFC9110、RFC9205、RFC7807

## 2. 批量操作规范

### 2.1 基础分类

**用途**：统一处理批量数据操作
**场景**：POST/PUT/DELETE 请求
**限制**：控制批量大小、确保原子性

**核心类型：**

- **批量创建**：一次性创建多个资源
- **批量更新**：一次性更新多个资源
- **批量删除**：一次性删除多个资源
- **批量查询**：一次性查询多个资源

### 2.2 批量创建规范

**设计理念**：统一处理批量创建，确保原子性和一致性
**适用场景**：批量创建用户、群组、消息等

**请求格式：**

```http
POST /api/v1/{resource}/batch
content-type: application/json

{
  "batch": true,
  "batch_size": 100,
  "batch_mode": "atomic",
  "data": [
    { "field1": "value1" },
    { "field2": "value2" }
  ]
}
```

**参数说明：**

| 参数         | 说明         | 示例                  | 适用场景         |
| ------------ | ------------ | --------------------- | ---------------- |
| `batch`      | 批量操作标识 | `batch=true`          | 所有批量操作     |
| `batch_size` | 每批处理数量 | `batch_size=100`      | 大数据量分批处理 |
| `batch_mode` | 处理模式     | `batch_mode=atomic`   | 事务性批量操作   |
| `data`       | 批量数据     | `data=[{...}, {...}]` | 批量创建数据     |

**处理模式：**

- **atomic**：原子性处理，全部成功或全部失败
- **sequential**：顺序处理，失败后停止
- **parallel**：并行处理，继续处理其他项

### 2.3 批量更新规范

**设计理念**：统一处理批量更新，确保数据一致性
**适用场景**：批量更新状态、设置等

**请求格式：**

```http
PUT /api/v1/{resource}/batch
content-type: application/json

{
  "batch": true,
  "batch_size": 100,
  "batch_mode": "atomic",
  "data": [
    { "id": "id1", "field1": "value1" },
    { "id": "id2", "field2": "value2" }
  ]
}
```

**参数说明：**

| 参数         | 说明         | 示例                      | 适用场景         |
| ------------ | ------------ | ------------------------- | ---------------- |
| `batch`      | 批量操作标识 | `batch=true`              | 所有批量操作     |
| `batch_size` | 每批处理数量 | `batch_size=100`          | 大数据量分批处理 |
| `batch_mode` | 处理模式     | `batch_mode=atomic`       | 事务性批量操作   |
| `data`       | 批量数据     | `data=[{id: "...", ...}]` | 批量更新数据     |

### 2.4 批量删除规范

**设计理念**：统一处理批量删除，确保操作安全
**适用场景**：批量删除资源

**请求格式：**

```http
DELETE /api/v1/{resource}/batch?ids=id1,id2,id3
```

**参数说明：**

| 参数         | 说明         | 示例                | 适用场景         |
| ------------ | ------------ | ------------------- | ---------------- |
| `ids`        | 批量 ID 列表 | `ids=id1,id2,id3`   | 批量删除指定资源 |
| `batch_mode` | 处理模式     | `batch_mode=atomic` | 事务性批量操作   |

### 2.5 批量查询规范

**设计理念**：统一处理批量查询，提高查询效率
**适用场景**：批量获取资源详情

**请求格式：**

```http
POST /api/v1/{resource}/batch/query
content-type: application/json

{
  "ids": ["id1", "id2", "id3"],
  "fields": ["field1", "field2"],
  "include": ["relation1", "relation2"]
}
```

**参数说明：**

| 参数      | 说明         | 示例                    | 适用场景         |
| --------- | ------------ | ----------------------- | ---------------- |
| `ids`     | 查询 ID 列表 | `ids=["id1", "id2"]`    | 批量查询指定资源 |
| `fields`  | 返回字段     | `fields=["field1"]`     | 指定返回字段     |
| `include` | 关联数据     | `include=["relation1"]` | 包含关联数据     |

### 2.6 处理模式选择策略

**设计理念**：根据业务场景选择最合适的处理模式
**目标**：在数据一致性和操作效率之间取得最佳平衡

#### 2.6.1 处理模式对比

| 处理模式     | 特点                     | 适用场景                   | 注意事项                 |
| ------------ | ------------------------ | -------------------------- | ------------------------ |
| `atomic`     | 全部成功或全部失败       | 需要强一致性的关键业务操作 | 性能开销较大，失败率较高 |
| `sequential` | 顺序处理，失败即停止     | 依赖顺序的批量操作         | 处理时间较长，部分成功   |
| `parallel`   | 并行处理，继续处理其他项 | 容错性要求高的非关键操作   | 需要额外的错误处理机制   |

#### 2.6.2 场景选择指南

**推荐使用 atomic 模式的场景：**

- 用户账户管理：创建、更新、删除用户账户
- 群组核心操作：创建群组、修改群组关键设置
- 消息撤回：批量撤回消息
- 支付相关操作：批量支付、退款等
- 数据迁移：需要保证数据一致性的迁移操作

**推荐使用 sequential 模式的场景：**

- 消息发送：需要按顺序处理的消息发送
- 状态更新：需要按顺序执行的状态变更
- 数据导入：需要保持顺序的数据导入
- 定时任务：需要按顺序执行的任务队列

**推荐使用 parallel 模式的场景：**

- 消息推送：离线消息推送
- 状态同步：非关键状态同步
- 数据统计：批量数据统计
- 通知发送：批量发送通知

#### 2.6.3 性能考虑

- **atomic 模式**：

  - 优点：数据一致性最强
  - 缺点：性能开销大，并发能力受限
  - 建议：批量大小控制在 100 以内

- **sequential 模式**：

  - 优点：顺序保证，部分成功
  - 缺点：处理时间较长
  - 建议：批量大小控制在 500 以内

- **parallel 模式**：
  - 优点：处理速度快，并发能力强
  - 缺点：需要额外的错误处理
  - 建议：批量大小可达到 1000 以上

#### 2.6.4 错误处理策略

**atomic 模式错误处理：**

```json
{
  "error": {
    "code": "BATCH_OPERATION_FAILED",
    "message": "批量操作失败",
    "details": {
      "failed_index": 2,
      "reason": "数据验证失败"
    }
  }
}
```

**sequential/parallel 模式错误处理：**

```json
{
  "data": {
    "success": [...],
    "failed": [...],
    "statistics": {
      "total": 100,
      "success": 95,
      "failed": 5
    }
  }
}
```

## 3. IM 业务场景示例

### 3.1 用户管理

**批量创建用户：**

```http
POST /api/v1/users/batch
{
  "batch": true,
  "batch_size": 100,
  "batch_mode": "atomic",
  "data": [
    { "username": "user1", "email": "user1@example.com" },
    { "username": "user2", "email": "user2@example.com" }
  ]
}
```

**批量更新用户状态：**

```http
PUT /api/v1/users/batch
{
  "batch": true,
  "batch_size": 100,
  "batch_mode": "atomic",
  "data": [
    { "id": "user1", "status": "active" },
    { "id": "user2", "status": "inactive" }
  ]
}
```

**批量删除用户：**

```http
DELETE /api/v1/users/batch?ids=user1,user2,user3
```

### 3.2 群组管理

**批量创建群组：**

```http
POST /api/v1/groups/batch
{
  "batch": true,
  "batch_size": 50,
  "batch_mode": "atomic",
  "data": [
    { "name": "群组1", "type": "public" },
    { "name": "群组2", "type": "private" }
  ]
}
```

**批量更新群组设置：**

```http
PUT /api/v1/groups/batch
{
  "batch": true,
  "batch_size": 50,
  "batch_mode": "atomic",
  "data": [
    { "id": "group1", "is_public": true },
    { "id": "group2", "is_public": false }
  ]
}
```

**批量删除群组：**

```http
DELETE /api/v1/groups/batch?ids=group1,group2,group3
```

### 3.3 消息管理

**批量发送消息：**

```http
POST /api/v1/messages/batch
{
  "batch": true,
  "batch_size": 100,
  "batch_mode": "atomic",
  "data": [
    { "chat_id": "user1", "content": "消息1" },
    { "chat_id": "user2", "content": "消息2" }
  ]
}
```

**批量更新消息状态：**

```http
PUT /api/v1/messages/batch
{
  "batch": true,
  "batch_size": 100,
  "batch_mode": "atomic",
  "data": [
    { "id": "msg1", "status": "read" },
    { "id": "msg2", "status": "deleted" }
  ]
}
```

**批量删除消息：**

```http
DELETE /api/v1/messages/batch?ids=msg1,msg2,msg3
```
