# 用户体系集成接口评价

## 1. 术语统一问题

用户、账户、联系人概念混乱，建议统一使用"账户"来表示登录实体。

## 2. HTTP 方法选择问题

### 2.1 修改用户密码方法错误

- **当前**: `PUT /users/{username}/password`
- **问题**: PUT 用于完整更新，修改密码属于部分更新
- **建议**: 改为 `PATCH /users/{username}/password`

### 2.2 强制下线方法不当

- **当前**: `GET /users/{username}/disconnect`
- **问题**: GET 应该是安全的，不应有副作用
- **建议**: 改为 `POST /users/{username}/disconnect`

### 2.3 封禁/解禁接口设计不规范

- **当前**: `POST /users/{username}/deactivate` 和 `POST /users/{username}/activate`
- **问题**: URL 中包含动词，不符合 RESTful 规范
- **建议**: 改为 `PATCH /users/{username}` 通过 body 中的状态字段控制

## 3. 响应格式不符合规范

### 3.1 成功响应格式问题

- **问题**: 当前响应格式不符合推荐的标准格式
- **建议**: 采用标准 data/meta 结构：

```json
{
  "data": {
    // 实际数据
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "v1"
  }
}
```

### 3.2 错误响应格式问题

- **问题**: 缺少双重错误码机制和国际化支持
- **建议**: 采用标准错误格式：

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
      "field": "username",
      "reason": "Invalid format"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  }
}
```

## 4. URL 设计不规范

### 4.1 动词形式 URL

- **问题**: 使用了 deactivate、activate、disconnect 等动词
- **建议**:
  - `POST /users/{username}/deactivate` → `PATCH /users/{username}` (body: {"status": "inactive"})
  - `POST /users/{username}/activate` → `PATCH /users/{username}` (body: {"status": "active"})
  - `GET /users/{username}/disconnect` → `POST /users/{username}/sessions/logout`

### 4.2 批量操作 URL 不一致

- **问题**: 批量状态查询使用 `/users/batch/status`，不一致
- **建议**: 统一使用 `/users/batch` 形式或查询参数方式

## 5. 批量操作响应不标准化

### 5.1 批量注册响应

- **问题**: 成功和失败混合在不同字段中
- **建议**: 采用标准批量操作响应格式：

```json
{
  "data": {
    "success": [{ "id": 123, "status": "created" }],
    "failed": [
      {
        "input": { "username": "user1", "password": "123" },
        "error": {
          "code": 40001,
          "type": "DUPLICATE_USERNAME",
          "message": "Username already exists"
        }
      }
    ]
  },
  "meta": {
    "totalCount": 2,
    "successCount": 1,
    "failedCount": 1
  }
}
```

## 6. 缺少标准响应头

- **问题**: 缺少推荐的响应头
- **建议**: 添加标准响应头：

```http
X-Request-ID: req-123456789
X-Response-Time: 123ms
X-API-Version: v1
Content-Language: zh-CN
```

## 7. 安全规范问题

### 7.1 敏感信息处理

- **问题**: 响应中可能包含敏感信息
- **建议**: 确保密码等敏感信息永不返回，用户信息适当脱敏

### 7.2 强制下线接口安全性

- **问题**: 使用 GET 方法进行有副作用的操作
- **建议**: 改为 POST 方法，避免 CSRF 攻击

## 8. 国际化支持不足

- **问题**: 错误消息和用户提示缺少多语言支持
- **建议**:
  - 请求头支持 `Accept-Language`
  - 响应中提供多语言错误消息
  - 响应头包含 `Content-Language`

## 9. 分页机制不完整

- **问题**: 缺少分页相关的响应头
- **建议**: 添加分页响应头：

```http
X-Pagination-Page: 1
X-Pagination-Limit: 10
X-Pagination-Total: 50
X-Pagination-Total-Pages: 5
Link: <https://api.example.com/users?page=2>; rel="next"
```

## 10. 建议新增功能

1. 字段选择支持：`GET /users?fields=id,username,nickname`
2. 搜索功能：`GET /users?q=keyword&status=active`
3. 排序功能：`GET /users?sort=-created_at`
4. 数据压缩支持：启用 gzip 压缩

## 11. 错误响应格式详细评价（基于错误格式规范）

### 11.1 当前错误响应格式的主要问题

#### 11.1.1 缺少双重错误码机制

- **问题**: 当前只有简单的错误类型字符串（如 `illegal_argument`、`unauthorized`），没有数字错误码
- **标准要求**: 应采用"数字错误码 + 字符串错误码"的组合方式
- **建议**:

  ```json
  // 当前格式（不规范）
  {
    "error": "illegal_argument",
    "error_description": "username XXX is not legal"
  }

  // 标准格式
  {
    "error": {
      "code": 40001,
      "type": "VALIDATION_ERROR",
      "message": "Request validation failed"
    }
  }
  ```

#### 11.1.2 错误码分段不规范

- **问题**: 当前错误码没有按照业务类型进行分段管理
- **标准要求**: 应按照 5 位数字进行分段：
  - 40000-40999: 客户端请求错误
  - 41000-41999: 认证授权错误
  - 42000-42999: 业务逻辑错误
  - 43000-43999: 资源状态错误
  - 50000-50999: 服务器内部错误
- **建议**: 重新设计错误码体系，例如：
  - 用户名格式错误: 40001 (VALIDATION_ERROR)
  - Token 无效: 41002 (INVALID_TOKEN)
  - 用户已存在: 42001 (USER_ALREADY_EXISTS)
  - 用户不存在: 43001 (USER_NOT_FOUND)

#### 11.1.3 缺少国际化支持

- **问题**: 错误消息只有英文，没有多语言支持
- **标准要求**: 应提供 `localizedMessage` 字段支持多语言
- **建议**:
  ```json
  {
    "error": {
      "code": 40001,
      "type": "VALIDATION_ERROR",
      "message": "Request validation failed",
      "localizedMessage": {
        "zh-CN": "请求参数验证失败",
        "en-US": "Request validation failed"
      }
    }
  }
  ```

#### 11.1.4 缺少详细错误上下文

- **问题**: 错误信息不够详细，缺少具体的字段信息和错误原因
- **标准要求**: 应提供 `details` 字段包含具体错误上下文
- **建议**:
  ```json
  {
    "error": {
      "code": 40001,
      "type": "VALIDATION_ERROR",
      "message": "Request validation failed",
      "details": {
        "field": "username",
        "reason": "Invalid username format",
        "value": "invalid-user@name",
        "allowedPattern": "^[a-z0-9._-]+$"
      }
    }
  }
  ```

#### 11.1.5 缺少标准错误响应字段

- **问题**: 缺少时间戳、请求 ID、文档链接等标准字段
- **标准要求**: 应包含完整的错误响应字段
- **建议**:
  ```json
  {
    "error": {
      "code": 40001,
      "type": "VALIDATION_ERROR",
      "message": "Request validation failed",
      "timestamp": "2024-01-01T12:00:00Z",
      "requestId": "req-123456789",
      "documentation": "https://docs.easemob.com/errors/40001"
    }
  }
  ```

### 11.2 具体接口错误码评价

#### 11.2.1 用户注册接口错误码问题

- **当前问题**:
  - 错误类型命名不规范：`illegal_argument` 应改为 `VALIDATION_ERROR`
  - 缺少具体的错误码数字：应该有 40001, 40002 等
  - 重复错误类型：多个不同错误都使用 `illegal_argument`
- **改进建议**:

  ```json
  // 用户名不合法
  {
    "error": {
      "code": 40001,
      "type": "INVALID_USERNAME_FORMAT",
      "message": "Username format is invalid",
      "localizedMessage": {
        "zh-CN": "用户名格式不正确",
        "en-US": "Username format is invalid"
      },
      "details": {
        "field": "username",
        "reason": "Contains invalid characters",
        "value": "user@name",
        "allowedPattern": "^[a-z0-9._-]+$"
      }
    }
  }

  // 用户已存在
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
        "username": "user1",
        "existingSince": "2023-01-01T10:00:00Z"
      }
    }
  }
  ```

#### 11.2.2 认证错误码问题

- **当前问题**:
  - `unauthorized` 类型过于宽泛
  - 缺少具体的认证失败原因分类
- **改进建议**:

  ```json
  // Token 无效
  {
    "error": {
      "code": 41002,
      "type": "INVALID_TOKEN",
      "message": "Invalid or expired token",
      "localizedMessage": {
        "zh-CN": "Token 无效或已过期",
        "en-US": "Invalid or expired token"
      }
    }
  }

  // 需要认证
  {
    "error": {
      "code": 41001,
      "type": "AUTHENTICATION_REQUIRED",
      "message": "Authentication required",
      "localizedMessage": {
        "zh-CN": "需要身份认证",
        "en-US": "Authentication required"
      }
    }
  }
  ```

#### 11.2.3 资源不存在错误码问题

- **当前问题**:
  - `service_resource_not_found` 命名不规范
  - 缺少具体的资源类型区分
- **改进建议**:
  ```json
  // 用户不存在
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
        "userId": "user123",
        "requestedResource": "user profile"
      }
    }
  }
  ```

### 11.3 错误码统一规划建议

基于用户体系集成的业务特点，建议采用以下错误码分段：

#### 11.3.1 客户端请求错误 (40000-40999)

- 40001: VALIDATION_ERROR - 请求参数验证失败
- 40002: MISSING_REQUIRED_FIELD - 缺少必填字段
- 40003: INVALID_USERNAME_FORMAT - 用户名格式不正确
- 40004: INVALID_PASSWORD_FORMAT - 密码格式不正确
- 40005: USERNAME_TOO_LONG - 用户名长度超限
- 40006: NICKNAME_TOO_LONG - 昵称长度超限

#### 11.3.2 认证授权错误 (41000-41999)

- 41001: AUTHENTICATION_REQUIRED - 需要身份认证
- 41002: INVALID_TOKEN - Token 无效或已过期
- 41003: INSUFFICIENT_PERMISSIONS - 权限不足
- 41004: TOKEN_EXPIRED - Token 已过期

#### 11.3.3 业务逻辑错误 (42000-42999)

- 42001: USER_ALREADY_EXISTS - 用户已存在
- 42002: DUPLICATE_OPERATION - 重复操作
- 42003: ACCOUNT_SUSPENDED - 账户已被暂停
- 42004: OPERATION_NOT_ALLOWED - 当前状态下不允许此操作

#### 11.3.4 资源状态错误 (43000-43999)

- 43001: USER_NOT_FOUND - 用户不存在
- 43002: RESOURCE_DELETED - 资源已被删除
- 43003: RESOURCE_LOCKED - 资源已被锁定

#### 11.3.5 限流和配额错误 (44000-44999)

- 44001: RATE_LIMIT_EXCEEDED - 请求频率超限
- 44002: QUOTA_EXCEEDED - 配额已用完
- 44003: USER_LIMIT_EXCEEDED - 用户数量超限

### 11.4 错误响应格式标准化建议

所有错误响应都应遵循统一格式：

```json
{
  "error": {
    "code": 40001, // 数字错误码（便于程序处理）
    "type": "VALIDATION_ERROR", // 字符串错误码（便于开发者理解）
    "message": "Request validation failed",
    "localizedMessage": {
      "zh-CN": "请求参数验证失败",
      "en-US": "Request validation failed"
    },
    "details": {
      "field": "username",
      "reason": "Invalid format",
      "value": "invalid-user"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789",
    "documentation": "https://docs.easemob.com/errors/40001"
  }
}
```

## 12. 用户体系业务逻辑错误码详细建议

### 12.1 用户注册业务逻辑错误码

#### 12.1.1 用户名验证相关错误码

```json
// 40001: 用户名格式验证失败
{
  "error": {
    "code": 40001,
    "type": "INVALID_USERNAME_FORMAT",
    "message": "Username format is invalid",
    "localizedMessage": {
      "zh-CN": "用户名格式不正确",
      "en-US": "Username format is invalid"
    },
    "details": {
      "field": "username",
      "value": "user@domain.com",
      "reason": "Username contains invalid characters",
      "allowedPattern": "^[a-z0-9._-]{3,64}$",
      "allowedCharacters": "小写字母、数字、下划线、点号、短横线",
      "minLength": 3,
      "maxLength": 64,
      "invalidCharacters": ["@", " ", "中文"]
    },
    "suggestions": [
      "使用3-64位小写字母、数字、下划线、点号或短横线",
      "避免使用特殊字符和空格",
      "不要使用邮箱地址作为用户名"
    ]
  }
}

// 40002: 用户名长度不符合要求
{
  "error": {
    "code": 40002,
    "type": "USERNAME_LENGTH_INVALID",
    "message": "Username length is invalid",
    "localizedMessage": {
      "zh-CN": "用户名长度不符合要求",
      "en-US": "Username length is invalid"
    },
    "details": {
      "field": "username",
      "value": "ab",
      "currentLength": 2,
      "minLength": 3,
      "maxLength": 64,
      "reason": "Username is too short"
    }
  }
}

// 40003: 用户名包含敏感词汇
{
  "error": {
    "code": 40003,
    "type": "USERNAME_CONTAINS_SENSITIVE_WORDS",
    "message": "Username contains sensitive or reserved words",
    "localizedMessage": {
      "zh-CN": "用户名包含敏感或保留词汇",
      "en-US": "Username contains sensitive or reserved words"
    },
    "details": {
      "field": "username",
      "value": "admin123",
      "reason": "Contains reserved word 'admin'",
      "sensitiveWords": ["admin"],
      "reservedWords": ["admin", "root", "system", "api", "test"]
    }
  }
}
```

#### 12.1.2 密码验证相关错误码

```json
// 40010: 密码强度不足
{
  "error": {
    "code": 40010,
    "type": "WEAK_PASSWORD",
    "message": "Password does not meet security requirements",
    "localizedMessage": {
      "zh-CN": "密码不符合安全要求",
      "en-US": "Password does not meet security requirements"
    },
    "details": {
      "field": "password",
      "reason": "Password is too weak",
      "requirements": {
        "minLength": 8,
        "maxLength": 64,
        "requireUppercase": true,
        "requireLowercase": true,
        "requireNumbers": true,
        "requireSpecialChars": true,
        "allowedSpecialChars": "!@#$%^&*()_+-=[]{}|;:,.<>?"
      },
      "currentStatus": {
        "length": 6,
        "hasUppercase": false,
        "hasLowercase": true,
        "hasNumbers": true,
        "hasSpecialChars": false
      },
      "missingRequirements": [
        "至少8个字符",
        "包含大写字母",
        "包含特殊字符"
      ]
    }
  }
}

// 40011: 密码包含常见弱密码
{
  "error": {
    "code": 40011,
    "type": "COMMON_WEAK_PASSWORD",
    "message": "Password is too common and easily guessed",
    "localizedMessage": {
      "zh-CN": "密码过于常见，容易被猜测",
      "en-US": "Password is too common and easily guessed"
    },
    "details": {
      "field": "password",
      "reason": "Password appears in common password list",
      "suggestions": [
        "避免使用123456、password等常见密码",
        "使用字母、数字、特殊字符的组合",
        "避免使用个人信息作为密码"
      ]
    }
  }
}
```

#### 12.1.3 昵称验证相关错误码

```json
// 40020: 昵称长度超限
{
  "error": {
    "code": 40020,
    "type": "NICKNAME_LENGTH_INVALID",
    "message": "Nickname length is invalid",
    "localizedMessage": {
      "zh-CN": "昵称长度不符合要求",
      "en-US": "Nickname length is invalid"
    },
    "details": {
      "field": "nickname",
      "value": "这是一个非常非常长的昵称，超过了系统允许的最大长度限制，应该被拒绝",
      "currentLength": 45,
      "maxLength": 20,
      "reason": "Nickname is too long"
    }
  }
}

// 40021: 昵称包含不当内容
{
  "error": {
    "code": 40021,
    "type": "NICKNAME_INAPPROPRIATE_CONTENT",
    "message": "Nickname contains inappropriate content",
    "localizedMessage": {
      "zh-CN": "昵称包含不当内容",
      "en-US": "Nickname contains inappropriate content"
    },
    "details": {
      "field": "nickname",
      "reason": "Contains inappropriate or offensive content",
      "moderationResult": {
        "flagged": true,
        "categories": ["profanity", "harassment"],
        "confidence": 0.95
      }
    }
  }
}
```

### 12.2 用户认证授权业务逻辑错误码

#### 12.2.1 Token 相关错误码

```json
// 41001: Token缺失
{
  "error": {
    "code": 41001,
    "type": "TOKEN_MISSING",
    "message": "Authentication token is required",
    "localizedMessage": {
      "zh-CN": "缺少认证令牌",
      "en-US": "Authentication token is required"
    },
    "details": {
      "requiredHeader": "Authorization",
      "expectedFormat": "Bearer <token>",
      "instructions": "请在请求头中包含有效的认证令牌"
    }
  }
}

// 41002: Token格式错误
{
  "error": {
    "code": 41002,
    "type": "TOKEN_FORMAT_INVALID",
    "message": "Invalid token format",
    "localizedMessage": {
      "zh-CN": "令牌格式不正确",
      "en-US": "Invalid token format"
    },
    "details": {
      "receivedFormat": "Basic dXNlcjpwYXNz",
      "expectedFormat": "Bearer <JWT_TOKEN>",
      "reason": "Token must be a valid JWT with Bearer prefix"
    }
  }
}

// 41003: Token已过期
{
  "error": {
    "code": 41003,
    "type": "TOKEN_EXPIRED",
    "message": "Authentication token has expired",
    "localizedMessage": {
      "zh-CN": "认证令牌已过期",
      "en-US": "Authentication token has expired"
    },
    "details": {
      "expiredAt": "2024-01-01T12:00:00Z",
      "currentTime": "2024-01-01T13:00:00Z",
      "expiredDuration": "1 hour ago",
      "refreshEndpoint": "/api/v1/auth/refresh",
      "instructions": "请重新获取令牌或使用刷新令牌"
    }
  }
}

// 41004: Token被撤销
{
  "error": {
    "code": 41004,
    "type": "TOKEN_REVOKED",
    "message": "Authentication token has been revoked",
    "localizedMessage": {
      "zh-CN": "认证令牌已被撤销",
      "en-US": "Authentication token has been revoked"
    },
    "details": {
      "revokedAt": "2024-01-01T11:30:00Z",
      "reason": "User password changed",
      "instructions": "请重新登录获取新的令牌"
    }
  }
}
```

#### 12.2.2 权限相关错误码

```json
// 41010: 权限不足
{
  "error": {
    "code": 41010,
    "type": "INSUFFICIENT_PERMISSIONS",
    "message": "Insufficient permissions to perform this action",
    "localizedMessage": {
      "zh-CN": "权限不足，无法执行此操作",
      "en-US": "Insufficient permissions to perform this action"
    },
    "details": {
      "requiredPermission": "user:delete",
      "userPermissions": ["user:read", "user:update"],
      "action": "delete_user",
      "resource": "user:12345",
      "instructions": "请联系管理员获取相应权限"
    }
  }
}

// 41011: 操作频率限制
{
  "error": {
    "code": 41011,
    "type": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded for this operation",
    "localizedMessage": {
      "zh-CN": "操作频率超过限制",
      "en-US": "Rate limit exceeded for this operation"
    },
    "details": {
      "operation": "user_registration",
      "limit": 10,
      "window": "1 hour",
      "current": 11,
      "resetTime": "2024-01-01T13:00:00Z",
      "retryAfter": 3600,
      "instructions": "请稍后再试或联系管理员提高限制"
    }
  }
}
```

### 12.3 用户状态管理业务逻辑错误码

#### 12.3.1 用户生命周期错误码

```json
// 42001: 用户已存在
{
  "error": {
    "code": 42001,
    "type": "USER_ALREADY_EXISTS",
    "message": "User with this username already exists",
    "localizedMessage": {
      "zh-CN": "用户名已存在",
      "en-US": "User with this username already exists"
    },
    "details": {
      "username": "john_doe",
      "existingSince": "2023-06-15T10:30:00Z",
      "conflictField": "username",
      "suggestions": [
        "尝试使用不同的用户名",
        "如果这是您的账户，请尝试登录",
        "使用用户名.数字的格式，如：john_doe_2024"
      ],
      "alternativeSuggestions": [
        "john_doe_2024",
        "john.doe.2024",
        "john-doe-2024"
      ]
    }
  }
}

// 42002: 用户账户被锁定
{
  "error": {
    "code": 42002,
    "type": "ACCOUNT_LOCKED",
    "message": "User account is temporarily locked",
    "localizedMessage": {
      "zh-CN": "用户账户已被临时锁定",
      "en-US": "User account is temporarily locked"
    },
    "details": {
      "username": "john_doe",
      "lockedAt": "2024-01-01T10:00:00Z",
      "lockReason": "Multiple failed login attempts",
      "unlockTime": "2024-01-01T11:00:00Z",
      "remainingTime": "45 minutes",
      "failedAttempts": 5,
      "maxAttempts": 5,
      "instructions": "请等待锁定时间结束后重试，或联系管理员"
    }
  }
}

// 42003: 用户账户被永久封禁
{
  "error": {
    "code": 42003,
    "type": "ACCOUNT_PERMANENTLY_SUSPENDED",
    "message": "User account has been permanently suspended",
    "localizedMessage": {
      "zh-CN": "用户账户已被永久封禁",
      "en-US": "User account has been permanently suspended"
    },
    "details": {
      "username": "john_doe",
      "suspendedAt": "2024-01-01T10:00:00Z",
      "reason": "Violation of terms of service",
      "violationType": "spam_behavior",
      "appealEndpoint": "/api/v1/appeals",
      "supportContact": "support@easemob.com",
      "instructions": "如有异议，请通过申诉渠道联系客服"
    }
  }
}

// 42004: 用户处于注销状态
{
  "error": {
    "code": 42004,
    "type": "ACCOUNT_DEACTIVATED",
    "message": "User account has been deactivated",
    "localizedMessage": {
      "zh-CN": "用户账户已被注销",
      "en-US": "User account has been deactivated"
    },
    "details": {
      "username": "john_doe",
      "deactivatedAt": "2024-01-01T10:00:00Z",
      "canReactivate": true,
      "reactivationPeriod": "30 days",
      "permanentDeletionDate": "2024-01-31T10:00:00Z",
      "reactivationEndpoint": "/api/v1/users/reactivate",
      "instructions": "可在30天内重新激活账户，否则将被永久删除"
    }
  }
}
```

#### 12.3.2 操作状态冲突错误码

```json
// 42010: 重复操作
{
  "error": {
    "code": 42010,
    "type": "DUPLICATE_OPERATION",
    "message": "This operation has already been performed",
    "localizedMessage": {
      "zh-CN": "此操作已经执行过",
      "en-US": "This operation has already been performed"
    },
    "details": {
      "operation": "account_activation",
      "performedAt": "2024-01-01T10:00:00Z",
      "currentStatus": "active",
      "allowedOperations": ["deactivate", "update_profile"],
      "instructions": "账户已处于激活状态，无需重复激活"
    }
  }
}

// 42011: 操作状态冲突
{
  "error": {
    "code": 42011,
    "type": "INVALID_STATE_TRANSITION",
    "message": "Cannot perform this operation in current state",
    "localizedMessage": {
      "zh-CN": "当前状态下无法执行此操作",
      "en-US": "Cannot perform this operation in current state"
    },
    "details": {
      "currentState": "suspended",
      "requestedOperation": "password_change",
      "reason": "Password cannot be changed while account is suspended",
      "allowedOperations": ["appeal_suspension"],
      "requiredStateTransition": "suspended -> active",
      "instructions": "请先申诉解封账户，然后再修改密码"
    }
  }
}
```

### 12.4 资源查找业务逻辑错误码

#### 12.4.1 用户查找错误码

```json
// 43001: 用户不存在
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
      "searchCriteria": {
        "username": "nonexistent_user"
      },
      "searchedAt": "2024-01-01T12:00:00Z",
      "suggestions": [
        "检查用户名拼写是否正确",
        "确认用户账户是否已被删除",
        "尝试使用其他查找方式（如邮箱、手机号）"
      ],
      "alternativeEndpoints": [
        "/api/v1/users/search?email=user@example.com",
        "/api/v1/users/search?phone=+1234567890"
      ]
    }
  }
}

// 43002: 用户已被删除
{
  "error": {
    "code": 43002,
    "type": "USER_DELETED",
    "message": "User has been permanently deleted",
    "localizedMessage": {
      "zh-CN": "用户已被永久删除",
      "en-US": "User has been permanently deleted"
    },
    "details": {
      "username": "deleted_user",
      "deletedAt": "2024-01-01T10:00:00Z",
      "deletionReason": "User requested account deletion",
      "retentionPeriod": "90 days",
      "canRecover": false,
      "instructions": "用户数据已被永久删除，无法恢复"
    }
  }
}
```

### 12.5 业务限制相关错误码

#### 12.5.1 容量限制错误码

```json
// 44001: 用户注册数量超限
{
  "error": {
    "code": 44001,
    "type": "USER_REGISTRATION_LIMIT_EXCEEDED",
    "message": "User registration limit exceeded",
    "localizedMessage": {
      "zh-CN": "用户注册数量超过限制",
      "en-US": "User registration limit exceeded"
    },
    "details": {
      "currentCount": 10001,
      "maxAllowed": 10000,
      "planType": "standard",
      "upgradeOptions": [
        {
          "plan": "premium",
          "maxUsers": 50000,
          "price": "$299/month"
        },
        {
          "plan": "enterprise",
          "maxUsers": "unlimited",
          "price": "Contact sales"
        }
      ],
      "contactSales": "sales@easemob.com",
      "instructions": "请升级到更高级别的套餐以增加用户数量限制"
    }
  }
}

// 44002: 并发操作限制
{
  "error": {
    "code": 44002,
    "type": "CONCURRENT_OPERATION_LIMIT_EXCEEDED",
    "message": "Too many concurrent operations",
    "localizedMessage": {
      "zh-CN": "并发操作数量过多",
      "en-US": "Too many concurrent operations"
    },
    "details": {
      "operation": "batch_user_creation",
      "currentConcurrent": 6,
      "maxConcurrent": 5,
      "queuePosition": 3,
      "estimatedWaitTime": "2 minutes",
      "retryAfter": 120,
      "instructions": "请稍后重试或减少并发请求数量"
    }
  }
}
```

### 12.6 业务逻辑错误码使用指南

#### 12.6.1 错误码优先级

1. **安全相关错误**：优先返回安全相关错误码（41xxx）
2. **权限验证错误**：其次处理权限相关错误（41xxx）
3. **参数验证错误**：然后处理参数格式错误（40xxx）
4. **业务逻辑错误**：最后处理业务逻辑错误（42xxx）

#### 12.6.2 错误码组合策略

```json
// 多字段验证失败时的响应
{
  "error": {
    "code": 40001,
    "type": "MULTIPLE_VALIDATION_ERRORS",
    "message": "Multiple validation errors occurred",
    "localizedMessage": {
      "zh-CN": "多个字段验证失败",
      "en-US": "Multiple validation errors occurred"
    },
    "details": {
      "failedFields": [
        {
          "field": "username",
          "code": 40001,
          "type": "INVALID_USERNAME_FORMAT",
          "reason": "Contains invalid characters"
        },
        {
          "field": "password",
          "code": 40010,
          "type": "WEAK_PASSWORD",
          "reason": "Password is too weak"
        }
      ],
      "totalErrors": 2
    }
  }
}
```

#### 12.6.3 错误恢复建议

每个错误码都应提供明确的恢复建议：

- **即时修复**：用户可以立即修正的问题
- **延时重试**：需要等待一段时间后重试的问题
- **人工介入**：需要联系客服或管理员的问题
- **升级服务**：需要升级服务套餐的问题

#### 12.6.4 错误码监控建议

- **错误频率监控**：监控各错误码的出现频率
- **用户体验分析**：分析错误对用户体验的影响
- **业务指标跟踪**：跟踪错误码对业务指标的影响
- **优化策略制定**：基于错误数据制定优化策略

# 问题表格

| 接口名称                                 | 方法选择评价  | 状态保存评价 | 认证方式评价 | 标准头评价 | 成功响应格式评价 | 错误响应格式评价 | 动词使用评价 | 数据传递规则评价 | HTTP 状态码使用评价 | 分页评价 | URL 命名规范评价 | 数据压缩评价 | 嵌套评价 | 批量操作评价 | 国际化评价 | 搜索评价 | 文件操作评价 | 安全规范评价  | 错误码规范评价 |
| :--------------------------------------- | :------------ | :----------- | :----------- | :--------- | :--------------- | :--------------- | :----------- | :--------------- | :------------------ | :------- | :--------------- | :----------- | :------- | :----------- | :--------- | :------- | :----------- | :------------ | :------------- |
| POST /users (注册)                       | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | N/A      | ✅ 合理          | ❌ 缺少      | ✅ 合理  | ⚠️ 格式问题  | ❌ 缺少    | N/A      | N/A          | ⚠️ 基本       | ❌ 缺少双重码  |
| GET /users/{username}                    | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | N/A      | ✅ 合理          | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | ❌ 缺少  | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| GET /users (批量获取)                    | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | ✅ 支持  | ✅ 合理          | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | ❌ 缺少  | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| DELETE /users/{username}                 | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | N/A      | ✅ 合理          | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| DELETE /users (批量删除)                 | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | ✅ 支持  | ✅ 合理          | ❌ 缺少      | ✅ 合理  | ⚠️ 格式问题  | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| PUT /users/{username}/password           | ❌ 应用 PATCH | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | N/A      | ✅ 合理          | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ⚠️ 敏感信息   | ❌ 缺少双重码  |
| POST /users/{username}/deactivate        | ⚠️ 应重构     | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ❌ 有动词    | ✅ 合理          | ✅ 合理             | N/A      | ❌ 有动词        | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| POST /users/{username}/activate          | ⚠️ 应重构     | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ❌ 有动词    | ✅ 合理          | ✅ 合理             | N/A      | ❌ 有动词        | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| GET /users/{username}/disconnect         | ❌ 应用 POST  | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ❌ 有动词    | ✅ 合理          | ✅ 合理             | N/A      | ❌ 有动词        | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ❌ GET 副作用 | ❌ 缺少双重码  |
| DELETE /users/{username}/disconnect/{id} | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ❌ 有动词    | ✅ 合理          | ✅ 合理             | N/A      | ❌ 有动词        | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| GET /users/{username}/status             | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | N/A      | ✅ 合理          | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| POST /users/batch/status                 | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | N/A      | ⚠️ batch 位置    | ❌ 缺少      | ✅ 合理  | ⚠️ 格式问题  | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |
| GET /users/{username}/resources          | ✅ 合理       | ✅ 无状态    | ✅ Bearer    | ⚠️ 基本    | ❌ 不规范        | ❌ 完全不符合    | ✅ 名词      | ✅ 合理          | ✅ 合理             | N/A      | ✅ 合理          | ❌ 缺少      | ✅ 合理  | N/A          | ❌ 缺少    | N/A      | N/A          | ✅ 合理       | ❌ 缺少双重码  |

## 评价说明

- ✅ 合理：符合规范要求
- ⚠️ 部分：部分符合，有改进空间
- ❌ 不符合：不符合规范，需要修改
- N/A：不适用该评价项

### 错误码规范评价说明

- ✅ 合理：采用双重错误码机制，有完整的错误响应格式，支持国际化
- ⚠️ 部分：部分采用规范格式，但缺少某些标准字段
- ❌ 缺少双重码：完全未采用标准错误格式，缺少数字错误码和字符串错误码组合
- ❌ 完全不符合：错误响应格式完全不符合标准要求
