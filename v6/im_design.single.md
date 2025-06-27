# IM RESTFUL API 开发设计规范

本文档基于 `RFC9110`、`RFC9205` 等国际标准，结合 `Claude-4-Sonnet` 的最佳实践，深度分析 IM 业务场景特点，形成了这套较为完整的可落地的规范建议方案。

# 1. HTTP URL

URL 设计遵循 RESTFUL API 原则，确保资源定位的准确性和可读性。

## 1.1 URL 结构规则

**URL 结构 = 协议 + 主机 + 版本 + 组织名 + 应用名 + 资源路径 + 参数等**

**分层结构：**

- **协议**：http/https
- **主机**：API 服务域名或者 IP 地址
  - 示例：api.easemob.com
  - 开发示例：api.dev-{env}.easemob.com
- **版本**：API 版本号（v1/v2 等）
- **组织标识**：组织标识（租户标识）
  - 用于多租户隔离
  - 系统生成
- **应用名**：应用名称（应用标识）
  - 用于多应用隔离
  - 用户创建时填写，创建后无法修改
- **资源路径**：资源层级
  - 名称使用小写字母、数字和下划线组成
  - 名称使用名词，不要使用动词，通过 HTTP 方法来表示操作意图
- **参数**：过滤条件
  - 名称使用小写字母、数字和下划线组成
  - 通过 `?<query>` 来传递条件
  - 通过 `&` 来分隔多个条件

**URL 结构示例：**

```http
https://{host}/{version}/{org_id}/{app_name}/auth
https://{host}/{version}/{org_id}/{app_name}/groups/threads
```

**URL 版本示例：**

```http
# ✅ 正确:
https://api.easemob.com/v1/myorg/myapp/users // 1.0版本
https://api.easemob.com/v2/myorg/myapp/users // 2.0版本

# ❌ 错误:
https://api.easemob.com/v1.2/myorg/myapp/users // 不支持次要版本号
https://api.easemob.com/v1.2.3/myorg/myapp/users // 不支持补丁版本号
```

**URL 资源示例：**

```http
# ✅ 正确：使用名词 + HTTP 方法
GET /api/v1/users           # 获取用户列表
POST /api/v1/users          # 创建用户
PUT /api/v1/users/123       # 更新用户
DELETE /api/v1/users/123    # 删除用户

# ❌ 错误：使用动词
GET /api/v1/getUsers
POST /api/v1/createUser
PUT /api/v1/updateUser/123
DELETE /api/v1/deleteUser/123
```

**URL 参数示例：**

```http
# ✅ 正确
GET /api/v1/users?name=John&age=30

# ❌ 错误
GET /api/v1/users?name=John;age=30 // 分隔符不是 `&`
GET /api/v1/users!name=John&age=30 // 没有使用 `?`
```

## 1.2 公开和私有接口

公开接口 和 私有接口 通过资源进行区分。

**接口示例：**

```
# 私有接口
/api/internal/v1/users
# 公开接口
/api/v1/users
```

# 2. HTTP Method

**CRUD 操作映射**

- **增加(Create)**: POST
- **查询(Read)**: GET
- **修改(Update)**: PUT（完整更新）
- **删除(Delete)**: DELETE

**HTTP 方法说明**

| 方法   | 幂等性     | 缓存性     | 主要用途 | 说明                   |
| ------ | ---------- | ---------- | -------- | ---------------------- |
| GET    | 幂等       | 可缓存     | 获取资源 | 不应有副作用           |
| POST   | 通常非幂等 | 通常不缓存 | 创建资源 | 可通过幂等键设计为幂等 |
| PUT    | 幂等       | 条件缓存   | 完整更新 | 响应可在某些情况下缓存 |
| DELETE | 幂等       | 条件缓存   | 删除资源 | 重复删除不应报错       |

**特殊情况**

- url 最后以 `/get` 结束表示实际为 `GET` 使用。

**幂等性说明**:

- **幂等**：多次调用产生相同结果
- **非幂等**：多次调用可能产生不同结果
- **取决于实现**：根据具体操作语义确定

**缓存性说明**:

- **可缓存**：响应可以被缓存
- **不缓存**：响应不应被缓存
- **条件缓存**：在特定条件下可以缓存

**服务无状态**

服务不保存客户端会话状态，每个请求包含所有必要信息，提高可扩展性和可靠性。

```http
# ✅ 正确:
GET /api/v1/users/123/posts?page=2&limit=10
authorization: Bearer token123

# ❌ 错误:
GET /api/v1/getNextPage  # 依赖服务器端状态
```

# 3. HTTP Header

HTTP Header 用于传递关于请求或响应的元数据，包括身份验证、缓存策略、内容协商等。

## 3.1 常用 Header 分类

| 类别     | 主要作用         | 常用 Header                              |
| -------- | ---------------- | ---------------------------------------- |
| 身份认证 | 验证请求合法性   | Authorization, X-Token                   |
| 内容协商 | 确定响应内容格式 | Accept, Content-Type                     |
| 语言协商 | 确定响应语言     | Accept-Language, Content-Language        |
| 缓存控制 | 控制缓存行为     | Cache-Control, ETag, Last-Modified       |
| 资源压缩 | 优化传输效率     | Accept-Encoding, Content-Encoding        |
| 跨域资源 | 控制跨域访问     | Access-Control-Allow-Origin              |
| 请求跟踪 | 调试与监控       | X-Request-ID, X-Trace-ID                 |
| 速率限制 | 限流相关信息     | X-RateLimit-Limit, X-RateLimit-Remaining |

## 3.2 表示标头

- **内容类型**:
  - `Content-Type`: 如果使用请求体并且内容是文本, 则设置值为 `application/json; charset=utf-8`
  - `Content-Type`: 如果使用请求体并且内容是文件, 则设置值为 `multipart/form-data; boundary={unique_boundary_string}`
- **内容编码**:
  - `Content-Encoding`: 当请求体较大时，可使用 `gzip` 等编码方式压缩传输内容，适用于 JSON、XML 等文本格式（参考 RFC 9110 Section 8.4.1）
- **内容语言**:
  - `Content-Language`: 内容语言, 值为 `zh-CN` 或 `en-US` 等

## 3.3 请求头

**必要**

- **身份验证**: `Authorization` 用于身份验证，推荐采用 Token 验证

**可选**

- **文本请求**:
  - `Accept`: 期望响应格式, 值为 `application/json; charset=utf-8`
- **文件请求**:
  - `Accept`: 期望响应格式, 例如: `multipart/form-data; boundary={unique_boundary_string}`
- **语言协商**:
  - `Accept-Language`: 期望响应语言, 例如: `zh-CN,zh,en-US,en`
- **压缩请求**:
  - `Accept-Encoding`: 响应体压缩格式, 值为 `gzip`
- **缓存控制请求**:
  - `Cache-Control`: 缓存控制，值可以为 `no-cache` `no-store` 等
- **请求跟踪**:
  - `X-Request-ID`: 请求 ID, 值为 UUID

## 3.4 响应头

**可选**

- **文本响应**:
  - `Accept`: 期望响应格式, 值为 `application/json; charset=utf-8`
- **文件响应**:

  - `Accept`: 期望响应格式, 值为: `multipart/form-data; boundary={unique_boundary_string}`

- **压缩响应**:
  - `Accept-Encoding`: 响应体压缩格式, 值为 `gzip`
- **缓存控制响应**:
  - `Cache-Control`: 缓存控制，值可以为 `no-cache` `no-store` 等
- **资源压缩响应**:
  - `Accept-Encoding`: 响应体压缩格式, 值为 `gzip`
- **跨域资源响应**:
  - `Access-Control-Allow-Origin`: 允许跨域访问的域名
  - `Access-Control-Allow-Methods`: 允许跨域访问的方法
  - `Access-Control-Allow-Headers`: 允许跨域访问的请求头
- **请求跟踪**:
  - `X-Request-ID`: 请求 ID, 值为 UUID

## 3.5 示例

**请求示例：**:

```http
# ✅ 推荐
Content-Type: application/json; charset=utf-8
Accept: application/json; charset=utf-8
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Encoding: gzip                 # 请求体压缩
Accept-Encoding: gzip                  # 响应体压缩
Cache-Control: no-cache                # 缓存控制
```

**响应示例：**:

```http
# ✅ 推荐
Accept: application/json; charset=utf-8               # 期望响应格式
Content-Type: application/json; charset=utf-8         # 响应体格式
Content-Language: zh-CN                               # 响应语言
Content-Encoding: gzip                 # 压缩响应
Cache-Control: no-cache                # 缓存控制
```

# 4. HTTP Body

## 4.1 请求体

请求体是在 HTTP 请求中传输数据的主要载体，主要用于 POST、PUT 等方法。

**基本原则**

- **适用 HTTP 方法**：主要用于 POST、PUT 等方法，DELETE 可选，GET 不支持
- **内容类型声明**：必须设置`Content-Type`头部

**使用场景**

- **复杂查询**：当参数结构复杂或 url 长度过长时
- **批量操作**：需要一次性处理多个资源
- **大数据传输**：请求体过大时，推荐使用文件上传方式

**请求体格式：**

- data: 请求内容。支持对象、数组、空值等
- meta: 元数据，用于记录请求信息
  - timestamp: 请求时间戳
  - requestId: 请求唯一标识（可选）

```json
{
  "data": {
    "userId": "123",
    "username": "zhangsan"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "1704110400456abc123def78912"
  }
}
```

## 4.2 对象格式响应体

**响应结构 = 核心数据(data)/错误信息(error) + 元数据(meta)**

- **data 字段**: 包含实际的业务数据（成功响应必需）
- **error 字段**: 包含错误信息（失败响应必需）
- **meta 字段**: 包含响应元数据信息（推荐）
  - timestamp: 毫秒级 UTC 时间戳
  - requestId: 请求追踪 ID

**命名风格**

- json 请求体对象采用首字母小写的驼峰命名风格，例如：`user`、`userList`、`userListPage` 等。

**成功响应格式：**:

- data: 响应内容。支持数组、对象、空值等
- meta: 元数据
  - timestamp: 毫秒级 UTC 时间戳
  - requestId: 请求追踪 ID（可选）

```json
{
  "data": {
    "userId": "123",
    "username": "zhangsan"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "1704110400456abc123def78912"
  }
}
```

**失败响应格式：**:

- error: 错误信息
  - code: 错误码
  - message: 错误信息
- meta: 元数据
  - timestamp: 毫秒级 UTC 时间戳
  - requestId: 请求追踪 ID（可选）

```json
{
  "error": {
    "code": 4010101,
    "message": "用户名或密码错误"
  },
  "meta": {
    "timestamp": 1704110400000,
    "requestId": "1704110400789def456abc12345"
  }
}
```

# 5. 错误码

采用 7 位错误码结构，确保错误信息的准确性和可追踪性。

## 5.1 错误码构成规则

**7 位错误码 = HTTP 状态码(3 位) + 具体错误(4 位)**

**分层结构：**

- **第 1-3 位**：HTTP 状态码（404, 409, 500 等）
- **第 4-7 位**：具体错误编号（0001-0999 为通用错误码，1000+ 为自定义错误码）

**示例解析：**

- `4040001` = `404` + `0001` (状态码 + 通用错误：名称不合法)
- `4090401` = `409` + `0401` (状态码 + 通用错误：资源已存在)
- `5000701` = `500` + `0701` (状态码 + 通用错误：服务器内部错误)
- `4001001` = `400` + `1001` (状态码 + 自定义错误)

## 5.2 错误码分类定义

**常见 HTTP 状态码**

- **100 Continue**: 客户端应继续发送请求
- **101 Switching Protocols**: 协议切换

- **200 OK**: 请求成功
- **201 Created**: 资源创建成功
- **202 Accepted**: 请求已接受，异步处理
- **204 No Content**: 请求成功，无返回内容

- **301 Moved Permanently**: 资源永久移动
- **302 Found**: 资源临时移动
- **304 Not Modified**: 缓存有效
- **307 Temporary Redirect**: 临时重定向，保持方法
- **308 Permanent Redirect**: 永久重定向，保持方法

- **400 Bad Request**: 请求格式错误
- **401 Unauthorized**: 认证失败
- **403 Forbidden**: 权限不足
- **404 Not Found**: 资源不存在
- **405 Method Not Allowed**: HTTP 方法不支持
- **406 Not Acceptable**: 不可接受的内容类型
- **408 Request Timeout**: 请求超时
- **409 Conflict**: 资源冲突
- **410 Gone**: 资源已永久删除
- **413 Payload Too Large**: 请求体过大
- **415 Unsupported Media**: 不支持的媒体类型
- **422 Unprocessable Entity**: 语义错误
- **429 Too Many Requests**: 请求过于频繁

- **500 Internal Server Error**: 服务器内部错误
- **501 Not Implemented**: 功能未实现
- **502 Bad Gateway**: 网关错误
- **503 Service Unavailable**: 服务不可用
- **504 Gateway Timeout**: 网关超时

**通用具体错误码 (0001-0999)**

以下错误码可在不同业务场景中复用：

> 参数验证类错误 (0001-0099)

- **0001**: 名称不合法（包含非法字符或违规内容）
- **0002**: 字段长度超过限制
- **0003**: 批量操作数量超过限制
- **0004**: 分页参数错误（limit、pagenum、pagesize 等）
- **0005**: 请求体格式错误或缺少必需参数
- **0006**: ID 格式错误或无效
- **0007**: 时间参数格式错误
- **0008**: 文件格式错误或未提供
- **0009**: 扩展字段格式错误
- **0010**: 内容不合法（包含违规内容）
- **0011**: 密码格式不符合要求
- **0012**: 禁言时长参数错误
- **0013**: 修改的参数不合法
- **0014**: 新旧操作对象不能相同
- **0015**: content-type 不正确

> 认证授权类错误 (0101-0199)

- **0101**: Token 不合法或已过期
- **0102**: OAuth 认证失败
- **0103**: 权限验证失败
- **0104**: Token 权限不足
- **0105**: 认证模式错误或缺少 Token

> 权限不足类错误 (0201-0299)

- **0201**: 功能未开通
- **0202**: 数量达到上限
- **0203**: 操作权限不足
- **0204**: 访问权限不足
- **0205**: 数据大小超过限制
- **0206**: 频率超出限制

> 资源状态类错误 (0301-0399)

- **0301**: 用户不存在
- **0302**: 应用不存在
- **0303**: 资源不存在
- **0304**: 关系不存在
- **0305**: 记录不存在
- **0306**: 文件不存在
- **0307**: 设备资源不存在

> 资源冲突类错误 (0401-0499)

- **0401**: 资源已存在
- **0402**: 用户已在目标中
- **0403**: 并发操作冲突
- **0404**: 状态冲突
- **0405**: 重复操作

> 频率限制类错误 (0501-0599)

- **0501**: API 调用频率超出限制
- **0502**: 操作频率超出限制
- **0503**: 套餐版本限制

> 服务器错误类 (0701-0799)

- **0701**: 服务器内部错误
- **0702**: 数据库操作失败
- **0703**: 创建操作失败
- **0704**: 更新操作失败
- **0705**: 删除操作失败
- **0706**: 查询操作失败

> 第三方服务错误类 (0801-0899)

- **0801**: 第三方服务连接失败
- **0802**: 第三方服务认证失败
- **0803**: 第三方服务响应超时

**自定义错误码 (1000-9999)**

- **1000+**: 各业务模块可根据特殊需求定义的专用错误码

# 6. 文件

**基本原则**

- **传输方式**：分片上传
- **编码方式**：使用原始二进制数据，避免 Base64 编码带来的 33% 数据量增加
- **Boundary 处理**：使用足够复杂的 boundary 字符串（包含时间戳和随机字符），降低与文件内容冲突的概率

**使用场景**

- **大视频**：聊天视频、语音消息
- **大图片**：高清图片、长图
- **大文档**：PDF、Word、Excel 等
- **压缩包**：ZIP、RAR 等

**请求头**

- `Authorization`: `Bearer {token}`
- `Content-Type`: `multipart/form-data; boundary={unique_boundary_string}`
- `Accept`: `application/json; charset=utf-8`

**响应头**

- `Content-Type`: `application/json; charset=utf-8`

**请求体**

```raw
Content-Type: multipart/form-data; boundary={unique_boundary_string}

--{unique_boundary_string}
Content-Disposition: form-data; name="property1"

[value1]
--{unique_boundary_string}
Content-Disposition: form-data; name="property2"

[value2]
--{unique_boundary_string}
Content-Disposition: form-data; name="property3"; filename="{file_name}"
Content-Type: {Content-Type}

[binary contents of the file]
--{unique_boundary_string}--
```

**响应体**

- fileId: 文件唯一标识
- chunkIndex: 已上传分片索引。当 `chunkIndex = totalChunks - 1`
- uploadedChunks: 已上传分片数量
- totalChunks: 总分片数

```json
{
  "data": {
    "fileId": "file_123456789",
    "chunkIndex": 2,
    "uploadedChunks": 3,
    "totalChunks": 5
  },
  "meta": {...}
}
```

_如果上传失败，请参考章节 4.2_

# 7. 查询

查询功能通过定义排序、分页和搜索等规范，来高效地控制和筛选返回的资源结果集。

## 7.1 排序

支持单字段和多字段排序，使用 `sort` 参数标记使用排序规则，例如：`sort=created_at:desc`。

**排序方向**：`asc`（升序）或 `desc`（降序）

- **典型：时间戳排序**：规定 整数值从小到大（asc），从大到小（desc）。
- **典型：字符串排序**：规定 字符串从小到大（asc）或从大到小（desc）。
- **默认排序**： 如果字段没有指定排序规则，则默认排序规则需要在文档中说明。

**多值排序**： 支持多字段排序，通过 `,` 分割字段。

## 7.2 分页

支持偏移分页和游标分页两种方式，适用于不同场景。

1. 偏移分页

**参数说明：**

| 参数   | 说明     | 要求                           | 适用场景   |
| ------ | -------- | ------------------------------ | ---------- |
| `page` | 页码     | 从 1 开始。例如：`page=1`      | 常规分页   |
| `size` | 每页数量 | 大于 0 的整数。例如：`size=20` | 控制返回量 |

**使用示例：**

```
GET /api/v1/users?page=1&size=20
```

**响应格式：**

- list: 数组类型
- pagination: 页码属性
  - totalPages: 总页数（可选）
  - noMore: 是否有下一页
  - count: 实际数量大小
  - page: 请求的页码

```json
{
  "data": {
    list: [...],
    "pagination": {
      "totalPages": 20,
      "page": 20,
      "count": 18,
      "noMore": true
    }
  },
  "meta": {...}
}
```

1. 游标分页

**参数说明：**

| 参数     | 说明     | 要求                                      | 适用场景   |
| -------- | -------- | ----------------------------------------- | ---------- |
| `cursor` | 游标     | 唯一标识，例如：`cursor=eyJpZCI6IjEyMyJ8` | 大数据集   |
| `limit`  | 限制数量 | 大于 0 的整数。例如：`limit=20`           | 控制返回量 |

**使用示例：**

```
GET /api/v1/users?cursor=eyJpZCI6IjEyMyJ9&limit=20
```

**响应格式：**

- list: 数组类型
- pagination: 页码属性
  - totalPages: 总页数（可选）
  - noMore: 是否有下一页
  - count: 实际数量大小
  - cursor: 下一页游标, 如果没有下一页，返回空字符串

```json
{
  "data": {
    "list": [...],
    "pagination": {
      "totalPages": 20,
      "count": 18,
      "cursor": "",
      "noMore": true
    }
  },
  "meta": {...}
}
```

## 7.3 搜索

支持单值、多值、组合排序等多种搜索方式。

**支持能力**：

- 支持精确匹配
- 支持多值匹配
- 支持多字段联合查询

**操作符规则**：

- **相等匹配**：`status=active`
- **多值查询**：`status=active,pending`
- **联合查询**：使用符号 `&` 间隔

**搜索示例：**：

```http
# 单值搜索
GET /api/v1/users?status=active
GET /api/v1/messages?chat_type=single

# 多值搜索
GET /api/v1/users?status=active,pending
GET /api/v1/messages?chat_type=single,group

# 搜索+排序
GET /api/v1/users?status=active&sort=created_at:desc
GET /api/v1/groups?filter=is_public==true;member_count=ge=10&sort=activity:desc,created_at:desc
GET /api/v1/messages?chat_id=123&sort=created_at:desc
```

# 8. 批处理

**批量操作构成 = URL 路径标识(/batch) + 操作关键字(可选)**

**操作类型分类：**

- **批量创建**：POST /api/v1/{resource}/batch
- **批量更新**：PUT /api/v1/{resource}/batch
- **批量删除**：POST /api/v1/{resource}/batch/delete
- **批量获取**：POST /api/v1/{resource}/batch/get

**事务类型：**

_由服务端决策_

- **原子性事务**：所有操作要么全部成功，要么全部失败
- **非原子性事务**：部分操作成功，部分操作失败

**请求格式：**

- data: 数组类型，数组元素用户自定义。

```json
{
  "data": [
    { "id": "id1" },
    { "id": "id2" }
  ],
  "meta": {...}
}
```

**响应格式：**

- success: 成功结果
- failed: 失败结果
- summary: 总计结果(可选)
  - totalCount: 总数
  - successCount: 成功数
  - failedCount: 失败数

```json
{
  "data": {
    "success": [
      { "id": "id1", "status": "created", "createdAt": 1704110400000 }
    ],
    "failed": [
      { "id": "id2", "error": { "code": 4000001, "message": "名称不合法" } }
    ],
    "summary": {
      "totalCount": 2,
      "successCount": 1,
      "failedCount": 1
    }
  },
  "meta": {...}
}
```

# 9. 安全策略

涵盖认证授权、数据安全、访问控制、传输安全和审计监控等核心安全方面。

## 9.1 传输安全

- **强制 HTTPS**：生产环境所有 API 请求必须通过 HTTPS 加密传输，禁止使用 HTTP 明文通信（除非内网调试）
  - 强制使用 TLS 1.2（含）以上版本，推荐 1.3
  - 使用强密码套件，优先采用 ECDHE（P-256 即可）算法提供前向保密性
  - 确保 SSL/TLS 证书由可信 CA 签发，定期更新，避免使用自签名证书
  - 配置适当的证书密钥长度（RSA 至少 2048 位）和合理有效期（不超过 1 年）
  - 实施 HSTS（HTTP Strict Transport Security）头，防止降级攻击
  - 服务器应当正确配置强制使用 HTTPS

## 9.2 身份验证与授权

- **身份验证机制**：所有 API 必须有某种足够安全的身份验证机制（比如 JWT token/OAuth 2.0）

## 9.3 输入验证

- **数据验证**：必须假定客户端数据是完全不可信的。对所有输入数据（包括请求参数、请求体和标头）必须进行严格验证和过滤，防止 SQL 注入、XSS 等常见攻击。
  - 参数必须有明确的类型、格式、长度要求

## 9.4 访问控制

- **限流措施**：所有 API 必须考虑实现某种限流措施
  - 可基于 App 级、IP 级、User 级等粒度
  - 可以由 API Gateway 统一实施
- **敏感信息保护**：避免在 URL 中暴露敏感信息：敏感数据（如密码、API 密钥等）不应出现在 URL 中，因为它们可能会被记录在服务器日志或浏览器历史记录中。

## 9.5 日志与监控

- **安全日志**：对敏感操作、授权失败等行为应确保有日志记录。
- **敏感数据处理**：生产环境中，一般不得在日志中输出 accesskey、password、手机号等；如确实有必要，敏感字段应当做脱敏处理。
- **日志最小化原则**：只记录必要的信息，避免过度记录导致日志量过大，信息冗余，影响性能等。

# 10. 速率限制

实施基于身份、IP 和资源的多层级限流等策略，保障系统稳定性和可靠性。

## 10.1 限流的目的

- **保护后端资源**： 防止因意外或恶意的高流量请求而导致服务器过载、崩溃或性能下降。
- **保证服务质量**： 确保所有用户都能获得公平的 API 访问机会，避免个别用户过度消耗资源影响其他用户体验。
- **成本控制**： 限制资源消耗，降低不必要的运营成本。
- **安全性**： 帮助抵御某些类型的拒绝服务 (DoS) 攻击。

## 10.2 基本原则

原则上所有 API 都应该有限流的考虑。所有有限流机制的 API 必须在文档中说明存在限流，并尽力给出具体的限制数量。

## 10.3 限流维度和策略

**限流维度**

- **用户级别**（Per User）：特指 IM 系统中租户的用户，以 userId 为唯一标识。
- **IP 地址级别**（Per IP）：一个 IP 地址背后可能有多个 userId。
- **APP 级别**（Per App）：影响该 APP 的所有用户。
- **集群级别**（Per Cluster）：影响该集群中的所有租户。
- **端点级别**（Per Endpoint）：影响所有调用该端点的调用方。

**建议：开发者根据业务场景自行决定使用哪些限流方式。**

**时间单位**

限流的统计时间单位可以为每秒、每分钟、每小时和每天。

**错误处理**

- 发生限流时应该返回标准的 HTTP 错误码 429。
- 如果可能，可以在响应的错误信息中明确告知限流规则，比如"该函数调用频率不得超过 XXX 次/s"。
- 可以在响应中包含 Retry-After 头部，告知调用方多久后再进行重试。

## 10.4 最佳实践

**算法选择**

限流算法可以从常见的算法中选择：

- 令牌桶（Token Bucket）
- 漏桶算法（Leaky Bucket）
- 固定窗口计数器（Fixed Window Counter）
- 滑动窗口计数（Sliding Window Counter）

一般而言，令牌桶是一个不错的算法。固定窗口计数器过于简单，不能处理边界处的突发流量，一般不建议使用。

**监控和预警**

对发生限流的调用应该有记录 log，频繁发生的限流需要有预警。

**数值设置**

- 设定限流数值时要重复考虑调用该 API 的用户行为和需求。
- 对于不同服务等级的租户可以设定不同的限流数值。

# todo: 增量更新

1. 降低客户端和服务器的数据交互
2. 保底机制

# todo: 大数据处理：

1. 客户端立刻返回（较少）
2. 客户端等待返回（通常）

# todo: 是否只采用 jwt token？

1. rongcloud 采用自定义签名的机制https://docs.rongcloud.cn/platform-chat-api
2. 腾讯采用自定义签名的机制 https://cloud.tencent.cn/document/product/269/32688
3. sendbird： 采用主备签名的机制: https://sendbird.com/docs/chat/platform-api/v3/prepare-to-use-api
4. getstream：采用 jwt token 的机制 https://getstream.github.io/protocol/?urls.primaryName=Chat#/product%3Achat/GetApp
