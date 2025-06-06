# 错误处理策略

> **重要性**: 高优先级，影响 API 可用性和用户体验  
> **适用范围**: 所有 API 响应和异常处理

## 概览

系统化的错误处理是保证 API 稳定性和用户体验的关键。本文档定义了完整的错误处理策略，包括错误分类、编码规范、传播机制和恢复策略。

## 1. 错误分类体系

### 1.1 按来源分类

#### 客户端错误 (4xx)

- **验证错误**: 输入数据不符合要求
- **认证错误**: 身份认证失败
- **授权错误**: 权限不足
- **资源错误**: 请求的资源不存在

#### 服务器错误 (5xx)

- **业务逻辑错误**: 业务规则冲突
- **系统错误**: 服务内部异常
- **依赖服务错误**: 外部服务不可用
- **基础设施错误**: 数据库、网络等故障

### 1.2 按严重程度分类

| 级别     | 描述             | HTTP 状态码        | 响应策略              |
| -------- | ---------------- | ------------------ | --------------------- |
| **致命** | 系统无法继续运行 | 500, 502, 503      | 立即告警，快速修复    |
| **严重** | 功能不可用       | 500, 409, 422      | 告警，优先修复        |
| **警告** | 部分功能受限     | 400, 401, 403, 404 | 记录日志，定期 review |
| **信息** | 用户操作提示     | 400, 422           | 用户友好提示          |

## 2. 错误响应格式

### 2.1 标准错误格式

**基于 RFC 7807 Problem Details 标准**:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "请求数据验证失败",
    "details": {
      "field": "email",
      "reason": "格式不正确",
      "value": "invalid-email"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "path": "/api/v1/users",
    "requestId": "req-123456789"
  }
}
```

### 2.2 字段说明

| 字段        | 类型   | 必需 | 说明                   |
| ----------- | ------ | ---- | ---------------------- |
| `code`      | string | ✅   | 错误代码，用于程序判断 |
| `message`   | string | ✅   | 用户友好的错误描述     |
| `details`   | object | ❌   | 详细错误信息           |
| `timestamp` | string | ✅   | 错误发生时间           |
| `path`      | string | ✅   | 请求路径               |
| `requestId` | string | ✅   | 请求追踪 ID            |

### 2.3 多语言支持

```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "账户余额不足",
    "localizedMessage": {
      "zh-CN": "账户余额不足",
      "en-US": "Insufficient account balance",
      "ja-JP": "口座残高が不足しています"
    },
    "details": {
      "currentBalance": 100.0,
      "requiredAmount": 150.0,
      "currency": "CNY"
    }
  }
}
```

## 3. 错误代码规范

### 3.1 编码规则

**格式**: `{CATEGORY}_{SPECIFIC_ERROR}`

**示例**:

- `VALIDATION_REQUIRED_FIELD` - 验证类：必填字段缺失
- `AUTH_INVALID_TOKEN` - 认证类：无效令牌
- `BUSINESS_INSUFFICIENT_STOCK` - 业务类：库存不足

### 3.2 分类代码

#### 验证错误 (VALIDATION\_\*)

```json
{
  "VALIDATION_REQUIRED_FIELD": "缺少必填字段",
  "VALIDATION_INVALID_FORMAT": "字段格式不正确",
  "VALIDATION_OUT_OF_RANGE": "字段值超出有效范围",
  "VALIDATION_DUPLICATE_VALUE": "字段值重复",
  "VALIDATION_INVALID_LENGTH": "字段长度不符合要求"
}
```

#### 认证错误 (AUTH\_\*)

```json
{
  "AUTH_MISSING_TOKEN": "缺少认证令牌",
  "AUTH_INVALID_TOKEN": "认证令牌无效",
  "AUTH_EXPIRED_TOKEN": "认证令牌已过期",
  "AUTH_INVALID_CREDENTIALS": "用户名或密码错误",
  "AUTH_ACCOUNT_LOCKED": "账户已被锁定"
}
```

#### 授权错误 (AUTHZ\_\*)

```json
{
  "AUTHZ_INSUFFICIENT_PERMISSIONS": "权限不足",
  "AUTHZ_RESOURCE_FORBIDDEN": "禁止访问该资源",
  "AUTHZ_OPERATION_NOT_ALLOWED": "不允许执行该操作",
  "AUTHZ_QUOTA_EXCEEDED": "超出配额限制"
}
```

#### 资源错误 (RESOURCE\_\*)

```json
{
  "RESOURCE_NOT_FOUND": "资源不存在",
  "RESOURCE_ALREADY_EXISTS": "资源已存在",
  "RESOURCE_GONE": "资源已不可用",
  "RESOURCE_LOCKED": "资源被锁定",
  "RESOURCE_VERSION_CONFLICT": "资源版本冲突"
}
```

#### 业务错误 (BUSINESS\_\*)

```json
{
  "BUSINESS_INSUFFICIENT_BALANCE": "余额不足",
  "BUSINESS_ORDER_CANCELLED": "订单已取消",
  "BUSINESS_INVENTORY_SHORTAGE": "库存不足",
  "BUSINESS_OPERATION_TIMEOUT": "操作超时",
  "BUSINESS_RULE_VIOLATION": "违反业务规则"
}
```

#### 系统错误 (SYSTEM\_\*)

```json
{
  "SYSTEM_INTERNAL_ERROR": "系统内部错误",
  "SYSTEM_SERVICE_UNAVAILABLE": "服务不可用",
  "SYSTEM_DATABASE_ERROR": "数据库错误",
  "SYSTEM_EXTERNAL_SERVICE_ERROR": "外部服务错误",
  "SYSTEM_CONFIGURATION_ERROR": "配置错误"
}
```

## 4. 具体场景处理

### 4.1 数据验证错误

**单字段验证失败**:

```http
HTTP/1.1 400 Bad Request
{
  "error": {
    "code": "VALIDATION_INVALID_FORMAT",
    "message": "邮箱格式不正确",
    "details": {
      "field": "email",
      "value": "invalid-email",
      "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "path": "/api/v1/users",
    "requestId": "req-123456789"
  }
}
```

**多字段验证失败**:

```http
HTTP/1.1 400 Bad Request
{
  "error": {
    "code": "VALIDATION_MULTIPLE_ERRORS",
    "message": "多个字段验证失败",
    "details": {
      "errors": [
        {
          "field": "email",
          "code": "VALIDATION_INVALID_FORMAT",
          "message": "邮箱格式不正确"
        },
        {
          "field": "age",
          "code": "VALIDATION_OUT_OF_RANGE",
          "message": "年龄必须在18-65之间"
        }
      ]
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "path": "/api/v1/users",
    "requestId": "req-123456789"
  }
}
```

### 4.2 业务逻辑错误

**库存不足示例**:

```http
HTTP/1.1 409 Conflict
{
  "error": {
    "code": "BUSINESS_INSUFFICIENT_STOCK",
    "message": "商品库存不足",
    "details": {
      "productId": 123,
      "requestedQuantity": 5,
      "availableQuantity": 2,
      "productName": "iPhone 15"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "path": "/api/v1/orders",
    "requestId": "req-123456789"
  }
}
```

### 4.3 系统内部错误

**数据库连接失败**:

```http
HTTP/1.1 503 Service Unavailable
{
  "error": {
    "code": "SYSTEM_DATABASE_UNAVAILABLE",
    "message": "服务暂时不可用，请稍后重试",
    "details": {
      "retryAfter": 30,
      "errorId": "db-conn-timeout-001"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "path": "/api/v1/users",
    "requestId": "req-123456789"
  }
}
```

### 4.4 外部服务错误

**第三方支付服务错误**:

```http
HTTP/1.1 502 Bad Gateway
{
  "error": {
    "code": "SYSTEM_PAYMENT_SERVICE_ERROR",
    "message": "支付服务暂时不可用",
    "details": {
      "service": "alipay",
      "errorCode": "SYSTEM_ERROR",
      "retryable": true,
      "estimatedRecoveryTime": "2024-01-01T12:05:00Z"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "path": "/api/v1/payments",
    "requestId": "req-123456789"
  }
}
```

## 5. 错误传播策略

### 5.1 上游错误处理

**原则**:

1. **不透传内部错误详情**
2. **转换为用户友好的消息**
3. **保留足够的调试信息**

**示例**:

```javascript
// ❌ 错误做法 - 直接透传数据库错误
{
  "error": "SQLSTATE[23000]: Integrity constraint violation: 1062 Duplicate entry 'user@example.com' for key 'email'"
}

// ✅ 正确做法 - 转换为业务错误
{
  "error": {
    "code": "VALIDATION_DUPLICATE_VALUE",
    "message": "该邮箱已被注册",
    "details": {
      "field": "email"
    }
  }
}
```

### 5.2 错误日志记录

**分层记录**:

```javascript
// 用户层日志（记录业务错误）
logger.warn("User registration failed", {
  error: "VALIDATION_DUPLICATE_VALUE",
  field: "email",
  userId: null,
  requestId: "req-123456789",
});

// 系统层日志（记录技术错误）
logger.error("Database constraint violation", {
  error: error.message,
  stack: error.stack,
  query: "INSERT INTO users...",
  requestId: "req-123456789",
});
```

## 6. 重试和恢复机制

### 6.1 可重试错误识别

**可重试的错误类型**:

- 网络超时（408, 504）
- 服务不可用（503）
- 限流错误（429）
- 临时的系统错误（500）

**不可重试的错误类型**:

- 客户端错误（400, 401, 403, 404）
- 业务逻辑错误（409, 422）
- 永久性系统错误

### 6.2 重试响应头

```http
HTTP/1.1 503 Service Unavailable
Retry-After: 30
X-Retry-Limit: 3
X-Retry-Remaining: 2
{
  "error": {
    "code": "SYSTEM_SERVICE_OVERLOADED",
    "message": "服务暂时过载，请稍后重试",
    "details": {
      "retryable": true,
      "retryAfter": 30,
      "maxRetries": 3
    }
  }
}
```

### 6.3 客户端重试策略

**指数退避重试**:

```javascript
class APIClient {
  async requestWithRetry(url, options, maxRetries = 3) {
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        const response = await fetch(url, options);

        if (response.ok) {
          return response;
        }

        const error = await response.json();

        // 检查是否可重试
        if (!this.isRetryable(response.status, error)) {
          throw new APIError(error);
        }

        // 最后一次尝试，直接抛出错误
        if (attempt === maxRetries) {
          throw new APIError(error);
        }

        // 计算退避时间
        const delay = this.calculateBackoff(attempt, error);
        await this.sleep(delay);
      } catch (error) {
        if (attempt === maxRetries) {
          throw error;
        }
      }
    }
  }

  isRetryable(status, error) {
    const retryableStatuses = [408, 429, 500, 502, 503, 504];
    return (
      retryableStatuses.includes(status) || error.error?.details?.retryable
    );
  }

  calculateBackoff(attempt, error) {
    // 服务器指定的重试时间
    if (error.error?.details?.retryAfter) {
      return error.error.details.retryAfter * 1000;
    }

    // 指数退避：1s, 2s, 4s, 8s...
    return Math.min(1000 * Math.pow(2, attempt), 30000);
  }
}
```

## 7. 监控和告警

### 7.1 错误指标监控

**关键指标**:

- 错误率（总错误数/总请求数）
- 不同错误类型的分布
- 错误响应时间
- 错误恢复时间

**监控配置**:

```yaml
alerts:
  - name: high_error_rate
    condition: error_rate > 0.05 # 错误率超过5%
    duration: 5m
    severity: critical

  - name: auth_failures
    condition: auth_error_rate > 0.10 # 认证错误率超过10%
    duration: 2m
    severity: warning

  - name: system_errors
    condition: system_error_count > 100 # 系统错误超过100次
    duration: 1m
    severity: critical
```

### 7.2 错误趋势分析

**错误分析维度**:

- 时间维度：小时、天、周趋势
- 用户维度：新用户 vs 老用户
- 功能维度：不同 API 端点的错误分布
- 地理维度：不同地区的错误情况

## 8. 错误测试策略

### 8.1 错误场景测试

**测试用例类型**:

```javascript
describe("Error Handling", () => {
  test("should return validation error for invalid email", async () => {
    const response = await api.post("/users", {
      email: "invalid-email",
      name: "Test User",
    });

    expect(response.status).toBe(400);
    expect(response.data.error.code).toBe("VALIDATION_INVALID_FORMAT");
    expect(response.data.error.details.field).toBe("email");
  });

  test("should return 409 for duplicate email", async () => {
    // 先创建一个用户
    await api.post("/users", {
      email: "test@example.com",
      name: "Test User",
    });

    // 尝试创建重复邮箱用户
    const response = await api.post("/users", {
      email: "test@example.com",
      name: "Another User",
    });

    expect(response.status).toBe(409);
    expect(response.data.error.code).toBe("VALIDATION_DUPLICATE_VALUE");
  });
});
```

### 8.2 故障注入测试

**模拟系统故障**:

```javascript
// 模拟数据库连接失败
test("should handle database connection failure", async () => {
  // 使用测试工具模拟数据库故障
  await mockDatabase.simulateFailure("connection_timeout");

  const response = await api.get("/users");

  expect(response.status).toBe(503);
  expect(response.data.error.code).toBe("SYSTEM_DATABASE_UNAVAILABLE");
  expect(response.headers["retry-after"]).toBeDefined();
});
```

## 9. 最佳实践

### 9.1 错误设计原则

1. **一致性**: 所有错误响应使用统一格式
2. **明确性**: 错误信息明确说明问题和解决方案
3. **安全性**: 不泄露敏感的系统内部信息
4. **可操作性**: 提供用户可以采取的行动指导
5. **可追踪性**: 包含足够信息用于问题诊断

### 9.2 错误信息编写指南

**好的错误信息特征**:

- 使用用户理解的语言
- 说明具体问题
- 提供解决建议
- 包含相关上下文

**示例对比**:

```json
// ❌ 糟糕的错误信息
{
  "error": "Error 500"
}

// ✅ 优秀的错误信息
{
  "error": {
    "code": "BUSINESS_INSUFFICIENT_BALANCE",
    "message": "账户余额不足，无法完成支付",
    "details": {
      "currentBalance": 100.00,
      "requiredAmount": 150.00,
      "currency": "CNY"
    },
    "suggestions": [
      "请充值后重试",
      "选择其他支付方式",
      "联系客服获取帮助"
    ]
  }
}
```

## 总结

完善的错误处理策略是 API 设计的重要组成部分，需要考虑：

1. **系统化分类**: 建立清晰的错误分类体系
2. **标准化格式**: 使用一致的错误响应格式
3. **智能化处理**: 实现错误传播和恢复机制
4. **可观测性**: 建立完善的监控和告警体系
5. **用户友好**: 提供清晰的错误信息和操作指导
