# IM REST 响应格式设计

本文档基于 `RFC7807`、`JSON:API` 等国际标准，结合 `Claude-4-Sonnet` 的最佳实践，深度分析 IM 业务场景特点，形成了这套完整的响应格式设计方案。

## 1. 响应格式构成规则

**标准响应结构 = 核心数据(data)/错误信息(error) + 元数据(meta)**

**分层结构：**

- **data 字段**：包含实际的业务数据（成功响应必需）
- **error 字段**：包含错误信息（失败响应必需）
- **meta 字段**：包含响应元数据信息（推荐）

**基础示例：**

```json
{
  "data": { "id": 123, "name": "张三" },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req1704110400012abc456def78901"
  }
}
```

## 2. 字段分类定义

### 2.1 核心字段 (data)

**用途**：包含实际业务数据
**类型**：对象、数组、基本类型或 null
**必需性**：所有成功响应必需

**数据类型规则：**

- **单个资源**：对象格式
- **资源列表**：数组格式
- **空结果**：null 或空数组

**data 为空 示例：**

```json
{
  "data": null,
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req1704110400012abc456def78901"
  }
}
```

### 2.2 错误字段 (error)

**用途**：请求失败时的错误信息
**类型**：对象
**必需性**：所有失败响应必需

**核心子字段：**

- **code**：错误代码（业务错误码）
- **message**：用户友好的错误消息

### 2.3 元数据字段 (meta)

**用途**：响应相关的元数据信息
**类型**：对象
**必需性**：强烈推荐包含

**核心子字段：**

- **timestamp**：响应生成时间（毫秒级时间戳）
- **requestId**：请求追踪标识
- **responseTime**：响应耗时（可选）

## 3. 最佳实践

### 3.1 requestId 生成规则(仅供参考)

确保请求唯一性和可追踪性，支持分布式环境下的请求标识和问题排查。

**组成结构 = 前缀(3 位) + 时间戳(13 位) + 设备标识(6 位) + 随机数(8 位)**

**分层结构：**

- **前缀(3 位)**：业务模块标识（req、usr、grp、msg 等）
- **时间戳(13 位)**：毫秒级时间戳，保证时序性和唯一性
- **设备标识(6 位)**：设备 ID 的 hash 值后 6 位，支持分布式追踪
- **随机数(8 位)**：随机字符串，避免高并发碰撞

**前缀映射规则：**

- **auth**: aut（认证模块）
- **users**: usr（用户模块）
- **groups**: grp（群组模块）
- **messages**: msg（消息模块）
- **push**: psh（推送模块）
- **default**: req（通用请求）

**生成步骤：**

- **第 1 步**: 根据业务模块获取 3 位前缀
- **第 2 步**: 获取当前 13 位毫秒时间戳
- **第 3 步**: 取设备 ID 后 6 位作为设备标识（不足补 0）
- **第 4 步**: 生成 8 位随机字符串
- **第 5 步**: 按顺序拼接所有部分

**生成示例：**

```
aut1704110400123abc456defghi12  # 认证模块请求
usr1704110400456def789ghiabc34  # 用户模块请求
grp1704110400789ghi123abcdef56  # 群组模块请求
msg1704110400012abc456def78901  # 消息模块请求
```

### 3.2 meta 参数

推荐在 IM 业务场景中 在 meta 中包含一些元数据信息，如请求时间、请求 ID 等，方便后续问题排查和跟踪。

```json
{
  "data": {
    "userId": "123",
    "username": "zhangsan"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "aut1704110400456abc123def78912"
  }
}
```

### 3.3 error 参数

推荐在 IM 业务场景中 在 error 中包含一些错误信息，如错误码、错误信息等，方便后续问题排查和跟踪。

- `error.code`: 错误码(整数)
- `error.message`: 错误信息(字符串，或者 json 的序列化的字符串等)

```json
{
  "error": {
    "code": 4010101,
    "message": "用户名或密码错误"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "aut1704110400789def456abc12345"
  }
}
```

## 4. IM 业务场景响应示例

### 4.1 auth 认证模块

#### 用户登录成功

```json
{
  "data": {
    "userId": "123",
    "username": "zhangsan",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 7200,
    "refreshToken": "refresh_token_string"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "aut1704110400456abc123def78912"
  }
}
```

#### 用户登录失败

```json
{
  "error": {
    "code": 4010101,
    "message": "用户名或密码错误"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "aut1704110400789def456abc12345"
  }
}
```

#### Token 刷新成功

```json
{
  "data": {
    "token": "new_access_token_string",
    "expiresIn": 7200,
    "refreshToken": "new_refresh_token_string"
  },
  "meta": {
    "timestamp": 1704112200000,
    "requestId": "aut1704112200012ghi789abc54321"
  }
}
```

### 4.2 users 用户模块

#### 单个用户查询

```json
{
  "data": {
    "userId": "123",
    "username": "zhangsan",
    "nickname": "张三",
    "email": "zhangsan@example.com",
    "avatar": "https://example.com/avatars/123.jpg",
    "status": "online",
    "createdAt": 1704103200000,
    "lastLoginAt": 1704108600000
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "usr1704110400123def456ghi78901"
  }
}
```

#### 用户列表查询（带分页）

```json
{
  "data": [
    {
      "userId": "123",
      "username": "zhangsan",
      "nickname": "张三",
      "status": "online"
    },
    {
      "userId": "124",
      "username": "lisi",
      "nickname": "李四",
      "status": "offline"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 156,
      "totalPages": 16,
      "hasNext": true,
      "hasPrev": false
    },
    "timestamp": 1704110400000,
    "requestId": "usr1704110400234ghi567abc12345"
  }
}
```

#### 创建用户成功

```json
{
  "data": {
    "userId": "125",
    "username": "wangwu",
    "nickname": "王五",
    "email": "wangwu@example.com",
    "status": "active",
    "createdAt": 1704110400000
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "usr1704110400345abc678def12345"
  }
}
```

### 4.3 groups 群组模块

#### 群组详情查询（包含成员信息）

```json
{
  "data": {
    "groupId": "group_001",
    "groupName": "技术讨论群",
    "description": "前端技术交流群",
    "avatar": "https://example.com/group-avatars/group_001.jpg",
    "memberCount": 25,
    "maxMembers": 200,
    "isPublic": true,
    "createdAt": 1704096000000,
    "ownerId": 123,
    "members": [
      { "userId": "123", "role": "owner" },
      { "userId": "124", "role": "admin" }
    ]
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "grp1704110400456def789ghi12345"
  }
}
```

#### 群组列表查询

```json
{
  "data": [
    {
      "groupId": "group_001",
      "groupName": "技术讨论群",
      "memberCount": 25,
      "isPublic": true
    },
    {
      "groupId": "group_002",
      "groupName": "项目协作群",
      "memberCount": 12,
      "isPublic": false
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 5,
      "totalPages": 1,
      "hasNext": false,
      "hasPrev": false
    },
    "timestamp": 1704110400000,
    "requestId": "grp1704110400567abc890def56789"
  }
}
```

### 4.4 messages 消息模块

#### 发送消息成功

```json
{
  "data": {
    "messageId": "msg_789",
    "from": "zhangsan",
    "to": "lisi",
    "type": "text",
    "body": {
      "text": "Hello, how are you?"
    },
    "timestamp": 1704110400000,
    "status": "sent"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "msg1704110400678def901abc67890",
    "responseTime": "150ms"
  }
}
```

#### 消息历史查询

```json
{
  "data": [
    {
      "messageId": "msg_789",
      "from": "zhangsan",
      "to": "lisi",
      "type": "text",
      "body": { "text": "Hello, how are you?" },
      "timestamp": 1704110400000
    },
    {
      "messageId": "msg_788",
      "from": "lisi",
      "to": "zhangsan",
      "type": "text",
      "body": { "text": "I'm fine, thanks!" },
      "timestamp": 1704110280000
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 156,
      "totalPages": 4,
      "hasNext": true
    },
    "timestamp": 1704110700000,
    "requestId": "msg1704110700789ghi012def78901"
  }
}
```

### 4.5 批量操作响应

#### 批量添加群成员

```json
{
  "data": {
    "success": [
      {
        "userId": "126",
        "username": "user1",
        "status": "added",
        "joinedAt": 1704110400000
      },
      {
        "userId": "127",
        "username": "user2",
        "status": "added",
        "joinedAt": 1704110400000
      }
    ],
    "failed": [
      {
        "userId": "128",
        "username": "user3",
        "error": {
          "code": 4030202,
          "message": "群成员数量已达上限"
        }
      }
    ]
  },
  "meta": {
    "summary": {
      "totalCount": 3,
      "successCount": 2,
      "failedCount": 1
    },
    "timestamp": 1704110400000,
    "requestId": "grp1704110400890abc123ghi45678"
  }
}
```

### 4.6 异步操作响应

#### 文件上传（异步处理）

```json
{
  "data": {
    "taskId": "upload_task_456",
    "status": "processing",
    "progress": 65,
    "fileId": "file_123",
    "fileName": "document.pdf",
    "fileSize": 2048576,
    "estimatedTime": 180
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req1704110400901def234abc78901"
  }
}
```

### 4.7 空结果响应

#### 空消息列表

```json
{
  "data": [],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 0,
      "totalPages": 0,
      "hasNext": false,
      "hasPrev": false
    },
    "timestamp": 1704110400000,
    "requestId": "msg1704110400012ghi345abc78901"
  }
}
```

#### 用户不存在

```json
{
  "data": null,
  "meta": {
    "timestamp": 1704110700000,
    "requestId": "usr1704110400123abc456def78901"
  }
}
```
