# 环信 好友管理 REST API

## API 概览

基于 [环信即时通讯 REST API 概览](https://doc.easemob.com/document/server-side/overview.html) 的好友管理相关接口。

### 好友关系管理

#### 获取用户好友列表
```http
GET https://{host}/{org_name}/{app_name}/users/{owner_username}/contacts/users
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/contacts/users",
  "entities": [],
  "data": ["user2", "user3", "user4"],
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 3
}
```

#### 添加好友
```http
POST https://{host}/{org_name}/{app_name}/users/{owner_username}/contacts/users/{friend_username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/contacts/users/user2",
  "entities": [],
  "data": {
    "result": true
  },
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 删除好友
```http
DELETE https://{host}/{org_name}/{app_name}/users/{owner_username}/contacts/users/{friend_username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/contacts/users/user2",
  "entities": [],
  "data": {
    "result": true
  },
  "timestamp": 1542795987470,
  "duration": 5,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 黑名单管理

#### 获取黑名单列表
```http
GET https://{host}/{org_name}/{app_name}/users/{owner_username}/blocks/users
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/blocks/users",
  "entities": [],
  "data": ["user5", "user6"],
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp",
  "count": 2
}
```

#### 添加用户到黑名单
```http
POST https://{host}/{org_name}/{app_name}/users/{owner_username}/blocks/users
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "usernames": ["user5", "user6"]
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/blocks/users",
  "entities": [],
  "data": {
    "user5": "success",
    "user6": "success"
  },
  "timestamp": 1542795987470,
  "duration": 12,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 从黑名单移除用户
```http
DELETE https://{host}/{org_name}/{app_name}/users/{owner_username}/blocks/users/{blocked_username}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/blocks/users/user5",
  "entities": [],
  "data": {
    "result": true
  },
  "timestamp": 1542795987470,
  "duration": 5,
  "organization": "XXXX",
  "applicationName": "testapp"
}
``` 