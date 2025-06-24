# IM URL 规范

本文档基于 `RFC3986`、`RFC7230` 等国际标准，结合 IM 业务场景特点，形成了这套完整的 URL 编码规范。

## 1. URL 结构规则

**URL 结构 = 协议 + 主机 + 版本 + 组织名 + 应用名 + 资源路径 + 查询参数等**

**分层结构：**

- **协议**：http/https
- **主机**：API 服务域名
  - 正式环境：api.easemob.com
  - 沙箱环境：api.dev-{env}.easemob.com
- **版本**：API 版本号（v1/v2 等）
- **组织名**：组织名称（租户标识）
  - 用于多租户隔离
  - 支持跨组织数据访问控制
- **应用名**：应用名称（应用标识）
  - 用于多应用隔离
  - 支持应用级别的配置管理
- **资源路径**：资源层级
- **查询参数**：过滤条件

**环信 REST 接口示例：**

_仅供参考_

```http
https://{host}/{version}/{org_id}/{app_id}/auth
https://{host}/{version}/{org_id}/{app_id}/users
https://{host}/{version}/{org_id}/{app_id}/groups
https://{host}/{version}/{org_id}/{app_id}/groups/threads // ??? 仅供参考
https://{host}/{version}/{org_id}/{app_id}/rooms
https://{host}/{version}/{org_id}/{app_id}/messages
https://{host}/{version}/{org_id}/{app_id}/messages/reactions
https://{host}/{version}/{org_id}/{app_id}/push
```

**完整 URL 示例：**

```http
# ✅ 正确:
https://api.easemob.com/v1/myorg/myapp/users // 1.0版本
https://api.easemob.com/v2/myorg/myapp/users // 2.0版本
https://api.dev-a1.easemob.com/v1/myorg/myapp/users    // 沙箱环境
https://api.dev-a61.easemob.com/v1/myorg/myapp/users   // 沙箱环境

# ❌ 错误:
https://api.easemob.com/v1.2/myorg/myapp/users // 不支持次要版本号
https://api.easemob.com/v1.2.3/myorg/myapp/users // 不支持补丁版本号
```

## 2. URL 组件规范

### 2.1 协议规范

- **http**：开发测试环境
- **https**：生产环境（强制使用）

### 2.2 主机规范

- **生产环境**：`api.easemob.com`
- **沙箱环境**：`api.dev-{env}.easemob.com`
- **IP 地址**：仅用于内部测试

### 2.3 版本规范

- **格式**：`/v{number}`
- **示例**：`/v1`、`/v2`
- **规则**：主版本号递增

### 2.4 组织名规范

- **格式**：`/{org_id}`
- **示例**：`/myorg`、`/testorg`
- **规则**：系统分配，用户无法修改

### 2.5 应用名规范

- **格式**：`/{app_id}`
- **示例**：`/myapp`、`/testapp`
- **规则**：系统分配，用户无法修改

### 2.6 资源路径规范

- **格式**：`/resource[/sub-resource]`
- **示例**：
  - `/users`：用户管理
  - `/groups`：群组管理
  - `/rooms`：聊天室管理
  - `/messages`：消息管理
  - `/messages/threads`：消息线程
  - `/messages/reactions`：消息表情回复
  - `/push`：推送通知
- **规则**：
  - 使用小写字母，用连字符分隔单词
  - 使用名词表示资源，避免使用动词
  - 通过 HTTP 方法表达操作意图

**示例：**

```http
# ❌ 错误：使用动词
GET /api/v1/getUsers
POST /api/v1/createUser
PUT /api/v1/updateUser/123
DELETE /api/v1/deleteUser/123

# ✅ 正确：使用名词 + HTTP 方法
GET /api/v1/users           # 获取用户列表
POST /api/v1/users          # 创建用户
PUT /api/v1/users/123       # 更新用户
DELETE /api/v1/users/123    # 删除用户
```

### 2.7 查询参数规范

- **格式**：`?param1=value1&param2=value2`
- **示例**：
  ```http
  GET /api/v1/users?status=active&page=1&limit=10&sort=created_at&order=desc
  ```
- **规则**：使用小写字母，用下划线分隔单词

## 3. 编码规范

### 3.1 字符分类

根据 RFC3986 第二章，URI 中的字符分为以下几类：

- **保留字符(Reserved Characters)**：在 URI 中具有特殊含义的字符，分为：

  - **通用分隔符(gen-delims)**：`:` `/` `?` `#` `[` `]` `@`
  - **子分隔符(sub-delims)**：`!` `$` `&` `'` `(` `)` `*` `+` `,` `;` `=`

- **非保留字符(Unreserved Characters)**：可以直接使用，无需编码的字符：

  - 大小写字母(A-Z, a-z)
  - 数字(0-9)
  - 连字符(`-`)
  - 点号(`.`)
  - 下划线(`_`)
  - 波浪号(`~`)

- **其他字符**：不在上述两类中的字符，包括非 ASCII 字符（如中文）和不在允许字符集中的字符，必须进行百分比编码。

### 3.2 百分比编码规则

- **编码格式**：`%xx`（xx 为字符对应 ASCII 码的十六进制值）
- **字符集**：UTF-8
- **编码时机**：
  - 当保留字符用作数据而非分隔符时
  - 当使用非 ASCII 字符时
  - 当使用不在 URI 允许字符集中的字符时
- **编码示例**：
  - 空格：`%20`（不应使用`+`，除非在 application/x-www-form-urlencoded 媒体类型中）
  - 中文"张"：`%E5%BC%A0`（UTF-8 编码）

[编码参考](https://developer.mozilla.org/zh-CN/docs/Glossary/Percent-encoding)

### 3.3 编码注意事项

1. **编码一致性**：

   - 非保留字符无需编码，即使被编码也应被视为等效
   - 保留字符用作分隔符时不应编码，用作数据时必须编码
   - 已编码的字符串不应再次编码（避免重复编码）

2. **编码示例**：

**路径参数**：

```
# 原始
/api/v1/users/张三

# 编码后
/api/v1/users/%E5%BC%A0%E4%B8%89

# ❌ 错误：重复编码
/api/v1/users/%25E5%25BC%25A0%25E4%25B8%2589
```

**查询参数**：

```
# 原始
?name=张三&type=技术

# 编码后
?name=%E5%BC%A0%E4%B8%89&type=%E6%8A%80%E6%9C%AF

# ❌ 错误：重复编码
?name=%25E5%25BC%25A0%25E4%25B8%2589&type=%25E6%25A0%2593%25E6%259C%25AF
```

3. **解码规则**：
   - 解码应在 URI 组件被解析和分离后进行
   - 百分比编码的非保留字符可以随时解码
   - 百分比编码的保留字符只有在确保不会被误解为分隔符时才能解码
   - 解码应只进行一次

## 4. 常见问题

### 4.1 编码问题

**问题：**

- 编码不一致
- 字符集不匹配
- 特殊字符处理
- 浏览器兼容性

**解决方案：**

- 统一使用 UTF-8
- 显式处理编码
- 避免特殊字符
- 充分测试验证

### 4.2 安全性问题

**问题：**

- 注入攻击
- 参数篡改
- 编码绕过
- 特殊字符攻击

**解决方案：**

- 参数验证
- 编码检查
- 安全过滤
- 使用 HTTPS

## 5. 工具支持

### 5.1 编码工具

**JavaScript：**

```javascript
// URL 编码
encodeURIComponent("张三"); // %E5%BC%A0%E4%B8%89

// URL 解码
decodeURIComponent("%E5%BC%A0%E4%B8%89"); // 张三
```

**Python：**

```python
# URL 编码
from urllib.parse import quote
quote('张三')  // %E5%BC%A0%E4%B8%89

# URL 解码
from urllib.parse import unquote
unquote('%E5%BC%A0%E4%B8%89')  // 张三
```
