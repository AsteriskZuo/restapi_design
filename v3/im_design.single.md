# IM REST 设计规范

本文档基于 `RFC9110`、`RFC9205` 等国际标准，结合 `Claude-4-Sonnet` 的最佳实践，深度分析 IM 业务场景特点，形成了这套较为完整的可落地的规范建议方案。

## 1. 版本

URL 路径中明确指定版本号，支持向后兼容。

**URL 结构说明：**

- **host**: API 服务域名
  - 正式环境：api.easemob.com
  - 沙箱环境：api.dev-{env}.easemob.com
- **version**: API 版本号（v1/v2 等）
- **org_name**: 组织名称（租户标识）
  - 用于多租户隔离
  - 支持跨组织数据访问控制
- **app_name**: 应用名称（应用标识）
  - 用于多应用隔离
  - 支持应用级别的配置管理

**环信 REST 接口示例**

```http
https://{host}/{version}/{org_name}/{app_name}/auth
https://{host}/{version}/{org_name}/{app_name}/users
https://{host}/{version}/{org_name}/{app_name}/groups
https://{host}/{version}/{org_name}/{app_name}/rooms
https://{host}/{version}/{org_name}/{app_name}/messages
https://{host}/{version}/{org_name}/{app_name}/messages/threads
https://{host}/{version}/{org_name}/{app_name}/messages/reactions
https://{host}/{version}/{org_name}/{app_name}/push
```

**完整 URL 示例：**

```http
https://api.easemob.com/v1/myorg/myapp/users // 正式
https://api.dev-a1.easemob.com/v1/myorg/myapp/users // 沙箱
https://api.dev-a61.easemob.com/v1/myorg/myapp/users // 沙箱
```

## 2. HTTP 方法选择

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

## 3. 无状态设计

服务不保存客户端会话状态，每个请求包含所有必要信息，提高可扩展性和可靠性。

```http
# ✅ 正确:
GET /api/v1/users/123/posts?page=2&limit=10
authorization: Bearer token123

# ❌ 错误:
GET /api/v1/getNextPage  # 依赖服务器端状态
```

## 4. 认证方式

推荐使用 JWT Bearer Token，符合 RFC 6750 规范，支持无状态认证和分布式部署。

```http
authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 5. HTTP 头部

规范 HTTP 头部使用，确保客户端和服务端正确处理请求内容格式、编码和认证信息。

### 基础请求头

```http
Accept: application/json
User-Agent: YourApp/1.0
```

### 带请求体时

```http
content-type: application/json; charset=utf-8
```

### 需要认证时

```http
authorization: Bearer token123
```

### 响应头格式

```http
content-type: application/json; charset=utf-8
```

## 6. 响应格式

采用标准响应结构，确保数据的一致性和可扩展性，支持 HATEOAS 和分页等高级特性。

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

**详细规范**：完整的响应格式设计和 IM 业务场景示例请参考 [响应格式设计文档](./im_response_format.md)

## 7. URL 资源命名

URL 使用名词表示资源，通过 HTTP 方法表达操作意图，符合 RESTful 设计原则。

```http
# ❌ 错误：动词形式的 URL
GET /api/v1/getUsers
POST /api/v1/createUser
PUT /api/v1/updateUser/123
DELETE /api/v1/deleteUser/123

# ✅ 正确：名词形式 + HTTP 方法
GET /api/v1/users           # 获取用户列表
POST /api/v1/users          # 创建用户
PUT /api/v1/users/123       # 更新用户
DELETE /api/v1/users/123    # 删除用户
```

## 8. 数据传递

URL 使用查询参数传递简单数据，请求体传递复杂数据，确保数据传递的安全性和高效性。

_这部分已经包括了批量处理和搜索等内容_

**查询参数示例：**

```http
GET /api/v1/users?status=active&page=1&limit=10&sort=created_at&order=desc
```

**请求体示例：**

```http
POST /api/v1/users
content-type: application/json

{
  "username": "zhangsan",
  "email": "zhangsan@example.com",
  "profile": {
    "age": 25,
    "interests": ["技术", "读书"]
  }
}
```

**详细规范**：完整的数据传递规范和 IM 业务场景示例请参考 [数据传递规范文档](./im_data_transfer_specification.md)

## 9. 响应错误码

采用 11 位错误码结构，确保错误信息的准确性和可追踪性，支持快速定位问题和统一错误处理。

```json
{
  "error": {
    "code": "40042020301",
    "type": "USER_NOT_FOUND",
    "message": "用户不存在",
    "details": {
      "field": "userId",
      "value": "123"
    }
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "usr1704110400234def567ghi89012"
  }
}
```

**错误码解析**：`40042020301` = `400`(请求错误) + `42`(业务逻辑模块) + `02`(用户子模块) + `0301`(用户不存在)

**详细规范**：完整的错误码分类和 IM 业务场景示例请参考 [错误码设计文档](./im_detail_error_format.md)

## 100. IM 业务分类建议

- auth: 用户权限认证管理。
- users: 用户管理。
- groups: 群组管理。
- rooms: 聊天室管理。
- messages: 消息管理。
- push: 推送通知管理。
- others

当前分类：

表示用户的关键字有： account、user、contact
表示群组的关键字：group
表示聊天室的关键字：chatroom（群组不带 chat）
表示消息的关键字：message、reaction（消息子类）、thread（消息子类）等
