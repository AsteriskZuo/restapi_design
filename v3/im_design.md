# IM REST 设计规范

本文档基于 `RFC9110`、`RFC9205` 等国际标准，结合 `Claude-4-Sonnet` 的最佳实践，深度分析 IM 业务场景特点，形成了这套较为完整的可落地的规范建议方案。

## 1. API 版本

URL 路径中明确指定版本号，支持向后兼容。

**URL 带有版本的示例：**

```http
# ✅ 正确:
https://api.easemob.com/v1/myorg/myapp/users // 1.0版本
https://api.easemob.com/v2/myorg/myapp/users // 2.0版本

# ❌ 错误:
https://api.easemob.com/v1.2/myorg/myapp/users // 不支持次要版本号
https://api.easemob.com/v1.2.3/myorg/myapp/users // 不支持补丁版本号
```

**详细规范**：完整的 URL 设计规范请参考 [URL 规范文档](./im_url_specification.md)

## 2. 服务无状态

服务不保存客户端会话状态，每个请求包含所有必要信息，提高可扩展性和可靠性。

```http
# ✅ 正确:
GET /api/v1/users/123/posts?page=2&limit=10
authorization: Bearer token123

# ❌ 错误:
GET /api/v1/getNextPage  # 依赖服务器端状态
```

## 3. 安全规范

涵盖认证授权、数据安全、访问控制、传输安全和审计监控等核心安全方面。

详细设计请参考 [安全规范文档](./im_security_overview.md)

## 4. 速率限制规则

实施基于身份、IP 和资源的多层级限流等策略，保障系统稳定性和可靠性。

详细设计请参考 [速率限制文档](./im_rate_limiting.md)

## 5. HTTP 方法选择

**CRUD 操作映射**

- **增加(Create)**: POST
- **查询(Read)**: GET
- **修改(Update)**: PUT（完整更新）或 PATCH（部分更新）
- **删除(Delete)**: DELETE

**HTTP 方法特性**

| 方法   | 幂等性     | 缓存性     | 主要用途 | 说明                         |
| ------ | ---------- | ---------- | -------- | ---------------------------- |
| GET    | 幂等       | 可缓存     | 获取资源 | 不应有副作用                 |
| POST   | 通常非幂等 | 通常不缓存 | 创建资源 | 可通过幂等键设计为幂等       |
| PUT    | 幂等       | 条件缓存   | 完整更新 | 响应可在某些情况下缓存       |
| PATCH  | 取决于实现 | 通常不缓存 | 部分更新 | 设置操作幂等，增量操作非幂等 |
| DELETE | 幂等       | 条件缓存   | 删除资源 | 重复删除不应报错             |

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

## 6. URL 规范

URL 设计遵循 RESTful 原则，确保资源定位的准确性和可读性。

**示例讲解：**

```http
https://api.easemob.com/v1/myorg/myapp/users           // 正式环境
https://api.dev-a1.easemob.com/v1/myorg/myapp/users    // 沙箱环境
```

- **协议**：https:// - 使用安全的 HTTPS 协议
- **主机**：
  - 正式环境：api.easemob.com
  - 沙箱环境：api.dev-a1.easemob.com
- **版本**：/v1 - API 版本号
- **组织名**：/myorg - 租户标识
- **应用名**：/myapp - 应用标识
- **资源路径**：/users - 资源层级

**详细规范**：完整的 URL 设计规范请参考 [URL 规范文档](./im_url_specification.md)

## 7. 认证方式选择

推荐使用 JWT Bearer Token，符合 RFC 6750 规范，支持无状态认证和分布式部署。

```http
authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 8. HTTP 头部要求

规范 HTTP 头部使用，确保客户端和服务端正确处理请求内容格式、编码和认证信息。

**基础请求头**

```http
accept: application/json; charset=utf-8
user-agent: YourApp/1.0 // 格式和内容，仅供参考
```

**可选请求头**

```http:
accept-encoding: gzip, deflate
```

**带请求体时**

```http
content-type: application/json; charset=utf-8
```

**需要认证时**

```http
authorization: Bearer token123
```

**响应头格式**

```http
content-type: application/json; charset=utf-8
```

## 9. 参数规范

统一参数传递方式，包括路径参数、查询参数、请求体参数等，支持 UTF-8 编码。

详细设计请参考 [参数规范文档](./im_parameter_specification.md)

## 10. 命名规范

统一命名风格，包括 URL、参数、字段等命名规则。

**示例：**

```http
# URL 命名
GET /api/v1/users/{userId}/messages
GET /api/v1/users/{userId}/user_profile

# 请求参数命名
GET /api/v1/users?status=active&sort=created_at:desc

# 请求体参数命名
{
  "userName": "zhangsan",
  "nickName": "张三"
}

# 响应参数命名
{
  "data": {
    "userId": "123",
    "userName": "zhangsan"
  }
}
```

详细设计请参考 [命名规范文档](./im_naming_conventions.md)

## 10. 响应体格式设计

采用标准响应结构，确保数据的一致性和可扩展性，支持分页等高级特性。

**响应成功示例：**

```json
{
  "data": {
    "userId": 123,
    "username": "zhangsan",
    "nickname": "张三"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "usr1704110400123abc456def78901"
  }
}
```

**响应失败示例：**

```json
{
  "error": {
    "code": "40042020301",
    "type": "USER_NOT_FOUND",
    "message": "用户不存在"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "usr1704110400234def567ghi89012"
  }
}
```

**详细规范**：完整的响应格式设请参考 [响应格式设计文档](./im_response_format.md)

## 11. 错误码规范

采用 7 位错误码结构，确保错误信息的准确性和可追踪性。

**示例：**

```json
{
  "error": {
    "code": "4000301",
    "type": "USER_NOT_FOUND",
    "message": "用户不存在"
  }
}
```

**错误码解析**：`4000301` = `400`(请求错误) + `0301`(用户不存在)

**详细规范**：完整的错误码请参考 [错误码设计文档](./im_error_code.md)

## 12. 查询：排序设计

支持单字段和多字段排序，使用 `sort` 参数指定排序字段和方向。

**示例：**

```http
# 单字段升序
GET /api/v1/users?sort=created_at:asc

# 多字段降序
GET /api/v1/users?sort=status:desc,created_at:desc
```

详细设计请参考 [排序规范文档](./im_sort_specification.md)

## 13. 查询：分页设计

支持偏移分页和游标分页两种方式，适用于不同场景。

**示例：**

```http
# 偏移分页
GET /api/v1/users?page=1&size=20

# 游标分页
GET /api/v1/users?cursor=eyJpZCI6IjEyMyJ9&limit=20
```

详细设计请参考 [分页规范文档](./im_pagination_specification.md)

## 14. 查询：搜索设计

支持基础搜索、高级搜索、全文搜索和语义搜索等多种搜索方式。

**示例：**

```http
# 单值搜索（简单搜索模式）
GET /api/v1/users?status=active
GET /api/v1/messages?chat_type=single

# 多值搜索（简单搜索模式）
GET /api/v1/users?status=active,pending
GET /api/v1/groups?is_public=true&name=*技术*

# 高级搜索（复杂搜索模式）
GET /api/v1/users?filter=status==active;age=ge=18
GET /api/v1/messages?filter=(chat_type==single,chat_type==group);content==*重要*

# 搜索+排序
GET /api/v1/users?status=active&sort=created_at:desc
GET /api/v1/groups?filter=is_public==true;member_count=ge=10&sort=activity:desc,created_at:desc
GET /api/v1/messages?chat_id=123&sort=created_at:desc
```

详见 [搜索规范文档](./im_search_specification.md)

详见 [排序规范文档](./im_sort_specification.md)

## 15. 批量操作

支持原子性和非原子性批量操作，适用于创建、更新、删除和获取等场景。

**示例：**

```http
# 批量创建
POST /api/v1/users/batch
{
  "batch_mode": "atomic",
  "data": [
    { "username": "user1", "email": "user1@example.com" },
    { "username": "user2", "email": "user2@example.com" }
  ]
}

# 批量更新
PUT /api/v1/users/batch
{
  "batch_mode": "non_atomic",
  "data": [
    { "id": "user1", "status": "active" },
    { "id": "user2", "status": "inactive" }
  ]
}

# 批量删除
DELETE /api/v1/users/batch?ids=user1,user2,user3&batch_mode=atomic

# 批量获取
POST /api/v1/users/batch/get
{
  "batch_mode": "non_atomic",
  "data": ["id1", "id2", "id3"]
}
```

详细设计请参考 [批量操作规范文档](./im_batch_specification.md)
