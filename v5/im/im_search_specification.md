# IM 搜索规范

## 1. 搜索规范

### 1.1 搜索模式概述

**设计理念**：提供两种搜索模式，满足从简单到复杂的不同场景需求
**核心标识**：`filter` 关键字作为启用专业语法的标记
**适用场景**：所有 GET 请求的数据查询和过滤

**模式分类：**

1. **简单搜索模式**：无 `filter` 关键字，使用直观的参数名查询，不支持 大于（大于等于）、小于（小于等于）、包含、不包含、嵌套等复杂操作
2. **复杂搜索模式**：使用 `filter` 关键字，启用完整 RSQL/FIQL 语法

### 1.2 简单搜索模式

**特征**：

- 不使用 `filter` 参数
- 直接使用字段名作为查询参数
- 语法简单，易于理解和使用

**支持能力**：

- 支持精确匹配
- 支持多值匹配
- 支持多字段联合查询

**操作符规则**：

- **相等匹配**：`status=active`
- **多值查询**：`status=active,pending`
- **联合查询**：使用符号 `&` 间隔

**示例**：

```http
# 单值搜索（简单搜索模式）
GET /api/v1/users?status=active
GET /api/v1/messages?chat_type=single

# 多值搜索（简单搜索模式）
GET /api/v1/users?status=active,pending
GET /api/v1/messages?chat_type=single,group

# 搜索+排序
GET /api/v1/users?status=active&sort=created_at:desc
GET /api/v1/groups?filter=is_public==true;member_count=ge=10&sort=activity:desc,created_at:desc
GET /api/v1/messages?chat_id=123&sort=created_at:desc
```

**使用限制**：

- 不支持范围查询（大于、小于等）
- 不支持排除匹配（!=）
- 不支持模糊匹配（\*）
- 不支持 OR 逻辑组合
- 不支持复杂的括号分组
- 不支持嵌套字段查询

### 1.3 复杂搜索模式（可选）

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

### 1.4 操作符规范

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

**注意** 操作符 `=`,`!`,`*` 需要编码，[detail](https://developer.mozilla.org/zh-CN/docs/Glossary/Percent-encoding)

### 1.5 逻辑组合

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

### 1.6 模糊搜索

支持 `name==*zhang`,`name==*zhang*`, `name==zhang*`, `name==zha*g` 等形式的模糊搜索。

## 2. 最佳实践

推荐 优先实现 简单搜索，根据业务需求等因素，再逐步实现复杂搜索，以提高用户体验和服务器性能。

### 2.1 限制搜索结果数量

综合导出的优缺点，建议在请求中添加 limit 或者 count 等类似字段，限制搜索数量。

例如：limit 通过请求体或者查询参数的方式来限制返回最大数量。count 通过响应体的方式来说明返回结果实际数量。

### 2.2 搜索结果标记

如果是分页或者游标搜索，那么可能需要多次请求和响应，建议响应体添加 `isFinished` 等类似字段，表示搜索是否完成。

### 2.3 模糊搜索 (暂不实现)

- 使用额外标记表示这是模糊搜索
- 使用搜索关键字添加 `*`符号表示模糊搜索
