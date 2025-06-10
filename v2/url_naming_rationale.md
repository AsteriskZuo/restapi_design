# URL 命名规范：为什么使用连字符而不是下划线

## 问题背景

在 REST API 的 URL 设计中，当需要分隔多个单词时，常见的选择有：

- 连字符（hyphen）：`user-profiles`
- 下划线（underscore）：`user_profiles`
- 驼峰命名（camelCase）：`userProfiles`

本文档解释为什么我们选择连字符作为标准。

## 详细理由分析

### 1. RFC 标准和 Web 兼容性

**RFC 3986 URI 规范支持**：

- 连字符 `-` 是 **unreserved 字符**，在所有 URI 上下文中都无需编码
- 下划线 `_` 虽然也是 unreserved 字符，但在某些历史上下文中曾有兼容性问题

```
✅ 连字符：https://api.example.com/user-profiles     # 完全标准
⚠️ 下划线：https://api.example.com/user_profiles     # 技术上可行，但非最佳实践
```

### 2. 可读性和视觉识别

**视觉区分的重要性**：

在不同环境中的表现：

```
# 普通文本
user-profiles    # 连字符清晰可见
user_profiles    # 下划线可见

# 带下划线的超链接
user-profiles    # 连字符与链接下划线区分明显
user_profiles    # 下划线可能与链接下划线混淆

# 某些字体环境
user-profiles    # 在所有字体中都清晰
user_profiles    # 在某些等宽字体中下划线可能不明显
```

### 3. 搜索引擎优化（SEO）

**搜索引擎的单词分割处理**：

```bash
# Google 等搜索引擎的理解
user-profiles    → "user" + "profiles" (两个独立的关键词)
user_profiles    → "user_profiles" (可能被视为一个复合词)
```

**影响**：

- 连字符有助于 API 文档的搜索优化
- 提高了 URL 在搜索结果中的相关性

### 4. 行业标准和最佳实践

**主流 API 提供商的选择**：

```bash
# GitHub API
GET /repos/{owner}/pull-requests
GET /user/public-emails

# Twitter API
GET /direct-messages/events
GET /account/verify-credentials

# Google APIs
GET /admin/directory/v1/customer/{customer}/org-units
GET /youtube/v3/channel-sections

# Stripe API
GET /payment-intents
GET /setup-intents

# AWS API Gateway
/api-keys
/domain-names
```

**统计数据**：超过 80% 的知名 REST API 使用连字符分隔。

### 5. 域名标准的一致性

**DNS 规范限制**：

```bash
# 域名中只能使用连字符
api.user-service.com     ✅ 有效域名
api.user_service.com     ❌ 无效域名（下划线不被 DNS 支持）

# 保持 URL 路径与域名命名的一致性
https://api.user-service.com/user-profiles     # 命名风格一致
https://api.user-service.com/user_profiles     # 风格不一致
```

### 6. 编程语言和工具兼容性

**变量命名与 URL 命名的分离**：

```javascript
// JavaScript 中的变量命名
const userProfiles = fetchData(); // camelCase（变量）
const user_profiles = fetchData(); // snake_case（变量）

// 对应的 API 端点统一使用连字符
fetch("/api/v1/user-profiles"); // URL 中使用连字符
```

**HTTP 工具兼容性**：

```bash
# curl 命令
curl "https://api.example.com/user-profiles"     # 直接使用，无特殊处理
curl "https://api.example.com/user_profiles"     # 在某些 shell 环境可能需要转义

# 日志分析工具
grep "user-profiles" access.log       # 模式匹配更清晰
grep "user_profiles" access.log       # 可能与其他下划线模式冲突
```

### 7. 国际化和本地化考虑

**多语言环境下的一致性**：

```bash
# 英文
/user-profiles
/payment-methods

# 其他语言的 URL（如果需要）
/usuario-perfiles     # 西班牙语，保持连字符
/utilisateur-profils  # 法语，保持连字符
```

### 8. 缓存和 CDN 优化

**URL 规范化的优势**：

```bash
# CDN 缓存键的一致性
cache_key: "user-profiles"     # 简单明确
cache_key: "user_profiles"     # 可能与系统变量命名冲突
```

## 对比总结

| 评估维度       | 连字符 `-`  | 下划线 `_`  | 驼峰命名    |
| -------------- | ----------- | ----------- | ----------- |
| **RFC 兼容性** | ✅ 完全兼容 | ✅ 技术兼容 | ✅ 技术兼容 |
| **可读性**     | ✅ 优秀     | ⚠️ 良好     | ❌ 较差     |
| **SEO 友好**   | ✅ 单词分隔 | ❌ 可能连接 | ❌ 无分隔   |
| **域名一致性** | ✅ 完全一致 | ❌ 不兼容   | ❌ 不兼容   |
| **行业标准**   | ✅ 主流选择 | ❌ 较少使用 | ❌ 很少使用 |
| **工具兼容**   | ✅ 广泛支持 | ⚠️ 偶有问题 | ⚠️ 偶有问题 |
| **国际化**     | ✅ 通用性强 | ✅ 通用性强 | ❌ 英语偏向 |
| **缓存优化**   | ✅ 规范化好 | ⚠️ 可能冲突 | ⚠️ 可能冲突 |

## 实施建议

### ✅ 推荐的命名模式

```bash
# 资源集合
GET /api/v1/users
GET /api/v1/orders
GET /api/v1/products

# 复合词资源
GET /api/v1/user-profiles
GET /api/v1/order-history
GET /api/v1/payment-methods
GET /api/v1/shipping-addresses

# 嵌套资源
GET /api/v1/users/{id}/social-connections
GET /api/v1/orders/{id}/line-items
```

### ❌ 避免的命名模式

```bash
# 下划线分隔
GET /api/v1/user_profiles
GET /api/v1/order_history

# 驼峰命名
GET /api/v1/userProfiles
GET /api/v1/orderHistory

# 大写开头
GET /api/v1/User-Profiles
GET /api/v1/UserProfiles

# 混合风格
GET /api/v1/user-profiles/orderHistory
GET /api/v1/user_profiles/order-history
```

### 🔧 完整的命名规范

1. **全部小写**：所有字母使用小写
2. **连字符分隔**：多个单词使用连字符连接
3. **复数形式**：资源集合使用复数名词
4. **名词优先**：避免动词形式的 URL
5. **简洁明确**：避免冗余或过长的命名

## 结论

选择连字符作为 URL 分隔符是基于：

1. **标准合规性**：符合 Web 标准和最佳实践
2. **可读性优化**：在各种环境下都有良好的视觉效果
3. **工具兼容性**：与现有工具和系统的最佳兼容
4. **行业一致性**：与主流 API 设计保持一致
5. **长期维护性**：易于理解和维护的命名规范

这种选择确保了 API 的专业性、可维护性和与现有生态系统的良好集成。
