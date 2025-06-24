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
# ❌ 错误示例
混用命名风格：URL 路径使用驼峰命名法（`/userProfiles`）
混用命名风格：JSON 属性使用下划线（`"user_id": "123"`）
不一致的命名：同一概念在不同场景中使用不同术语（URL 中用`user_id`，JSON 中用`userId`）
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
- 分页参数：`page`、`size`、`cursor`、`limit`
- 排序参数：`sort`

**示例：**

```http
# ✅ 正确
✅ ?status=active&created_at=2024-01-01
✅ ?sort_by=created_at&order=desc
✅ ?page=1&limit=20

# ❌ 错误
?createdAt=2024-01-01
?created-at=2024-01-01
?sortBy=created_at
```

## 4. HTTP 头部命名

**规则：**

- 标准头部保持原格式：`Content-Type`、`Authorization`
- 自定义头部使用 `X-` 前缀 + 连字符：`X-Request-ID`

**示例：**

```http
# ✅ 正确
Content-Type: application/json
X-Request-ID: req123
X-Client-Version: 1.0.0

# ❌ 错误
x-request-id: req123
X_Client_Version: 1.0.0
```

## 5. JSON 属性命名

**规则：**

- 使用驼峰命名法：`userId`、`createdAt`、`isOnline`
- 请求体和响应体保持一致

**示例：**

```json
# ✅ 正确
{
  "userId": "123",
  "createdAt": "2024-01-01T12:00:00Z",
  "isOnline": true
}

# ❌ 错误
{
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

## 7. 最佳实践

在 IM 业务中，建议 产品设计文档、开发 HLD 文档、restapi 接口说明、sdk 接口说明、集成文档、测试文档等文档中保持**术语一致**。 例如：用户术语使用 user（当前有 account、user、contact 等），群组术语使用 group（当前有 group、chatgroup 等）。

### 7.1 唯一标识符

用户 ID、群组 ID、聊天室 ID、消息 ID、文件 ID、群成员 ID 等，建议使用 Id 作为名称的后缀。

- **生成方式**：建议在客户端或服务端使用分布式算法生成全局唯一 ID，避免依赖数据库自增主键，确保水平扩展能力。
  - **UUIDv4**：基于随机数，128 bit，碰撞概率极低。
  - **Snowflake**：基于时间戳的 64 bit 整数，按时间趋势递增，方便排序。
  - **ULID**：可读性更高的 Base-32 编码，按时间排序且 URL-safe。
- **无语义性**：Id 不应包含业务或隐私信息，仅承担唯一性职能，防止数据泄漏。

推荐 ✅
例如：userId，groupId、roomId、messageId、fileId、memberId。

不推荐 ❌
例如：userName，groupName。

### 7.2 名字

用户名、群组名、聊天室名、消息名、文件名、群成员名等，建议使用 Name 作为名称的后缀。

- **命名原则**：Name 字段面向人类展示，应当可读、具备业务含义，并符合以下要求：
  - **长度限制**：建议长度控制在合理范围。例如： 4-64 个字符。
  - **安全合规**：禁止包含敏感词、XSS 或 SQL 注入相关字符，必要时进行内容审核。

推荐 ✅
例如：userName，groupName。

不推荐 ❌
例如：user，group。

### 7.3 备注

好友备注、群组备注、聊天室备注、群成员备注等，建议使用 Remark 作为名称的后缀。 （当前有 nickName 等）

推荐 ✅
例如：userRemark，groupRemark, memberRemark。

### 7.4 时间戳

统一采用 毫秒级为单位的时间戳（`1704110400000`），建议使用 timestamp 作为名称的后缀。

采用 UTC 标准。

推荐 ✅
例如：timestamp， serverTimestamp, localTimestamp。

### 7.5 状态

用户状态、消息状态、文件状态等，建议使用 status 作为名称的后缀。（不要使用 state）

### 7.6 其它关键字

资源的动作关键字，增删改查，例如：add、delete、update、delete 等。 (资源包括、用户、群组、聊天室 等)

资源的更新方式：updateToLocal, updateToServer 等。（SDK 参考）
资源的获取方式：getFromLocal, getFromServer 等。（SDK 参考）

消息的动作关键字，例如：send、receive、delete、recall、forward、update、insert、get、fetch 等。
消息的类型枚举值，例如：text、image、file、location、voice、video、custom 等。
消息的角色关键字，例如：senderId、receiverId、conversionId 等。(不要使用 from，to 等其他关键字)

会话的特有关键字，例如：mute、unmute、pin、unpin、read、unread 等。

群组的特有关键字，例如：thread、reaction 等。

# 参考文档

[tencent_user_naming](https://cloud.tencent.com/document/product/269/38417)

[tencent_group_naming](https://cloud.tencent.com/document/product/269/1615#.E8.AF.B7.E6.B1.82.E5.8C.85.E7.A4.BA.E4.BE.8B)
