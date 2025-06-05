# HTTP 方法使用指南和状态码规范

## HTTP 方法详解

### 1. GET - 获取资源

**用途**: 获取资源，不应有副作用

**特性**:
- 安全 (Safe): 不修改服务器状态
- 幂等 (Idempotent): 多次调用结果相同
- 可缓存 (Cacheable)

**使用场景**:
```
GET /api/v1/users              # 获取所有用户
GET /api/v1/users/123          # 获取特定用户
GET /api/v1/users/123/posts    # 获取用户的文章
GET /api/v1/posts?status=published&page=1  # 带过滤条件
```

**最佳实践**:
```
✅ 正确：
GET /api/v1/users?page=1&limit=10&sort=name

❌ 错误：
GET /api/v1/deleteUser/123     # 不应该有副作用
GET /api/v1/users              # 返回所有数据而不分页
```

### 2. POST - 创建资源

**用途**: 创建新资源，处理数据

**特性**:
- 不安全: 会修改服务器状态
- 非幂等: 重复调用可能产生不同结果
- 不可缓存

**使用场景**:
```
POST /api/v1/users             # 创建新用户
POST /api/v1/users/123/posts   # 为用户创建文章
POST /api/v1/orders/123/payments  # 处理订单支付
POST /api/v1/users/search      # 复杂搜索（查询条件在body中）
```

**请求示例**:
```
POST /api/v1/users
Content-Type: application/json

{
  "name": "张三",
  "email": "zhangsan@example.com",
  "age": 25
}
```

**响应示例**:
```
HTTP/1.1 201 Created
Location: /api/v1/users/124
Content-Type: application/json

{
  "id": 124,
  "name": "张三",
  "email": "zhangsan@example.com",
  "age": 25,
  "createdAt": "2024-01-01T12:00:00Z"
}
```

### 3. PUT - 完整更新资源

**用途**: 完整替换现有资源

**特性**:
- 不安全: 会修改服务器状态
- 幂等: 多次调用结果相同
- 不可缓存

**使用场景**:
```
PUT /api/v1/users/123          # 完整更新用户信息
PUT /api/v1/posts/456          # 完整更新文章内容
```

**请求示例**:
```
PUT /api/v1/users/123
Content-Type: application/json

{
  "name": "李四",
  "email": "lisi@example.com",
  "age": 30,
  "status": "active"
}
```

**注意事项**:
- 必须提供资源的完整数据
- 缺失的字段会被设置为默认值或null
- 如果资源不存在，某些实现可能会创建新资源

### 4. PATCH - 部分更新资源

**用途**: 部分更新现有资源

**特性**:
- 不安全: 会修改服务器状态
- 非幂等 (通常)
- 不可缓存

**使用场景**:
```
PATCH /api/v1/users/123        # 部分更新用户信息
PATCH /api/v1/posts/456        # 更新文章的某些字段
```

**请求示例**:
```
PATCH /api/v1/users/123
Content-Type: application/json

{
  "age": 31,
  "status": "inactive"
}
```

**JSON Patch 格式**:
```
PATCH /api/v1/users/123
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/age", "value": 31 },
  { "op": "add", "path": "/skills", "value": ["Java", "Python"] },
  { "op": "remove", "path": "/temporaryField" }
]
```

### 5. DELETE - 删除资源

**用途**: 删除资源

**特性**:
- 不安全: 会修改服务器状态
- 幂等: 多次删除同一资源结果相同
- 不可缓存

**使用场景**:
```
DELETE /api/v1/users/123       # 删除特定用户
DELETE /api/v1/posts/456       # 删除特定文章
DELETE /api/v1/users/123/posts/456  # 删除用户的特定文章
```

**响应示例**:
```
HTTP/1.1 204 No Content
```

或者返回删除的资源信息：
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "张三",
  "deletedAt": "2024-01-01T12:00:00Z"
}
```

### 6. HEAD - 获取资源元信息

**用途**: 获取资源的元信息，不返回实体内容

**特性**:
- 安全
- 幂等
- 可缓存

**使用场景**:
```
HEAD /api/v1/users/123         # 检查用户是否存在
HEAD /api/v1/files/image.jpg   # 获取文件大小和修改时间
```

### 7. OPTIONS - 获取资源支持的方法

**用途**: 获取资源支持的HTTP方法

**特性**:
- 安全
- 幂等
- 可缓存

**响应示例**:
```
OPTIONS /api/v1/users/123
HTTP/1.1 200 OK
Allow: GET, PUT, PATCH, DELETE
Access-Control-Allow-Methods: GET, PUT, PATCH, DELETE
```

## HTTP 状态码规范

### 1xx 信息性状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 100 | Continue | 客户端应继续发送请求 |
| 101 | Switching Protocols | 协议切换 |

### 2xx 成功状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 200 | OK | GET, PUT, PATCH 成功 |
| 201 | Created | POST 创建成功 |
| 202 | Accepted | 请求已接受，异步处理 |
| 204 | No Content | DELETE 成功，或 PUT/PATCH 无返回内容 |
| 206 | Partial Content | 范围请求成功 |

**详细说明**:

#### 200 OK
```
GET /api/v1/users/123
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "张三"
}
```

#### 201 Created
```
POST /api/v1/users
HTTP/1.1 201 Created
Location: /api/v1/users/124
Content-Type: application/json

{
  "id": 124,
  "name": "张三",
  "createdAt": "2024-01-01T12:00:00Z"
}
```

#### 202 Accepted
```
POST /api/v1/reports/generate
HTTP/1.1 202 Accepted
Content-Type: application/json

{
  "taskId": "task-123",
  "status": "processing",
  "estimatedTime": "5 minutes"
}
```

#### 204 No Content
```
DELETE /api/v1/users/123
HTTP/1.1 204 No Content
```

### 3xx 重定向状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 301 | Moved Permanently | 资源永久移动 |
| 302 | Found | 资源临时移动 |
| 304 | Not Modified | 缓存有效 |
| 307 | Temporary Redirect | 临时重定向，保持方法 |
| 308 | Permanent Redirect | 永久重定向，保持方法 |

### 4xx 客户端错误状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 400 | Bad Request | 请求格式错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 405 | Method Not Allowed | HTTP方法不支持 |
| 409 | Conflict | 资源冲突 |
| 422 | Unprocessable Entity | 请求格式正确但语义错误 |
| 429 | Too Many Requests | 请求过于频繁 |

**详细说明**:

#### 400 Bad Request
```
POST /api/v1/users
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数验证失败",
    "details": [
      {
        "field": "email",
        "message": "邮箱格式不正确"
      }
    ]
  }
}
```

#### 401 Unauthorized
```
GET /api/v1/users/123
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer
Content-Type: application/json

{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "需要有效的访问令牌"
  }
}
```

#### 403 Forbidden
```
DELETE /api/v1/users/123
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "error": {
    "code": "FORBIDDEN",
    "message": "您没有权限删除此用户"
  }
}
```

#### 404 Not Found
```
GET /api/v1/users/999
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "error": {
    "code": "NOT_FOUND",
    "message": "用户不存在"
  }
}
```

#### 409 Conflict
```
POST /api/v1/users
HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "error": {
    "code": "CONFLICT",
    "message": "用户名已存在",
    "details": {
      "conflictField": "username",
      "conflictValue": "zhangsan"
    }
  }
}
```

#### 422 Unprocessable Entity
```
POST /api/v1/users
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "数据验证失败",
    "details": [
      {
        "field": "age",
        "message": "年龄必须在18-100之间",
        "code": "OUT_OF_RANGE"
      }
    ]
  }
}
```

### 5xx 服务器错误状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 500 | Internal Server Error | 服务器内部错误 |
| 501 | Not Implemented | 功能未实现 |
| 502 | Bad Gateway | 网关错误 |
| 503 | Service Unavailable | 服务不可用 |
| 504 | Gateway Timeout | 网关超时 |

**详细说明**:

#### 500 Internal Server Error
```
GET /api/v1/users/123
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "服务器内部错误",
    "requestId": "req-123456"
  }
}
```

#### 503 Service Unavailable
```
GET /api/v1/users
HTTP/1.1 503 Service Unavailable
Retry-After: 60
Content-Type: application/json

{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "服务临时不可用，请稍后重试"
  }
}
```

## 方法选择决策树

```
是否修改服务器状态？
├── 否 → GET (获取资源)
└── 是 → 
    ├── 创建新资源？ → POST
    ├── 完整替换现有资源？ → PUT
    ├── 部分更新现有资源？ → PATCH
    └── 删除资源？ → DELETE
```

## 状态码选择指南

### 成功场景
- 获取数据成功: 200
- 创建成功: 201
- 异步处理接受: 202
- 操作成功无返回内容: 204

### 客户端错误
- 请求格式问题: 400
- 认证问题: 401
- 权限问题: 403
- 资源不存在: 404
- 方法不支持: 405
- 数据冲突: 409
- 业务逻辑错误: 422
- 频率限制: 429

### 服务器错误
- 程序异常: 500
- 功能未实现: 501
- 服务不可用: 503

## 常见误用和最佳实践

### 误用案例1: 用GET进行数据修改
```
❌ 错误：
GET /api/v1/users/123/activate

✅ 正确：
PATCH /api/v1/users/123
{
  "status": "active"
}
```

### 误用案例2: 错误的状态码
```
❌ 错误：
POST /api/v1/users
HTTP/1.1 200 OK  # 创建应该返回201

✅ 正确：
POST /api/v1/users
HTTP/1.1 201 Created
Location: /api/v1/users/124
```

### 误用案例3: PUT vs PATCH
```
❌ 错误使用PUT进行部分更新：
PUT /api/v1/users/123
{
  "name": "新名字"
}
# 这会把其他字段设为null

✅ 正确使用PATCH：
PATCH /api/v1/users/123
{
  "name": "新名字"
}
# 只更新指定字段
``` 