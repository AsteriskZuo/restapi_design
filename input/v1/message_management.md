# 环信 消息管理 REST API

## API 概览

基于 [环信即时通讯 REST API 概览](https://doc.easemob.com/document/server-side/overview.html) 的消息管理相关接口。

### 发送消息

#### 发送文本消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1", "user2"],
  "msg": {
    "type": "txt",
    "msg": "Hello World"
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success",
    "user2": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 发送图片消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1"],
  "msg": {
    "type": "img",
    "url": "http://XXXX/XXXX/image.jpg",
    "filename": "image.jpg",
    "size": {
      "width": 480,
      "height": 720
    }
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 发送语音消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1"],
  "msg": {
    "type": "audio",
    "url": "http://XXXX/XXXX/audio.mp3",
    "filename": "audio.mp3",
    "length": 10,
    "secret": "VfEpSmSvEeS7yU8dwa9uEAAAAAAAAA"
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 发送视频消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1"],
  "msg": {
    "type": "video",
    "url": "http://XXXX/XXXX/video.mp4",
    "filename": "video.mp4",
    "thumb": "http://XXXX/XXXX/thumb.jpg",
    "length": 10,
    "file_length": 58103
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 发送文件消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1"],
  "msg": {
    "type": "file",
    "url": "http://XXXX/XXXX/file.pdf",
    "filename": "document.pdf",
    "secret": "VfEpSmSvEeS7yU8dwa9uEAAAAAAAAA"
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 发送位置消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1"],
  "msg": {
    "type": "loc",
    "lat": "39.966",
    "lng": "116.322",
    "addr": "中国北京市海淀区中关村"
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 发送透传消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1"],
  "msg": {
    "type": "cmd",
    "action": "action1"
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 发送自定义消息
```http
POST https://{host}/{org_name}/{app_name}/messages
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "target_type": "users",
  "target": ["user1"],
  "msg": {
    "type": "custom",
    "customEvent": "gift",
    "customExts": {
      "gift_id": "001",
      "gift_name": "flower"
    }
  },
  "from": "admin"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages",
  "entities": {},
  "data": {
    "user1": "success"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 获取消息历史

#### 获取服务器历史消息
```http
GET https://{host}/{org_name}/{app_name}/chatmessages?ql=select+*+where+timestamp>1542795187256+and+timestamp<1542795487256
Authorization: Bearer YourAppToken
```

查询参数：
- `ql`: 查询语言，指定查询条件
- `limit`: 返回的消息数量，默认10，最大1000
- `cursor`: 分页游标

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatmessages",
  "uri": "https://XXXX/XXXX/testapp/chatmessages",
  "entities": [
    {
      "uuid": "5b90f240-d28e-11e8-a73c-9bf650a6ce3c",
      "type": "chatmessage",
      "created": 1542795987456,
      "modified": 1542795987456,
      "from": "user1",
      "to": "user2",
      "chat_type": "chat",
      "payload": {
        "msg": "Hello World"
      }
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

#### 导出历史消息文件
```http
GET https://{host}/{org_name}/{app_name}/chatmessages/export
Authorization: Bearer YourAppToken
```

查询参数：
- `time_range`: 时间范围，格式：2018-06-01_2018-06-30
- `file_type`: 文件类型，csv 或 json

返回值：
```json
{
  "action": "export",
  "timestamp": 1542795987470,
  "duration": 10,
  "organization": "XXXX",
  "applicationName": "testapp",
  "data": {
    "file_url": "http://XXXX/XXXX/export_file.csv",
    "expire_time": 1542882387470
  }
}
```

### 消息撤回

#### 撤回消息
```http
POST https://{host}/{org_name}/{app_name}/messages/recall
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "msg_id": "995112912316343512",
  "to": "user2",
  "from": "user1",
  "chat_type": "chat"
}
```

返回值：
```json
{
  "action": "recall",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/messages",
  "uri": "https://XXXX/XXXX/testapp/messages/recall",
  "entities": {},
  "data": {
    "result": true,
    "msg_id": "995112912316343512"
  },
  "timestamp": 1542795987470,
  "duration": 18,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 单向删除消息

#### 单向删除会话消息
```http
DELETE https://{host}/{org_name}/{app_name}/users/{username}/user_channel
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "channel": "user2",
  "type": "chat",
  "delete_roam": true
}
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/user_channel",
  "entities": {},
  "data": {
    "result": true
  },
  "timestamp": 1542795987470,
  "duration": 15,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 消息状态管理

#### 设置消息已读
```http
POST https://{host}/{org_name}/{app_name}/users/{username}/offline_msg_status/{msg_id}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "type": "read"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/offline_msg_status/995112912316343512",
  "entities": {},
  "data": {
    "result": "ok"
  },
  "timestamp": 1542795987470,
  "duration": 3,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 获取未读消息数量
```http
GET https://{host}/{org_name}/{app_name}/users/{username}/offline_msg_count
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/offline_msg_count",
  "entities": {},
  "data": {
    "user1": 5
  },
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 消息投递

#### 获取消息送达状态
```http
GET https://{host}/{org_name}/{app_name}/users/{username}/offline_msg_status/{msg_id}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/offline_msg_status/995112912316343512",
  "entities": {},
  "data": {
    "delivered": true,
    "timestamp": 1542795987456
  },
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 消息管理扩展

#### 查询指定时间段的消息数量
```http
GET https://{host}/{org_name}/{app_name}/chatmessages?ql=select+count(*)+where+timestamp>1542795187256+and+timestamp<1542795487256
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatmessages",
  "uri": "https://XXXX/XXXX/testapp/chatmessages",
  "entities": [],
  "aggregates": {
    "count": 1000
  },
  "timestamp": 1542795987470,
  "duration": 10,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 修改消息
```http
PUT https://{host}/{org_name}/{app_name}/chatmessages/{msg_id}
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "msg": {
    "type": "txt",
    "msg": "Modified message content"
  }
}
```

返回值：
```json
{
  "action": "put",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatmessages",
  "uri": "https://XXXX/XXXX/testapp/chatmessages/995112912316343512",
  "entities": [
    {
      "uuid": "995112912316343512",
      "type": "chatmessage",
      "created": 1542795987456,
      "modified": 1542796087456,
      "from": "user1",
      "to": "user2",
      "chat_type": "chat",
      "payload": {
        "msg": "Modified message content"
      }
    }
  ],
  "timestamp": 1542796087470,
  "duration": 15,
  "organization": "XXXX",
  "applicationName": "testapp"
}
``` 