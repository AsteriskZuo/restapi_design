# REST API 搜索规范比较分析

## 概述

本文档详细比较了五种主流的 REST API 搜索和过滤规范：OData、RSQL/FIQL、RQL、JSON API 和 MongoDB 查询语法。通过对比分析，帮助开发者选择最适合的搜索规范。

## 官方参考资料

- **[OData](https://www.odata.org/)** - 最成熟和完整的标准，由 Microsoft 制定 ([规范文档](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html))
- **[RSQL/FIQL](https://github.com/jirutka/rsql-parser)** - 轻量级、URI 友好的查询语言 ([FIQL 规范](https://datatracker.ietf.org/doc/draft-nottingham-atompub-fiql/))
- **[RQL](https://github.com/persvr/rql)** - Resource Query Language，语法灵活 ([规范草案](https://dundalek.com/rql/draft-zyp-rql-00))
- **[JSON API](https://jsonapi.org/)** - 规范化的 API 设计标准 ([过滤规范](https://jsonapi.org/format/#fetching-filtering))
- **[MongoDB Query Language](https://docs.mongodb.com/manual/tutorial/query-documents/)** - NoSQL 查询语法 ([官方文档](https://docs.mongodb.com/manual/reference/operator/query/))

## 规范概览

| 规范      | 发起方    | 成熟度     | 复杂度 | 主要特点               |
| --------- | --------- | ---------- | ------ | ---------------------- |
| OData     | Microsoft | ⭐⭐⭐⭐⭐ | 高     | 功能完整、标准化程度高 |
| RSQL/FIQL | 社区      | ⭐⭐⭐⭐   | 中     | URI 友好、语法简洁     |
| RQL       | SitePen   | ⭐⭐⭐     | 中     | 灵活扩展、支持嵌套     |
| JSON API  | 社区      | ⭐⭐⭐⭐   | 低-中  | 规范化、结构清晰       |
| MongoDB   | MongoDB   | ⭐⭐⭐⭐⭐ | 中-高  | NoSQL 原生、表达力强   |

## 基本语法对比

### 语法结构对比

| 规范      | 基本结构                  | 操作符风格 | 逻辑连接符      |
| --------- | ------------------------- | ---------- | --------------- |
| OData     | `$filter=field op value`  | 空格分隔   | `and`, `or`     |
| RSQL/FIQL | `field=op=value`          | 等号包围   | `;`, `,`        |
| RQL       | `op(field,value)`         | 函数式     | `and()`, `or()` |
| JSON API  | `filter[field][op]=value` | 方括号嵌套 | `&` (隐式 AND)  |
| MongoDB   | `field[op]=value`         | 方括号包围 | `&` (隐式 AND)  |

### 比较操作符对比

| 操作     | OData        | RSQL/FIQL      | RQL          | JSON API  | MongoDB    |
| -------- | ------------ | -------------- | ------------ | --------- | ---------- |
| 等于     | `eq`         | `==`           | `eq()`       | `=`       | `=`        |
| 不等于   | `ne`         | `!=`           | `ne()`       | `[ne]=`   | `[ne]=`    |
| 大于     | `gt`         | `=gt=` 或 `>`  | `gt()`       | `[gt]=`   | `[gt]=`    |
| 大于等于 | `ge`         | `=ge=` 或 `>=` | `ge()`       | `[gte]=`  | `[gte]=`   |
| 小于     | `lt`         | `=lt=` 或 `<`  | `lt()`       | `[lt]=`   | `[lt]=`    |
| 小于等于 | `le`         | `=le=` 或 `<=` | `le()`       | `[lte]=`  | `[lte]=`   |
| 包含     | `in`         | `=in=`         | `in()`       | `[in]=`   | `[in]=`    |
| 模糊匹配 | `contains()` | `==*value*`    | `contains()` | `[like]=` | `[regex]=` |

### 语法示例对比

#### 1. 简单等值查询

```
# 查找状态为活跃的用户
OData:     GET /api/users?$filter=status eq 'active'
RSQL:      GET /api/users?filter=status==active
RQL:       GET /api/users?query=eq(status,active)
JSON API:  GET /api/users?filter[status]=active
MongoDB:   GET /api/users?status=active
```

#### 2. 范围查询

```
# 查找年龄在18-65之间的用户
OData:     GET /api/users?$filter=age ge 18 and age le 65
RSQL:      GET /api/users?filter=age=ge=18;age=le=65
RQL:       GET /api/users?query=and(ge(age,18),le(age,65))
JSON API:  GET /api/users?filter[age][gte]=18&filter[age][lte]=65
MongoDB:   GET /api/users?age[gte]=18&age[lte]=65
```

#### 3. 多条件 AND 查询

```
# 查找活跃的成年用户
OData:     GET /api/users?$filter=status eq 'active' and age ge 18
RSQL:      GET /api/users?filter=status==active;age=ge=18
RQL:       GET /api/users?query=and(eq(status,active),ge(age,18))
JSON API:  GET /api/users?filter[status]=active&filter[age][gte]=18
MongoDB:   GET /api/users?status=active&age[gte]=18
```

#### 4. OR 逻辑查询

```
# 查找VIP用户或管理员
OData:     GET /api/users?$filter=role eq 'vip' or role eq 'admin'
RSQL:      GET /api/users?filter=role==vip,role==admin
RQL:       GET /api/users?query=or(eq(role,vip),eq(role,admin))
JSON API:  GET /api/users?filter[role]=vip,admin (取决于实现)
MongoDB:   GET /api/users?role[in]=vip,admin
```

#### 5. 模糊匹配

```
# 查找姓名包含"John"的用户
OData:     GET /api/users?$filter=contains(name,'John')
RSQL:      GET /api/users?filter=name==*John*
RQL:       GET /api/users?query=contains(name,John)
JSON API:  GET /api/users?filter[name][like]=*John*
MongoDB:   GET /api/users?name[regex]=John
```

#### 6. 数组包含查询

```
# 查找拥有特定技能的用户
OData:     GET /api/users?$filter=skills/any(s: s eq 'JavaScript')
RSQL:      GET /api/users?filter=skills=in=(JavaScript,Python)
RQL:       GET /api/users?query=in(skills,JavaScript)
JSON API:  GET /api/users?filter[skills][in]=JavaScript,Python
MongoDB:   GET /api/users?skills[in]=JavaScript,Python
```

#### 7. 排序

```
# 按年龄升序，姓名降序排列
OData:     GET /api/users?$orderby=age asc, name desc
RSQL:      GET /api/users?sort=+age,-name
RQL:       GET /api/users?query=sort(+age,-name)
JSON API:  GET /api/users?sort=age,-name
MongoDB:   GET /api/users?sort=age:1,name:-1
```

#### 8. 分页

```
# 获取第2页，每页10条
OData:     GET /api/users?$skip=10&$top=10
RSQL:      GET /api/users?offset=10&limit=10
RQL:       GET /api/users?query=limit(10,10)
JSON API:  GET /api/users?page[number]=2&page[size]=10
MongoDB:   GET /api/users?skip=10&limit=10
```

### 复杂度分析

| 规范      | 语法复杂度 | 学习难度 | 表达能力 | URL 友好度 |
| --------- | ---------- | -------- | -------- | ---------- |
| OData     | 高         | 高       | 极强     | 中         |
| RSQL/FIQL | 低         | 低       | 中       | 高         |
| RQL       | 中         | 中       | 强       | 中         |
| JSON API  | 低         | 低       | 中       | 中         |
| MongoDB   | 中         | 中       | 强       | 高         |

### 特殊字符处理

| 规范      | 转义方式       | 特殊字符            | URL 编码需求 |
| --------- | -------------- | ------------------- | ------------ |
| OData     | 单引号包围     | `'`, `()`, 空格     | 必需         |
| RSQL/FIQL | 引号包围       | `=`, `;`, `,`, `()` | 可选         |
| RQL       | 引号或类型前缀 | `,`, `()`, 空格     | 部分需要     |
| JSON API  | URL 编码       | `[]`, `=`, `&`      | 必需         |
| MongoDB   | URL 编码       | `[]`, `=`, `&`      | 必需         |

## 详细比较

### 1. OData (Open Data Protocol)

#### 特点

- **标准化程度最高**：OASIS 官方标准，有完整的规范文档
- **功能最完整**：支持查询、排序、分页、聚合、函数等
- **生态系统成熟**：微软全力支持，工具链完善

#### 语法示例

```
# 基础过滤
GET /api/users?$filter=age gt 18

# 复杂查询
GET /api/users?$filter=startswith(name,'John') and age ge 21&$orderby=name desc&$top=10

# 函数支持
GET /api/users?$filter=contains(email,'@gmail.com') and year(birthDate) eq 1990

# 展开关联
GET /api/users?$expand=orders($filter=total gt 100)
```

#### 优点

- ✅ 功能最全面，支持复杂查询
- ✅ 标准化程度高，互操作性好
- ✅ 工具支持丰富（Visual Studio、Postman 等）
- ✅ 支持元数据发现
- ✅ 类型安全

#### 缺点

- ❌ 学习曲线陡峭
- ❌ 实现复杂度高
- ❌ 对于简单场景过于复杂
- ❌ URI 长度可能过长

#### 适用场景

- 企业级应用
- 复杂数据查询需求
- 微软技术栈
- 需要标准化的大型系统

---

### 2. RSQL/FIQL (Feed Item Query Language)

#### 特点

- **URI 友好**：无需 URL 编码，安全字符集
- **语法简洁**：易学易用
- **社区活跃**：Java、.NET、Node.js 等多语言支持

#### 语法示例

```
# 基础比较
GET /api/users?filter=name==John

# 范围查询
GET /api/users?filter=age=gt=18;age=lt=65

# 逻辑组合
GET /api/users?filter=name==John,name==Jane;status==active

# 包含查询
GET /api/users?filter=tags=in=(developer,manager)

# 模糊匹配
GET /api/users?filter=email==*@gmail.com
```

#### 优点

- ✅ 语法简洁直观
- ✅ URI 友好，无需编码
- ✅ 学习成本低
- ✅ 多语言支持好
- ✅ 性能开销小

#### 缺点

- ❌ 功能相对有限
- ❌ 不支持复杂函数
- ❌ 缺乏官方标准
- ❌ 元数据支持弱

#### 适用场景

- 中小型项目
- 快速开发
- 移动应用 API
- 简单到中等复杂度查询

---

### 3. RQL (Resource Query Language)

#### 特点

- **高度灵活**：支持自定义操作符
- **嵌套支持**：可以构建复杂的嵌套查询
- **语法清晰**：函数式语法风格

#### 语法示例

```
# 函数式语法
GET /api/users?query=eq(name,John)

# 嵌套查询
GET /api/users?query=and(gt(age,18),lt(age,65))

# 排序和分页
GET /api/users?query=sort(+name,-age)&limit(0,10)

# 聚合查询
GET /api/users?query=aggregate(department,sum(salary))

# 组合查询
GET /api/users?query=or(eq(status,active),and(eq(status,pending),gt(createdAt,date:2023-01-01)))
```

#### 优点

- ✅ 灵活性最高
- ✅ 支持复杂嵌套
- ✅ 可扩展性强
- ✅ 表达能力强

#### 缺点

- ❌ 语法相对复杂
- ❌ 社区支持有限
- ❌ 标准化程度低
- ❌ 学习成本较高

#### 适用场景

- 需要高度定制的查询
- 复杂数据分析
- 灵活性要求高的系统
- 有充足开发资源的项目

---

### 4. JSON API

#### 特点

- **规范化设计**：统一的 API 设计规范
- **结构清晰**：明确的参数命名规则
- **社区驱动**：活跃的开源社区

#### 语法示例

```
# 基础过滤
GET /api/users?filter[age][gt]=18

# 多条件过滤
GET /api/users?filter[name]=John&filter[status]=active

# 排序
GET /api/users?sort=name,-age

# 分页
GET /api/users?page[number]=1&page[size]=10

# 字段选择
GET /api/users?fields[user]=name,email&fields[profile]=avatar
```

#### 优点

- ✅ 结构化程度高
- ✅ 易于理解和实现
- ✅ 社区支持好
- ✅ 与 JSON API 规范整体一致

#### 缺点

- ❌ 查询能力相对有限
- ❌ URI 可能较长
- ❌ 复杂查询支持不足
- ❌ 灵活性有限

#### 适用场景

- 标准化要求高的项目
- 团队协作项目
- RESTful API 设计
- 中等复杂度的查询需求

---

### 5. MongoDB 查询语法

#### 特点

- **NoSQL 原生**：来自最流行的 NoSQL 数据库
- **表达力强**：丰富的操作符和查询能力
- **REST 适配**：广泛被 REST API 借鉴的语法风格
- **社区庞大**：MongoDB 生态系统成熟

#### 语法示例

```
# MongoDB 原生查询
db.users.find({ age: { $gte: 18, $lt: 65 }, status: "active" })

# REST API 中的 MongoDB 风格
GET /api/users?age[gte]=18&age[lt]=65&status=active

# 复杂查询
GET /api/users?$or=[{"age[gte]": 18}, {"status": "vip"}]&name[regex]=^John

# 数组查询
GET /api/users?tags[in]=developer,manager&skills[exists]=true

# 嵌套字段查询
GET /api/users?profile.age[gte]=18&address.city=beijing
```

#### 优点

- ✅ 表达能力强，支持复杂查询
- ✅ MongoDB 开发者熟悉度高
- ✅ 支持丰富的数据类型和操作
- ✅ 社区生态成熟
- ✅ 适合 NoSQL 数据结构

#### 缺点

- ❌ 语法相对复杂
- ❌ 不是正式的 REST 标准
- ❌ URL 编码复杂查询困难
- ❌ 学习成本较高

#### 适用场景

- 使用 MongoDB 作为主数据库
- NoSQL 数据结构的项目
- 需要复杂查询能力
- 团队熟悉 MongoDB 的项目

## 实际案例对比

### 查询需求：查找年龄在 18-65 岁之间的活跃用户，按姓名排序

#### OData

```
GET /api/users?$filter=age ge 18 and age le 65 and status eq 'active'&$orderby=name asc
```

#### RSQL/FIQL

```
GET /api/users?filter=age=ge=18;age=le=65;status==active&sort=+name
```

#### RQL

```
GET /api/users?query=and(ge(age,18),le(age,65),eq(status,active))&sort(+name)
```

#### JSON API

```
GET /api/users?filter[age][gte]=18&filter[age][lte]=65&filter[status]=active&sort=name
```

#### MongoDB 风格

```
GET /api/users?age[gte]=18&age[lte]=65&status=active&sort=name:1
```

## 性能对比

| 规范      | 解析复杂度 | 内存占用 | 执行效率 | 缓存友好度 |
| --------- | ---------- | -------- | -------- | ---------- |
| OData     | 高         | 高       | 中       | 低         |
| RSQL/FIQL | 中         | 低       | 高       | 高         |
| RQL       | 中         | 中       | 中       | 中         |
| JSON API  | 低         | 低       | 高       | 高         |
| MongoDB   | 中         | 中       | 高       | 中         |

## 选择建议

### 选择 OData 如果：

- 构建企业级应用
- 需要复杂查询功能
- 使用微软技术栈
- 对标准化要求很高

### 选择 RSQL/FIQL 如果：

- 追求简洁性和性能
- 快速开发和部署
- 中小型项目
- 需要多语言支持

### 选择 RQL 如果：

- 需要高度定制化
- 复杂嵌套查询
- 有充足的开发资源
- 灵活性是首要考虑

### 选择 JSON API 如果：

- 遵循 JSON API 规范
- 注重结构化和一致性
- 团队协作项目
- 中等复杂度需求

### 选择 MongoDB 风格如果：

- 使用 MongoDB 或其他 NoSQL 数据库
- 团队熟悉 MongoDB 查询语法
- 需要表达复杂的数据结构查询
- 希望保持与数据库查询的一致性

## 混合方案

在实际项目中，可以考虑混合使用多种规范：

```javascript
// 基础查询使用简单的键值对
GET /api/users?status=active&city=beijing

// 范围查询使用 MongoDB 风格
GET /api/users?age[gte]=18&age[lt]=65&status=active

// 复杂查询使用 RSQL
GET /api/users?filter=age=gt=18;age=lt=65;(status==active,status==pending)

// 高级功能使用 OData 语法
GET /api/users?$filter=contains(tags,'developer')&$expand=profile
```

## 实现建议

1. **选择核心规范**：根据项目需求选择一个主要规范
2. **逐步增强**：从简单语法开始，逐步添加复杂功能
3. **提供文档**：详细的 API 文档和示例
4. **考虑向后兼容**：保持 API 的向后兼容性
5. **性能监控**：监控查询性能，优化热点查询

## 总结

每种搜索规范都有其特定的优势和适用场景。选择时应考虑：

- **项目复杂度**：简单项目选择 RSQL/FIQL，复杂项目选择 OData
- **数据库类型**：MongoDB 项目优先考虑 MongoDB 风格
- **团队能力**：考虑团队的学习成本和维护能力
- **性能要求**：高性能场景优先考虑 RSQL/FIQL 或 MongoDB 风格
- **标准化需求**：企业级应用优先考虑 OData 或 JSON API
- **扩展性要求**：需要高度定制时选择 RQL

最终目标是在功能性、性能、可维护性和开发效率之间找到最佳平衡点。
