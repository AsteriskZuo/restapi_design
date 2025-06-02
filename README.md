# REST API 设计规范文档

## 目录

1. [REST API 基础概念](01-基础概念.md)
2. [核心设计原则](#核心设计原则)
3. [URL 设计规范](02-URL设计规范.md)
4. [HTTP 方法使用指南](03-HTTP方法和状态码.md)
5. [状态码使用规范](03-HTTP方法和状态码.md#http-状态码规范)
6. [请求和响应格式](04-响应格式和错误处理.md)
7. [错误处理机制](04-响应格式和错误处理.md#错误代码规范)
8. [版本控制策略](#版本控制策略)
9. [安全性考虑](#安全性考虑)
10. [性能优化](05-最佳实践和示例.md#性能优化最佳实践)
11. [常见反模式和避免方法](#常见反模式和避免方法)
12. [示例和最佳实践](05-最佳实践和示例.md)

---

## 概述

本文档旨在提供一套全面的REST API设计规范，帮助开发团队构建一致性、可维护、易用的API接口。

### 为什么需要规范？

- **一致性**: 统一的设计模式让API更容易理解和使用
- **可维护性**: 清晰的规范降低维护成本
- **开发效率**: 减少设计决策时间，加快开发速度
- **用户体验**: 直观的API设计提升开发者体验

### 反面案例分析

你提到的这个URL设计：`https://xxx.domain.com/user1/add_contact/user2`

**问题分析：**
1. 将动态参数直接嵌入URL路径中
2. 当参数为空时URL结构破坏
3. 语义不清晰，不符合REST资源导向的设计
4. 难以扩展和维护

**正确的设计应该是：**
```
POST /api/v1/users/{userId}/contacts
Body: { "contactUserId": "user2" }
```

这种设计避免了空参数问题，并且语义清晰。

---

## 核心设计原则

### 1. 资源导向 (Resource-Oriented)

REST API应该围绕资源设计，而不是操作：

```
✅ 正确：
GET /api/v1/users/123        # 获取用户资源
POST /api/v1/posts           # 创建文章资源
DELETE /api/v1/comments/456  # 删除评论资源

❌ 错误：
GET /api/v1/getUser?id=123   # 面向操作的设计
POST /api/v1/createPost      # 动词形式
DELETE /api/v1/deleteComment/456  # 冗余的动词
```

### 2. 统一接口 (Uniform Interface)

使用标准的HTTP方法和状态码：

- **GET**: 获取资源（安全、幂等、可缓存）
- **POST**: 创建资源（非安全、非幂等）
- **PUT**: 完整更新资源（非安全、幂等）
- **PATCH**: 部分更新资源（非安全、通常非幂等）
- **DELETE**: 删除资源（非安全、幂等）

### 3. 无状态 (Stateless)

每个请求必须包含处理该请求所需的所有信息：

```
✅ 正确：
GET /api/v1/users/123/posts?page=2&limit=10
Authorization: Bearer token123

❌ 错误：
GET /api/v1/getNextPage  # 依赖服务器端状态
```

### 4. 可缓存 (Cacheable)

适当使用HTTP缓存机制：

```http
# 响应头
Cache-Control: public, max-age=3600
ETag: "resource-version-123"
Last-Modified: Tue, 01 Jan 2024 12:00:00 GMT

# 条件请求
If-None-Match: "resource-version-123"
If-Modified-Since: Tue, 01 Jan 2024 12:00:00 GMT
```

## 版本控制策略

### 1. URL版本控制（推荐）

```
/api/v1/users    # 版本1
/api/v2/users    # 版本2
/api/v3/users    # 版本3
```

**优点：**
- 简单明了
- 易于理解和实现
- 支持不同版本的独立缓存

### 2. Header版本控制

```http
Accept: application/vnd.api+json;version=1
X-API-Version: 2
```

**适用场景：**
- 需要保持URL稳定
- 版本变化较少
- 客户端可以灵活处理版本头

### 3. 查询参数版本控制

```
/api/users?version=1
```

**不推荐原因：**
- 缓存复杂
- URL变得不整洁
- 容易被忽略

### 版本兼容性原则

1. **向后兼容**: 新版本应保持向后兼容
2. **渐进弃用**: 提前通知API废弃计划
3. **迁移指南**: 提供版本升级指导
4. **并行支持**: 同时支持多个版本

## 安全性考虑

### 1. 认证 (Authentication)

```http
# JWT Bearer Token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# API Key
X-API-Key: your-secret-api-key

# Basic Auth (仅HTTPS)
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

### 2. 授权 (Authorization)

```json
{
  "error": {
    "code": "PERMISSION_DENIED",
    "message": "权限不足",
    "details": {
      "resource": "users",
      "action": "delete",
      "requiredRole": "admin",
      "currentRole": "user"
    }
  }
}
```

### 3. 输入验证

```javascript
// 验证规则示例
{
  "email": {
    "required": true,
    "type": "email",
    "maxLength": 255
  },
  "age": {
    "type": "integer",
    "minimum": 0,
    "maximum": 150
  }
}
```

### 4. 频率限制

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
Retry-After: 3600
```

### 5. HTTPS和数据加密

- 所有API必须使用HTTPS
- 敏感数据传输加密
- 避免在URL中传递敏感信息

## 常见反模式和避免方法

### 1. URL设计反模式

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

### 2. 参数传递反模式

```
❌ 关键参数直接嵌入URL：
GET /api/v1/user1/add_contact/user2

✅ 正确的参数传递：
POST /api/v1/users/user1/contacts
Body: { "contactUserId": "user2" }
```

### 3. 错误处理反模式

```
❌ 不一致的错误格式：
HTTP 200 OK
{
  "success": false,
  "error": "User not found"
}

✅ 正确的错误处理：
HTTP 404 Not Found
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "用户不存在",
    "requestId": "req-123"
  }
}
```

### 4. 状态码滥用反模式

```
❌ 所有响应都返回200：
HTTP 200 OK
{
  "status": "error",
  "message": "User not found"
}

✅ 正确使用HTTP状态码：
HTTP 404 Not Found
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "用户不存在"
  }
}
```

## 开发工具和资源

### 1. 设计工具

- **Swagger/OpenAPI**: API设计和文档
- **Postman**: API测试和调试
- **Insomnia**: REST客户端
- **REST Client**: VS Code扩展

### 2. 验证工具

- **JSON Schema**: 数据结构验证
- **Spectral**: OpenAPI规范检查
- **Newman**: Postman集合自动化测试

### 3. 监控工具

- **API Gateway**: 流量管理和监控
- **ELK Stack**: 日志分析
- **Prometheus**: 性能监控

## 快速开始清单

设计新API时，请检查以下项目：

### URL设计 ✓
- [ ] 使用名词而不是动词
- [ ] 使用复数形式的资源名
- [ ] 使用小写字母和连字符
- [ ] 避免在URL中嵌入可变参数
- [ ] 合理的资源层级（不超过3-4层）

### HTTP方法 ✓
- [ ] GET用于获取资源
- [ ] POST用于创建资源
- [ ] PUT用于完整更新
- [ ] PATCH用于部分更新
- [ ] DELETE用于删除资源

### 响应格式 ✓
- [ ] 统一的JSON响应格式
- [ ] 包含适当的元数据
- [ ] 一致的错误响应结构
- [ ] 使用camelCase字段命名

### 状态码 ✓
- [ ] 200 用于成功的GET/PUT/PATCH
- [ ] 201 用于成功的POST创建
- [ ] 204 用于成功的DELETE
- [ ] 400 用于请求错误
- [ ] 401 用于认证失败
- [ ] 403 用于权限不足
- [ ] 404 用于资源不存在
- [ ] 500 用于服务器错误

### 安全性 ✓
- [ ] 使用HTTPS
- [ ] 实现适当的认证机制
- [ ] 添加权限控制
- [ ] 输入验证和过滤
- [ ] 频率限制

---

## 结语

良好的REST API设计是一门艺术，需要在简洁性、功能性和可维护性之间找到平衡。这份规范文档提供了经过实践验证的最佳实践，特别是解决了你提到的URL参数为空导致的问题。

通过遵循这些规范，你可以：

1. **避免常见陷阱**: 如URL设计问题、状态码滥用等
2. **提高开发效率**: 减少设计决策时间
3. **改善用户体验**: 让API更容易理解和使用
4. **降低维护成本**: 一致的设计降低维护复杂度

记住，规范不是死板的规则，而是指导原则。在特定场景下，可能需要根据实际需求进行适当调整，但应该有充分的理由和文档说明。

希望这份规范能帮助你设计出优秀的REST API！ 