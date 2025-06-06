# API 安全规范

> **重要性**: 最高优先级，所有生产 API 必须遵循  
> **适用范围**: 所有 API 接口和数据传输

## 概览

API 安全是保护系统和用户数据的第一道防线。本文档提供全面的安全规范，确保 API 在各种威胁下的安全性。

## 1. 传输安全

### 1.1 强制 HTTPS

**规则**: 生产环境必须使用 HTTPS，禁止 HTTP 传输

```http
# ✅ 正确 - 使用HTTPS
https://api.example.com/v1/users

# ❌ 错误 - 使用HTTP
http://api.example.com/v1/users
```

**实施要求**:

- 使用 TLS 1.2 或更高版本
- 配置 HSTS 头部强制 HTTPS
- 重定向 HTTP 到 HTTPS

```http
# 响应头配置
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### 1.2 证书要求

- 使用有效的 SSL/TLS 证书
- 证书必须覆盖所有 API 域名
- 定期更新证书，避免过期

## 2. 认证和授权

### 2.1 认证机制

**推荐方案**: JWT Bearer Token + OAuth 2.0

```http
# 标准认证头
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# API密钥认证（仅限服务间调用）
X-API-Key: your-api-key-here
```

**JWT 配置要求**:

```json
{
  "alg": "RS256", // 使用RSA签名算法
  "typ": "JWT",
  "exp": 1640995200, // 设置过期时间
  "iat": 1640908800, // 签发时间
  "iss": "your-api", // 签发者
  "aud": "your-app" // 受众
}
```

### 2.2 令牌管理

**安全要求**:

- 访问令牌有效期：15-60 分钟
- 刷新令牌有效期：7-30 天
- 支持令牌撤销机制

```http
# 令牌刷新
POST /api/v1/auth/refresh
{
  "refreshToken": "refresh-token-here"
}

# 令牌撤销
POST /api/v1/auth/revoke
{
  "token": "token-to-revoke"
}
```

### 2.3 权限控制

**基于角色的访问控制 (RBAC)**:

```json
{
  "user": {
    "id": 123,
    "roles": ["user", "editor"],
    "permissions": ["posts:read", "posts:write", "users:read"]
  }
}
```

**权限检查示例**:

```http
# 需要特定权限的操作
DELETE /api/v1/posts/123
Authorization: Bearer token123
# 需要 "posts:delete" 权限

# 403 权限不足响应
HTTP/1.1 403 Forbidden
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "缺少所需权限",
    "requiredPermissions": ["posts:delete"]
  }
}
```

## 3. 输入验证和防护

### 3.1 输入验证

**验证所有输入数据**:

```javascript
// 输入验证示例
const validateUserInput = {
  name: {
    type: "string",
    minLength: 1,
    maxLength: 100,
    pattern: "^[a-zA-Z0-9\\u4e00-\\u9fa5\\s-_]+$",
  },
  email: {
    type: "string",
    format: "email",
    maxLength: 255,
  },
  age: {
    type: "integer",
    minimum: 0,
    maximum: 150,
  },
};
```

### 3.2 SQL 注入防护

**使用参数化查询**:

```sql
-- ✅ 正确 - 参数化查询
SELECT * FROM users WHERE id = ? AND status = ?

-- ❌ 错误 - 字符串拼接
SELECT * FROM users WHERE id = ' + userId + ' AND status = 'active'
```

### 3.3 XSS 防护

**输出编码**:

```http
# 响应头防护
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

**数据输出处理**:

```json
{
  "comment": "&lt;script&gt;alert('xss')&lt;/script&gt;", // HTML编码
  "rawContent": "<script>alert('xss')</script>" // 标记为原始内容
}
```

## 4. 数据保护

### 4.1 敏感数据处理

**永不返回的字段**:

- 密码（即使是哈希值）
- 加密密钥
- 内部系统标识

**需要脱敏的字段**:

```json
{
  "user": {
    "phone": "138****1234", // 手机号脱敏
    "email": "j***@example.com", // 邮箱脱敏
    "idCard": "110***********01" // 身份证脱敏
  }
}
```

### 4.2 数据加密

**静态数据加密**:

- 数据库中的敏感字段加密
- 文件存储加密
- 备份数据加密

**传输数据加密**:

- 使用 HTTPS
- 敏感字段额外加密

```json
{
  "user": {
    "id": 123,
    "name": "张三",
    "sensitiveData": {
      "encrypted": true,
      "algorithm": "AES-256-GCM",
      "data": "encrypted-content-here"
    }
  }
}
```

## 5. 跨域资源共享 (CORS)

### 5.1 CORS 配置

**安全的 CORS 设置**:

```http
# 严格的CORS配置
Access-Control-Allow-Origin: https://trusted-domain.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, X-Requested-With
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

**动态 CORS 验证**:

```javascript
// 白名单验证
const allowedOrigins = ["https://app.example.com", "https://admin.example.com"];

function validateOrigin(origin) {
  return allowedOrigins.includes(origin);
}
```

### 5.2 预检请求处理

```http
# OPTIONS预检请求
OPTIONS /api/v1/users
Origin: https://app.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type

# 预检响应
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 86400
```

## 6. 安全攻击防护

### 6.1 DDoS 防护

**实施层级**:

1. 网络层防护（云服务商）
2. 应用层防护（API 网关）
3. 业务层防护（限流规则）

**配置示例**:

```yaml
# API网关限流配置
rateLimiting:
  global:
    requests: 10000
    window: 60s
  perIP:
    requests: 100
    window: 60s
  perUser:
    requests: 1000
    window: 60s
```

### 6.2 CSRF 防护

**同源检查**:

```http
# 检查Referer头
Referer: https://app.example.com/page

# 检查Origin头
Origin: https://app.example.com
```

**CSRF Token**:

```http
# 获取CSRF Token
GET /api/v1/csrf-token
{
  "csrfToken": "csrf-token-value"
}

# 使用CSRF Token
POST /api/v1/users
X-CSRF-Token: csrf-token-value
```

### 6.3 暴力破解防护

**账户锁定策略**:

```json
{
  "loginAttempts": {
    "maxAttempts": 5,
    "lockoutDuration": 900, // 15分钟
    "progressiveLockout": true
  }
}
```

**响应示例**:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 900
{
  "error": {
    "code": "ACCOUNT_LOCKED",
    "message": "账户已锁定",
    "unlockTime": "2024-01-01T12:15:00Z"
  }
}
```

## 7. 审计和监控

### 7.1 安全日志

**必须记录的事件**:

- 认证成功/失败
- 权限检查失败
- 敏感操作
- 异常访问模式

```json
{
  "timestamp": "2024-01-01T12:00:00Z",
  "event": "authentication_failed",
  "userId": null,
  "ip": "192.168.1.100",
  "userAgent": "Mozilla/5.0...",
  "reason": "invalid_credentials",
  "severity": "warning"
}
```

### 7.2 异常监控

**告警触发条件**:

- 短时间内大量认证失败
- 异常 IP 访问模式
- 敏感数据大量访问
- 系统错误率激增

```json
{
  "alert": {
    "type": "security_incident",
    "severity": "high",
    "description": "检测到来自IP 192.168.1.100的暴力破解攻击",
    "metrics": {
      "failedAttempts": 50,
      "timeWindow": "5m",
      "affectedAccounts": 10
    }
  }
}
```

## 8. 安全配置清单

### 8.1 服务器配置

**HTTP 安全头部**:

```http
# 完整的安全头部配置
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self'; script-src 'self'
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

### 8.2 环境变量安全

**敏感配置管理**:

```bash
# ✅ 正确 - 使用环境变量
export DATABASE_PASSWORD="secure-password"
export JWT_SECRET="jwt-secret-key"
export API_KEY="api-key-value"

# ❌ 错误 - 硬编码在代码中
const password = "secure-password";  // 不要这样做
```

### 8.3 依赖安全

**定期安全检查**:

```bash
# 依赖漏洞扫描
npm audit
yarn audit

# 安全更新
npm update
yarn upgrade
```

## 9. 应急响应

### 9.1 安全事件响应

**响应流程**:

1. **检测** - 发现安全事件
2. **评估** - 评估影响范围
3. **隔离** - 隔离受影响系统
4. **修复** - 修复安全漏洞
5. **恢复** - 恢复正常服务
6. **总结** - 事后分析改进

### 9.2 令牌撤销

**紧急撤销机制**:

```http
# 撤销特定用户的所有令牌
POST /api/v1/admin/revoke-user-tokens
{
  "userId": 123,
  "reason": "security_incident"
}

# 全局令牌撤销（紧急情况）
POST /api/v1/admin/revoke-all-tokens
{
  "reason": "security_breach",
  "excludeAdmins": true
}
```

## 10. 合规性要求

### 10.1 数据保护法规

**GDPR 合规**:

- 用户数据访问权
- 数据删除权（被遗忘权）
- 数据可携带权
- 数据处理透明度

```http
# 用户数据导出
GET /api/v1/users/me/export
Accept: application/json

# 用户数据删除
DELETE /api/v1/users/me
{
  "confirmation": "DELETE_MY_DATA",
  "reason": "user_request"
}
```

### 10.2 审计要求

**审计日志保留**:

- 访问日志：6 个月
- 安全事件：2 年
- 敏感操作：5 年

## 总结

API 安全是一个多层次的防护体系，需要从传输、认证、授权、数据保护等多个维度综合考虑。关键要点：

1. **深度防护**: 多层安全措施
2. **最小权限**: 仅授予必要权限
3. **定期更新**: 及时修补安全漏洞
4. **持续监控**: 实时安全监控和告警
5. **应急准备**: 制定安全事件响应计划
