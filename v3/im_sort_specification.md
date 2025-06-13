# IM 排序规范

## 1. 设计理念

**核心原则**：统一、灵活、高效
**适用场景**：结果集排序
**技术标准**：基于 sort 参数的标准排序语法

## 2. 排序规范

### 2.1 基础语法

**参数名**：`sort`
**格式**：`字段名:排序方向`
**方向**：`asc`（升序）或 `desc`（降序）

### 2.2 排序类型

1. **单字段排序**

   ```
   GET /api/v1/users?sort=created_at:desc
   ```

2. **多字段排序**

   ```
   GET /api/v1/users?sort=status:asc,created_at:desc
   ```

3. **特殊排序**
   - **相关度排序**：`sort=relevance`
   - **随机排序**：`sort=random`
   - **自定义排序**：`sort=custom_field:asc`

### 2.3 排序规则

**默认规则：**

- 未指定排序时，默认按主键降序
- 多字段排序时，按字段顺序优先级排序
- 空值处理：统一放在最后

**特殊处理：**

- 中文排序：使用拼音或笔画
- 时间排序：统一使用 UTC 时间
- 数值排序：考虑精度和单位

### 2.4 性能优化

**优化策略：**

- 优先使用索引字段排序
- 避免对非索引字段排序
- 控制排序字段数量
- 大数据量时使用游标分页

**使用限制：**

- 排序字段数：建议不超过 3 个
- 排序方向：必须明确指定
- 特殊字符：需要进行 URL 编码

## 3. IM 业务场景示例

### 3.1 用户排序

**基础排序：**

```
GET /api/v1/users?sort=created_at:desc
```

**多字段排序：**

```
GET /api/v1/users?sort=status:asc,last_login:desc
```

### 3.2 群组排序

**成员数排序：**

```
GET /api/v1/groups?sort=member_count:desc
```

**活跃度排序：**

```
GET /api/v1/groups?sort=activity:desc,created_at:desc
```

### 3.3 消息排序

**时间排序：**

```
GET /api/v1/messages?sort=created_at:desc
```

**优先级排序：**

```
GET /api/v1/messages?sort=priority:desc,created_at:desc
```
