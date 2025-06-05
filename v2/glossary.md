# 重要概念

## 安全 (Safe)

**定义**: HTTP 方法是安全的，意味着它不会修改服务器上的资源状态，不会产生副作用。

**特点**:

- 只读操作，不改变服务器数据
- 可以被缓存、预取、爬虫访问
- 用户可以放心地重复调用

**示例**:

```
✅ 安全的操作：
GET /api/v1/users/123    # 只是获取用户信息
HEAD /api/v1/users/123   # 只获取响应头信息

❌ 不安全的操作：
POST /api/v1/users       # 创建新用户，改变了服务器状态
DELETE /api/v1/users/123 # 删除用户，改变了服务器状态
```

## 幂等 (Idempotent)

**定义**: 多次执行同一个 HTTP 请求，产生的效果和执行一次是相同的。

**特点**:

- 可以安全地重试
- 网络故障时可以自动重发
- 结果具有可预测性

**示例**:

```
✅ 幂等的操作：
GET /api/v1/users/123     # 多次获取，结果相同
PUT /api/v1/users/123     # 多次完整更新，最终状态相同
DELETE /api/v1/users/123  # 多次删除，结果都是用户被删除

❌ 非幂等的操作：
POST /api/v1/users        # 每次调用都会创建新用户
PATCH /api/v1/users/123   # 部分更新可能产生累积效应
```

**实际应用**:

```javascript
// 幂等的PUT操作
PUT /api/v1/users/123
{
  "name": "张三",
  "age": 25,
  "status": "active"
}
// 无论调用多少次，用户123的最终状态都是相同的

// 非幂等的POST操作
POST /api/v1/orders/123/items
{
  "productId": 456,
  "quantity": 1
}
// 每次调用都会向订单中添加一个商品
```

## 可缓存 (Cacheable)

**定义**: HTTP 响应可以被客户端、代理服务器或 CDN 缓存，以提高性能和减少服务器负载。

**特点**:

- 减少网络请求次数
- 提高响应速度
- 降低服务器压力
- 需要合适的缓存策略

**缓存控制示例**:

```http
GET /api/v1/users/123
HTTP/1.1 200 OK
Cache-Control: max-age=3600, public
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT

{
  "id": 123,
  "name": "张三"
}
```

**缓存策略**:

```http
# 不缓存
Cache-Control: no-cache, no-store

# 缓存1小时
Cache-Control: max-age=3600

# 私有缓存（只能被用户浏览器缓存）
Cache-Control: private, max-age=300

# 公共缓存（可以被CDN等缓存）
Cache-Control: public, max-age=86400
```
