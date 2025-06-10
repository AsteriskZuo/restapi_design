# 环信 聊天室管理 REST API

## API 概览

基于 [环信即时通讯 REST API 概览](https://doc.easemob.com/document/server-side/overview.html) 的聊天室管理相关接口。

### 聊天室创建

#### 创建聊天室
```http
POST https://{host}/{org_name}/{app_name}/chatrooms
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "name": "testchatroom",
  "description": "聊天室描述",
  "maxusers": 300,
  "owner": "user1",
  "members": ["user2", "user3"],
  "custom": "自定义属性"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms",
  "entities": [],
  "data": {
    "id": "142391095541777"
  },
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 聊天室信息管理

#### 获取APP下的聊天室
```http
GET https://{host}/{org_name}/{app_name}/chatrooms?limit=20&cursor=xxx
Authorization: Bearer YourAppToken
```

查询参数：
- `limit`: 返回的聊天室数量，默认10，最大1000
- `cursor`: 分页游标

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms",
  "entities": [],
  "data": [
    {
      "id": "142391095541777",
      "name": "testchatroom",
      "description": "聊天室描述",
      "membersonly": false,
      "allowinvites": true,
      "maxusers": 300,
      "owner": "user1",
      "created": 1542795987456,
      "custom": "自定义属性",
      "affiliations_count": 3
    }
  ],
  "timestamp": 1542795987470,
  "duration": 5,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 1,
  "cursor": "LTU5MDEzODQ4MjU"
}
```

#### 获取聊天室详情
```http
GET https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777",
  "entities": [],
  "data": [
    {
      "id": "142391095541777",
      "name": "testchatroom",
      "description": "聊天室描述",
      "membersonly": false,
      "allowinvites": true,
      "maxusers": 300,
      "owner": "user1",
      "created": 1542795987456,
      "custom": "自定义属性",
      "affiliations_count": 3,
      "affiliations": [
        {
          "owner": "user1"
        },
        {
          "member": "user2"
        },
        {
          "member": "user3"
        }
      ]
    }
  ],
  "timestamp": 1542795987470,
  "duration": 3,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 修改聊天室信息
```http
PUT https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "name": "新聊天室名称",
  "description": "新的聊天室描述",
  "maxusers": 500,
  "custom": "新的自定义属性"
}
```

返回值：
```json
{
  "action": "put",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777",
  "entities": [],
  "data": {
    "description": true,
    "maxusers": true,
    "name": true
  },
  "timestamp": 1542795987470,
  "duration": 15,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 删除聊天室
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777",
  "entities": [],
  "data": {
    "success": true,
    "id": "142391095541777"
  },
  "timestamp": 1542795987470,
  "duration": 25,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 聊天室成员管理

#### 获取聊天室成员
```http
GET https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/users?pagenum=1&pagesize=10
Authorization: Bearer YourAppToken
```

查询参数：
- `pagenum`: 页码，从1开始
- `pagesize`: 每页大小，默认10，最大1000

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/users",
  "entities": [],
  "data": [
    {
      "owner": "user1"
    },
    {
      "member": "user2"
    },
    {
      "member": "user3"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 3
}
```

#### 添加聊天室成员 (单个)
```http
POST https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/users/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/users/user4",
  "entities": [],
  "data": {
    "result": true,
    "action": "add_member",
    "user": "user4",
    "id": "142391095541777"
  },
  "timestamp": 1542795987470,
  "duration": 12,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量添加聊天室成员
```http
POST https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/users
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "usernames": ["user4", "user5", "user6"]
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/users",
  "entities": [],
  "data": {
    "newmembers": [
      {
        "result": "success",
        "user": "user4"
      },
      {
        "result": "success", 
        "user": "user5"
      },
      {
        "result": "failed",
        "user": "user6",
        "reason": "user not found"
      }
    ],
    "action": "add_member",
    "id": "142391095541777"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 移除聊天室成员 (单个)
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/users/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/users/user4",
  "entities": [],
  "data": {
    "result": true,
    "action": "remove_member",
    "user": "user4",
    "id": "142391095541777"
  },
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量移除聊天室成员
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/users/{usernames}
Authorization: Bearer YourAppToken
```

路径参数：
- `usernames`: 用户名列表，用逗号分隔

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/users/user4,user5",
  "entities": [],
  "data": [
    {
      "result": "success",
      "action": "remove_member",
      "user": "user4",
      "id": "142391095541777"
    },
    {
      "result": "success",
      "action": "remove_member", 
      "user": "user5",
      "id": "142391095541777"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 15,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 聊天室管理员管理

#### 设置聊天室管理员
```http
POST https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/admin
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "newadmin": "user2"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/admin",
  "entities": [],
  "data": {
    "result": "success",
    "newadmin": "user2"
  },
  "timestamp": 1542795987470,
  "duration": 5,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 移除聊天室管理员
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/admin/{oldadmin}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/admin/user2",
  "entities": [],
  "data": {
    "result": "success",
    "oldadmin": "user2"
  },
  "timestamp": 1542795987470,
  "duration": 3,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 聊天室封禁管理

#### 获取聊天室黑名单
```http
GET https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/blocks/users
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/blocks/users",
  "entities": [],
  "data": ["user4", "user5"],
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 2
}
```

#### 添加用户至聊天室黑名单 (单个)
```http
POST https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/blocks/users/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/blocks/users/user4",
  "entities": [],
  "data": {
    "result": true,
    "action": "add_blocks",
    "user": "user4",
    "id": "142391095541777"
  },
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量添加用户至聊天室黑名单
```http
POST https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/blocks/users
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "usernames": ["user4", "user5"]
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/blocks/users",
  "entities": [],
  "data": [
    {
      "result": "success",
      "action": "add_blocks",
      "user": "user4",
      "id": "142391095541777"
    },
    {
      "result": "success",
      "action": "add_blocks",
      "user": "user5", 
      "id": "142391095541777"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 12,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 从聊天室黑名单移除用户 (单个)
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/blocks/users/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/blocks/users/user4",
  "entities": [],
  "data": {
    "result": true,
    "action": "remove_blocks",
    "user": "user4",
    "id": "142391095541777"
  },
  "timestamp": 1542795987470,
  "duration": 5,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量从聊天室黑名单移除用户
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/blocks/users/{usernames}
Authorization: Bearer YourAppToken
```

路径参数：
- `usernames`: 用户名列表，用逗号分隔

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/blocks/users/user4,user5",
  "entities": [],
  "data": [
    {
      "result": "success",
      "action": "remove_blocks",
      "user": "user4",
      "id": "142391095541777"
    },
    {
      "result": "success",
      "action": "remove_blocks",
      "user": "user5",
      "id": "142391095541777"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 聊天室禁言管理

#### 获取聊天室禁言列表
```http
GET https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/mute
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/mute",
  "entities": [],
  "data": [
    {
      "user": "user4",
      "expire": 1542799587470
    },
    {
      "user": "user5",
      "expire": 1542799587470
    }
  ],
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 禁言聊天室成员 (单个)
```http
POST https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/mute
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "mute_duration": 86400000,
  "username": "user4"
}
```

参数说明：
- `mute_duration`: 禁言时长，单位毫秒，-1表示永久禁言

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/mute",
  "entities": [],
  "data": {
    "result": true,
    "expire": 1542882387470,
    "user": "user4"
  },
  "timestamp": 1542795987470,
  "duration": 6,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量禁言聊天室成员
```http
POST https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/mute
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "mute_duration": 86400000,
  "usernames": ["user4", "user5"]
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/mute",
  "entities": [],
  "data": [
    {
      "result": "success",
      "expire": 1542882387470,
      "user": "user4"
    },
    {
      "result": "success",
      "expire": 1542882387470,
      "user": "user5"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 10,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 解除聊天室成员禁言 (单个)
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/mute/{username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/mute/user4",
  "entities": [],
  "data": {
    "result": true,
    "user": "user4"
  },
  "timestamp": 1542795987470,
  "duration": 3,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 批量解除聊天室成员禁言
```http
DELETE https://{host}/{org_name}/{app_name}/chatrooms/{chatroom_id}/mute/{usernames}
Authorization: Bearer YourAppToken
```

路径参数：
- `usernames`: 用户名列表，用逗号分隔

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatrooms",
  "uri": "https://XXXX/XXXX/testapp/chatrooms/142391095541777/mute/user4,user5",
  "entities": [],
  "data": [
    {
      "result": "success",
      "user": "user4"
    },
    {
      "result": "success",
      "user": "user5"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 5,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 用户聊天室相关

#### 获取用户加入的聊天室
```http
GET https://{host}/{org_name}/{app_name}/users/{username}/joined_chatrooms
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/joined_chatrooms",
  "entities": [],
  "data": [
    {
      "id": "142391095541777",
      "name": "testchatroom"
    },
    {
      "id": "142391095541778",
      "name": "testchatroom2"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 3,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 2
}
``` 