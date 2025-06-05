# URL 设计规范

## 基本原则

### 1. 资源导向设计

URL应该表示资源，而不是操作。

```
✅ 正确：
GET /api/v1/users/123
POST /api/v1/users
PUT /api/v1/users/123

❌ 错误：
GET /api/v1/getUser?id=123
POST /api/v1/createUser
PUT /api/v1/updateUser?id=123
```

### 2. 使用名词而不是动词

```
✅ 正确：
/users
/posts
/comments

❌ 错误：
/getUsers
/createPost
/deleteComment
```

### 3. 使用复数形式

```
✅ 正确：
/users        (获取所有用户)
/users/123    (获取特定用户)

❌ 错误：
/user
/user/123
```

## URL 结构规范

### 基本结构

```
https://{domain}/api/{version}/{resource}/{id}/{sub-resource}/{sub-id}
```

**示例：**
```
https://api.example.com/api/v1/users/123/posts/456
```

### 层级关系

正确表达资源之间的层级关系：

```
✅ 正确的层级设计：
/users/123/posts           # 用户123的所有文章
/users/123/posts/456       # 用户123的文章456
/posts/456/comments        # 文章456的所有评论
/posts/456/comments/789    # 文章456的评论789

❌ 错误的层级设计：
/user123/add_post          # 动词形式，参数嵌入
/get_user_posts/123        # 动词开头
/123/456/comments          # 缺少资源名称
```

## 解决参数为空的问题

### 问题案例分析

你提到的问题URL：
```
https://xxx.domain.com/user1/add_contact/user2
```

**问题：**
1. 当 user1 或 user2 为空时，URL变成了：
   - `https://xxx.domain.com//add_contact/user2`
   - `https://xxx.domain.com/user1/add_contact/`
2. 这会导致路由匹配失败和语义不清

### 正确的解决方案

#### 方案1：使用请求体传递参数（推荐）

```
POST /api/v1/users/{userId}/contacts
Content-Type: application/json

{
  "contactUserId": "user2",
  "relationship": "friend",
  "note": "通过XX认识"
}
```

**优点：**
- 避免URL中的空参数问题
- 支持复杂的数据结构
- 易于扩展
- 参数验证清晰

#### 方案2：使用查询参数

```
POST /api/v1/contacts?userId=user1&contactUserId=user2
```

**适用场景：**
- 参数较少
- 参数都是简单类型
- 需要支持GET方法

#### 方案3：RESTful资源设计

```
POST /api/v1/users/{userId}/contacts
```

将"添加联系人"理解为"在用户的联系人列表中创建一个新项目"。

## 参数传递最佳实践

### 1. 路径参数 (Path Parameters)

用于标识特定资源：

```
/users/{userId}           # 用户ID
/posts/{postId}           # 文章ID
/users/{userId}/posts     # 特定用户的文章
```

**规则：**
- 必填参数
- 用于资源标识
- 不能为空

### 2. 查询参数 (Query Parameters)

用于过滤、排序、分页等：

```
/users?page=1&limit=10&sort=createdAt&order=desc
/posts?category=tech&status=published&author=123
```

**规则：**
- 可选参数
- 用于修改返回结果
- 可以为空（使用默认值）

### 3. 请求体参数 (Body Parameters)

用于创建或更新资源：

```
POST /api/v1/users
{
  "name": "张三",
  "email": "zhangsan@example.com"
}
```

## 特殊情况处理

### 1. 搜索接口

```
✅ 推荐：
GET /api/v1/search/users?q=张三&type=name
GET /api/v1/users/search?q=张三

❌ 不推荐：
GET /api/v1/searchUsers?query=张三
```

### 2. 批量操作

```
✅ 推荐：
POST /api/v1/users/batch
{
  "action": "delete",
  "userIds": [1, 2, 3]
}

POST /api/v1/users:batchDelete
{
  "userIds": [1, 2, 3]
}

❌ 不推荐：
DELETE /api/v1/users/1,2,3
```

### 3. 复杂查询

```
✅ 推荐：
POST /api/v1/users/search
{
  "filters": {
    "age": {"min": 18, "max": 65},
    "city": ["北京", "上海"],
    "skills": ["Java", "Python"]
  },
  "sort": [{"field": "createdAt", "order": "desc"}],
  "pagination": {"page": 1, "limit": 20}
}
```

## 命名约定

### 1. 大小写规则

```
✅ 正确：
/api/v1/user-profiles      # kebab-case（推荐）
/api/v1/userprofiles       # 全小写
/api/v1/user_profiles      # snake_case

❌ 错误：
/api/v1/UserProfiles       # PascalCase
/api/v1/userProfiles       # camelCase
```

### 2. 资源命名

```
✅ 正确：
/users                     # 简单名词复数
/user-profiles            # 复合词用连字符
/api-keys                 # 缩写词用连字符

❌ 错误：
/Users                    # 大写
/user_profiles            # 下划线
/apiKeys                  # 驼峰
```

### 3. 版本号

```
✅ 推荐方式：
/api/v1/users             # URL路径版本
/api/v2/users             # 新版本

可选方式：
Accept: application/vnd.api+json;version=1
X-API-Version: 1
```

## 长度和复杂度限制

### URL长度限制

- 总长度不超过2048字符
- 路径段不超过255字符
- 查询字符串不超过1024字符

### 嵌套深度限制

```
✅ 推荐（2-3层）：
/users/123/posts
/users/123/posts/456/comments

❌ 过深（避免超过4层）：
/companies/123/departments/456/teams/789/members/101/skills
```

## 错误案例和改进

### 案例1：原始设计问题

```
❌ 问题设计：
https://xxx.domain.com/user1/add_contact/user2

问题：
1. 动词形式 "add_contact"
2. 参数直接嵌入路径
3. 空参数导致URL破坏
4. 难以扩展
```

```
✅ 改进方案：
POST /api/v1/users/{userId}/contacts
{
  "contactUserId": "user2",
  "metadata": {}
}

优点：
1. 符合REST规范
2. 避免空参数问题
3. 易于扩展
4. 语义清晰
```

### 案例2：其他常见问题

```
❌ 问题：
GET /api/getUserInfo?userId=&type=basic
# userId为空，导致逻辑错误

✅ 改进：
GET /api/v1/users/{userId}?fields=basic
# 必填参数在路径中，可选参数在查询中
```

## 工具和验证

### URL验证清单

- [ ] 使用名词而不是动词
- [ ] 使用复数形式
- [ ] 使用小写字母和连字符
- [ ] 避免文件扩展名
- [ ] 必填参数在路径中
- [ ] 可选参数在查询中或请求体中
- [ ] 处理空参数的情况
- [ ] 符合版本控制规范
- [ ] 嵌套层级合理（不超过4层）
- [ ] URL长度在限制范围内

### 常用工具

1. **URL设计验证工具**
   - Postman
   - Swagger/OpenAPI
   - REST Client扩展

2. **自动化测试**
   - 参数验证测试
   - 边界条件测试
   - 空值处理测试 