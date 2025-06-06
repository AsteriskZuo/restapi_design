# 数据验证规范

> **重要性**: 中优先级，保证数据质量和系统安全  
> **适用范围**: 所有 API 请求和响应数据

## 概览

数据验证是确保 API 数据质量、安全性和一致性的重要环节。本文档定义了完整的数据验证规范，包括输入验证、输出格式、类型约定等。

## 1. 输入数据验证

### 1.1 验证原则

**全面验证原则**:

- 验证所有用户输入
- 服务器端验证为准
- 客户端验证为辅助
- 先验证再处理

**安全验证原则**:

- 白名单优于黑名单
- 严格验证敏感字段
- 防止注入攻击
- 限制数据长度

### 1.2 JSON Schema 验证

**用户注册示例**:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "用户注册请求",
  "required": ["email", "password", "name"],
  "properties": {
    "email": {
      "type": "string",
      "format": "email",
      "maxLength": 255,
      "description": "用户邮箱"
    },
    "password": {
      "type": "string",
      "minLength": 8,
      "maxLength": 128,
      "pattern": "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]",
      "description": "密码：至少8位，包含大小写字母、数字和特殊字符"
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100,
      "pattern": "^[a-zA-Z0-9\\u4e00-\\u9fa5\\s-_]+$",
      "description": "用户姓名"
    },
    "age": {
      "type": "integer",
      "minimum": 18,
      "maximum": 120,
      "description": "年龄"
    },
    "phone": {
      "type": "string",
      "pattern": "^1[3-9]\\d{9}$",
      "description": "中国大陆手机号"
    },
    "address": {
      "type": "object",
      "properties": {
        "country": {
          "type": "string",
          "enum": ["CN", "US", "JP", "UK"],
          "description": "国家代码"
        },
        "province": {
          "type": "string",
          "maxLength": 50,
          "description": "省份"
        },
        "city": {
          "type": "string",
          "maxLength": 50,
          "description": "城市"
        },
        "detail": {
          "type": "string",
          "maxLength": 200,
          "description": "详细地址"
        }
      },
      "required": ["country", "province", "city"]
    }
  },
  "additionalProperties": false
}
```

### 1.3 常用验证规则

#### 字符串验证

```json
{
  "username": {
    "type": "string",
    "minLength": 3,
    "maxLength": 20,
    "pattern": "^[a-zA-Z0-9_-]+$",
    "description": "用户名：3-20位字母数字下划线中划线"
  },
  "description": {
    "type": "string",
    "maxLength": 500,
    "description": "描述信息"
  },
  "url": {
    "type": "string",
    "format": "uri",
    "maxLength": 2048,
    "description": "URL地址"
  }
}
```

#### 数字验证

```json
{
  "price": {
    "type": "number",
    "minimum": 0,
    "maximum": 99999999.99,
    "multipleOf": 0.01,
    "description": "价格：精确到分"
  },
  "quantity": {
    "type": "integer",
    "minimum": 1,
    "maximum": 9999,
    "description": "数量"
  },
  "rating": {
    "type": "number",
    "minimum": 0,
    "maximum": 5,
    "description": "评分：0-5"
  }
}
```

#### 时间日期验证

```json
{
  "birthday": {
    "type": "string",
    "format": "date",
    "description": "生日：YYYY-MM-DD"
  },
  "createdAt": {
    "type": "string",
    "format": "date-time",
    "description": "创建时间：ISO 8601格式"
  },
  "expireTime": {
    "type": "string",
    "format": "date-time",
    "description": "过期时间"
  }
}
```

#### 数组验证

```json
{
  "tags": {
    "type": "array",
    "items": {
      "type": "string",
      "maxLength": 50
    },
    "minItems": 1,
    "maxItems": 10,
    "uniqueItems": true,
    "description": "标签列表"
  },
  "permissions": {
    "type": "array",
    "items": {
      "type": "string",
      "enum": ["read", "write", "delete", "admin"]
    },
    "minItems": 1,
    "description": "权限列表"
  }
}
```

## 2. 输出数据格式

### 2.1 响应数据规范

**成功响应格式**:

```json
{
  "data": {
    "id": 123,
    "email": "user@example.com",
    "name": "张三",
    "status": "active",
    "createdAt": "2024-01-01T12:00:00Z",
    "updatedAt": "2024-01-01T12:00:00Z"
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "v1",
    "requestId": "req-123456789"
  }
}
```

**分页响应格式**:

```json
{
  "data": [
    {
      "id": 1,
      "name": "用户1"
    },
    {
      "id": 2,
      "name": "用户2"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 50,
      "totalPages": 5,
      "hasNext": true,
      "hasPrev": false
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  }
}
```

### 2.2 数据类型约定

#### 标识符 (ID)

```json
{
  "id": 123, // 整数ID
  "uuid": "550e8400-e29b-41d4-a716-446655440000", // UUID
  "code": "USER_001", // 业务编码
  "slug": "user-profile" // URL友好标识
}
```

#### 时间戳

```json
{
  "createdAt": "2024-01-01T12:00:00Z", // ISO 8601 UTC时间
  "updatedAt": "2024-01-01T12:00:00Z", // ISO 8601 UTC时间
  "timestamp": 1640995200, // Unix时间戳（秒）
  "timestampMs": 1640995200000 // Unix时间戳（毫秒）
}
```

#### 金额和数量

```json
{
  "price": 123.45, // 浮点数，单位为元
  "priceInCents": 12345, // 整数，单位为分（避免浮点数精度问题）
  "quantity": 10, // 整数数量
  "percentage": 85.5 // 百分比（0-100）
}
```

#### 状态和枚举

```json
{
  "status": "active", // 字符串枚举
  "type": 1, // 数字枚举（配合文档说明）
  "isEnabled": true, // 布尔值
  "visibility": "public" // 可见性枚举
}
```

### 2.3 敏感数据处理

**数据脱敏规则**:

```json
{
  "user": {
    "phone": "138****1234", // 手机号脱敏
    "email": "j***@example.com", // 邮箱脱敏
    "idCard": "110***********01", // 身份证脱敏
    "bankCard": "6225***********1234" // 银行卡脱敏
  }
}
```

**字段排除规则**:

```json
{
  "user": {
    "id": 123,
    "name": "张三",
    "email": "user@example.com"
    // password 字段永远不返回
    // internalId 内部字段不返回
    // deletedAt 软删除字段按需返回
  }
}
```

## 3. 自定义验证规则

### 3.1 业务规则验证

**订单验证示例**:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "创建订单请求",
  "required": ["items", "deliveryAddress"],
  "properties": {
    "items": {
      "type": "array",
      "minItems": 1,
      "maxItems": 50,
      "items": {
        "type": "object",
        "required": ["productId", "quantity"],
        "properties": {
          "productId": {
            "type": "integer",
            "minimum": 1,
            "description": "商品ID"
          },
          "quantity": {
            "type": "integer",
            "minimum": 1,
            "maximum": 999,
            "description": "购买数量"
          },
          "price": {
            "type": "number",
            "minimum": 0,
            "maximum": 99999999.99,
            "description": "单价"
          }
        }
      }
    },
    "couponCode": {
      "type": "string",
      "pattern": "^[A-Z0-9]{6,12}$",
      "description": "优惠券代码"
    },
    "deliveryAddress": {
      "type": "object",
      "required": ["recipient", "phone", "address"],
      "properties": {
        "recipient": {
          "type": "string",
          "minLength": 2,
          "maxLength": 20,
          "description": "收件人"
        },
        "phone": {
          "type": "string",
          "pattern": "^1[3-9]\\d{9}$",
          "description": "收件人电话"
        },
        "address": {
          "type": "string",
          "minLength": 10,
          "maxLength": 200,
          "description": "详细地址"
        }
      }
    }
  }
}
```

### 3.2 跨字段验证

**密码确认验证**:

```javascript
// 服务器端验证逻辑
function validatePasswordConfirmation(data) {
  if (data.password !== data.confirmPassword) {
    return {
      valid: false,
      error: {
        code: "VALIDATION_PASSWORD_MISMATCH",
        message: "密码确认不匹配",
        field: "confirmPassword",
      },
    };
  }
  return { valid: true };
}
```

**日期范围验证**:

```javascript
function validateDateRange(data) {
  const startDate = new Date(data.startDate);
  const endDate = new Date(data.endDate);

  if (startDate >= endDate) {
    return {
      valid: false,
      error: {
        code: "VALIDATION_INVALID_DATE_RANGE",
        message: "开始日期必须早于结束日期",
        fields: ["startDate", "endDate"],
      },
    };
  }
  return { valid: true };
}
```

## 4. 文件上传验证

### 4.1 文件类型验证

**允许的文件类型**:

```json
{
  "imageUpload": {
    "allowedTypes": ["image/jpeg", "image/png", "image/gif", "image/webp"],
    "maxSize": 5242880, // 5MB
    "maxWidth": 4096,
    "maxHeight": 4096
  },
  "documentUpload": {
    "allowedTypes": ["application/pdf", "application/msword", "text/plain"],
    "maxSize": 10485760, // 10MB
    "allowedExtensions": [".pdf", ".doc", ".docx", ".txt"]
  }
}
```

### 4.2 文件验证规则

```javascript
// 文件验证示例
function validateFileUpload(file, rules) {
  const errors = [];

  // 类型验证
  if (!rules.allowedTypes.includes(file.mimetype)) {
    errors.push({
      code: "VALIDATION_INVALID_FILE_TYPE",
      message: `不支持的文件类型: ${file.mimetype}`,
      allowedTypes: rules.allowedTypes,
    });
  }

  // 大小验证
  if (file.size > rules.maxSize) {
    errors.push({
      code: "VALIDATION_FILE_TOO_LARGE",
      message: `文件大小超过限制: ${file.size} > ${rules.maxSize}`,
      maxSize: rules.maxSize,
    });
  }

  // 扩展名验证
  if (rules.allowedExtensions) {
    const ext = path.extname(file.originalname).toLowerCase();
    if (!rules.allowedExtensions.includes(ext)) {
      errors.push({
        code: "VALIDATION_INVALID_FILE_EXTENSION",
        message: `不支持的文件扩展名: ${ext}`,
        allowedExtensions: rules.allowedExtensions,
      });
    }
  }

  return {
    valid: errors.length === 0,
    errors,
  };
}
```

## 5. 数据转换和净化

### 5.1 输入数据清理

```javascript
// 数据清理函数
function sanitizeInput(data) {
  return {
    email: data.email?.toLowerCase().trim(),
    name: data.name?.trim(),
    phone: data.phone?.replace(/\D/g, ""), // 只保留数字
    description: data.description?.trim().slice(0, 500), // 限制长度
    tags: data.tags?.map((tag) => tag.trim().toLowerCase()).filter(Boolean), // 清理标签
  };
}
```

### 5.2 HTML 内容净化

```javascript
// 使用类似 DOMPurify 的库净化 HTML 内容
function sanitizeHTML(html) {
  const allowedTags = ["p", "br", "strong", "em", "ul", "ol", "li"];
  const allowedAttributes = {
    a: ["href", "title"],
    img: ["src", "alt", "width", "height"],
  };

  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: allowedTags,
    ALLOWED_ATTR: allowedAttributes,
  });
}
```

## 6. 国际化验证

### 6.1 多语言字段验证

```json
{
  "title": {
    "type": "object",
    "required": ["zh-CN"],
    "properties": {
      "zh-CN": {
        "type": "string",
        "minLength": 1,
        "maxLength": 100,
        "description": "中文标题"
      },
      "en-US": {
        "type": "string",
        "maxLength": 100,
        "description": "英文标题"
      },
      "ja-JP": {
        "type": "string",
        "maxLength": 100,
        "description": "日文标题"
      }
    },
    "additionalProperties": false
  }
}
```

### 6.2 地区特定验证

```javascript
// 不同地区的手机号验证
const phonePatterns = {
  CN: /^1[3-9]\d{9}$/, // 中国大陆
  HK: /^[5-9]\d{7}$/, // 香港
  US: /^\+1[2-9]\d{2}[2-9]\d{6}$/, // 美国
  JP: /^0[789]0-?\d{4}-?\d{4}$/, // 日本
};

function validatePhoneByRegion(phone, region) {
  const pattern = phonePatterns[region];
  if (!pattern) {
    return { valid: false, error: "Unsupported region" };
  }

  return {
    valid: pattern.test(phone),
    error: pattern.test(phone) ? null : "Invalid phone format for region",
  };
}
```

## 7. 性能优化

### 7.1 验证缓存

```javascript
// 缓存编译后的 JSON Schema
const schemaCache = new Map();

function getCompiledSchema(schemaName) {
  if (!schemaCache.has(schemaName)) {
    const schema = loadSchema(schemaName);
    const compiled = ajv.compile(schema);
    schemaCache.set(schemaName, compiled);
  }
  return schemaCache.get(schemaName);
}
```

### 7.2 批量验证

```javascript
// 批量数据验证
function validateBatch(items, schema) {
  const validator = getCompiledSchema(schema);
  const results = [];

  for (let i = 0; i < items.length; i++) {
    const item = items[i];
    const valid = validator(item);

    results.push({
      index: i,
      valid,
      errors: valid ? null : validator.errors,
    });

    // 早期退出策略
    if (!valid && results.filter((r) => !r.valid).length > 10) {
      break; // 错误太多，停止验证
    }
  }

  return results;
}
```

## 8. 测试策略

### 8.1 验证测试用例

```javascript
describe("User Validation", () => {
  test("should accept valid user data", () => {
    const validUser = {
      email: "test@example.com",
      password: "SecurePass123!",
      name: "张三",
      age: 25,
    };

    const result = validateUser(validUser);
    expect(result.valid).toBe(true);
  });

  test("should reject invalid email", () => {
    const invalidUser = {
      email: "invalid-email",
      password: "SecurePass123!",
      name: "张三",
    };

    const result = validateUser(invalidUser);
    expect(result.valid).toBe(false);
    expect(result.errors[0].code).toBe("VALIDATION_INVALID_FORMAT");
  });

  test("should reject weak password", () => {
    const weakPasswordUser = {
      email: "test@example.com",
      password: "123456",
      name: "张三",
    };

    const result = validateUser(weakPasswordUser);
    expect(result.valid).toBe(false);
    expect(result.errors[0].field).toBe("password");
  });
});
```

### 8.2 边界值测试

```javascript
describe("Boundary Value Testing", () => {
  test("should handle minimum length strings", () => {
    const minLengthData = {
      name: "A", // 最小长度
      description: "", // 空字符串
    };

    const result = validate(minLengthData, schema);
    // 根据业务规则验证结果
  });

  test("should handle maximum values", () => {
    const maxValueData = {
      age: 120, // 最大年龄
      price: 99999999.99, // 最大价格
    };

    const result = validate(maxValueData, schema);
    expect(result.valid).toBe(true);
  });
});
```

## 总结

完善的数据验证规范需要考虑：

1. **全面性**: 覆盖所有输入输出数据
2. **安全性**: 防范各种注入攻击
3. **一致性**: 统一的验证规则和错误格式
4. **性能**: 高效的验证机制
5. **可维护性**: 清晰的规则定义和文档
6. **国际化**: 支持多语言和地区差异
7. **用户友好**: 清晰的错误提示

通过系统化的数据验证，可以显著提高 API 的安全性、稳定性和用户体验。
