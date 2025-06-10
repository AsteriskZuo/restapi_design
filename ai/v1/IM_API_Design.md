# IM 即时通讯 REST API 设计文档

## 文档概述

本文档基于 [基本设计规范](../../v2/basic_design.md) 和 [高级设计规范](../../v2/advance_design.md)，为 IM 即时通讯系统设计完整的 REST API 接口。

## 系统架构

### URL 基础结构

```
https://{host}/{version}/{org_name}/{app_name}/{resource}
```

**示例**：
```
https://api.example.com/v1/myorg/mychat/auth
https://api.example.com/v1/myorg/mychat/users
https://api.example.com/v1/myorg/mychat/messages
```

### 核心模块

- **auth**: 认证授权模块
- **users**: 用户管理模块  
- **groups**: 群组管理模块
- **rooms**: 聊天室管理模块
- **messages**: 消息管理模块
- **push**: 推送通知模块

## 1. 认证授权模块 (Auth)

### 1.1 用户登录

```http
POST /v1/{org_name}/{app_name}/auth/login
```

**请求体**：
```json
{
  "username": "zhangsan",
  "password": "password123",
  "deviceId": "device_12345",
  "deviceType": "iOS",
  "clientVersion": "1.0.0"
}
```

**响应**：
```json
{
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600,
    "tokenType": "Bearer",
    "user": {
      "id": "user_123",
      "username": "zhangsan",
      "nickname": "张三",
      "avatar": "https://cdn.example.com/avatars/123.jpg",
      "status": "online"
    }
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "v1"
  }
}
```

### 1.2 刷新令牌

```http
POST /v1/{org_name}/{app_name}/auth/refresh
```

**请求体**：
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 1.3 登出

```http
POST /v1/{org_name}/{app_name}/auth/logout
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "deviceId": "device_12345"
}
```

### 1.4 注册

```http
POST /v1/{org_name}/{app_name}/auth/register
```

**请求体**：
```json
{
  "username": "lisi",
  "password": "password123", 
  "email": "lisi@example.com",
  "phone": "+8613800138000",
  "nickname": "李四",
  "avatar": "https://cdn.example.com/avatars/default.jpg"
}
```

## 2. 用户管理模块 (Users)

### 2.1 获取用户信息

```http
GET /v1/{org_name}/{app_name}/users/{user_id}
Authorization: Bearer {access_token}
```

**响应**：
```json
{
  "data": {
    "id": "user_123",
    "username": "zhangsan",
    "nickname": "张三",
    "avatar": "https://cdn.example.com/avatars/123.jpg",
    "email": "zhangsan@example.com",
    "phone": "+8613800138000",
    "status": "online",
    "lastSeen": "2024-01-01T12:00:00Z",
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T12:00:00Z"
  }
}
```

### 2.2 更新用户信息

```http
PATCH /v1/{org_name}/{app_name}/users/{user_id}
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "nickname": "新昵称",
  "avatar": "https://cdn.example.com/avatars/new.jpg",
  "signature": "个性签名"
}
```

### 2.3 搜索用户

```http
GET /v1/{org_name}/{app_name}/users/search?q={keyword}&page=1&limit=10
Authorization: Bearer {access_token}
```

**响应**：
```json
{
  "data": [
    {
      "id": "user_123",
      "username": "zhangsan",
      "nickname": "张三",
      "avatar": "https://cdn.example.com/avatars/123.jpg",
      "status": "online"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 1,
      "totalPages": 1,
      "hasNext": false,
      "hasPrev": false
    }
  }
}
```

### 2.4 用户状态管理

#### 设置在线状态

```http
PATCH /v1/{org_name}/{app_name}/users/{user_id}/status
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "status": "online|offline|away|busy",
  "statusMessage": "忙碌中"
}
```

### 2.5 好友关系管理

#### 获取好友列表

```http
GET /v1/{org_name}/{app_name}/users/{user_id}/friends?page=1&limit=50
Authorization: Bearer {access_token}
```

#### 发送好友请求

```http
POST /v1/{org_name}/{app_name}/users/{user_id}/friend-requests
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "targetUserId": "user_456",
  "message": "我是张三，想和你成为好友"
}
```

#### 处理好友请求

```http
PATCH /v1/{org_name}/{app_name}/users/{user_id}/friend-requests/{request_id}
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "action": "accept|reject"
}
```

#### 删除好友

```http
DELETE /v1/{org_name}/{app_name}/users/{user_id}/friends/{friend_id}
Authorization: Bearer {access_token}
```

## 3. 群组管理模块 (Groups)

### 3.1 创建群组

```http
POST /v1/{org_name}/{app_name}/groups
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "name": "技术讨论群",
  "description": "技术交流与讨论",
  "avatar": "https://cdn.example.com/groups/tech.jpg",
  "type": "public|private",
  "maxMembers": 500,
  "memberIds": ["user_123", "user_456"]
}
```

**响应**：
```json
{
  "data": {
    "id": "group_123",
    "name": "技术讨论群",
    "description": "技术交流与讨论",
    "avatar": "https://cdn.example.com/groups/tech.jpg",
    "type": "public",
    "memberCount": 2,
    "maxMembers": 500,
    "ownerId": "user_123",
    "createdAt": "2024-01-01T12:00:00Z"
  }
}
```

### 3.2 获取群组信息

```http
GET /v1/{org_name}/{app_name}/groups/{group_id}
Authorization: Bearer {access_token}
```

### 3.3 更新群组信息

```http
PATCH /v1/{org_name}/{app_name}/groups/{group_id}
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "name": "新群组名称",
  "description": "新的群组描述",
  "avatar": "https://cdn.example.com/groups/new.jpg"
}
```

### 3.4 群组成员管理

#### 获取群组成员列表

```http
GET /v1/{org_name}/{app_name}/groups/{group_id}/members?page=1&limit=50
Authorization: Bearer {access_token}
```

#### 添加群组成员

```http
POST /v1/{org_name}/{app_name}/groups/{group_id}/members
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "userIds": ["user_789", "user_101"]
}
```

#### 移除群组成员

```http
DELETE /v1/{org_name}/{app_name}/groups/{group_id}/members/{user_id}
Authorization: Bearer {access_token}
```

#### 设置群组管理员

```http
PATCH /v1/{org_name}/{app_name}/groups/{group_id}/members/{user_id}
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "role": "admin|member",
  "permissions": ["manage_members", "delete_messages"]
}
```

### 3.5 退出群组

```http
DELETE /v1/{org_name}/{app_name}/groups/{group_id}/members/me
Authorization: Bearer {access_token}
```

### 3.6 解散群组

```http
DELETE /v1/{org_name}/{app_name}/groups/{group_id}
Authorization: Bearer {access_token}
```

## 4. 聊天室管理模块 (Rooms)

### 4.1 创建聊天室

```http
POST /v1/{org_name}/{app_name}/rooms
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "type": "private|group",
  "participants": ["user_123", "user_456"],
  "groupId": "group_123",
  "settings": {
    "allowAnonymous": false,
    "messageRetention": 30
  }
}
```

### 4.2 获取聊天室列表

```http
GET /v1/{org_name}/{app_name}/rooms?type=all&page=1&limit=20
Authorization: Bearer {access_token}
```

**响应**：
```json
{
  "data": [
    {
      "id": "room_123",
      "type": "private",
      "participants": [
        {
          "id": "user_123",
          "nickname": "张三",
          "avatar": "https://cdn.example.com/avatars/123.jpg"
        }
      ],
      "lastMessage": {
        "id": "msg_789",
        "content": "你好",
        "type": "text",
        "senderId": "user_456",
        "timestamp": "2024-01-01T12:00:00Z"
      },
      "unreadCount": 3,
      "updatedAt": "2024-01-01T12:00:00Z"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 5,
      "totalPages": 1
    }
  }
}
```

### 4.3 获取聊天室详情

```http
GET /v1/{org_name}/{app_name}/rooms/{room_id}
Authorization: Bearer {access_token}
```

### 4.4 更新聊天室设置

```http
PATCH /v1/{org_name}/{app_name}/rooms/{room_id}
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "settings": {
    "muteUntil": "2024-01-02T12:00:00Z",
    "pinned": true
  }
}
```

## 5. 消息管理模块 (Messages)

### 5.1 发送消息

```http
POST /v1/{org_name}/{app_name}/messages
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "roomId": "room_123",
  "type": "text|image|file|audio|video|location",
  "content": "Hello World",
  "replyToId": "msg_456",
  "metadata": {
    "fileName": "document.pdf",
    "fileSize": 1024000,
    "duration": 30
  }
}
```

**响应**：
```json
{
  "data": {
    "id": "msg_789",
    "roomId": "room_123",
    "senderId": "user_123",
    "type": "text",
    "content": "Hello World",
    "replyTo": {
      "id": "msg_456",
      "content": "原消息内容",
      "senderId": "user_456"
    },
    "status": "sent",
    "timestamp": "2024-01-01T12:00:00Z",
    "editedAt": null
  }
}
```

### 5.2 获取消息历史

```http
GET /v1/{org_name}/{app_name}/messages?roomId={room_id}&before={timestamp}&limit=50
Authorization: Bearer {access_token}
```

**响应**：
```json
{
  "data": [
    {
      "id": "msg_789",
      "roomId": "room_123",
      "sender": {
        "id": "user_123",
        "nickname": "张三",
        "avatar": "https://cdn.example.com/avatars/123.jpg"
      },
      "type": "text",
      "content": "Hello World",
      "status": "read",
      "timestamp": "2024-01-01T12:00:00Z",
      "readBy": [
        {
          "userId": "user_456",
          "readAt": "2024-01-01T12:01:00Z"
        }
      ]
    }
  ],
  "meta": {
    "hasMore": true,
    "nextCursor": "2024-01-01T11:00:00Z"
  }
}
```

### 5.3 编辑消息

```http
PATCH /v1/{org_name}/{app_name}/messages/{message_id}
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "content": "编辑后的消息内容"
}
```

### 5.4 删除消息

```http
DELETE /v1/{org_name}/{app_name}/messages/{message_id}
Authorization: Bearer {access_token}
```

### 5.5 标记消息已读

```http
PATCH /v1/{org_name}/{app_name}/messages/{message_id}/read
Authorization: Bearer {access_token}
```

### 5.6 批量标记已读

```http
PATCH /v1/{org_name}/{app_name}/messages/read
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "roomId": "room_123",
  "lastReadMessageId": "msg_789"
}
```

### 5.7 搜索消息

```http
GET /v1/{org_name}/{app_name}/messages/search?q={keyword}&roomId={room_id}&type={message_type}&page=1&limit=20
Authorization: Bearer {access_token}
```

### 5.8 消息反应 (表情回应)

#### 添加反应

```http
POST /v1/{org_name}/{app_name}/messages/{message_id}/reactions
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "emoji": "👍",
  "unified": "1f44d"
}
```

#### 移除反应

```http
DELETE /v1/{org_name}/{app_name}/messages/{message_id}/reactions/{emoji}
Authorization: Bearer {access_token}
```

## 6. 推送通知模块 (Push)

### 6.1 注册设备令牌

```http
POST /v1/{org_name}/{app_name}/push/devices
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "deviceToken": "device_token_123",
  "platform": "iOS|Android|Web",
  "deviceId": "device_12345",
  "appVersion": "1.0.0"
}
```

### 6.2 更新推送设置

```http
PATCH /v1/{org_name}/{app_name}/push/settings
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "enabled": true,
  "sound": true,
  "vibration": true,
  "badge": true,
  "messagePreview": true,
  "quietHours": {
    "enabled": true,
    "start": "22:00",
    "end": "07:00"
  },
  "roomSettings": {
    "room_123": {
      "enabled": false
    }
  }
}
```

### 6.3 获取推送设置

```http
GET /v1/{org_name}/{app_name}/push/settings
Authorization: Bearer {access_token}
```

## 7. 文件管理

### 7.1 上传文件

```http
POST /v1/{org_name}/{app_name}/files
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**表单数据**：
- `file`: 文件内容
- `type`: 文件类型 (avatar|message|group)
- `metadata`: 额外元数据

**响应**：
```json
{
  "data": {
    "id": "file_123",
    "filename": "image.jpg",
    "size": 1024000,
    "mimeType": "image/jpeg",
    "url": "https://cdn.example.com/files/file_123.jpg",
    "thumbnailUrl": "https://cdn.example.com/files/file_123_thumb.jpg",
    "expiresAt": "2024-02-01T12:00:00Z"
  }
}
```

### 7.2 下载文件

```http
GET /v1/{org_name}/{app_name}/files/{file_id}
Authorization: Bearer {access_token}
```

## 8. 统计和分析

### 8.1 获取用户统计

```http
GET /v1/{org_name}/{app_name}/stats/users
Authorization: Bearer {access_token}
```

**响应**：
```json
{
  "data": {
    "totalUsers": 10000,
    "activeUsers": {
      "today": 1500,
      "thisWeek": 3000,
      "thisMonth": 5000
    },
    "newUsers": {
      "today": 10,
      "thisWeek": 50,
      "thisMonth": 200
    }
  }
}
```

### 8.2 获取消息统计

```http
GET /v1/{org_name}/{app_name}/stats/messages
Authorization: Bearer {access_token}
```

## 错误处理

### 错误响应格式

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
      "reason": "Invalid email format"
    },
    "timestamp": "2024-01-01T12:00:00Z",
    "requestId": "req-123456789"
  }
}
```

### 常见错误码

| 错误码 | 类型 | 说明 |
|--------|------|------|
| 40001 | VALIDATION_ERROR | 参数验证失败 |
| 40101 | AUTHENTICATION_REQUIRED | 需要认证 |
| 40102 | INVALID_TOKEN | 无效令牌 |
| 40103 | TOKEN_EXPIRED | 令牌过期 |
| 40301 | PERMISSION_DENIED | 权限不足 |
| 40401 | RESOURCE_NOT_FOUND | 资源不存在 |
| 40901 | RESOURCE_CONFLICT | 资源冲突 |
| 42001 | USER_NOT_FOUND | 用户不存在 |
| 42002 | ROOM_NOT_FOUND | 聊天室不存在 |
| 42003 | MESSAGE_NOT_FOUND | 消息不存在 |
| 44001 | RATE_LIMIT_EXCEEDED | 请求频率超限 |
| 50001 | INTERNAL_ERROR | 服务器内部错误 |

## 实时通信

### WebSocket 连接

```
wss://ws.example.com/v1/{org_name}/{app_name}/ws?token={access_token}
```

### 实时事件

**新消息事件**：
```json
{
  "type": "message.new",
  "data": {
    "message": {
      "id": "msg_789",
      "roomId": "room_123",
      "senderId": "user_456",
      "content": "Hello",
      "timestamp": "2024-01-01T12:00:00Z"
    }
  }
}
```

**用户状态变化**：
```json
{
  "type": "user.status_changed",
  "data": {
    "userId": "user_123",
    "status": "online",
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

## 批量操作

### 批量发送消息

```http
POST /v1/{org_name}/{app_name}/messages/batch
Authorization: Bearer {access_token}
```

**请求体**：
```json
{
  "messages": [
    {
      "roomId": "room_123",
      "content": "消息1",
      "type": "text"
    },
    {
      "roomId": "room_456", 
      "content": "消息2",
      "type": "text"
    }
  ]
}
```

## 安全说明

1. **HTTPS**: 所有 API 调用必须使用 HTTPS
2. **认证**: 除注册和登录外，所有接口都需要 Bearer Token 认证
3. **权限控制**: 用户只能访问自己有权限的资源
4. **速率限制**: 实施适当的 API 调用频率限制
5. **数据脱敏**: 敏感信息在响应中适当脱敏

## 性能优化

1. **分页**: 列表接口支持分页，默认限制 20 条记录
2. **字段选择**: 支持 `fields` 参数选择返回字段
3. **缓存**: 适当使用缓存头进行 HTTP 缓存
4. **压缩**: 支持 gzip 压缩

## 版本控制

- 当前版本：v1
- 版本策略：语义化版本控制
- 向后兼容：维护最新版本和前一个版本 