# 环信 用户管理 REST API

## API 概览

基于 [环信即时通讯 REST API 概览](https://doc.easemob.com/document/server-side/overview.html) 的用户管理相关接口。

### 用户基本操作

#### 注册用户
```http
POST https://{host}/{org_name}/{app_name}/users
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "username": "user1",
  "password": "123456",
  "nickname": "testuser"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": true,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542795196515,
  "duration": 0,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量注册用户
```http
POST https://{host}/{org_name}/{app_name}/users
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
[
  {
    "username": "user1",
    "password": "123456",
    "nickname": "testuser1"
  },
  {
    "username": "user2", 
    "password": "123456",
    "nickname": "testuser2"
  }
]
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": true,
      "nickname": "testuser1"
    },
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f2",
      "type": "user",
      "created": 1542795196505,
      "modified": 1542795196505,
      "username": "user2",
      "activated": true,
      "nickname": "testuser2"
    }
  ],
  "timestamp": 1542795196515,
  "duration": 0,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 获取用户信息
```http
GET https://{host}/{org_name}/{app_name}/users/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": true,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542795196515,
  "duration": 0,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 1
}
```

#### 获取多个用户信息
```http
GET https://{host}/{org_name}/{app_name}/users?limit=20&cursor=xxx
Authorization: Bearer YourAppToken
```

查询参数：
- `limit`: 返回的用户数量，默认值为 10，取值范围为 [1,100]
- `cursor`: 数据查询的起始位置

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": true,
      "nickname": "testuser1"
    },
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f2",
      "type": "user",
      "created": 1542795196505,
      "modified": 1542795196505,
      "username": "user2",
      "activated": true,
      "nickname": "testuser2"
    }
  ],
  "timestamp": 1542795196515,
  "duration": 0,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 2,
  "cursor": "LTgzNDgyODQ4"
}
```

#### 修改用户密码
```http
PUT https://{host}/{org_name}/{app_name}/users/{username}/password
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "newpassword": "newpassword"
}
```

返回值：
```json
{
  "action": "set user password",
  "timestamp": 1542795987470,
  "duration": 8
}
```

#### 修改用户昵称
```http
PUT https://{host}/{org_name}/{app_name}/users/{username}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "nickname": "新昵称"
}
```

返回值：
```json
{
  "action": "put",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795987473,
      "username": "user1",
      "activated": true,
      "nickname": "新昵称"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 删除用户
```http
DELETE https://{host}/{org_name}/{app_name}/users/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": true,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542796177147,
  "duration": 32,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量删除用户
```http
DELETE https://{host}/{org_name}/{app_name}/users
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "usernames": ["user1", "user2", "user3"]
}
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": true
    },
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f2", 
      "type": "user",
      "created": 1542795196505,
      "modified": 1542795196505,
      "username": "user2",
      "activated": true
    }
  ],
  "timestamp": 1542796177147,
  "duration": 45,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 用户Token管理

#### 获取用户Token
```http
POST https://{host}/{org_name}/{app_name}/token
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "grant_type": "password",
  "username": "user1", 
  "password": "123456"
}
```

返回值：
```json
{
  "access_token": "YWMtxc6K0L1aEeiTkNP4grL7BwAAAAAAAAAAAAFWChJZGHvqCxMSzqlYQOzEuBnWjrOOgOOjAgMAAAFnKdc-ZgBPGgBMDAKKANxUW7Cco3C1I9nwGJIFHMSaU3Cg2O2Q",
  "expires_in": 604800,
  "user": {
    "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
    "type": "user",
    "created": 1542795196504,
    "modified": 1542795196504,
    "username": "user1",
    "activated": true,
    "nickname": "testuser"
  }
}
```

#### 强制用户下线
```http
DELETE https://{host}/{org_name}/{app_name}/users/{username}/disconnect
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "Disconnect user",
  "result": true,
  "uri": "https://XXXX/XXXX/testapp",
  "timestamp": 1542796177147,
  "duration": 45,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 用户在线状态

#### 查看用户在线状态
```http
GET https://{host}/{org_name}/{app_name}/users/{username}/status
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "uri": "https://XXXX/XXXX/testapp",
  "timestamp": 1542795987470,
  "duration": 1,
  "organization": "XXXX",
  "applicationName": "testapp",
  "data": {
    "user1": "online"
  }
}
```

#### 批量查看用户在线状态
```http
POST https://{host}/{org_name}/{app_name}/users/batch/status
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "usernames": ["user1", "user2", "user3"]
}
```

返回值：
```json
{
  "action": "get batch user status",
  "data": [
    {
      "result": {
        "user1": "online",
        "user2": "offline",
        "user3": "online"
      }
    }
  ],
  "timestamp": 1542795987470,
  "duration": 2
}
```

### 用户属性管理

#### 设置用户属性
```http
PUT https://{host}/{org_name}/{app_name}/metadata/user/{username}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "nickname": "昵称",
  "avatar": "头像URL",
  "phone": "手机号",
  "gender": 1,
  "sign": "个性签名",
  "email": "邮箱",
  "birth": "生日"
}
```

返回值：
```json
{
  "timestamp": 1542795987470,
  "duration": 4,
  "organization": "XXXX",
  "applicationName": "testapp",
  "action": "put",
  "data": {
    "nickname": "昵称",
    "avatar": "头像URL",
    "phone": "手机号",
    "gender": 1,
    "sign": "个性签名",
    "email": "邮箱",
    "birth": "生日"
  }
}
```

#### 获取用户属性
```http
GET https://{host}/{org_name}/{app_name}/metadata/user/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "timestamp": 1542795987470,
  "duration": 1,
  "organization": "XXXX",
  "applicationName": "testapp",
  "action": "get",
  "data": {
    "nickname": "昵称",
    "avatar": "头像URL",
    "phone": "手机号",
    "gender": 1,
    "sign": "个性签名",
    "email": "邮箱",
    "birth": "生日"
  }
}
```

#### 获取指定用户属性
```http
POST https://{host}/{org_name}/{app_name}/metadata/user/get
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "targets": ["user1", "user2"],
  "properties": ["nickname", "avatar"]
}
```

返回值：
```json
{
  "timestamp": 1542795987470,
  "duration": 3,
  "organization": "XXXX",
  "applicationName": "testapp",
  "action": "get",
  "data": {
    "user1": {
      "nickname": "昵称1",
      "avatar": "头像URL1"
    },
    "user2": {
      "nickname": "昵称2", 
      "avatar": "头像URL2"
    }
  }
}
```

#### 删除用户属性
```http
DELETE https://{host}/{org_name}/{app_name}/metadata/user/{username}
Authorization: Bearer YourAppToken
```

查询参数：
- `properties`: 要删除的属性名，多个属性用逗号分隔

返回值：
```json
{
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp",
  "action": "delete",
  "data": {}
}
```

### 用户封禁管理

#### 封禁用户
```http
POST https://{host}/{org_name}/{app_name}/users/{username}/deactivate
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "Deactivate user",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": false,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 12
}
```

#### 解封用户
```http
POST https://{host}/{org_name}/{app_name}/users/{username}/activate
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "activate user",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795196504,
      "username": "user1",
      "activated": true,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 8
}
```

### 全局禁言

#### 设置全局禁言
```http
POST https://{host}/{org_name}/{app_name}/mutes
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "username": "user1",
  "chat": 3600,
  "groupchat": 3600,
  "chatroom": 3600
}
```

参数说明：
- `chat`: 单聊禁言时长，单位：秒，-1表示永久禁言
- `groupchat`: 群聊禁言时长，单位：秒，-1表示永久禁言  
- `chatroom`: 聊天室禁言时长，单位：秒，-1表示永久禁言

返回值：
```json
{
  "action": "mute user",
  "data": {
    "result": true,
    "expire": 1542795987470,
    "user": "user1"
  },
  "timestamp": 1542795987470,
  "duration": 6
}
```

#### 查询全局禁言
```http
GET https://{host}/{org_name}/{app_name}/mutes/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get user mute",
  "data": {
    "userid": "user1",
    "chat": 3600,
    "groupchat": 3600,
    "chatroom": 3600,
    "unmuteTime": 1542795987470
  },
  "timestamp": 1542795987470,
  "duration": 1
}
```

#### 解除全局禁言
```http
DELETE https://{host}/{org_name}/{app_name}/mutes/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "remove user mute",
  "data": {
    "result": true,
    "user": "user1"
  },
  "timestamp": 1542795987470,
  "duration": 3
}
```

### 推送设置

#### 设置推送昵称
```http
PUT https://{host}/{org_name}/{app_name}/users/{username}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "pushname": "推送昵称"
}
```

返回值：
```json
{
  "action": "put",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795987473,
      "username": "user1",
      "activated": true,
      "pushname": "推送昵称"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 设置推送免打扰
```http
PUT https://{host}/{org_name}/{app_name}/users/{username}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "notification_no_disturbing": true,
  "notification_no_disturbing_start": "22:00",
  "notification_no_disturbing_end": "8:00"
}
```

返回值：
```json
{
  "action": "put",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795987473,
      "username": "user1",
      "activated": true,
      "notification_no_disturbing": true,
      "notification_no_disturbing_start": "22:00",
      "notification_no_disturbing_end": "8:00"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 设置推送展示方式
```http
PUT https://{host}/{org_name}/{app_name}/users/{username}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "notification_display_style": 1
}
```

参数说明：
- `0`: 仅显示"您有一条新消息"
- `1`: 显示消息发送方的昵称和消息内容

返回值：
```json
{
  "action": "put",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1",
  "entities": [
    {
      "uuid": "b2aade90-e978-11e8-a974-f3368f82e4f1",
      "type": "user",
      "created": 1542795196504,
      "modified": 1542795987473,
      "username": "user1",
      "activated": true,
      "notification_display_style": 1
    }
  ],
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
} 