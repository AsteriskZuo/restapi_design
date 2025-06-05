# 基本规则

## 规则 1: 指定版本号

```
/api/v1/users    # 版本1
/api/v2/users    # 版本2
/api/v3/users    # 版本3
```

## 规则 2: 使用 HTTP 标准方法

- **GET**: 获取资源（安全、幂等、可缓存）
- **POST**: 创建资源（非安全、非幂等）
- **PUT**: 完整更新资源（非安全、幂等）
- **PATCH**: 部分更新资源（非安全、非幂等）
- **DELETE**: 删除资源（非安全、幂等）

## 规则 3: 服务不保存状态

```
✅ 正确：
GET /api/v1/users/123/posts?page=2&limit=10
Authorization: Bearer token123

❌ 错误：
GET /api/v1/getNextPage  # 依赖服务器端状态
```

## 规则 4: 认证方式

采用 JWT Bearer Token 进行调用验证。

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 规则 5: 不使用动词形式的 URL

```
❌ 动词形式的URL：
GET /api/v1/getUsers
POST /api/v1/createUser
DELETE /api/v1/deleteUser/123

✅ 正确的名词形式：
GET /api/v1/users
POST /api/v1/users
DELETE /api/v1/users/123
```

## 规则 6: 数据传递规范

**简单参数**: 使用查询参数

```
GET /api/v1/users?status=active&limit=10
```

**复杂数据**: 使用请求体

```
POST /api/v1/users
{
  "name": "张三",
  "email": "zhangsan@example.com",
  "profile": {
    "age": 25,
    "interests": ["技术", "旅行"]
  }
}
```

## 规则 7: 内容分类

按业务领域对资源进行分类：

```
# 用户类别
GET /api/v1/users

# 群组类别
GET /api/v1/groups

# 消息类别
GET /api/v1/messages
```

## 规则 8: 规范 URL 命名

**设计原则：**

- 使用名词而不是动词
- 使用复数形式
- 使用小写字母
- 使用连字符而不是下划线

```
✅ 正确示例：
GET /api/v1/users
GET /api/v1/user-profiles
GET /api/v1/order-items

❌ 错误示例：
GET /api/v1/getUsers
GET /api/v1/user_profiles
GET /api/v1/users.json
```

## 规则 9: HTTP 方法选择指南

### CRUD 操作映射

- **增加(Create)**: POST
- **查询(Read)**: GET 或 POST（复杂查询）
- **修改(Update)**: PUT（完整更新）或 PATCH（部分更新）
- **删除(Delete)**: DELETE

### 方法特性

| 方法   | 安全性 | 幂等性   | 缓存性 | 主要用途 |
| ------ | ------ | -------- | ------ | -------- |
| GET    | 安全   | 幂等     | 可缓存 | 获取资源 |
| POST   | 不安全 | 非幂等   | 不缓存 | 创建资源 |
| PUT    | 不安全 | 幂等     | 不缓存 | 完整更新 |
| PATCH  | 不安全 | 非幂等\* | 不缓存 | 部分更新 |
| DELETE | 不安全 | 幂等     | 不缓存 | 删除资源 |

**注意**: \*PATCH 方法根据 RFC 5789 规范，本质上是非幂等的。但在特定实现中，可以设计为幂等的（例如：设置字段值、使用条件请求等）。

## 规则 10: HTTP 状态码

### 成功状态码

| 状态码 | 含义       | 使用场景                  |
| ------ | ---------- | ------------------------- |
| 200    | OK         | GET, PUT, PATCH 成功      |
| 201    | Created    | POST 创建成功             |
| 204    | No Content | DELETE 成功，或无返回内容 |

### 客户端错误状态码

| 状态码 | 含义                 | 使用场景               |
| ------ | -------------------- | ---------------------- |
| 400    | Bad Request          | 请求格式错误           |
| 401    | Unauthorized         | 未认证                 |
| 403    | Forbidden            | 无权限                 |
| 404    | Not Found            | 资源不存在             |
| 409    | Conflict             | 资源冲突               |
| 422    | Unprocessable Entity | 请求格式正确但语义错误 |

### 服务器错误状态码

| 状态码 | 含义                  | 使用场景       |
| ------ | --------------------- | -------------- |
| 500    | Internal Server Error | 服务器内部错误 |
| 503    | Service Unavailable   | 服务不可用     |

## 规则 11: 响应格式

### 成功响应格式

**单一资源响应**

```json
{
  "data": {
    "id": 123,
    "name": "张三",
    "email": "zhangsan@example.com",
    "createdAt": "2024-01-01T12:00:00Z"
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "v1"
  }
}
```

**列表资源响应**

```json
{
  "data": [
    {
      "id": 123,
      "name": "张三"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 50,
      "totalPages": 5
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "v1"
  }
}
```

### 错误响应格式

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数验证失败",
    "details": {
      "field": "email",
      "reason": "邮箱格式不正确"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  }
}
```

### 核心原则

- **data**: 包含实际业务数据
- **meta**: 包含元数据信息（时间戳、版本、分页等）
- **error**: 包含错误信息（错误码、消息、详情等）
- 字段命名保持一致的风格（camelCase 或 snake_case）
- 时间字段统一使用 ISO 8601 格式
- 错误代码使用分层结构：`{CATEGORY}_{SPECIFIC_ERROR}`

### 常见错误代码分类

- `AUTH_*`: 认证相关错误
- `PERMISSION_*`: 权限相关错误
- `VALIDATION_*`: 验证相关错误
- `BUSINESS_*`: 业务逻辑错误
- `SYSTEM_*`: 系统错误

## 规则 12: URL 长度和嵌套限制

### URL 长度规则

- 总长度不超过 2048 字符
- 路径段不超过 255 字符
- 查询字符串不超过 1024 字符

### 嵌套深度限制

```
✅ 推荐（2-3层）：
/api/v1/users/123/posts
/api/v1/users/123/posts/456/comments

❌ 过深（避免超过4层）：
/api/v1/companies/123/departments/456/teams/789/members/101/skills
```

### 替代方案

对于深层嵌套，使用查询参数或独立端点：

```
# 替代深层嵌套
GET /api/v1/members?companyId=123&departmentId=456&teamId=789

# 或者使用独立端点
GET /api/v1/team-members/789
```

## 规则 13: 批量操作

### 批量创建

```
POST /api/v1/users/batch
[
  {
    "name": "张三",
    "email": "zhangsan@example.com"
  },
  {
    "name": "李四",
    "email": "lisi@example.com"
  }
]
```

### 批量更新

```
PATCH /api/v1/users/batch
[
  {
    "id": 123,
    "name": "张三新"
  },
  {
    "id": 124,
    "status": "inactive"
  }
]
```

### 批量删除

```
DELETE /api/v1/users/batch
{
  "ids": [123, 124, 125]
}
```

### 批量操作响应

```json
{
  "data": {
    "success": [
      {
        "id": 123,
        "status": "updated"
      }
    ],
    "failed": [
      {
        "id": 124,
        "error": "用户不存在"
      }
    ]
  },
  "meta": {
    "totalCount": 2,
    "successCount": 1,
    "failedCount": 1
  }
}
```

## 规则 14: 分页规范

### 基础分页参数

```
GET /api/v1/users?page=1&limit=10
```

### 分页响应

```json
{
  "data": [...],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 50,
      "totalPages": 5,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```
