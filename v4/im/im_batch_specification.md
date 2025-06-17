# IM 批量操作规范

## 1. 设计理念

**批量操作构成 = URL 路径标识(/batch) + 一致性模式(必需) + 业务数据(必需)**

**核心原则**：统一、原子、高效
**适用场景**：批量创建、更新、删除、获取等操作
**技术标准**：RFC9110、RFC9205、RFC7807

**操作类型分类：**

- **批量创建**：POST /api/v1/{resource}/batch
- **批量更新**：PUT /api/v1/{resource}/batch
- **批量删除**：DELETE /api/v1/{resource}/batch
- **批量获取**：POST /api/v1/{resource}/batch/get

## 2. 批量操作规范

### 2.1 通用参数说明

**通用参数表：**

| 参数         | 类型   | 必需 | 可选值                   | 默认值   | 说明         | 适用操作         |
| ------------ | ------ | ---- | ------------------------ | -------- | ------------ | ---------------- |
| `batch_mode` | string | 是   | `atomic` or `non_atomic` | `atomic` | 一致性模式   | 所有操作         |
| `data`       | array  | 是   | -                        | -        | 批量数据数组 | 创建、更新、获取 |

### 2.2 批量创建 (POST /resource/batch)

**请求格式：**

```http
POST /api/v1/{resource}/batch
Content-Type: application/json

{
  "batch_mode": "atomic",
  "data": [
    { "id": "foo", "name": "value1" },
    { "id": "bar", "name": "value2" }
  ]
}
```

**响应格式：**

```json
{
  "data": {
    "success": [
      { "id": "id1", "status": "created", "createdAt": 1704110400000 }
    ],
    "failed": [
      { "index": 1, "error": { "code": "4000001", "message": "名称不合法" } }
    ],
    "summary": {
      "totalCount": 2,
      "successCount": 1,
      "failureCount": 1
    }
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req1704110400012abc456def78901"
  }
}
```

### 2.3 批量更新 (PUT /resource/batch)

**请求格式：**

```http
PUT /api/v1/{resource}/batch
Content-Type: application/json

{
  "batch_mode": "atomic",
  "data": [
    { "id": "id1", "name": "new_value1" },
    { "id": "id2", "name": "new_value2" }
  ]
}
```

**响应格式：**

```json
{
  "data": {
    "success": [
      { "id": "id1", "status": "updated", "updatedAt": 1704110400000 }
    ],
    "failed": [
      { "index": 1, "error": { "code": "4040301", "message": "用户不存在" } }
    ],
    "summary": {
      "totalCount": 2,
      "successCount": 1,
      "failureCount": 1
    }
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req1704110400012abc456def78901"
  }
}
```

### 2.4 批量删除 (DELETE /resource/batch)

**请求格式：**

```http
DELETE /api/v1/{resource}/batch?ids=id1,id2,id3&batch_mode=atomic
```

**特殊说明：**

- ids: 唯一标识，也可以是其它的唯一标识，如：用户名、手机号等

**响应格式：**

```json
{
  "data": {
    "success": [
      { "id": "id1", "status": "deleted", "deletedAt": 1704110400000 },
      { "id": "id2", "status": "deleted", "deletedAt": 1704110400000 }
    ],
    "failed": [
      { "id": "id3", "error": { "code": "4040301", "message": "用户不存在" } }
    ],
    "summary": {
      "totalCount": 3,
      "successCount": 2,
      "failureCount": 1
    }
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req1704110400012abc456def78901"
  }
}
```

### 2.5 批量获取 (POST /resource/batch/get)

**请求格式：**

```http
POST /api/v1/{resource}/batch/get
Content-Type: application/json

{
  "batch_mode": "non_atomic",
  "data": ["id1", "id2", "id3"]
}
```

**特殊说明：**

- `data` 数组包含资源 ID 字符串，不是对象
- 默认返回核心字段，不支持字段筛选和关联数据

**响应格式：**

```json
{
  "data": {
    "success": [
      {
        "id": "id1",
        "username": "user1",
        "email": "user1@example.com",
        "createdAt": 1704110400000
      },
      {
        "id": "id2",
        "username": "user2",
        "email": "user2@example.com",
        "createdAt": 1704110300000
      }
    ],
    "failed": [
      { "id": "id3", "error": { "code": "4040301", "message": "用户不存在" } }
    ],
    "summary": {
      "totalCount": 3,
      "successCount": 2,
      "failureCount": 1
    }
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req1704110400012abc456def78901"
  }
}
```

## 3. 一致性模式规则

### 3.1 模式对比

| 模式         | 事务特性           | 错误处理           | 性能影响 | 批量大小限制          | 适用场景       |
| ------------ | ------------------ | ------------------ | -------- | --------------------- | -------------- |
| `atomic`     | 全部成功或全部失败 | 任一失败则全部回滚 | 较低     | 数量较少，例如：≤100  | 关键业务操作   |
| `non_atomic` | 允许部分成功       | 继续处理其他项     | 较高     | 数量较大，例如：≤1000 | 非关键批量操作 |

### 3.2 选择指南

**atomic 模式适用场景：**

| 业务场景     | 具体操作                     | 原因                   |
| ------------ | ---------------------------- | ---------------------- |
| 用户账户管理 | 批量创建、更新、删除用户账户 | 需要保证账户数据一致性 |
| 群组核心操作 | 创建群组、修改群组关键设置   | 群组状态必须保持一致   |
| 消息撤回     | 批量撤回消息                 | 撤回操作不能部分成功   |
| 支付相关操作 | 批量支付、退款等             | 财务数据必须严格一致   |
| 数据迁移     | 关键数据迁移                 | 数据完整性要求高       |

**non_atomic 模式适用场景：**

| 业务场景 | 具体操作         | 原因                     |
| -------- | ---------------- | ------------------------ |
| 消息推送 | 离线消息批量推送 | 部分推送失败不影响其他   |
| 状态同步 | 批量同步用户状态 | 非关键数据，允许部分失败 |
| 数据统计 | 批量数据统计处理 | 统计数据允许部分处理     |
| 通知发送 | 批量发送通知     | 通知失败不影响其他通知   |
| 内容同步 | 批量同步用户内容 | 内容同步允许重试机制     |

### 3.3 性能考虑

**data 数组大小建议：**

| 一致性模式   | 推荐大小       | 最大限制    | 性能特点                 | 使用建议                     |
| ------------ | -------------- | ----------- | ------------------------ | ---------------------------- |
| `atomic`     | 50-100 个元素  | 100 个元素  | 事务开销大，并发能力受限 | 关键操作，数据量不大         |
| `non_atomic` | 200-500 个元素 | 1000 个元素 | 处理速度快，容错能力强   | 大批量操作，对实时性要求不高 |

## 4. IM 业务场景示例

### 4.1 用户管理

**批量创建用户：**

```http
POST /api/v1/users/batch
{
  "batch_mode": "atomic",
  "data": [
    { "username": "user1", "email": "user1@example.com", "nickname": "用户1" },
    { "username": "user2", "email": "user2@example.com", "nickname": "用户2" }
  ]
}
```

**批量更新用户状态：**

```http
PUT /api/v1/users/batch
{
  "batch_mode": "non_atomic",
  "data": [
    { "id": "user1", "status": "active" },
    { "id": "user2", "status": "inactive" }
  ]
}
```

**批量删除用户：**

```http
DELETE /api/v1/users/batch?ids=user1,user2,user3&batch_mode=atomic
```

**批量获取用户：**

```http
POST /api/v1/users/batch/get
{
  "ids": ["user1", "user2", "user3"],
}
```

### 4.2 群组管理

**批量创建群组：**

```http
POST /api/v1/groups/batch
{
  "batch_mode": "atomic",
  "data": [
    { "name": "技术讨论群", "type": "public", "description": "前端技术交流" },
    { "name": "项目协作群", "type": "private", "description": "项目内部讨论" }
  ]
}
```

**批量更新群组设置：**

```http
PUT /api/v1/groups/batch
{
  "batch_mode": "non_atomic",
  "data": [
    { "id": "group1", "is_public": true, "max_members": 200 },
    { "id": "group2", "is_public": false, "max_members": 50 }
  ]
}
```

### 4.3 消息管理

**批量发送消息：**

```http
POST /api/v1/messages/batch
{
  "batch_mode": "non_atomic",
  "data": [
    { "to": "user1", "type": "text", "body": { "text": "欢迎加入！" } },
    { "to": "user2", "type": "text", "body": { "text": "欢迎加入！" } }
  ]
}
```

**批量更新消息状态：**

```http
PUT /api/v1/messages/batch
{
  "batch_mode": "non_atomic",
  "data": [
    { "id": "msg1", "status": "read" },
    { "id": "msg2", "status": "deleted" }
  ]
}
```
