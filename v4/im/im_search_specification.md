# IM 搜索规范

## 1. 设计理念

**核心原则**：统一、灵活、高效
**适用场景**：从简单过滤到复杂搜索的各种场景
**技术标准**：RSQL/FIQL 语法规范

## 2. 搜索规范

### 2.1 搜索模式概述

**设计理念**：提供两种搜索模式，满足从简单到复杂的不同场景需求
**核心标识**：`filter` 关键字作为启用专业语法的标记
**适用场景**：所有 GET 请求的数据查询和过滤

**模式分类：**

1. **简单搜索模式**：无 `filter` 关键字，使用直观的参数名查询
2. **复杂搜索模式**：使用 `filter` 关键字，启用完整 RSQL/FIQL 语法

### 2.2 简单搜索模式

**特征**：

- 不使用 `filter` 参数
- 直接使用字段名作为查询参数
- 语法简单，易于理解和使用

**支持能力**：

- 支持精确匹配
- 支持排除匹配
- 支持模糊匹配
- 支持多值匹配
- 支持多字段联合查询，字段间使用 `&` 连接，表示两个字段之间是 AND 关系 // todo: 不合理，建议使用 `;`
- 支持多字段联合查询，字段间使用 `|` 连接，表示两个字段之间是 OR 关系 // todo: 不合理，建议使用 `,`

**操作符规则**：

- **相等匹配**：`status==active`（简写形式可用 `status=active`）
- **排除匹配**：`status!=deleted`（使用 `!=` 表示不等于）
- **多值查询**：`status==active,pending`（等同于 `status=in=(active,pending)`）
- **模糊匹配**：`name==*zhang*`（使用 `*` 表示通配符）
- **联合与匹配**： // todo:
- **联合或匹配**： // todo:

**示例**：

```
GET /api/v1/users?status==active
GET /api/v1/users?status==active&age==25
GET /api/v1/users?status==active&age==25&city==beijing
GET /api/v1/users?status==active,pending&name==*zhang*
```

**使用限制**：

- 不支持范围查询（大于、小于等）
- 不支持 OR 逻辑组合
- 不支持复杂的括号分组
- 不支持嵌套字段查询

### 2.3 复杂搜索模式（可选）

**特征**：

- 必须使用 `filter` 参数
- 启用完整的 RSQL/FIQL 语法规范
- 支持所有高级搜索功能

**支持能力**：

- 支持所有操作符和逻辑组合
- 支持嵌套字段和关联查询
- 无字段数量限制
- 支持复杂的条件分组

**示例**：

```
GET /api/v1/users?filter=status==active;age=ge=18
GET /api/v1/users?filter=(status==active,status==pending);age=gt=18;age=lt=65
GET /api/v1/users?filter=profile.age=ge=18;address.city==beijing
```

**冲突处理**：

- **互斥原则**：不能同时使用两种模式
- **优先级**：如果同时存在 `filter` 和其他字段参数，`filter` 优先，其他参数被忽略

### 2.4 操作符规范

**适用范围**：仅用于复杂搜索模式（Filter）

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

**注意** 操作符 `=`,`!`,`*` 需要编码，[detail](https://developer.mozilla.org/zh-CN/docs/Glossary/Percent-encoding)

### 2.5 逻辑组合

**适用范围**：仅用于复杂搜索模式（Filter）

**AND 逻辑**：使用分号（`;`）分隔

```
GET /api/v1/users?filter=status==active;age=ge=18
```

**OR 逻辑**：使用逗号（`,`）分隔

```
GET /api/v1/users?filter=status==active,status==pending
```

**复杂组合**：使用圆括号（`()`）分组

_非必要不使用_

```
GET /api/v1/users?filter=(age=gt=18;age=lt=65);(status==active,status==pending)
```

### 2.6 性能优化

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

**简单搜索示例：**

```
# 单字段搜索
GET /api/v1/users?status==active

# 多字段搜索
GET /api/v1/users?status==active&city==beijing

# 多值查询
GET /api/v1/users?status==active,pending

# 模糊搜索
GET /api/v1/users?nickname==*zhang*
```

**复杂搜索示例：**

```
# 基础过滤
GET /api/v1/users?filter=status==active;is_online==true

# 高级搜索
GET /api/v1/users?filter=(age=ge=18;age=lt=65);(status==active,status==pending);city==beijing

# 嵌套字段搜索
GET /api/v1/users?filter=profile.age=ge=18;profile.verified==true
```

### 3.2 群组搜索

**简单搜索示例：**

```
# 公共群组
GET /api/v1/groups?is_public==true

# 按名称搜索
GET /api/v1/groups?name==*技术*
```

**复杂搜索示例：**

```
# 复杂条件过滤
GET /api/v1/groups?filter=is_public==true;member_count=ge=10

# 组合搜索
GET /api/v1/groups?filter=name==*技术*;description==*交流*;member_count=gt=5
```

### 3.3 消息搜索

**简单搜索示例：**

```
# 按会话类型搜索
GET /api/v1/messages?chat_type==single

# 按内容模糊搜索
GET /api/v1/messages?content==*会议*
```

**复杂搜索示例：**

```
# 精确过滤
GET /api/v1/messages?filter=chat_type==single;chat_id==user_123

# 时间范围搜索
GET /api/v1/messages?filter=content==*会议*;created_at=ge=2024-01-01;created_at=lt=2024-12-31

# 组合条件搜索
GET /api/v1/messages?filter=(chat_type==single,chat_type==group);content==*重要*
```

## 4. 最佳实践

### 4.1 限制搜索数量

建议在请求体里面添加 limit 或者 count 等字段，限制搜索数量， 降低服务器压力。

### 4.2 搜索结果标记

如果是分页或者游标搜索，那么可能需要多次请求和响应，建议添加 isFinished 等字段，表示搜索是否完成。

### 4.3 模糊搜索

- 使用标记表示这是模糊搜索 // todo: 如果是多个字段，指定那个字段是模糊搜索呢？
- 模糊搜索仅支持单字段，不能同时支持多个字段。例如：支持 `name` 和 `age` 两个字段模糊搜索

# 关键字命名问题

- search
- filter (推荐理由: sendbird/getsteam/tencent 都使用 filter)

名字选择是非常主观的，所以，我用 AI 统计了各个大厂的使用情况， [详见](./cursor_sort_vs_orderby_keyword_preferen.md)

# 参考文档

tentcent 采用哪种搜索规范？ [detail](https://cloud.tencent.com/document/product/1709/112947)

rongcloud 采用哪种搜索规范？[detail](https://docs.rongcloud.cn/not_existed)

rsql 实际项目？[detail](https://github.com/search?q=RSQL&type=repositories) // todo: java 757 star
