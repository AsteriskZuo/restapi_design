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

## 规则 3: URL 长度和嵌套限制

### URL 长度规则

- 总长度不超过 2048 字符
- 路径段不超过 255 字符
- 查询字符串不超过 1024 字符

### 嵌套深度限制

```
✅ 推荐（2-3层）：
/api/v1/users/123/posts
/api/v1/users/123/posts/456/comments

❌ 过深（避免超过4层）：
/api/v1/companies/123/departments/456/teams/789/members/101/skills
```

### 替代方案

对于深层嵌套，使用查询参数或独立端点：

```
# 替代深层嵌套
GET /api/v1/members?companyId=123&departmentId=456&teamId=789

# 或者使用独立端点
GET /api/v1/team-members/789
```

## 规则 4: 批量操作

### 批量创建

```
POST /api/v1/users/batch
[
  {
    "name": "张三",
    "email": "zhangsan@example.com"
  },
  {
    "name": "李四",
    "email": "lisi@example.com"
  }
]
```

### 批量更新

```
PATCH /api/v1/users/batch
[
  {
    "id": 123,
    "name": "张三新"
  },
  {
    "id": 124,
    "status": "inactive"
  }
]
```

### 批量删除

```
DELETE /api/v1/users/batch
{
  "ids": [123, 124, 125]
}
```

### 批量操作响应

```json
{
  "data": {
    "success": [
      {
        "id": 123,
        "status": "updated"
      }
    ],
    "failed": [
      {
        "id": 124,
        "error": "用户不存在"
      }
    ]
  },
  "meta": {
    "totalCount": 2,
    "successCount": 1,
    "failedCount": 1
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
```

### 分页响应头

```http
X-Pagination-Page: 1
X-Pagination-Limit: 10
X-Pagination-Total: 50
X-Pagination-Total-Pages: 5
Link: <https://api.example.com/users?page=2>; rel="next"
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

## 规则 7: 异步操作

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

## 规则 8: 搜索和过滤

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

## 规则 9: 文件操作

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

## 规则 10: API 安全规范

### 敏感信息传递

- 敏感数据必须通过请求体传递，不得出现在 URL 中
- 强制使用 HTTPS

### 响应数据处理

- 密码字段永不返回
- 敏感信息适当脱敏

### 认证令牌

- 使用 Authorization 头传递
- 避免在查询参数中传递 token

[简单的代码示例]

# 其他规则

其他规则由于复杂性放在单独章节进行说明。

[缓存策略](./cache_advanced.md)
[速率限制](./rate_limiting.md)
