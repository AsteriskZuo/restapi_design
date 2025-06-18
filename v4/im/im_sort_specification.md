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

- 未指定排序时，由业务场景等因素决定升序还是降序
- 多字段排序时，按字段顺序优先级排序

### 2.4 性能优化

**优化策略：**

- 控制排序字段数量
- 大数据量时使用游标分页

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

## 4.最佳实践

### 4.1 未指定排序情况

当未指定排序字段时，服务接口应该提供默认排序。例如：升序或者降序。

排序字段的默认排序方向应该遵循业务场景的需求。

### 4.2 排序字段数量要求

当排序字段数量超过 n 个时，会影响服务性能，建议控制排序字段数量。

排序字段数量应该有业务场景和技术等因素来决定。

### 4.3 支持游标控制搜索结果

当搜索结果多于客户端需求时，可以使用游标更加精确的控制搜索结果。

// todo: 示例：搜索年龄 18 的用户，人数超过 100 个时，使用游标控制搜索结果。

### 4.4 排序字段的选择

建议选择时间戳等字段，减少服务压力，提高响应速度。

# 关键字命名问题

- sort (推荐, 搜索的关键字 filter、 分页关键字 page、cursor、都没有使用 by，这样可以保持一致的风格)
- sort_by
- order_by

**名字选择是非常主观的，所以，我用 AI 统计了各个大厂的使用情况。**
