# 高级规则

## 规则 1: 字段选择

支持通过查询参数选择返回字段：

```
GET /api/v1/users?fields=id,name,email
```

```json
{
  "data": [
    {
      "id": 123,
      "name": "张三",
      "email": "zhangsan@example.com"
    }
  ]
}
```

## 规则 2: 数据压缩

```http
Accept-Encoding: gzip, deflate
Content-Encoding: gzip
```

## 规则 3: 缓存控制

### 基础缓存策略

任何接口都应有明确的缓存策略。默认不缓存。

缓存策略枚举值：

- `no-cache`: 不缓存
- `time-based`: 基于时间的缓存（设置过期时间）
- `version-based`: 基于版本的缓存（通过 ETag 控制）
- `count-based`: 基于次数的缓存（设置最大缓存次数）

### 缓存响应头

```http
Cache-Control: public, max-age=3600
ETag: "abc123"
Last-Modified: Tue, 01 Jan 2024 12:00:00 GMT
```

### 条件请求

```http
# 客户端请求
If-None-Match: "abc123"
If-Modified-Since: Tue, 01 Jan 2024 12:00:00 GMT

# 服务器响应（资源未变化）
HTTP/1.1 304 Not Modified
```

## 规则 4: 接口调用频率限制

### 频率限制策略

任何接口都应有频率限制。默认不限制。

- 限制值为整数，0 表示不限制
- 通常以"每秒/每分钟/每小时"为单位
- 超过限制返回 429 状态码

### 频率限制响应头

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
Retry-After: 60
```

### 频率限制错误响应

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "请求频率超过限制",
    "details": {
      "limit": 100,
      "remaining": 0,
      "resetTime": "2024-01-01T13:00:00Z",
      "retryAfter": 3600
    }
  }
}
```

## 规则 5: 响应头规范

### 通用响应头

```http
Content-Type: application/json; charset=utf-8
X-Request-ID: req-123456789
X-Response-Time: 123ms
X-API-Version: v1
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
```

### 分页响应头

```http
X-Pagination-Page: 1
X-Pagination-Limit: 10
X-Pagination-Total: 50
X-Pagination-Total-Pages: 5
Link: <https://api.example.com/users?page=2>; rel="next",
      <https://api.example.com/users?page=5>; rel="last"
```

### 错误相关响应头

```http
# 认证错误
WWW-Authenticate: Bearer realm="api"

# 频率限制
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640995200
Retry-After: 3600

# 服务不可用
Retry-After: 60
```

## 规则 6: 国际化支持

### 多语言错误消息

```json
{
  "error": {
    "code": "VALIDATION_REQUIRED_FIELD",
    "message": "Required field is missing",
    "localizedMessage": {
      "zh-CN": "缺少必填字段",
      "en-US": "Required field is missing"
    },
    "details": {
      "field": "email"
    }
  }
}
```

### 请求头

```http
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Content-Language: zh-CN
```

## 规则 7: 详细状态码

### 1xx 信息性状态码

| 状态码 | 含义                | 使用场景             |
| ------ | ------------------- | -------------------- |
| 100    | Continue            | 客户端应继续发送请求 |
| 101    | Switching Protocols | 协议切换             |

### 3xx 重定向状态码

| 状态码 | 含义               | 使用场景             |
| ------ | ------------------ | -------------------- |
| 301    | Moved Permanently  | 资源永久移动         |
| 302    | Found              | 资源临时移动         |
| 304    | Not Modified       | 缓存有效             |
| 307    | Temporary Redirect | 临时重定向，保持方法 |
| 308    | Permanent Redirect | 永久重定向，保持方法 |

### 详细 4xx 状态码

| 状态码 | 含义               | 使用场景           |
| ------ | ------------------ | ------------------ |
| 405    | Method Not Allowed | HTTP 方法不支持    |
| 406    | Not Acceptable     | 不可接受的内容类型 |
| 408    | Request Timeout    | 请求超时           |
| 410    | Gone               | 资源已永久删除     |
| 413    | Payload Too Large  | 请求体过大         |
| 415    | Unsupported Media  | 不支持的媒体类型   |
| 429    | Too Many Requests  | 请求过于频繁       |

### 详细 5xx 状态码

| 状态码 | 含义            | 使用场景   |
| ------ | --------------- | ---------- |
| 501    | Not Implemented | 功能未实现 |
| 502    | Bad Gateway     | 网关错误   |
| 504    | Gateway Timeout | 网关超时   |

## 规则 8: 异步操作

### 异步任务创建

```
POST /api/v1/data/export
{
  "format": "csv",
  "filters": {...}
}

# 响应
HTTP/1.1 202 Accepted
{
  "data": {
    "taskId": "task-123456789",
    "status": "pending",
    "estimatedTime": 300
  }
}
```

### 任务状态查询

```
GET /api/v1/tasks/task-123456789

# 响应
{
  "data": {
    "id": "task-123456789",
    "status": "processing",
    "progress": 45,
    "result": null
  }
}
```

### 完成通知

```
# Webhook 通知
POST /your-webhook-url
{
  "taskId": "task-123456789",
  "status": "completed",
  "result": {
    "downloadUrl": "https://api.example.com/files/export-123.csv"
  }
}
```

## 规则 9: 搜索和过滤

### 基础搜索

```
GET /api/v1/users?q=zhang&fields=id,name,email
```

### 高级过滤

```
GET /api/v1/users?status=active&age[gte]=18&age[lt]=65&city=beijing
```

### 复杂查询

```
POST /api/v1/users/search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"name": "zhang"}},
        {"range": {"age": {"gte": 18, "lt": 65}}}
      ],
      "filter": [
        {"term": {"status": "active"}}
      ]
    }
  },
  "sort": [
    {"createdAt": {"order": "desc"}}
  ],
  "page": 1,
  "limit": 10
}
```

## 规则 10: 文件操作

### 文件上传

```
POST /api/v1/files
Content-Type: multipart/form-data

# 响应
{
  "data": {
    "id": "file-123456789",
    "filename": "document.pdf",
    "size": 1024000,
    "mimeType": "application/pdf",
    "url": "https://cdn.example.com/files/file-123456789.pdf"
  }
}
```

### 文件下载

```
GET /api/v1/files/file-123456789/download

# 响应头
Content-Disposition: attachment; filename="document.pdf"
Content-Type: application/pdf
Content-Length: 1024000
```

### 分片上传

```
# 1. 初始化分片上传
POST /api/v1/files/multipart
{
  "filename": "large-file.zip",
  "size": 104857600,
  "chunkSize": 1048576
}

# 2. 上传分片
PUT /api/v1/files/upload-123456789/chunks/1
Content-Type: application/octet-stream

# 3. 完成上传
POST /api/v1/files/upload-123456789/complete
{
  "chunks": [1, 2, 3, ..., 100]
}
```
