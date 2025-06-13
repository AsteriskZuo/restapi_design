# IM 访问控制规范

## 1. 速率限制

### 1.1 API 调用限制

```http
# 响应头
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1516239022
```

| 接口类型 | 限制规则    | 说明         |
| -------- | ----------- | ------------ |
| 登录接口 | 5 次/分钟   | 防止暴力破解 |
| 消息发送 | 60 次/分钟  | 防止垃圾消息 |
| 文件上传 | 10 次/分钟  | 防止资源滥用 |
| 普通接口 | 100 次/分钟 | 常规限制     |

### 1.2 并发请求限制

```json
{
  "limits": {
    "maxConcurrent": 10,
    "maxQueueSize": 100,
    "timeout": 30000
  }
}
```

## 2. IP 控制

### 2.1 IP 白名单

```http
# 管理白名单
POST /v1/admin/ip-whitelist
{
  "ip": "192.168.1.1",
  "description": "内部服务器",
  "expiresAt": "2024-12-31T23:59:59Z"
}
```

### 2.2 IP 黑名单

```http
# 管理黑名单
POST /v1/admin/ip-blacklist
{
  "ip": "1.2.3.4",
  "reason": "异常访问",
  "duration": 3600
}
```

## 3. 资源控制

### 3.1 配额管理

```json
{
  "quotas": {
    "storage": {
      "free": "100MB",
      "pro": "1GB",
      "enterprise": "10GB"
    },
    "message": {
      "free": "1000条/天",
      "pro": "10000条/天",
      "enterprise": "无限制"
    }
  }
}
```

### 3.2 资源使用统计

```http
# 获取使用统计
GET /v1/stats/usage

# 响应
{
  "data": {
    "storage": {
      "used": "50MB",
      "total": "100MB",
      "percentage": 50
    },
    "message": {
      "today": 500,
      "limit": 1000,
      "percentage": 50
    }
  }
}
```

## 4. 访问控制策略

### 4.1 时间控制

```json
{
  "timeRestrictions": {
    "maintenance": {
      "start": "02:00",
      "end": "04:00",
      "timezone": "Asia/Shanghai"
    },
    "peakHours": {
      "start": "09:00",
      "end": "18:00",
      "rateLimit": "50%"
    }
  }
}
```

### 4.2 地域控制

```json
{
  "geoRestrictions": {
    "allowed": ["CN", "HK", "TW"],
    "blocked": ["XX", "YY"],
    "default": "block"
  }
}
```

## 5. 异常处理

### 5.1 限流响应

```json
{
  "error": {
    "code": "429",
    "type": "RATE_LIMIT_EXCEEDED",
    "message": "请求过于频繁",
    "retryAfter": 60
  }
}
```

### 5.2 封禁响应

```json
{
  "error": {
    "code": "403",
    "type": "IP_BLOCKED",
    "message": "IP已被封禁",
    "expiresAt": "2024-01-01T00:00:00Z"
  }
}
```

## 6. 最佳实践

### 6.1 开发建议

- 实现优雅降级
- 添加重试机制
- 使用缓存减少请求
- 实现请求队列

### 6.2 运维建议

- 监控访问模式
- 及时调整限制
- 定期清理黑名单
- 记录异常访问
