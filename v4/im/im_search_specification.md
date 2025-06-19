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

1. **简单搜索模式**：无 `filter` 关键字，使用直观的参数名查询，不支持 大于（大于等于）、小于（小于等于）、包含、不包含、嵌套等复杂操作
2. **复杂搜索模式**：使用 `filter` 关键字，启用完整 RSQL/FIQL 语法

### 2.2 简单搜索模式

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

```
GET /api/v1/users?status=active
GET /api/v1/users?status=active&age=25
GET /api/v1/users?status=active&age=25&city=beijing
GET /api/v1/users?status=active,pending
```

**使用限制**：

- 不支持范围查询（大于、小于等）
- 不支持排除匹配（!=）
- 不支持模糊匹配（\*）
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

**注意** 操作符 `=`,`!`,`*` 需要编码，[detail](https://developer.mozilla.org/zh-CN/docs/Glossary/Percent-encoding)

[RSQL/FIQL 语法规范](https://github.com/imsys/IM-Specification/blob/master/IM_Search_Specification.md#rsqlfiql-语法规范)

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

### 2.6 模糊搜索

支持 `name==*zhang`,`name==*zhang*`, `name==zhang*`, `name==zha*g` 等形式的模糊搜索。

## 3. IM 业务场景示例

### 3.1 用户搜索

**简单搜索示例：**

```
# 单字段搜索
GET /api/v1/users?status=active

# 多字段搜索
GET /api/v1/users?status=active&city=beijing

# 多值查询
GET /api/v1/users?status=active,pending
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
GET /api/v1/groups?is_public=true

# 按名称搜索
GET /api/v1/groups?name=技术组
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
GET /api/v1/messages?chat_type=single

# 按内容搜索
GET /api/v1/messages?content=会议内容
```

**复杂搜索示例：**

```
# 精确过滤
GET /api/v1/messages?filter=chat_type==single;chat_id==user_123
# 编码后的URL
GET /api/v1/messages?filter=chat_type%3D%3Dsingle%3Bchat_id%3D%3Duser_123

# 时间范围搜索
GET /api/v1/messages?filter=content==*会议*;created_at=ge=2024-01-01;created_at=lt=2024-12-31
# 编码后的URL
/api/v1/messages?filter=content%3D%3D%2A%E4%BC%9A%E8%AE%AE%2A%3Bcreated_at%3Dge%3D2024-01-01%3Bcreated_at%3Dlt%3D2024-12-31

# 组合条件搜索
GET /api/v1/messages?filter=(chat_type==single,chat_type==group);content==*重要*
# 编码后的URL
GET /api/v1/messages?filter=(chat_type%3D%3Dsingle%2Cchat_type%3D%3Dgroup)%3Bcontent%3D%3D%2A%E9%87%8D%E8%A6%81%2A
```

## 4. 最佳实践

推荐 优先实现 简单搜索，根据业务需求等因素，再逐步实现复杂搜索，以提高用户体验和服务器性能。

### 4.1 限制搜索结果数量

综合导出的优缺点，建议在请求中添加 limit 或者 count 等类似字段，限制搜索数量。

优点如下:

1. **系统稳定性**

   - 防止单个请求消耗过多资源
   - 避免因大量数据处理导致系统响应变慢
   - 降低系统崩溃风险

2. **错误处理友好**

   - 失败后只需重试失败的部分
   - 重试代价小，用户体验好
   - 支持断点续传

3. **进度可控**
   - 用户可以看到处理进度
   - 可以及时反馈结果
   - 支持分批次处理

缺点如下:

1. **实现复杂度增加**

   - 需要实现分页或游标机制
   - 需要处理数据一致性问题（分页过程中数据变化）

2. **使用限制**
   - 无法一次性获取全量数据
   - 需要多次请求才能获取完整结果

根据不同业务场景自行决定，如果需要分页搜索，那么参考[分页章节](./im_pagination_specification.md)

### 4.2 搜索结果标记

如果是分页或者游标搜索，那么可能需要多次请求和响应，建议响应体添加 `isFinished` 等类似字段，表示搜索是否完成。

### 4.3 模糊搜索 (暂不实现)

- 使用额外标记表示这是模糊搜索
- 使用搜索关键字添加 `*`符号表示模糊搜索

# 关键字命名问题

- search
- filter (推荐理由: sendbird/getsteam/tencent 都使用 filter)

名字选择是非常主观的，所以，我用 AI 统计了各个大厂的使用情况， [详见](./cursor_sort_vs_orderby_keyword_preferen.md)

# 参考文档

tentcent 采用哪种搜索规范？ [detail](https://cloud.tencent.com/document/product/1709/112947)

rongcloud 采用哪种搜索规范？[detail](https://docs.rongcloud.cn/not_existed)

rsql 实际项目？[detail](https://github.com/search?q=RSQL&type=repositories) // java 757 star

RSQL 规范？ [detail](https://www.here.com/docs/bundle/data-client-library-developer-guide-java-scala/page/client/rsql.html)

# RSQL 语法解析器示例

## java

## javascript

## python
