# 基本规则

## 规则 1: 指定版本号

```
/api/v1/users    # 版本1
/api/v2/users    # 版本2
/api/v3/users    # 版本3
```

## 规则 2: HTTP 方法选择指南

### CRUD 操作映射

- **增加(Create)**: POST
- **查询(Read)**: GET
- **修改(Update)**: PUT（完整更新）或 PATCH（部分更新）
- **删除(Delete)**: DELETE

### HTTP 方法特性

| 方法   | 安全性 | 幂等性     | 缓存性     | 主要用途 | 说明                         |
| ------ | ------ | ---------- | ---------- | -------- | ---------------------------- |
| GET    | 安全   | 幂等       | 可缓存     | 获取资源 | 不应有副作用                 |
| POST   | 不安全 | 通常非幂等 | 通常不缓存 | 创建资源 | 可通过幂等键设计为幂等       |
| PUT    | 不安全 | 幂等       | 条件缓存   | 完整更新 | 响应可在某些情况下缓存       |
| PATCH  | 不安全 | 取决于实现 | 通常不缓存 | 部分更新 | 设置操作幂等，增量操作非幂等 |
| DELETE | 不安全 | 幂等       | 条件缓存   | 删除资源 | 重复删除不应报错             |

**安全性说明**:

- **安全**：不会修改服务器状态，可以安全地重复调用
- **不安全**：可能修改服务器状态

**幂等性说明**:

- **幂等**：多次调用产生相同结果
- **非幂等**：多次调用可能产生不同结果
- **取决于实现**：根据具体操作语义确定

**缓存性说明**:

- **可缓存**：响应可以被缓存
- **不缓存**：响应不应被缓存
- **条件缓存**：在特定条件下可以缓存

## 规则 3: 服务不保存状态

```
✅ 正确:
GET /api/v1/users/123/posts?page=2&limit=10
Authorization: Bearer token123

❌ 错误:
GET /api/v1/getNextPage  # 依赖服务器端状态
```

## 规则 4: 认证方式

认证方式有很多，可以参考 RFC 6750 规范。

推荐的认证方式是 采用 JWT Bearer Token 进行调用验证。

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 规则 4.1: 标准 HTTP 头部

### 必须包含的请求头

```http
Accept: application/json
Content-Type: application/json; charset=utf-8
User-Agent: YourApp/1.0
Authorization: Bearer token123
```

### 推荐的请求头

```http
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Accept-Encoding: gzip, deflate
Cache-Control: no-cache    # 对于敏感请求
```

### 标准响应头

```http
Content-Type: application/json; charset=utf-8
Content-Language: zh-CN
Content-Encoding: gzip     # 启用压缩时
X-Request-ID: req-123456789
X-Response-Time: 123ms
```

## 规则 5: 响应格式

相应格式有多种，可以参考 RFC 9110 规范。

推荐的响应格式是 JSON Schema 格式。

### 成功响应格式

```json
{
  "data": {
    "id": 123,
    "name": "张三",
    "email": "zhangsan@example.com"
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "v1"
  }
}
```

详细内容，请参考 [response_format_standard.md](./response_format_standard.md)

### 错误响应格式

采用 **数字错误码 + 字符串错误码** 的双重机制：

```json
{
  "error": {
    "code": 40001, // 数字错误码（程序处理）
    "type": "VALIDATION_ERROR", // 字符串错误码（开发者理解）
    "message": "Request validation failed",
    "localizedMessage": {
      "zh-CN": "请求参数验证失败",
      "en-US": "Request validation failed"
    },
    "details": {
      "field": "email",
      "reason": "Invalid email format"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  }
}
```

**错误码分段规划**：

- 40000-40999: 客户端请求错误
- 41000-41999: 认证授权错误
- 42000-42999: 业务逻辑错误
- 43000-43999: 资源状态错误
- 44000-44999: 限流配额错误
- 50000-50999: 服务器内部错误

详细错误码定义和示例，请参考 [detail_error_format.md](./detail_error_format.md)。

## 规则 6: 不使用动词形式的 URL

```
❌ 动词形式的URL:
GET /api/v1/getUsers
POST /api/v1/createUser

✅ 正确的名词形式:
GET /api/v1/users
POST /api/v1/users
```

## 规则 7: 数据传递规范

**简单参数**: 使用查询参数

- 单一值：字符串、数字、布尔值
- 过滤、排序、分页等控制参数

```
GET /api/v1/users?status=active&limit=10
```

**复杂数据**: 使用请求体

- 对象结构、嵌套数据
- 数组、列表数据
- 创建/更新的完整实体

```
POST /api/v1/users
{
  "name": "张三",
  "email": "zhangsan@example.com"
}
```

## 规则 8: HTTP 状态码

### 1xx 信息性状态码

| 状态码 | 含义                | 使用场景             |
| ------ | ------------------- | -------------------- |
| 100    | Continue            | 客户端应继续发送请求 |
| 101    | Switching Protocols | 协议切换             |

### 2xx 成功状态码

| 状态码 | 含义       | 使用场景             |
| ------ | ---------- | -------------------- |
| 200    | OK         | 请求成功             |
| 201    | Created    | 资源创建成功         |
| 202    | Accepted   | 请求已接受，异步处理 |
| 204    | No Content | 请求成功，无返回内容 |

### 3xx 重定向状态码

| 状态码 | 含义               | 使用场景             |
| ------ | ------------------ | -------------------- |
| 301    | Moved Permanently  | 资源永久移动         |
| 302    | Found              | 资源临时移动         |
| 304    | Not Modified       | 缓存有效             |
| 307    | Temporary Redirect | 临时重定向，保持方法 |
| 308    | Permanent Redirect | 永久重定向，保持方法 |

### 4xx 客户端错误状态码

| 状态码 | 含义                 | 使用场景           |
| ------ | -------------------- | ------------------ |
| 400    | Bad Request          | 请求格式错误       |
| 401    | Unauthorized         | 认证失败           |
| 403    | Forbidden            | 权限不足           |
| 404    | Not Found            | 资源不存在         |
| 405    | Method Not Allowed   | HTTP 方法不支持    |
| 406    | Not Acceptable       | 不可接受的内容类型 |
| 408    | Request Timeout      | 请求超时           |
| 409    | Conflict             | 资源冲突           |
| 410    | Gone                 | 资源已永久删除     |
| 413    | Payload Too Large    | 请求体过大         |
| 415    | Unsupported Media    | 不支持的媒体类型   |
| 422    | Unprocessable Entity | 语义错误           |
| 429    | Too Many Requests    | 请求过于频繁       |

### 详细 5xx 状态码

| 状态码 | 含义            | 使用场景   |
| ------ | --------------- | ---------- |
| 501    | Not Implemented | 功能未实现 |
| 502    | Bad Gateway     | 网关错误   |
| 504    | Gateway Timeout | 网关超时   |

## 规则 9: 分页规范

```
GET /api/v1/users?page=1&limit=10
```

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

## 规则 10: 内容分类

按业务领域对资源进行分类:

```
# 用户类别
GET /api/v1/users

# 群组类别
GET /api/v1/groups

# 消息类别
GET /api/v1/messages
```

## 规则 11: 规范 URL 命名

**设计原则:**

- 使用名词而不是动词
- 使用复数形式
- 使用小写字母和连字符

```
✅ 正确:
GET /api/v1/users
GET /api/v1/user-profiles

❌ 错误:
GET /api/v1/getUsers
GET /api/v1/user_profiles
```

为什么使用连字符的原因，请参考 [url_naming_rationale.md](./url_naming_rationale.md)。
