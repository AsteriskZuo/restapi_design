# IM REST API 命名规范

本文档定义了 IM REST API 的命名规范，旨在确保 API 的一致性和可维护性。这些规范基于 REST 最佳实践和行业标准，同时考虑了 IM 业务场景的特殊需求。

## 1. 概述

### 1.1 命名原则

- **一致性**：保持命名风格的一致性，便于开发者理解和使用
- **可读性**：使用清晰、直观的命名，避免歧义
- **简洁性**：在保持清晰的前提下，尽量使用简短的命名
- **语义化**：命名应准确反映其用途和含义
- **向后兼容**：命名变更需要考虑向后兼容性

### 1.2 命名风格选择

| 场景      | 命名风格             | 示例             |
| --------- | -------------------- | ---------------- |
| URL 路径  | 小写字母，连字符分隔 | `/user-profiles` |
| 查询参数  | 小写字母，下划线分隔 | `sort_by`        |
| JSON 属性 | 驼峰命名法           | `userId`         |
| HTTP 头部 | 标准格式或 X- 前缀   | `X-Request-ID`   |

## 2. URL 路径命名

### 2.1 资源命名规则

- 使用复数形式表示资源集合
- 使用小写字母
- 使用连字符（-）分隔单词
- 避免使用动词，除非表示操作

**示例：**

```http
# ✅ 正确
/users
/message-attachments
/user-profiles

# ❌ 错误
/getUsers
/messageAttachments
/user_profiles
```

### 2.2 子资源命名规则

- 使用连字符（-）连接主资源和子资源
- 保持命名层次清晰

**示例：**

```http
# ✅ 正确
/users/{userId}/messages
/groups/{groupId}/members
/messages/{messageId}/attachments

# ❌ 错误
/users/{userId}messages
/groups/{groupId}members
```

### 2.3 操作标识符命名规则

- 使用动词表示操作
- 使用连字符（-）分隔单词
- 放在资源路径之后

**示例：**

```http
# ✅ 正确
/users/search
/messages/batch-send
/groups/batch-create

# ❌ 错误
/searchUsers
/sendMessages
/createGroups
```

## 3. 查询参数命名

### 3.1 过滤参数命名规则

- 使用小写字母
- 使用下划线（\_）分隔单词
- 使用描述性名称

**示例：**

```http
# ✅ 正确
?status=active
?created_at=2024-01-01
?message_type=text

# ❌ 错误
?Status=active
?createdAt=2024-01-01
?messageType=text
```

### 3.2 排序参数命名规则

- 使用 `sort_by` 指定排序字段
- 使用 `order` 指定排序方向（asc/desc）

**示例：**

```http
# ✅ 正确
?sort_by=created_at&order=desc
?sort_by=message_count&order=asc

# ❌ 错误
?sort=createdAt
?orderBy=messageCount
```

### 3.3 分页参数命名规则

- 使用 `page` 表示页码
- 使用 `limit` 表示每页数量
- 使用 `cursor` 表示游标分页的游标值

**示例：**

```http
# ✅ 正确
?page=1&limit=20
?cursor=eyJjcmVhdGVkX2F0IjoiMjAyNC0wMS0wMSJ9

# ❌ 错误
?pageNumber=1
?pageSize=20
?nextCursor=eyJjcmVhdGVkX2F0IjoiMjAyNC0wMS0wMSJ9
```

## 4. HTTP 头部命名

### 4.1 标准头部使用规范

- 保持标准 HTTP 头部的原有格式
- 遵循 HTTP 规范的大小写规则

**示例：**

```http
Content-Type: application/json
Authorization: Bearer token123
Accept: application/json
```

### 4.2 自定义头部命名规则

- 使用 `X-` 前缀
- 使用连字符（-）分隔单词
- 使用描述性名称

**示例：**

```http
# ✅ 正确
X-Request-ID: req123
X-Client-Version: 1.0.0
X-Device-Type: ios

# ❌ 错误
x-request-id: req123
X_Client_Version: 1.0.0
XDeviceType: ios
```

## 5. JSON 数据命名

### 5.1 请求体命名规则

- 使用驼峰命名法（camelCase）
- 使用描述性名称
- 避免缩写（除非是广泛接受的缩写）

**示例：**

```json
{
  "userId": "123",
  "nickname": "张三",
  "avatarUrl": "https://example.com/avatar.jpg",
  "isOnline": true,
  "lastActiveAt": "2024-01-01T12:00:00Z"
}
```

### 5.2 响应体命名规则

- 使用驼峰命名法（camelCase）
- 保持与请求体命名风格一致
- 使用标准响应结构

**示例：**

```json
{
  "data": {
    "userId": "123",
    "nickname": "张三",
    "avatarUrl": "https://example.com/avatar.jpg",
    "isOnline": true,
    "lastActiveAt": "2024-01-01T12:00:00Z"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req123"
  }
}
```

### 5.3 错误响应命名规则

- 使用驼峰命名法（camelCase）
- 使用标准错误结构

**示例：**

```json
{
  "error": {
    "code": "40042020301",
    "type": "USER_NOT_FOUND",
    "message": "用户不存在",
    "details": {
      "userId": "123",
      "reason": "用户已被删除"
    }
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req123"
  }
}
```

## 6. 最佳实践

### 6.1 常见错误和注意事项

- 避免使用缩写（除非是广泛接受的缩写）
- 避免使用特殊字符
- 避免使用数字开头
- 避免使用保留字
- 避免使用过长的名称

### 6.2 命名冲突处理

- 使用更具体的名称避免冲突
- 使用命名空间区分不同模块
- 使用版本号区分不同版本

### 6.3 向后兼容性考虑

- 保持命名风格的一致性
- 避免频繁更改命名
- 提供命名变更的过渡期
- 在文档中明确说明命名变更

## 7. 示例

### 7.1 完整 API 示例

```http
# 创建用户
POST /v1/users
Content-Type: application/json
X-Request-ID: req123

{
  "nickname": "张三",
  "avatarUrl": "https://example.com/avatar.jpg",
  "isOnline": true
}

# 查询用户列表
GET /v1/users?status=active&sort_by=created_at&order=desc&page=1&limit=20
X-Request-ID: req124

# 获取用户消息
GET /v1/users/{userId}/messages?message_type=text&created_at=2024-01-01
X-Request-ID: req125

# 批量发送消息
POST /v1/messages/batch-send
Content-Type: application/json
X-Request-ID: req126

{
  "targetIds": ["user1", "user2"],
  "messageType": "text",
  "content": "Hello, World!"
}
```

### 7.2 响应示例

```json
{
  "data": {
    "userId": "123",
    "nickname": "张三",
    "avatarUrl": "https://example.com/avatar.jpg",
    "isOnline": true,
    "lastActiveAt": "2024-01-01T12:00:00Z",
    "messages": [
      {
        "messageId": "msg1",
        "content": "Hello, World!",
        "createdAt": "2024-01-01T12:00:00Z"
      }
    ]
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "req123",
    "pagination": {
      "total": 100,
      "page": 1,
      "limit": 20
    }
  }
}
```
