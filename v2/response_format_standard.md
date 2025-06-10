# REST API 响应格式标准

## 概述

本文档详细介绍 REST API 响应格式的标准字段、相关规范和最佳实践。

## 核心字段说明

### 1. `data` 字段

**用途**：包含实际的业务数据
**类型**：可以是对象、数组或基本类型
**必需性**：成功响应中通常必需

```json
{
  "data": {
    "id": 123,
    "name": "张三",
    "email": "zhangsan@example.com"
  }
}
```

**不同场景的 data 格式**：

```json
// 单个资源
{
  "data": {
    "id": 123,
    "name": "张三"
  }
}

// 资源列表
{
  "data": [
    {"id": 1, "name": "张三"},
    {"id": 2, "name": "李四"}
  ]
}

// 空结果
{
  "data": null
}
// 或
{
  "data": []
}

// 简单值
{
  "data": "success"
}
```

### 2. `meta` 字段

**用途**：包含关于响应的元数据信息
**类型**：对象
**必需性**：可选，但推荐包含

```json
{
  "data": [...],
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "v1",
    "requestId": "req-123456789",
    "responseTime": "123ms",
    "server": "api-server-01"
  }
}
```

**常见的 meta 子字段**：

- `timestamp`: 响应生成时间（ISO 8601 格式）
- `version`: API 版本
- `requestId`: 请求追踪 ID
- `responseTime`: 响应时间
- `server`: 服务器标识
- `environment`: 环境标识（dev/staging/prod）

### 3. `pagination` 字段

**用途**：分页相关的元数据
**位置**：通常放在 `meta` 内或作为顶级字段
**必需性**：分页查询时必需

```json
{
  "data": [...],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 150,
      "totalPages": 15,
      "hasNext": true,
      "hasPrev": false,
      "nextPage": 2,
      "prevPage": null
    }
  }
}
```

### 4. `links` 字段

**用途**：HATEOAS 相关链接
**类型**：对象
**标准**：符合 HAL (Hypertext Application Language) 规范

```json
{
  "data": {...},
  "links": {
    "self": "https://api.example.com/users/123",
    "edit": "https://api.example.com/users/123",
    "delete": "https://api.example.com/users/123",
    "avatar": "https://api.example.com/users/123/avatar",
    "friends": "https://api.example.com/users/123/friends"
  }
}
```

**分页场景下的 links**：

```json
{
  "data": [...],
  "links": {
    "first": "https://api.example.com/users?page=1",
    "last": "https://api.example.com/users?page=15",
    "prev": "https://api.example.com/users?page=1",
    "next": "https://api.example.com/users?page=3",
    "self": "https://api.example.com/users?page=2"
  }
}
```

### 5. `included` 字段

**用途**：包含关联的资源数据
**标准**：JSON:API 规范
**场景**：避免 N+1 查询问题

```json
{
  "data": {
    "id": 1,
    "title": "Hello World",
    "author": {
      "id": 9,
      "type": "people"
    }
  },
  "included": [
    {
      "type": "people",
      "id": 9,
      "attributes": {
        "firstName": "Dan",
        "lastName": "Gebhardt",
        "twitter": "dgeb"
      }
    }
  ]
}
```

### 6. `status` 字段

**用途**：业务层面的状态信息
**类型**：字符串或对象
**场景**：需要区分 HTTP 状态和业务状态时

```json
{
  "data": {...},
  "status": {
    "code": "SUCCESS",
    "message": "Operation completed successfully"
  }
}
```

### 7. `warnings` 字段

**用途**：非致命性警告信息
**类型**：数组
**场景**：操作成功但有需要注意的问题

```json
{
  "data": {...},
  "warnings": [
    {
      "code": "DEPRECATED_FIELD",
      "message": "Field 'old_field' is deprecated, use 'new_field' instead",
      "field": "old_field"
    }
  ]
}
```

## 相关标准规范

### 1. JSON:API 规范

**官方网站**：https://jsonapi.org/

**核心结构**：

```json
{
  "data": {...},        // 主要数据
  "included": [...],    // 关联数据
  "meta": {...},        // 元数据
  "links": {...},       // 链接信息
  "jsonapi": {          // JSON:API 版本信息
    "version": "1.0"
  }
}
```

### 2. HAL (Hypertext Application Language)

**规范**：RFC 8288

**核心结构**：

```json
{
  "_embedded": {...},   // 嵌入的资源
  "_links": {...},      // 超媒体链接
  "property1": "value1" // 资源属性
}
```

### 3. RFC 7807 - Problem Details

**用途**：专门用于错误响应
**结构**：

```json
{
  "type": "https://example.com/probs/out-of-credit",
  "title": "You do not have enough credit.",
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345/msgs/abc",
  "balance": 30,
  "accounts": ["/account/12345", "/account/67890"]
}
```

### 4. OpenAPI 规范

**版本**：3.x
**响应定义示例**：

```yaml
responses:
  200:
    description: Successful response
    content:
      application/json:
        schema:
          type: object
          properties:
            data:
              $ref: "#/components/schemas/User"
            meta:
              $ref: "#/components/schemas/Meta"
```

## 不同场景的响应格式

### 1. 单资源查询

```json
{
  "data": {
    "id": 123,
    "name": "张三",
    "email": "zhangsan@example.com",
    "createdAt": "2024-01-01T10:00:00Z"
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  },
  "links": {
    "self": "https://api.example.com/users/123",
    "edit": "https://api.example.com/users/123",
    "avatar": "https://api.example.com/users/123/avatar"
  }
}
```

### 2. 列表查询（带分页）

```json
{
  "data": [
    { "id": 1, "name": "张三" },
    { "id": 2, "name": "李四" }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 150,
      "totalPages": 15
    },
    "timestamp": "2024-01-01T12:00:00Z"
  },
  "links": {
    "first": "https://api.example.com/users?page=1",
    "last": "https://api.example.com/users?page=15",
    "next": "https://api.example.com/users?page=2",
    "self": "https://api.example.com/users?page=1"
  }
}
```

### 3. 创建资源

```json
{
  "data": {
    "id": 124,
    "name": "王五",
    "email": "wangwu@example.com",
    "createdAt": "2024-01-01T12:00:00Z"
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  },
  "links": {
    "self": "https://api.example.com/users/124",
    "edit": "https://api.example.com/users/124"
  }
}
```

### 4. 批量操作

```json
{
  "data": {
    "successful": [
      { "id": 1, "status": "updated" },
      { "id": 2, "status": "created" }
    ],
    "failed": [
      {
        "id": 3,
        "error": {
          "code": 40001,
          "type": "VALIDATION_ERROR",
          "message": "Invalid email format"
        }
      }
    ]
  },
  "meta": {
    "totalCount": 3,
    "successCount": 2,
    "failureCount": 1,
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

### 5. 空结果

```json
{
  "data": [],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 0,
      "totalPages": 0
    },
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

### 6. 异步操作

```json
{
  "data": {
    "taskId": "task-123456789",
    "status": "processing",
    "progress": 45,
    "estimatedTime": 300
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z"
  },
  "links": {
    "self": "https://api.example.com/tasks/task-123456789",
    "cancel": "https://api.example.com/tasks/task-123456789/cancel"
  }
}
```

## 字段使用指南

### 必需字段

| 字段             | 场景         | 说明             |
| ---------------- | ------------ | ---------------- |
| `data`           | 所有成功响应 | 包含实际业务数据 |
| `meta.timestamp` | 所有响应     | 响应生成时间     |
| `meta.requestId` | 生产环境     | 便于问题追踪     |
| `pagination`     | 分页查询     | 分页元数据       |

### 推荐字段

| 字段                | 场景        | 说明         |
| ------------------- | ----------- | ------------ |
| `links`             | RESTful API | 支持 HATEOAS |
| `meta.version`      | 版本化 API  | API 版本信息 |
| `meta.responseTime` | 性能监控    | 响应时间统计 |
| `warnings`          | 兼容性处理  | 非致命性警告 |

### 可选字段

| 字段          | 场景         | 说明           |
| ------------- | ------------ | -------------- |
| `included`    | 复杂关联查询 | 避免 N+1 查询  |
| `status`      | 复杂业务状态 | 业务层状态信息 |
| `meta.server` | 微服务架构   | 服务器标识     |

## 最佳实践

### 1. 保持一致性

```json
// ✅ 好的做法：字段名称和结构保持一致
{
  "data": {...},
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  }
}

// ❌ 避免：不一致的命名
{
  "result": {...},      // 有时用 data，有时用 result
  "metadata": {         // 有时用 meta，有时用 metadata
    "time": "...",      // 有时用 timestamp，有时用 time
    "request_id": "..." // 有时用 requestId，有时用 request_id
  }
}
```

### 2. 合理使用嵌套

```json
// ✅ 好的做法：逻辑分组
{
  "data": {...},
  "meta": {
    "pagination": {...},
    "performance": {
      "responseTime": "123ms",
      "cacheHit": true
    }
  }
}

// ❌ 避免：过度扁平化
{
  "data": {...},
  "page": 1,
  "limit": 10,
  "total": 100,
  "responseTime": "123ms",
  "cacheHit": true
}
```

### 3. 空值处理

```json
// ✅ 推荐：明确的空值表示
{
  "data": null,          // 单个资源不存在
  "meta": {...}
}

{
  "data": [],            // 列表为空
  "meta": {
    "pagination": {
      "total": 0
    }
  }
}

// ❌ 避免：省略字段
{
  // 缺少 data 字段，客户端无法判断是否有数据
  "meta": {...}
}
```

## 总结

选择合适的响应格式需要考虑：

1. **业务需求**：根据实际业务场景选择必要的字段
2. **客户端需求**：考虑客户端的使用便利性
3. **标准兼容**：尽量符合现有标准（JSON:API、HAL 等）
4. **性能考虑**：避免不必要的字段增加响应大小
5. **可扩展性**：为未来的需求预留扩展空间

建议采用 `data` + `meta` 的基础结构，根据具体需求添加 `links`、`included` 等字段。
