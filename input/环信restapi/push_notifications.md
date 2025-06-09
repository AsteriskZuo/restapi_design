# 环信 离线推送 REST API

## API 概览

基于 [环信即时通讯 REST API 概览](https://doc.easemob.com/document/server-side/overview.html) 的离线推送相关接口。

### 推送通知管理

#### 发送推送通知
```http
POST https://{host}/{org_name}/{app_name}/notification
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "targets": ["user1", "user2"],
  "notification": {
    "alert": "您有新消息",
    "badge": 1,
    "sound": "default"
  },
  "type": "users"
}
```

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/notification",
  "uri": "https://XXXX/XXXX/testapp/notification",
  "entities": [],
  "data": {
    "success": ["user1", "user2"],
    "failure": []
  },
  "timestamp": 1542795987470,
  "duration": 15,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 离线推送设置

#### 获取应用推送证书
```http
GET https://{host}/{org_name}/{app_name}/push_certificate
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/push_certificate",
  "uri": "https://XXXX/XXXX/testapp/push_certificate",
  "entities": [],
  "data": {
    "apns": {
      "certificate": "XXXX",
      "password": "XXXX",
      "environment": "development"
    },
    "gcm": {
      "api_key": "XXXX"
    }
  },
  "timestamp": 1542795987470,
  "duration": 2,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 上传推送证书
```http
POST https://{host}/{org_name}/{app_name}/push_certificate
Authorization: Bearer YourAppToken
Content-Type: multipart/form-data
```

请求体（multipart/form-data）：
- `certificate`: 证书文件
- `password`: 证书密码
- `environment`: 环境（development/production）
- `platform`: 平台（ios/android）

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/push_certificate",
  "uri": "https://XXXX/XXXX/testapp/push_certificate",
  "entities": [],
  "data": {
    "result": "success",
    "platform": "ios",
    "environment": "development"
  },
  "timestamp": 1542795987470,
  "duration": 25,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 推送设备令牌管理

#### 绑定设备令牌
```http
PUT https://{host}/{org_name}/{app_name}/users/{username}/notification/device_token
Authorization: Bearer YourAppToken
Content-Type: application/json
```

请求体：
```json
{
  "device_token": "device_token_string",
  "device_type": "ios",
  "notification_channel": "apns"
}
```

返回值：
```json
{
  "action": "put",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/notification/device_token",
  "entities": [],
  "data": {
    "result": "success"
  },
  "timestamp": 1542795987470,
  "duration": 8,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

#### 解绑设备令牌
```http
DELETE https://{host}/{org_name}/{app_name}/users/{username}/notification/device_token/{device_token}
Authorization: Bearer YourAppToken
```

返回值：
```json
{
  "action": "delete",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/users",
  "uri": "https://XXXX/XXXX/testapp/users/user1/notification/device_token/device_token_string",
  "entities": [],
  "data": {
    "result": "success"
  },
  "timestamp": 1542795987470,
  "duration": 5,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 推送统计

#### 获取推送统计
```http
GET https://{host}/{org_name}/{app_name}/notification/statistics
Authorization: Bearer YourAppToken
```

查询参数：
- `start_time`: 开始时间戳
- `end_time`: 结束时间戳

返回值：
```json
{
  "action": "get",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/notification",
  "uri": "https://XXXX/XXXX/testapp/notification/statistics",
  "entities": [],
  "data": {
    "total_sent": 1000,
    "total_success": 980,
    "total_failure": 20,
    "ios_sent": 600,
    "android_sent": 400
  },
  "timestamp": 1542795987470,
  "duration": 3,
  "organization": "XXXX",
  "applicationName": "testapp"
}
``` 