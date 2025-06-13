# IM 搜索规范

## 1. 设计理念

**核心原则**：统一、灵活、高效
**适用场景**：从简单过滤到复杂搜索的各种场景
**技术标准**：RSQL/FIQL 语法规范

## 2. 搜索规范

### 2.1 基础分类

**用途**：数据查询和过滤
**场景**：GET 请求
**限制**：避免敏感信息、控制查询复杂度

**搜索类型：**

1. **基础搜索（简单过滤）**

   - 精确匹配：`status==active`
   - 范围过滤：`age=ge=18;age=lt=65`
   - 多值过滤：`status=in=(active,pending)`
   - 布尔过滤：`is_online==true`

2. **高级搜索（复杂条件）**

   - 组合条件：`(age=gt=18;age=lt=65);(status==active,status==pending)`
   - 嵌套字段：`profile.age=ge=18;address.city==beijing`
   - 关联查询：`group.member_count=gt=10`

3. **全文搜索**

   - 关键词搜索：`q=zhang`
   - 模糊匹配：`name==*zhang*;email==*@gmail.com`
   - 多字段搜索：`(name==*zhang*,nickname==*zhang*)`

4. **语义搜索**
   - 相似度搜索：`description~=技术交流`
   - 智能推荐：`recommend=true`

### 2.2 操作符规范

**主要操作符：**

| 操作符  | 说明     | 示例                          | 适用场景 |
| ------- | -------- | ----------------------------- | -------- |
| `==`    | 等于     | `status==active`              | 精确匹配 |
| `!=`    | 不等于   | `status!=deleted`             | 排除匹配 |
| `=gt=`  | 大于     | `age=gt=18`                   | 范围过滤 |
| `=ge=`  | 大于等于 | `age=ge=18`                   | 范围过滤 |
| `=lt=`  | 小于     | `age=lt=65`                   | 范围过滤 |
| `=le=`  | 小于等于 | `age=le=65`                   | 范围过滤 |
| `=in=`  | 包含     | `status=in=(active,pending)`  | 多值过滤 |
| `=out=` | 不包含   | `status=out=(deleted,banned)` | 多值排除 |
| `==*`   | 模糊匹配 | `name==*zhang*`               | 模糊搜索 |
| `~=`    | 相似度   | `description~=技术交流`       | 语义搜索 |

### 2.3 逻辑组合

**AND 逻辑**：使用分号（`;`）分隔

```
GET /api/v1/users?filter=status==active;age=ge=18
```

**OR 逻辑**：使用逗号（`,`）分隔

```
GET /api/v1/users?filter=status==active,status==pending
```

**复杂组合**：使用圆括号（`()`）分组

```
GET /api/v1/users?filter=(age=gt=18;age=lt=65);(status==active,status==pending)
```

### 2.4 性能优化

**查询优化：**

- **索引字段**：优先使用已建立索引的字段
- **查询限制**：控制查询条件的复杂度
- **结果限制**：使用分页控制返回数量
- **字段选择**：只返回需要的字段

**使用限制：**

- **查询长度**：单个查询字符串不超过 1024 字符
- **条件数量**：建议不超过 10 个条件
- **嵌套深度**：建议不超过 3 层
- **特殊字符**：需要进行 URL 编码

## 3. IM 业务场景示例

### 3.1 用户搜索

**基础过滤：**

```
GET /api/v1/users?filter=status==active;is_online==true
```

**高级搜索：**

```
GET /api/v1/users?filter=(age=ge=18;age=lt=65);(status==active,status==pending);city==beijing
```

**全文搜索：**

```
GET /api/v1/users?q=zhang&fields=id,name,email
```

### 3.2 群组搜索

**群组过滤：**

```
GET /api/v1/groups?filter=is_public==true;member_count=ge=10
```

**群组搜索：**

```
GET /api/v1/groups?filter=name==*技术*;description==*交流*
```

### 3.3 消息搜索

**消息过滤：**

```
GET /api/v1/messages?filter=chat_type==single;chat_id==user_123
```

**消息搜索：**

```
GET /api/v1/messages?filter=content==*会议*;created_at=ge=2024-01-01
```
