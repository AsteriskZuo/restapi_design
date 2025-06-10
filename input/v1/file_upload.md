# 环信 文件上传 REST API

## API 概览

基于 [环信即时通讯 REST API 概览](https://doc.easemob.com/document/server-side/overview.html) 的文件上传相关接口。

### 文件上传

#### 上传文件
```http
POST https://{host}/{org_name}/{app_name}/chatfiles
Authorization: Bearer YourAppToken
Content-Type: multipart/form-data
```

请求体（multipart/form-data）：
- `file`: 文件二进制数据
- `restrict-access`: 是否限制访问，true或false

返回值：
```json
{
  "action": "post",
  "application": "8be024f0-e978-11e8-b697-5d598d5f8402",
  "path": "/chatfiles",
  "uri": "https://XXXX/XXXX/testapp/chatfiles",
  "entities": [
    {
      "uuid": "5b90f240-d28e-11e8-a73c-9bf650a6ce3c",
      "type": "chatfile",
      "share-secret": "VfEpSmSvEeS7yU8dwa9uEAAAAAAAAA"
    }
  ],
  "timestamp": 1542795987470,
  "duration": 89,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

### 文件下载

#### 下载文件
```http
GET https://{host}/{org_name}/{app_name}/chatfiles/{file_uuid}
Authorization: Bearer YourAppToken
```

查询参数：
- `share-secret`: 文件访问密钥（当restrict-access为true时必须）
- `thumbnail`: 是否下载缩略图，true或false

#### 下载缩略图
```http
GET https://{host}/{org_name}/{app_name}/chatfiles/{file_uuid}?thumbnail=true
Authorization: Bearer YourAppToken
```

注意：此接口会直接返回文件的二进制数据

### 文件信息

#### 获取文件信息
```http
HEAD https://{host}/{org_name}/{app_name}/chatfiles/{file_uuid}
Authorization: Bearer YourAppToken
```

查询参数：
- `share-secret`: 文件访问密钥（当restrict-access为true时必须）

返回值（HTTP Headers）：
```
Content-Type: image/jpeg
Content-Length: 13987
Accept-Ranges: bytes
``` 