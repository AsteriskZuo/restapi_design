# 改进的错误响应格式设计

## 1. 错误码设计原则

### 1.1 双重错误码机制

采用 **数字错误码 + 字符串错误码** 的组合方式：

```json
{
  "error": {
    "code": 40001,                    // 数字错误码（便于程序处理）
    "type": "VALIDATION_ERROR",       // 字符串错误码（便于开发者理解）
    "message": "Request validation failed",
    "localizedMessage": {
      "zh-CN": "请求参数验证失败",
      "en-US": "Request validation failed"
    },
    "details": {
      "field": "email",
      "reason": "Invalid email format",
      "value": "invalid-email"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/40001"
  }
}
```

### 1.2 数字错误码分段规划

按照 5 位数字进行分段管理：

| 错误码范围 | 错误类型 | 说明 |
|-----------|----------|------|
| 40000-40999 | 客户端请求错误 | 4xx HTTP 状态码对应 |
| 41000-41999 | 认证授权错误 | 身份验证、权限相关 |
| 42000-42999 | 业务逻辑错误 | 业务规则验证失败 |
| 43000-43999 | 资源状态错误 | 资源不存在、状态不正确 |
| 50000-50999 | 服务器内部错误 | 5xx HTTP 状态码对应 |
| 51000-51999 | 外部依赖错误 | 第三方服务调用失败 |
| 52000-52999 | 系统资源错误 | 存储、网络等资源问题 |

## 2. 具体错误码定义

### 2.1 客户端请求错误 (40000-40999)

```json
{
  "40001": {
    "type": "VALIDATION_ERROR",
    "httpStatus": 400,
    "message": "Request validation failed",
    "description": "请求参数验证失败"
  },
  "40002": {
    "type": "MISSING_REQUIRED_FIELD",
    "httpStatus": 400,
    "message": "Required field is missing",
    "description": "缺少必填字段"
  },
  "40003": {
    "type": "INVALID_FIELD_FORMAT",
    "httpStatus": 400,
    "message": "Invalid field format",
    "description": "字段格式不正确"
  },
  "40004": {
    "type": "INVALID_JSON_FORMAT",
    "httpStatus": 400,
    "message": "Invalid JSON format",
    "description": "JSON 格式不正确"
  },
  "40005": {
    "type": "REQUEST_TOO_LARGE",
    "httpStatus": 413,
    "message": "Request payload too large",
    "description": "请求体过大"
  }
}
```

### 2.2 认证授权错误 (41000-41999)

```json
{
  "41001": {
    "type": "AUTHENTICATION_REQUIRED",
    "httpStatus": 401,
    "message": "Authentication required",
    "description": "需要身份认证"
  },
  "41002": {
    "type": "INVALID_TOKEN",
    "httpStatus": 401,
    "message": "Invalid or expired token",
    "description": "Token 无效或已过期"
  },
  "41003": {
    "type": "INSUFFICIENT_PERMISSIONS",
    "httpStatus": 403,
    "message": "Insufficient permissions",
    "description": "权限不足"
  },
  "41004": {
    "type": "TOKEN_EXPIRED",
    "httpStatus": 401,
    "message": "Token has expired",
    "description": "Token 已过期"
  },
  "41005": {
    "type": "ACCOUNT_SUSPENDED",
    "httpStatus": 403,
    "message": "Account is suspended",
    "description": "账户已被暂停"
  }
}
```

### 2.3 业务逻辑错误 (42000-42999)

```json
{
  "42001": {
    "type": "USER_ALREADY_EXISTS",
    "httpStatus": 409,
    "message": "User already exists",
    "description": "用户已存在"
  },
  "42002": {
    "type": "GROUP_MEMBER_LIMIT_EXCEEDED",
    "httpStatus": 400,
    "message": "Group member limit exceeded",
    "description": "群组成员数量超限"
  },
  "42003": {
    "type": "MESSAGE_SEND_FAILED",
    "httpStatus": 400,
    "message": "Failed to send message",
    "description": "消息发送失败"
  },
  "42004": {
    "type": "DUPLICATE_OPERATION",
    "httpStatus": 409,
    "message": "Duplicate operation detected",
    "description": "检测到重复操作"
  },
  "42005": {
    "type": "OPERATION_NOT_ALLOWED",
    "httpStatus": 400,
    "message": "Operation not allowed in current state",
    "description": "当前状态下不允许此操作"
  },
  "42006": {
    "type": "FRIENDSHIP_ALREADY_EXISTS",
    "httpStatus": 409,
    "message": "Users are already friends",
    "description": "用户已经是好友关系"
  }
}
```

### 2.4 资源状态错误 (43000-43999)

```json
{
  "43001": {
    "type": "USER_NOT_FOUND",
    "httpStatus": 404,
    "message": "User not found",
    "description": "用户不存在"
  },
  "43002": {
    "type": "GROUP_NOT_FOUND",
    "httpStatus": 404,
    "message": "Group not found",
    "description": "群组不存在"
  },
  "43003": {
    "type": "MESSAGE_NOT_FOUND",
    "httpStatus": 404,
    "message": "Message not found",
    "description": "消息不存在"
  },
  "43004": {
    "type": "RESOURCE_DELETED",
    "httpStatus": 410,
    "message": "Resource has been deleted",
    "description": "资源已被删除"
  },
  "43005": {
    "type": "RESOURCE_LOCKED",
    "httpStatus": 423,
    "message": "Resource is locked",
    "description": "资源已被锁定"
  }
}
```

### 2.5 服务器内部错误 (50000-50999)

```json
{
  "50001": {
    "type": "INTERNAL_SERVER_ERROR",
    "httpStatus": 500,
    "message": "Internal server error",
    "description": "服务器内部错误"
  },
  "50002": {
    "type": "DATABASE_ERROR",
    "httpStatus": 500,
    "message": "Database operation failed",
    "description": "数据库操作失败"
  },
  "50003": {
    "type": "SERVICE_UNAVAILABLE",
    "httpStatus": 503,
    "message": "Service temporarily unavailable",
    "description": "服务暂时不可用"
  }
}
```

### 2.6 限流和配额错误 (44000-44999)

```json
{
  "44001": {
    "type": "RATE_LIMIT_EXCEEDED",
    "httpStatus": 429,
    "message": "Rate limit exceeded",
    "description": "请求频率超限"
  },
  "44002": {
    "type": "QUOTA_EXCEEDED",
    "httpStatus": 429,
    "message": "Quota exceeded",
    "description": "配额已用完"
  },
  "44003": {
    "type": "CONCURRENT_LIMIT_EXCEEDED",
    "httpStatus": 429,
    "message": "Concurrent request limit exceeded",
    "description": "并发请求数超限"
  }
}
```

## 3. 错误响应格式示例

### 3.1 基础错误响应

```json
{
  "error": {
    "code": 40001,
    "type": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "localizedMessage": {
      "zh-CN": "请求参数验证失败",
      "en-US": "Request validation failed"
    },
    "details": {
      "field": "email",
      "reason": "Invalid email format",
      "value": "invalid-email"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/40001"
  }
}
```

### 3.2 多字段验证错误

```json
{
  "error": {
    "code": 40001,
    "type": "VALIDATION_ERROR",
    "message": "Multiple validation errors",
    "localizedMessage": {
      "zh-CN": "多个字段验证失败",
      "en-US": "Multiple validation errors"
    },
    "details": [
      {
        "field": "email",
        "reason": "Invalid email format",
        "value": "invalid-email"
      },
      {
        "field": "password",
        "reason": "Password too short",
        "value": "***"
      }
    ],
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/40001"
  }
}
```

### 3.3 限流错误响应

```json
{
  "error": {
    "code": 44001,
    "type": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded",
    "localizedMessage": {
      "zh-CN": "请求频率超限",
      "en-US": "Rate limit exceeded"
    },
    "details": {
      "limit": 1000,
      "remaining": 0,
      "resetTime": "2024-01-01T13:00:00Z",
      "retryAfter": 3600
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/44001"
  }
}
```

### 3.4 业务逻辑错误

```json
{
  "error": {
    "code": 42001,
    "type": "USER_ALREADY_EXISTS",
    "message": "User already exists",
    "localizedMessage": {
      "zh-CN": "用户已存在",
      "en-US": "User already exists"
    },
    "details": {
      "username": "john_doe",
      "existingSince": "2023-01-01T10:00:00Z"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/42001"
  }
}
```

### 3.5 资源不存在错误

**场景**：客户端请求用户A的信息，但服务器没有该用户

**HTTP 状态码**：`404 Not Found`

```json
{
  "error": {
    "code": 43001,
    "type": "USER_NOT_FOUND",
    "message": "User not found",
    "localizedMessage": {
      "zh-CN": "用户不存在",
      "en-US": "User not found"
    },
    "details": {
      "userId": "userA",
      "requestedResource": "user profile",
      "suggestions": [
        "Check if the user ID is correct",
        "Verify the user hasn't been deleted"
      ]
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/43001"
  }
}
```

### 3.6 好友关系冲突错误

**场景**：客户端请求用户A添加用户B为好友，但服务器发现他们已经是好友

**HTTP 状态码**：`409 Conflict`

```json
{
  "error": {
    "code": 42006,
    "type": "FRIENDSHIP_ALREADY_EXISTS",
    "message": "Users are already friends",
    "localizedMessage": {
      "zh-CN": "用户已经是好友关系",
      "en-US": "Users are already friends"
    },
    "details": {
      "fromUserId": "userA",
      "toUserId": "userB",
      "existingSince": "2023-12-01T10:30:00Z",
      "relationshipType": "mutual_friends",
      "suggestions": [
        "Check friendship status before adding",
        "Use GET /api/v1/users/userA/friends to verify existing relationships"
      ]
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/42006"
  }
}
```

## 4. 实施建议

### 4.1 错误码管理

```yaml
# 错误码配置文件
error_codes:
  40001:
    type: "VALIDATION_ERROR"
    http_status: 400
    message: "Request validation failed"
    localized_messages:
      zh-CN: "请求参数验证失败"
      en-US: "Request validation failed"
    documentation: "https://docs.easemob.com/errors/40001"
    
  41001:
    type: "AUTHENTICATION_REQUIRED"
    http_status: 401
    message: "Authentication required"
    localized_messages:
      zh-CN: "需要身份认证"
      en-US: "Authentication required"
    documentation: "https://docs.easemob.com/errors/41001"
    
  42006:
    type: "FRIENDSHIP_ALREADY_EXISTS"
    http_status: 409
    message: "Users are already friends"
    localized_messages:
      zh-CN: "用户已经是好友关系"
      en-US: "Users are already friends"
    documentation: "https://docs.easemob.com/errors/42006"
    
  43001:
    type: "USER_NOT_FOUND"
    http_status: 404
    message: "User not found"
    localized_messages:
      zh-CN: "用户不存在"
      en-US: "User not found"
    documentation: "https://docs.easemob.com/errors/43001"
```



## 5. 优势总结

### 5.1 数字错误码的优势
- **程序化处理**：便于客户端进行数值比较和范围判断
- **分类管理**：通过数字段可以快速识别错误类型
- **排序和统计**：便于错误统计和排序分析
- **国际化兼容**：数字本身无需翻译

### 5.2 字符串错误码的优势
- **可读性强**：开发者可以直接理解错误含义
- **调试友好**：日志中更容易识别问题
- **文档化**：便于编写错误处理文档
- **向后兼容**：添加新错误码不影响现有逻辑

### 5.3 组合使用的优势
- **最佳实践**：结合了两种方式的优点
- **灵活处理**：客户端可以选择合适的处理方式
- **完整信息**：提供了完整的错误上下文
- **易于维护**：便于错误码的管理和扩展

这种设计既满足了程序化处理的需求，又保持了良好的开发者体验，是现代 API 设计的最佳实践。 