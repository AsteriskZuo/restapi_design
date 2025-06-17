# IM REST API 命名规范

本文档定义了 IM REST API 的命名规范，确保 API 的一致性和可维护性。

## 1. 命名规范总览

### 1.1 四类命名风格

| 场景      | 命名风格             | 示例             |
| --------- | -------------------- | ---------------- |
| URL 路径  | 小写字母，下划线分隔 | `/user_profiles` |
| 查询参数  | 小写字母，下划线分隔 | `user_id`        |
| JSON 属性 | 驼峰命名法           | `userId`         |
| HTTP 头部 | 标准格式或 X- 前缀   | `X-Request-ID`   |

### 1.2 通用原则

- **一致性**：在同一场景中统一使用对应的命名风格（例如，所有 URL 路径都使用下划线风格，所有 JSON 属性都使用驼峰风格）
- **可读性**：使用清晰、直观的命名
- **简洁性**：避免过长的名称
- **语义化**：准确反映用途和含义

> **重要**：不同场景必须严格遵循对应的命名风格。URL 路径和查询参数使用下划线风格，而 JSON 属性使用驼峰风格，这种差异是有意设计的，符合各自领域的最佳实践。

### 1.3 常见错误示例

```
❌ 混用命名风格：URL 路径使用驼峰命名法（`/userProfiles`）
❌ 混用命名风格：JSON 属性使用下划线（`"user_id": "123"`）
❌ 不一致的命名：同一概念在不同场景中使用不同术语（URL 中用`user_id`，JSON 中用`userId`）
```

## 2. URL 路径命名

**规则：**

- 使用复数形式：`/users`、`/messages`
- 小写字母 + 下划线：`/user_profiles`、`/message_attachments`
- 子资源用斜杠连接：`/users/{userId}/messages`
- 操作用动词：`/users/search`

**示例：**

```http
✅ /users/{userId}/messages
✅ /groups/{groupId}/members
✅ /messages/batch_send

❌ /getUsers
❌ /messageAttachments
❌ /user-profiles
```

## 3. 查询参数命名

**规则：**

- 小写字母 + 下划线：`created_at`、`message_type`
- 分页参数：`page`、`limit`、`cursor`
- 排序参数：`sort_by`、`order`

**示例：**

```http
✅ ?status=active&created_at=2024-01-01
✅ ?sort_by=created_at&order=desc
✅ ?page=1&limit=20

❌ ?createdAt=2024-01-01
❌ ?created-at=2024-01-01
❌ ?sortBy=created_at
```

## 4. HTTP 头部命名

**规则：**

- 标准头部保持原格式：`Content-Type`、`Authorization`
- 自定义头部使用 `X-` 前缀 + 连字符：`X-Request-ID`

**示例：**

```http
✅ Content-Type: application/json
✅ X-Request-ID: req123
✅ X-Client-Version: 1.0.0

❌ x-request-id: req123
❌ X_Client_Version: 1.0.0
```

## 5. JSON 属性命名

**规则：**

- 使用驼峰命名法：`userId`、`createdAt`、`isOnline`
- 请求体和响应体保持一致

**示例：**

```json
✅ {
  "userId": "123",
  "createdAt": "2024-01-01T12:00:00Z",
  "isOnline": true
}

❌ {
  "user_id": "123",
  "created-at": "2024-01-01T12:00:00Z"
}
```

## 6. 完整示例

### 6.1 API 请求示例

```http
# 创建用户
POST /v1/users
Content-Type: application/json
X-Request-ID: req123

{
  "nickname": "张三",
  "avatarUrl": "https://example.com/avatar.jpg"
}

# 查询用户消息
GET /v1/users/{userId}/messages?message_type=text&created_at=2024-01-01&sort_by=created_at&order=desc&page=1&limit=20
X-Request-ID: req124

# 批量发送消息
POST /v1/messages/batch_send
Content-Type: application/json
X-Request-ID: req125

{
  "targetIds": ["user1", "user2"],
  "messageType": "text",
  "content": "Hello, World!"
}
```

### 6.2 响应示例

```json
{
  "data": {
    "userId": 123,
    "username": "zhangsan"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "aut1704110400456abc123def78912"
  }
}
```

### 6.3 错误响应示例

```json
{
  "error": {
    "code": "40141010101",
    "type": "AUTHENTICATION_FAILED",
    "message": "用户名或密码错误"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "aut1704110400789def456abc12345"
  }
}
```

错误码响应格式，[详见](./im_response_format.md)
