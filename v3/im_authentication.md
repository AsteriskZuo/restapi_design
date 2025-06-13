# IM 认证与授权规范

## 1. 认证机制

### 1.1 JWT Token 认证

```http
# 请求头格式
Authorization: Bearer <token>

# Token 结构
{
  "sub": "user123",
  "org": "org1",
  "app": "app1",
  "iat": 1516239022,
  "exp": 1516242622,
  "scope": ["message:read", "message:write"]
}
```

### 1.2 Token 管理

- Token 有效期：默认 1 小时
- 刷新 Token：有效期 7 天
- Token 轮换：每次刷新生成新的 Token

### 1.3 多因素认证

```http
# 第一步：用户名密码认证
POST /v1/auth/login
{
  "username": "user123",
  "password": "******"
}

# 第二步：验证码认证
POST /v1/auth/verify
{
  "token": "temp-token-123",
  "code": "123456"
}
```

## 2. 授权控制

### 2.1 权限模型

```json
{
  "roles": {
    "admin": ["*"],
    "user": ["message:read", "message:write"],
    "guest": ["message:read"]
  }
}
```

### 2.2 资源访问控制

```http
# 检查权限
GET /v1/permissions/check
{
  "resource": "message",
  "action": "write",
  "userId": "user123"
}
```

### 2.3 操作权限控制

```http
# 消息发送权限
POST /v1/messages
{
  "content": "Hello",
  "to": "user456"
}
# 需要 message:write 权限

# 消息读取权限
GET /v1/messages
# 需要 message:read 权限
```

## 3. 会话管理

### 3.1 会话控制

- 单设备登录：新登录会踢掉旧会话
- 多设备登录：支持配置最大同时在线数
- 会话超时：空闲 30 分钟自动登出

### 3.2 会话状态

```http
# 获取会话状态
GET /v1/sessions/current

# 响应
{
  "data": {
    "sessionId": "sess-123",
    "userId": "user123",
    "deviceId": "device-456",
    "lastActive": 1516239022,
    "expiresAt": 1516242622
  }
}
```

## 4. 安全要求

### 4.1 密码策略

- 最小长度：8 位
- 复杂度要求：必须包含大小写字母、数字和特殊字符
- 定期更换：90 天强制更换
- 历史密码：不能使用最近 5 次使用过的密码

### 4.2 登录保护

- 失败次数限制：5 次/小时
- 锁定时间：30 分钟
- IP 限制：同一 IP 每小时最多 100 次尝试

## 5. 最佳实践

### 5.1 开发建议

- 使用 HTTPS 传输
- 实现 Token 自动刷新
- 敏感操作需要二次验证
- 记录关键操作日志

### 5.2 运维建议

- 定期轮换密钥
- 监控异常登录
- 及时处理安全事件
- 定期安全审计
