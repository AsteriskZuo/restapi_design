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

采用 **HTTP状态码(3位) + 具体错误码(5位)** 的组合方式，形成8位错误码：

| 完整错误码示例 | HTTP状态码 | 业务模块 | 错误类型 | 说明 |
|---------------|-----------|----------|----------|------|
| 40040001-40040999 | 400 | 40(通用) | 客户端请求错误 | 参数验证、格式错误等 |
| 40141001-40141999 | 401 | 41(认证) | 认证错误 | 身份验证失败 |
| 40341001-40341999 | 403 | 41(认证) | 授权错误 | 权限不足、账户状态异常 |
| 40441001-40441999 | 404 | 41(认证) | 认证资源不存在 | 认证相关资源找不到 |
| 40442001-40442999 | 404 | 42(业务) | 业务资源不存在 | 用户、群组、消息等不存在 |
| 40943001-40943999 | 409 | 43(资源) | 资源冲突 | 重复创建、状态冲突等 |
| 41340001-41340999 | 413 | 40(通用) | 请求过大 | 文件上传、请求体过大 |
| 42343001-42343999 | 423 | 43(资源) | 资源锁定 | 资源被锁定或冻结 |
| 42944001-42944999 | 429 | 44(限流) | 限流错误 | 频率限制、配额超限 |
| 50050001-50050999 | 500 | 50(服务器) | 服务器内部错误 | 系统内部异常 |
| 50250001-50250999 | 502 | 50(服务器) | 网关错误 | 上游服务异常 |
| 50350001-50350999 | 503 | 50(服务器) | 服务不可用 | 服务维护、过载 |
| 50450001-50450999 | 504 | 50(服务器) | 网关超时 | 上游服务超时 |

### 1.3 错误码构成规则

**8位错误码 = HTTP状态码(3位) + 业务模块(2位) + 具体错误(3位)**

**分层结构：**
- **第1-3位**：HTTP状态码（404, 409, 500等）
- **第4-5位**：业务模块分类（40=通用，41=认证，42=业务逻辑等）
- **第6-8位**：具体错误编号（001, 002, 003等）

**示例解析：**
- `40441001` = `404` + `41` + `001` (404状态码 + 认证模块 + 认证资源不存在)
- `40442001` = `404` + `42` + `001` (404状态码 + 业务逻辑模块 + 业务资源不存在)  
- `40943001` = `409` + `43` + `001` (409状态码 + 资源状态模块 + 资源冲突)
- `50050001` = `500` + `50` + `001` (500状态码 + 服务器模块 + 内部错误)

**业务模块分类：**
- **40XXX**：通用模块（参数验证、格式错误等）
- **41XXX**：认证授权模块（身份验证、权限等）
- **42XXX**：业务逻辑模块（用户、群组、消息等核心业务）
- **43XXX**：资源状态模块（资源存在性、状态等）
- **44XXX**：限流配额模块（频率限制、配额等）
- **50XXX**：服务器模块（内部错误、数据库等）
- **51XXX**：外部依赖模块（第三方服务等）

**具体错误码示例：**

| 完整错误码 | HTTP状态码 | 业务模块 | 具体编号 | 错误描述 |
|-----------|-----------|----------|----------|----------|
| 40040001 | 400 | 40(通用) | 001 | 参数验证失败 |
| 40040002 | 400 | 40(通用) | 002 | 缺少必填字段 |
| 40141001 | 401 | 41(认证) | 001 | Token无效 |
| 40141002 | 401 | 41(认证) | 002 | Token过期 |
| 40341001 | 403 | 41(认证) | 001 | 权限不足 |
| 40441001 | 404 | 41(认证) | 001 | 认证服务不可用 |
| 40442001 | 404 | 42(业务) | 001 | 用户不存在 |
| 40442002 | 404 | 42(业务) | 002 | 群组不存在 |
| 40442003 | 404 | 42(业务) | 003 | 消息不存在 |
| 40943001 | 409 | 43(资源) | 001 | 用户已存在 |
| 40943002 | 409 | 43(资源) | 002 | 好友关系已存在 |
| 42944001 | 429 | 44(限流) | 001 | 请求频率超限 |
| 42944002 | 429 | 44(限流) | 002 | 配额已用完 |
| 50050001 | 500 | 50(服务器) | 001 | 内部服务器错误 |
| 50050002 | 500 | 50(服务器) | 002 | 数据库连接失败 |

**优势：**
1. **状态码直观**：前3位直接对应HTTP状态码
2. **模块清晰**：第4-5位明确业务领域
3. **错误精确**：第6-8位提供具体错误定位
4. **扩展性强**：每个模块下可支持999个具体错误

## 2. 具体错误码定义

### 2.1 客户端请求错误 (400XXXXX)

```json
{
  "40040001": {
    "type": "VALIDATION_ERROR",
    "httpStatus": 400,
    "internalCode": 40001,
    "message": "Request validation failed",
    "description": "请求参数验证失败"
  },
  "40040002": {
    "type": "MISSING_REQUIRED_FIELD",
    "httpStatus": 400,
    "internalCode": 40002,
    "message": "Required field is missing",
    "description": "缺少必填字段"
  },
  "40040003": {
    "type": "INVALID_FIELD_FORMAT",
    "httpStatus": 400,
    "internalCode": 40003,
    "message": "Invalid field format",
    "description": "字段格式不正确"
  },
  "40040011": {
    "type": "INVALID_JSON_FORMAT",
    "httpStatus": 400,
    "internalCode": 40011,
    "message": "Invalid JSON format",
    "description": "JSON 格式不正确"
  },
  "41341301": {
    "type": "REQUEST_TOO_LARGE",
    "httpStatus": 413,
    "internalCode": 41301,
    "message": "Request payload too large",
    "description": "请求体过大"
  }
}
```

### 2.2 认证错误 (401XXXXX)

```json
{
  "40140101": {
    "type": "AUTHENTICATION_REQUIRED",
    "httpStatus": 401,
    "internalCode": 40101,
    "message": "Authentication required",
    "description": "需要身份认证"
  },
  "40140102": {
    "type": "INVALID_TOKEN",
    "httpStatus": 401,
    "internalCode": 40102,
    "message": "Invalid or expired token",
    "description": "Token 无效或已过期"
  },
  "40140103": {
    "type": "TOKEN_EXPIRED",
    "httpStatus": 401,
    "internalCode": 40103,
    "message": "Token has expired",
    "description": "Token 已过期"
  }
}
```

### 2.3 权限错误 (403XXXXX)

```json
{
  "40340301": {
    "type": "INSUFFICIENT_PERMISSIONS",
    "httpStatus": 403,
    "internalCode": 40301,
    "message": "Insufficient permissions",
    "description": "权限不足"
  },
  "40340302": {
    "type": "ACCOUNT_SUSPENDED",
    "httpStatus": 403,
    "internalCode": 40302,
    "message": "Account is suspended",
    "description": "账户已被暂停"
  }
}
```

### 2.4 资源不存在错误 (404XXXXX)

```json
{
  "40440401": {
    "type": "USER_NOT_FOUND",
    "httpStatus": 404,
    "internalCode": 40401,
    "message": "User not found",
    "description": "用户不存在"
  },
  "40440411": {
    "type": "GROUP_NOT_FOUND",
    "httpStatus": 404,
    "internalCode": 40411,
    "message": "Group not found",
    "description": "群组不存在"
  },
  "40440421": {
    "type": "MESSAGE_NOT_FOUND",
    "httpStatus": 404,
    "internalCode": 40421,
    "message": "Message not found",
    "description": "消息不存在"
  }
}
```

### 2.5 资源冲突错误 (409XXXXX)

```json
{
  "40940901": {
    "type": "USER_ALREADY_EXISTS",
    "httpStatus": 409,
    "internalCode": 40901,
    "message": "User already exists",
    "description": "用户已存在"
  },
  "40940911": {
    "type": "FRIENDSHIP_ALREADY_EXISTS",
    "httpStatus": 409,
    "internalCode": 40911,
    "message": "Users are already friends",
    "description": "用户已经是好友关系"
  },
  "40940921": {
    "type": "GROUP_MEMBER_LIMIT_EXCEEDED",
    "httpStatus": 409,
    "internalCode": 40921,
    "message": "Group member limit exceeded",
    "description": "群组成员数量超限"
  }
}
```

### 2.6 限流错误 (429XXXXX)

```json
{
  "42942901": {
    "type": "RATE_LIMIT_EXCEEDED",
    "httpStatus": 429,
    "internalCode": 42901,
    "message": "Rate limit exceeded",
    "description": "请求频率超限"
  },
  "42942902": {
    "type": "QUOTA_EXCEEDED",
    "httpStatus": 429,
    "internalCode": 42902,
    "message": "Quota exceeded",
    "description": "配额已用完"
  },
  "42942903": {
    "type": "CONCURRENT_LIMIT_EXCEEDED",
    "httpStatus": 429,
    "internalCode": 42903,
    "message": "Concurrent request limit exceeded",
    "description": "并发请求数超限"
  }
}
```

### 2.7 服务器错误 (500XXXXX)

```json
{
  "50050001": {
    "type": "INTERNAL_SERVER_ERROR",
    "httpStatus": 500,
    "internalCode": 50001,
    "message": "Internal server error",
    "description": "服务器内部错误"
  },
  "50050002": {
    "type": "DATABASE_ERROR",
    "httpStatus": 500,
    "internalCode": 50002,
    "message": "Database operation failed",
    "description": "数据库操作失败"
  },
  "50350301": {
    "type": "SERVICE_UNAVAILABLE",
    "httpStatus": 503,
    "internalCode": 50301,
    "message": "Service temporarily unavailable",
    "description": "服务暂时不可用"
  }
}
```

## 3. 错误响应格式示例

### 3.1 基础错误响应

```json
{
  "error": {
    "code": 40040001,
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
    "documentation": "https://docs.easemob.com/errors/40040001"
  }
}
```

### 3.2 多字段验证错误

```json
{
  "error": {
    "code": 40040001,
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
    "documentation": "https://docs.easemob.com/errors/40040001"
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

## 6. 新设计方案评估

### 6.1 与传统5位错误码对比

| 对比项 | 传统5位设计 | 新8位设计 | 优势分析 |
|--------|------------|-----------|----------|
| **直观性** | 40001, 41001 | 40040001, 40441001 | ✅ 新设计HTTP状态码更直观 |
| **业务分类** | 模糊分类 | 精确到业务模块 | ✅ 新设计业务领域划分更清晰 |
| **扩展性** | 每类999个错误 | 每模块999个错误 | ✅ 新设计扩展空间更大 |
| **维护性** | 需要记忆分段规则 | 自解释的分层结构 | ✅ 新设计更易理解和维护 |
| **兼容性** | - | 可保留原5位作为内部码 | ✅ 新设计向后兼容 |

### 6.2 实际应用场景

**场景1：用户不存在**
- 传统设计：`43001` (需要查表才知道是404错误)
- 新设计：`40442001` (一眼看出404状态码+业务模块+具体错误)

**场景2：认证服务异常**
- 传统设计：`50001` (无法区分具体服务)
- 新设计：`40441001` (明确是404+认证模块问题)

**场景3：错误统计分析**
```javascript
// 传统方式：需要维护映射表
const httpStatus = getHttpStatusByErrorCode(43001); // 查表获取

// 新方式：直接解析
const httpStatus = Math.floor(errorCode / 100000); // 404
const module = Math.floor((errorCode % 100000) / 1000); // 42
const specificError = errorCode % 1000; // 001
```

### 6.3 实施建议

1. **渐进式迁移**：新接口使用8位码，老接口保持兼容
2. **文档更新**：提供错误码对照表和解析工具
3. **SDK适配**：客户端SDK提供错误码解析辅助函数
4. **监控优化**：利用分层结构优化错误监控和报警

### 6.4 潜在挑战

1. **数字较长**：8位数字比5位稍长，但信息量更大
2. **理解成本**：初期需要开发者适应新的编码规则
3. **系统改造**：现有系统需要适配新的错误码格式

### 6.5 结论

**推荐采用新的8位错误码设计**，因为：

1. **业务语义更清晰**：HTTP状态码+业务模块+具体错误的三层结构
2. **开发效率更高**：减少查表操作，错误码自解释
3. **扩展性更强**：每个业务模块独立编号空间
4. **兼容性良好**：可以保留原错误码作为internalCode字段

这种设计特别适合像环信这样业务模块较多、错误类型复杂的IM平台。 