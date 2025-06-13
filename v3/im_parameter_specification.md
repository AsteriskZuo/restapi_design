# IM 参数规范

## 1. 设计理念

**核心原则**：统一、灵活、高效
**适用场景**：所有 API 请求和响应
**技术标准**：基于 HTTP 标准参数传递

## 2. 参数规范

### 2.1 参数类型

1. **路径参数**

   - 用于标识资源
   - 示例：`/api/v1/users/{userId}`

2. **查询参数**

   - 用于过滤、排序、分页
   - 示例：`?status=active&sort=createdAt:desc`

3. **请求体参数**

   - 用于复杂数据传递
   - 示例：JSON 格式的请求体

4. **请求头参数**

   - 用于认证、控制
   - 示例：`authorization: Bearer token`

5. **响应头参数**

   - 用于元数据、控制信息
   - 示例：`x-request-id`、`x-rate-limit-remaining`

6. **响应体参数**
   - 用于返回业务数据
   - 示例：JSON 格式的响应体

### 2.2 参数编码

**编码规则：**

- 使用 UTF-8 字符集
- URL 参数必须进行 URL 编码
- 请求体使用 JSON 格式时，字符串值需要 JSON 编码

**URL 参数编码：**

- 非 ASCII 字符：`%xx` 格式
- 特殊字符：`%xx` 格式
- 空格：`%20` 或 `+`
- 保留字符：`%xx` 格式

**示例：**

```
# 原始参数
name=张三&type=技术交流

# URL 编码后
name=%E5%BC%A0%E4%B8%89&type=%E6%8A%80%E6%9C%AF%E4%BA%A4%E6%B5%81
```

**JSON 编码：**

```json
{
  "name": "张三",
  "type": "技术交流"
}
```

**注意事项：**

- 避免在 URL 中使用中文等非 ASCII 字符
- 查询参数优先使用英文
- 必须使用中文时，确保正确编码
- 考虑不同浏览器的编码行为差异

## 3. IM 业务场景示例

### 3.1 用户管理

**创建用户：**

```
POST /api/v1/users
content-type: application/json

{
  "username": "zhangSan",
  "password": "******",
  "nickname": "张三",
  "avatar": "https://..."
}
```

**响应：**

```
HTTP/1.1 201 Created
content-type: application/json
x-rate-limit-remaining: 999

{
  "data": {
    "userId": "user123",
    "username": "zhangSan",
    "nickname": "张三",
    "avatar": "https://...",
    "createdAt": "2024-03-21T10:00:00Z"
  },
  "meta": {
    "requestId": "req_123456789",
    "timestamp": "2024-03-21T10:00:00Z"
  }
}
```

**更新用户：**

```
PUT /api/v1/users/{userId}
content-type: application/json

{
  "nickname": "新昵称",
  "avatar": "https://..."
}
```

### 3.2 群组管理

**创建群组：**

```
POST /api/v1/groups
content-type: application/json

{
  "name": "技术交流群",
  "description": "技术讨论",
  "isPublic": true,
  "maxMembers": 200
}
```

**响应：**

```
HTTP/1.1 201 Created
content-type: application/json
x-rate-limit-remaining: 999

{
  "data": {
    "groupId": "group123",
    "name": "技术交流群",
    "description": "技术讨论",
    "isPublic": true,
    "maxMembers": 200,
    "createdAt": "2024-03-21T10:00:00Z"
  },
  "meta": {
    "requestId": "req_123456789",
    "timestamp": "2024-03-21T10:00:00Z"
  }
}
```

**更新群组：**

```
PUT /api/v1/groups/{groupId}
content-type: application/json

{
  "name": "新群名",
  "description": "新描述"
}
```

### 3.3 消息管理

**发送消息：**

```
POST /api/v1/messages
content-type: application/json

{
  "chatType": "single",
  "chatId": "user123",
  "content": "Hello",
  "type": "text"
}
```

**响应：**

```
HTTP/1.1 201 Created
content-type: application/json
x-rate-limit-remaining: 999

{
  "data": {
    "messageId": "msg123",
    "chatType": "single",
    "chatId": "user123",
    "content": "Hello",
    "type": "text",
    "createdAt": "2024-03-21T10:00:00Z"
  },
  "meta": {
    "requestId": "req_123456789",
    "timestamp": "2024-03-21T10:00:00Z"
  }
}
```

**更新消息：**

```
PUT /api/v1/messages/{messageId}
content-type: application/json

{
  "content": "Updated content"
}
```
