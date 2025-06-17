# IM 审计与监控规范

## 1. 日志记录

### 1.1 操作日志

```json
{
  "log": {
    "timestamp": "2024-01-01T12:00:00Z",
    "level": "INFO",
    "type": "OPERATION",
    "userId": "user123",
    "action": "message.send",
    "resource": {
      "type": "message",
      "id": "msg-123"
    },
    "details": {
      "ip": "192.168.1.1",
      "userAgent": "Mozilla/5.0...",
      "status": "success"
    }
  }
}
```

### 1.2 安全日志

```json
{
  "log": {
    "timestamp": "2024-01-01T12:00:00Z",
    "level": "WARN",
    "type": "SECURITY",
    "event": "login.failed",
    "details": {
      "userId": "user123",
      "ip": "192.168.1.1",
      "reason": "invalid_password",
      "attempts": 3
    }
  }
}
```

## 2. 日志分类

### 2.1 日志级别

| 级别  | 说明     | 示例                     |
| ----- | -------- | ------------------------ |
| ERROR | 系统错误 | 服务崩溃、数据库连接失败 |
| WARN  | 警告信息 | 登录失败、资源不足       |
| INFO  | 普通信息 | 用户操作、系统状态       |
| DEBUG | 调试信息 | 详细流程、变量值         |

### 2.2 日志类型

| 类型     | 说明         | 保留时间 |
| -------- | ------------ | -------- |
| 操作日志 | 用户行为记录 | 90 天    |
| 安全日志 | 安全事件记录 | 180 天   |
| 系统日志 | 系统运行状态 | 30 天    |
| 审计日志 | 重要操作记录 | 365 天   |

## 3. 监控指标

### 3.1 系统监控

```json
{
  "metrics": {
    "cpu": {
      "usage": 45.2,
      "threshold": 80
    },
    "memory": {
      "used": "4.2GB",
      "total": "8GB"
    },
    "disk": {
      "used": "120GB",
      "total": "200GB"
    }
  }
}
```

### 3.2 业务监控

```json
{
  "metrics": {
    "message": {
      "qps": 1000,
      "latency": 50,
      "errorRate": 0.01
    },
    "user": {
      "online": 10000,
      "active": 5000
    }
  }
}
```

## 4. 告警机制

### 4.1 告警规则

```json
{
  "alerts": {
    "highCpu": {
      "metric": "cpu.usage",
      "threshold": 80,
      "duration": "5m",
      "severity": "critical"
    },
    "highErrorRate": {
      "metric": "error.rate",
      "threshold": 0.01,
      "duration": "1m",
      "severity": "warning"
    }
  }
}
```

### 4.2 告警通知

```json
{
  "notification": {
    "channels": ["email", "sms", "webhook"],
    "recipients": ["admin@example.com"],
    "template": {
      "title": "系统告警",
      "content": "CPU使用率超过80%"
    }
  }
}
```

## 5. 审计追踪

### 5.1 敏感操作

```json
{
  "audit": {
    "timestamp": "2024-01-01T12:00:00Z",
    "operator": "admin123",
    "action": "user.delete",
    "target": "user456",
    "reason": "违规操作",
    "approver": "superadmin"
  }
}
```

### 5.2 数据变更

```json
{
  "audit": {
    "timestamp": "2024-01-01T12:00:00Z",
    "operator": "user123",
    "action": "profile.update",
    "changes": {
      "before": { "name": "张三" },
      "after": { "name": "李四" }
    }
  }
}
```

## 6. 最佳实践

### 6.1 开发建议

- 统一日志格式
- 异步日志处理
- 日志分级存储
- 敏感信息脱敏

### 6.2 运维建议

- 日志集中管理
- 定期日志分析
- 监控指标优化
- 告警阈值调整
