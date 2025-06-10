# 缓存控制高级指南

> **适用人群**: 已掌握基础缓存概念的开发者  
> **前置阅读**: [基本规则 - 规则 10: 缓存控制](basic_design.md#规则-10-缓存控制)

## 概览

本文档深入介绍缓存适用场景、验证机制、条件请求、高级缓存策略等内容，帮助开发者构建更高效的缓存系统。

## 1. 缓存适用场景分析

### 1.1 场景评估标准

在决定是否使用缓存时，需要考虑以下几个维度：

| 评估维度         | 适合缓存      | 不适合缓存   |
| ---------------- | ------------- | ------------ |
| **数据变化频率** | 低频变化      | 高频变化     |
| **计算复杂度**   | 复杂查询/计算 | 简单查询     |
| **数据敏感性**   | 允许略微过期  | 必须实时准确 |
| **访问频率**     | 高频访问      | 低频访问     |
| **用户影响**     | 影响用户体验  | 不影响体验   |

### 1.2 适合缓存的典型场景

#### 📚 静态配置类数据

```http
# 系统配置
GET /api/v1/configs/system
Cache-Control: public, max-age=86400, immutable
# 特点：很少变化，全局共享，访问频繁

# 分类列表
GET /api/v1/categories
Cache-Control: public, max-age=3600
# 特点：相对稳定，展示用途，查询频繁
```

**缓存策略建议：**

- 缓存时间：1-24 小时
- 缓存类型：public（公共缓存）
- 失效机制：管理员更新时主动失效

#### 👤 用户基础信息

```http
# 用户资料
GET /api/v1/users/123/profile
Cache-Control: public, max-age=1800, must-revalidate
# 特点：变化不频繁，多处展示，查询多

# 用户头像
GET /api/v1/users/123/avatar
Cache-Control: public, max-age=2592000, immutable
# 特点：长期不变，大量访问，文件较大
```

**缓存策略建议：**

- 缓存时间：30 分钟-1 小时
- 缓存类型：public（头像）/ private（敏感信息）
- 失效机制：用户更新时失效

#### 📊 聚合统计数据

```http
# 文章阅读统计
GET /api/v1/posts/123/stats
Cache-Control: public, max-age=300, stale-while-revalidate=60
# 特点：计算复杂，允许延迟，频繁查询

# 热门排行榜
GET /api/v1/posts/trending
Cache-Control: public, max-age=600
# 特点：复杂算法，实时性要求不高，访问量大
```

**缓存策略建议：**

- 缓存时间：5-15 分钟
- 缓存类型：public
- 更新策略：后台定时更新

#### 🔍 搜索结果

```http
# 常见搜索词结果
GET /api/v1/search?q=javascript&type=posts
Cache-Control: public, max-age=900, stale-while-revalidate=300
# 特点：相同查询重复多，计算成本高

# 地理位置搜索
GET /api/v1/locations?city=beijing&radius=10km
Cache-Control: public, max-age=1800
# 特点：地理数据相对稳定，查询复杂
```

**缓存策略建议：**

- 缓存时间：15-30 分钟
- 缓存类型：public
- 缓存键：包含所有查询参数

#### 📄 内容管理类

```http
# 文章详情（已发布）
GET /api/v1/posts/123
Cache-Control: public, max-age=1800, must-revalidate
# 特点：内容稳定，访问频繁，SEO重要

# 页面模板
GET /api/v1/templates/homepage
Cache-Control: public, max-age=3600
# 特点：设计确定后很少改动，全站使用
```

**缓存策略建议：**

- 缓存时间：30 分钟-1 小时
- 缓存类型：public
- 失效机制：内容更新时失效

### 1.3 不适合缓存的场景

#### 🚫 实时性要求高的数据

```http
# 支付状态查询
GET /api/v1/payments/123/status
Cache-Control: no-cache, no-store, must-revalidate
# 原因：金融数据必须实时准确

# 实时消息推送
GET /api/v1/messages/unread
Cache-Control: no-cache, no-store
# 原因：消息延迟影响用户体验

# 库存查询（电商）
GET /api/v1/products/123/stock
Cache-Control: no-cache, max-age=0
# 原因：库存数据快速变化，影响交易
```

#### 🚫 高度个人化的数据

```http
# 个人消息列表
GET /api/v1/users/me/messages
Cache-Control: private, no-cache
# 原因：每个用户数据不同，变化频繁

# 个人通知
GET /api/v1/users/me/notifications
Cache-Control: private, max-age=30
# 原因：高度个性化，但可短时缓存
```

#### 🚫 敏感安全数据

```http
# 登录验证
POST /api/v1/auth/login
Cache-Control: no-cache, no-store, must-revalidate
# 原因：安全敏感，不能缓存

# 权限检查
GET /api/v1/users/me/permissions
Cache-Control: private, max-age=300, must-revalidate
# 原因：权限变化需要及时反映
```

### 1.4 特殊场景的缓存策略

#### 🎯 分级缓存策略

```http
# 新闻首页（不同用户群体）
GET /api/v1/news/homepage?region=beijing&category=tech
Cache-Control: public, max-age=300, stale-while-revalidate=60
Vary: Accept-Language, User-Agent
# 策略：按地区和分类分层缓存

# 用户动态（关注的人的动态）
GET /api/v1/users/me/timeline
Cache-Control: private, max-age=180
# 策略：个人时间线短时缓存
```

#### 🔄 条件缓存

```http
# 在线用户状态
GET /api/v1/users/123/online-status
Cache-Control: public, max-age=60
# 策略：1分钟短缓存，平衡实时性和性能

# 评论列表（有新评论时）
GET /api/v1/posts/123/comments
Cache-Control: public, max-age=300
If-Modified-Since: Wed, 01 Jan 2024 10:00:00 GMT
# 策略：使用条件请求检查更新
```

### 1.5 缓存场景决策树

```
是否需要缓存？
├── 数据变化频率 < 每分钟？
│   ├── 是 → 计算/查询复杂？
│   │   ├── 是 → 访问频率高？
│   │   │   ├── 是 → 强烈推荐缓存
│   │   │   └── 否 → 考虑缓存
│   │   └── 否 → 不推荐缓存
│   └── 否 → 不适合缓存
├── 实时性要求？
│   ├── 高 → 不适合缓存
│   ├── 中 → 短时缓存（< 5分钟）
│   └── 低 → 适合缓存
└── 安全敏感数据？
    ├── 是 → 不适合缓存
    └── 否 → 可以缓存
```

### 1.6 业务场景缓存建议

#### 电商平台

```http
# 商品基本信息（价格除外）
GET /api/v1/products/123
Cache-Control: public, max-age=1800

# 商品价格（频繁变动）
GET /api/v1/products/123/price
Cache-Control: public, max-age=60

# 用户购物车
GET /api/v1/users/me/cart
Cache-Control: private, max-age=300
```

#### 社交媒体

```http
# 用户资料
GET /api/v1/users/123/profile
Cache-Control: public, max-age=900

# 动态时间线
GET /api/v1/users/me/timeline
Cache-Control: private, max-age=180

# 热门话题
GET /api/v1/topics/trending
Cache-Control: public, max-age=600
```

#### 内容管理系统

```http
# 已发布文章
GET /api/v1/articles/123
Cache-Control: public, max-age=3600

# 草稿文章
GET /api/v1/articles/123?draft=true
Cache-Control: private, no-cache

# 文章列表
GET /api/v1/articles?category=tech
Cache-Control: public, max-age=900
```

## 2. 缓存验证机制

### 2.1 ETag（实体标签）验证

ETag 是资源的唯一标识符，类似于资源的"指纹"，用于验证缓存是否仍然有效。

#### 基本工作流程

**完整的 ETag 生命周期：**

```http
# 0. 创建用户时（ETag诞生）
POST /api/v1/users
{
  "name": "张三",
  "email": "zhangsan@example.com"
}

# 服务端处理：
# 1) 保存用户数据到数据库
# 2) 基于新数据生成ETag: "user123-v1-abc"
# 3) 将ETag存储（可选：存在数据库/缓存中）

Response:
HTTP/1.1 201 Created
ETag: "user123-v1-abc"  # 新生成的ETag
{
  "id": 123,
  "name": "张三",
  "email": "zhangsan@example.com",
  "created_at": "2024-01-01T10:00:00Z",
  "updated_at": "2024-01-01T10:00:00Z"
}
```

```http
# 1. 首次GET请求
GET /api/v1/users/123

# 服务端处理：
# 1) 从数据库获取用户数据
# 2) 基于当前数据生成ETag: "user123-v1-abc"
# 3) 返回数据和ETag

Response:
ETag: "user123-v1-abc"
Cache-Control: public, max-age=3600
{
  "id": 123,
  "name": "张三",
  "email": "zhangsan@example.com",
  "updated_at": "2024-01-01T10:00:00Z"
}
```

```http
# 2. 更新用户数据（ETag变化）
PUT /api/v1/users/123
{
  "name": "李四",
  "email": "zhangsan@example.com"
}

# 服务端处理：
# 1) 更新数据库中的用户数据
# 2) 基于新数据生成新ETag: "user123-v2-def"
# 3) 返回更新后的数据和新ETag

Response:
ETag: "user123-v2-def"  # 新的ETag
{
  "id": 123,
  "name": "李四",  # 内容已更新
  "email": "zhangsan@example.com",
  "updated_at": "2024-01-01T11:00:00Z"
}
```

```http
# 3. 缓存验证请求（使用旧ETag）
GET /api/v1/users/123
If-None-Match: "user123-v1-abc"  # 客户端的旧ETag

# 服务端处理：
# 1) 从数据库获取当前数据
# 2) 生成当前ETag: "user123-v2-def"
# 3) 比较请求中的ETag和当前ETag
# 4) 不匹配，返回新数据

Response:
HTTP/1.1 200 OK
ETag: "user123-v2-def"  # 返回当前ETag
{
  "id": 123,
  "name": "李四",  # 返回最新内容
  "email": "zhangsan@example.com",
  "updated_at": "2024-01-01T11:00:00Z"
}
```

```http
# 4. 缓存验证请求（使用新ETag）
GET /api/v1/users/123
If-None-Match: "user123-v2-def"  # 客户端的新ETag

# 服务端处理：
# 1) 从数据库获取当前数据
# 2) 生成当前ETag: "user123-v2-def"
# 3) 比较请求中的ETag和当前ETag
# 4) 匹配！数据未变化，返回304

Response:
HTTP/1.1 304 Not Modified
ETag: "user123-v2-def"
# 无响应体，节省带宽
```

**ETag 的存储方式：**

```javascript
// 方式1: 实时生成（推荐）
app.get("/api/v1/users/:id", async (req, res) => {
  // 1. 从数据库获取最新数据
  const user = await User.findById(req.params.id);

  // 2. 基于数据生成ETag
  const etag = generateETag(user);

  // 3. 检查客户端ETag
  if (req.headers["if-none-match"] === etag) {
    return res.status(304).set("ETag", etag).end();
  }

  // 4. 返回数据和ETag
  res.set("ETag", etag).json(user);
});

// 方式2: 预存储ETag
app.put("/api/v1/users/:id", async (req, res) => {
  // 1. 更新数据
  const user = await User.findByIdAndUpdate(req.params.id, req.body, {
    new: true,
  });

  // 2. 生成新ETag
  const etag = generateETag(user);

  // 3. 可选：将ETag存储到数据库
  await User.findByIdAndUpdate(req.params.id, { etag: etag });

  // 4. 返回数据和ETag
  res.set("ETag", etag).json(user);
});
```

#### ETag 生成策略

**方案 1: 基于内容哈希**

```javascript
const crypto = require("crypto");

function generateETag(user) {
  const content = JSON.stringify({
    id: user.id,
    name: user.name,
    email: user.email,
    updated_at: user.updated_at,
  });

  const hash = crypto.createHash("md5").update(content).digest("hex");
  return `"${hash}"`; // ETag需要用引号包围
}
```

**方案 2: 基于版本号**

```javascript
function generateETag(user) {
  return `"user-${user.id}-v${user.version}"`;
}

// 数据库表结构
// users: id, name, email, version, created_at, updated_at
// 每次更新时：UPDATE users SET ..., version = version + 1
```

**方案 3: 基于时间戳**

```javascript
function generateETag(user) {
  const timestamp = new Date(user.updated_at).getTime();
  return `"user-${user.id}-${timestamp}"`;
}
```

### 2.2 Last-Modified 验证

基于最后修改时间的缓存验证机制。

```http
# 首次请求
GET /api/v1/users/123
Response:
Last-Modified: Wed, 01 Jan 2024 10:00:00 GMT
Cache-Control: public, max-age=3600

# 验证请求
GET /api/v1/users/123
If-Modified-Since: Wed, 01 Jan 2024 10:00:00 GMT

# 未修改的响应
HTTP/1.1 304 Not Modified
Last-Modified: Wed, 01 Jan 2024 10:00:00 GMT
```

### 2.3 ETag vs Last-Modified 对比

| 特性         | ETag                    | Last-Modified          |
| ------------ | ----------------------- | ---------------------- |
| **精确度**   | 高（内容变化即变化）    | 低（时间精度限制）     |
| **性能开销** | 中等（需计算哈希）      | 低（直接使用时间戳）   |
| **适用场景** | 内容频繁微调            | 文件系统、时间敏感资源 |
| **优先级**   | 高（HTTP 规范优先使用） | 低                     |

## 3. 条件请求

### 3.1 条件请求头

| 请求头            | 对应响应头         | 条件     | 用途     |
| ----------------- | ------------------ | -------- | -------- |
| If-None-Match     | ETag               | 不匹配时 | 缓存验证 |
| If-Match          | ETag               | 匹配时   | 安全更新 |
| If-Modified-Since | Last-Modified      | 已修改时 | 缓存验证 |
| If-Range          | ETag/Last-Modified | 范围请求 | 断点续传 |

### 3.2 安全更新示例

```http
# 1. 获取资源进行编辑
GET /api/v1/posts/456
Response:
ETag: "post-456-v3"
{
  "id": 456,
  "title": "原标题",
  "content": "原内容"
}

# 2. 条件更新（防止并发修改冲突）
PUT /api/v1/posts/456
If-Match: "post-456-v3"  # 只有ETag匹配才更新
{
  "title": "新标题",
  "content": "新内容"
}

# 成功响应
HTTP/1.1 200 OK
ETag: "post-456-v4"

# 冲突响应（其他用户已修改）
HTTP/1.1 412 Precondition Failed
{
  "error": {
    "code": "PRECONDITION_FAILED",
    "message": "资源已被其他用户修改，请刷新后重试"
  }
}
```

## 4. 高级缓存策略

### 4.1 分层缓存策略

```http
# 长期缓存（配置类数据）
GET /api/v1/configs/system
Cache-Control: public, max-age=86400, immutable
ETag: "config-v5"

# 中期缓存（用户数据）
GET /api/v1/users/123
Cache-Control: public, max-age=3600, must-revalidate
ETag: "user123-v2"

# 短期缓存（动态数据）
GET /api/v1/posts/trending
Cache-Control: public, max-age=300, stale-while-revalidate=60
ETag: "trending-20240101-1200"

# 私有缓存（敏感数据）
GET /api/v1/users/123/private-info
Cache-Control: private, max-age=1800, no-transform
ETag: "private-user123-v1"
```

### 4.2 缓存指令详解

| 指令                   | 含义                       | 使用场景       |
| ---------------------- | -------------------------- | -------------- |
| public                 | 可被任何缓存存储           | 公开数据       |
| private                | 只能被用户缓存存储         | 用户特定数据   |
| no-cache               | 必须验证后才能使用         | 需要验证的数据 |
| no-store               | 不得存储任何缓存           | 敏感数据       |
| must-revalidate        | 过期后必须验证             | 重要数据       |
| stale-while-revalidate | 允许使用过期缓存并后台更新 | 用户体验优化   |
| immutable              | 内容永不改变               | 静态资源       |

### 4.3 组合缓存策略示例

```http
# 用户头像（长期 + 版本控制）
GET /api/v1/users/123/avatar?v=5
Cache-Control: public, max-age=2592000, immutable
ETag: "avatar-123-v5"

# 文章列表（短期 + 后台更新）
GET /api/v1/posts?category=tech
Cache-Control: public, max-age=180, stale-while-revalidate=60
ETag: "posts-tech-20240101-1205"

# 用户个人设置（私有 + 必须验证）
GET /api/v1/users/me/settings
Cache-Control: private, max-age=900, must-revalidate
ETag: "settings-user123-v7"
```

## 5. 缓存失效策略

### 5.1 主动失效

```http
# 更新用户信息时，提供缓存失效建议
PUT /api/v1/users/123
Response:
X-Cache-Invalidate: /api/v1/users/123, /api/v1/users/123/profile
ETag: "user123-v3"
```

### 5.2 依赖失效

```http
# 更新用户头像时，相关缓存失效
POST /api/v1/users/123/avatar
Response:
X-Cache-Invalidate: /api/v1/users/123, /api/v1/users/123/avatar
X-Cache-Tags: user-123, avatar-123
```

### 5.3 批量失效

```http
# 批量操作时的缓存失效
PATCH /api/v1/users/batch
Response:
X-Cache-Invalidate-Pattern: /api/v1/users/*
X-Cache-Tags: user-list, user-search
```

## 6. 性能监控

### 6.1 缓存性能头

```http
# 缓存命中情况
X-Cache-Status: HIT|MISS|BYPASS|EXPIRED
X-Cache-Age: 1800        # 缓存年龄（秒）
X-Cache-TTL: 1200        # 剩余TTL（秒）
X-Cache-Key: user:123:v2 # 缓存键
```

### 6.2 缓存统计

```http
# 在调试模式下提供详细统计
X-Cache-Debug: true
Response:
X-Cache-Hit-Ratio: 85.6%
X-Cache-Miss-Reason: cache-expired
X-Cache-Backend: redis-cluster-1
X-Cache-Latency: 2ms
```

## 7. 边缘情况处理

### 7.1 缓存穿透保护

```http
# 对不存在的资源也提供缓存控制
GET /api/v1/users/999999
HTTP/1.1 404 Not Found
Cache-Control: public, max-age=60  # 短时间缓存404响应
ETag: "not-found-users-999999"
```

### 7.2 缓存雪崩预防

```http
# 添加随机过期时间
Cache-Control: public, max-age=3600
X-Cache-Jitter: 300  # ±5分钟随机时间
```

### 7.3 热点数据处理

```http
# 热点数据特殊处理
GET /api/v1/posts/trending-hot
Response:
Cache-Control: public, max-age=30, stale-while-revalidate=300
X-Cache-Priority: high
X-Cache-Replicas: 3
```

## 8. 客户端实现建议

### 8.1 智能缓存客户端

```javascript
class APIClient {
  async fetchWithCache(url, options = {}) {
    const cachedData = this.getFromCache(url);
    const headers = { ...options.headers };

    // 添加条件请求头
    if (cachedData?.etag && !options.forceFresh) {
      headers["If-None-Match"] = cachedData.etag;
    }

    if (cachedData?.lastModified && !options.forceFresh) {
      headers["If-Modified-Since"] = cachedData.lastModified;
    }

    const response = await fetch(url, { ...options, headers });

    if (response.status === 304) {
      // 使用缓存数据
      return this.refreshCacheMetadata(url, response.headers);
    }

    // 更新缓存
    const data = await response.json();
    this.saveToCache(url, {
      data,
      etag: response.headers.get("ETag"),
      lastModified: response.headers.get("Last-Modified"),
      cacheControl: this.parseCacheControl(
        response.headers.get("Cache-Control"),
      ),
    });

    return data;
  }
}
```

### 8.2 缓存策略配置

```javascript
const cacheStrategies = {
  "user-profile": {
    maxAge: 3600,
    staleWhileRevalidate: 1800,
    validateOnBackground: true,
  },
  "real-time-data": {
    maxAge: 0,
    forceRevalidate: true,
  },
  "static-config": {
    maxAge: 86400,
    immutable: true,
  },
};
```

## 9. 调试和故障排除

### 9.1 缓存调试头

```http
# 开发环境下的调试信息
GET /api/v1/users/123
X-Debug-Cache: true

Response:
X-Cache-Debug-Info: {
  "backend": "redis",
  "key": "user:123:v2",
  "hit": true,
  "age": 1800,
  "ttl": 1200,
  "size": "2.5KB"
}
```

### 9.2 常见问题解决

**问题 1: 缓存不一致**

```http
# 强制刷新缓存
GET /api/v1/users/123
Cache-Control: no-cache
```

**问题 2: 缓存过期策略不当**

```http
# 检查缓存配置
GET /api/v1/debug/cache-config
Response:
{
  "policies": {
    "/api/v1/users/*": "max-age=3600, must-revalidate",
    "/api/v1/posts/*": "max-age=300, stale-while-revalidate=60"
  }
}
```

## 10. 条件请求和缓存验证

### 10.1 ETag 机制

ETag（实体标签）是HTTP协议中用于缓存验证的重要机制，符合RFC 9110标准。

#### 强ETag vs 弱ETag

```http
# 强ETag - 精确匹配
ETag: "v1.0.123"

# 弱ETag - 语义等价
ETag: W/"v1.0.123"
```

#### ETag 生成策略

```javascript
// 基于内容的哈希
const etag = `"${crypto.createHash('md5').update(JSON.stringify(data)).digest('hex')}"`;

// 基于版本号
const etag = `"v${resource.version}.${resource.updatedAt.getTime()}"`;

// 基于最后修改时间
const etag = `"${resource.id}-${resource.updatedAt.toISOString()}"`;
```

### 10.2 条件请求头部

#### If-None-Match（用于GET请求）

```http
# 客户端请求
GET /api/v1/users/123
If-None-Match: "v1.0.123"

# 服务器响应（数据未变化）
HTTP/1.1 304 Not Modified
ETag: "v1.0.123"
Cache-Control: public, max-age=3600
```

#### If-Match（用于PUT/PATCH/DELETE请求）

```http
# 客户端更新请求
PUT /api/v1/users/123
If-Match: "v1.0.123"
{
  "name": "新名称"
}

# 服务器响应（成功更新）
HTTP/1.1 200 OK
ETag: "v1.0.124"
{
  "id": 123,
  "name": "新名称",
  "version": "v1.0.124"
}
```

#### If-Match 冲突处理

```http
# 客户端使用过期的ETag
PUT /api/v1/users/123
If-Match: "v1.0.123"

# 服务器响应（版本冲突）
HTTP/1.1 412 Precondition Failed
ETag: "v1.0.125"
{
  "error": {
    "code": "VERSION_CONFLICT",
    "message": "资源已被其他客户端修改",
    "currentETag": "v1.0.125"
  }
}
```

### 10.3 Last-Modified 机制

#### 基于时间的缓存验证

```http
# 服务器响应
HTTP/1.1 200 OK
Last-Modified: Wed, 01 Jan 2024 12:00:00 GMT
Cache-Control: public, max-age=3600

# 客户端条件请求
GET /api/v1/posts/123
If-Modified-Since: Wed, 01 Jan 2024 12:00:00 GMT

# 服务器响应（未修改）
HTTP/1.1 304 Not Modified
Last-Modified: Wed, 01 Jan 2024 12:00:00 GMT
```

#### If-Unmodified-Since（用于更新操作）

```http
# 客户端更新请求
PUT /api/v1/posts/123
If-Unmodified-Since: Wed, 01 Jan 2024 12:00:00 GMT
{
  "title": "新标题"
}

# 服务器响应（资源已被修改）
HTTP/1.1 412 Precondition Failed
Last-Modified: Wed, 01 Jan 2024 14:00:00 GMT
```

### 10.4 条件请求最佳实践

#### 优先级规则

当同时存在多个条件头部时，按以下优先级处理：

1. **If-Match** > **If-Unmodified-Since**
2. **If-None-Match** > **If-Modified-Since**
3. **强ETag** > **弱ETag** > **Last-Modified**

#### 完整的条件请求流程

```http
# 1. 首次请求
GET /api/v1/articles/123
Accept: application/json

# 服务器响应
HTTP/1.1 200 OK
ETag: "article-123-v1.2.3"
Last-Modified: Thu, 02 Jan 2024 10:30:00 GMT
Cache-Control: public, max-age=1800
{
  "id": 123,
  "title": "文章标题",
  "content": "文章内容...",
  "version": "v1.2.3"
}

# 2. 缓存过期后的条件请求
GET /api/v1/articles/123
If-None-Match: "article-123-v1.2.3"
If-Modified-Since: Thu, 02 Jan 2024 10:30:00 GMT

# 服务器响应（内容未改变）
HTTP/1.1 304 Not Modified
ETag: "article-123-v1.2.3"
Last-Modified: Thu, 02 Jan 2024 10:30:00 GMT
Cache-Control: public, max-age=1800

# 3. 更新操作
PUT /api/v1/articles/123
If-Match: "article-123-v1.2.3"
Content-Type: application/json
{
  "title": "更新的标题"
}

# 服务器响应（更新成功）
HTTP/1.1 200 OK
ETag: "article-123-v1.2.4"
Last-Modified: Thu, 02 Jan 2024 15:45:00 GMT
{
  "id": 123,
  "title": "更新的标题",
  "content": "文章内容...",
  "version": "v1.2.4"
}
```

### 10.5 条件请求实现建议

#### 资源版本管理

```json
{
  "id": 123,
  "data": {...},
  "meta": {
    "version": "v1.2.3",
    "createdAt": "2024-01-01T12:00:00Z",
    "updatedAt": "2024-01-02T15:45:00Z",
    "etag": "article-123-v1.2.3"
  }
}
```

#### 条件请求状态码

| 状态码 | 使用场景 | 说明 |
|--------|----------|------|
| 304    | Not Modified | 条件请求资源未修改 |
| 412    | Precondition Failed | 条件不满足（如版本冲突） |
| 428    | Precondition Required | 要求提供条件头部 |

## 总结

高级缓存控制是现代 API 设计的重要组成部分，正确实现可以：

- 显著减少服务器负载
- 提高响应速度
- 降低网络带宽使用
- 改善用户体验

关键要点：

1. **选择合适的验证机制**：ETag 用于精确控制，Last-Modified 用于时间敏感场景
2. **设计分层缓存策略**：根据数据特性选择不同的缓存时间和策略
3. **处理边缘情况**：考虑缓存穿透、雪崩等问题
4. **监控缓存性能**：通过响应头提供缓存状态信息
5. **客户端智能化**：实现智能的客户端缓存逻辑
6. **实现条件请求**：使用ETag和Last-Modified进行缓存验证，减少不必要的数据传输
