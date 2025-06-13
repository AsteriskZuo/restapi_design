# IM 分页规范

## 1. 设计理念

**核心原则**：统一、灵活、高效
**适用场景**：大数据集分页查询
**技术标准**：支持偏移分页和游标分页

## 2. 分页规范

### 2.1 偏移分页

**参数说明：**

| 参数   | 说明     | 示例      | 适用场景   |
| ------ | -------- | --------- | ---------- |
| `page` | 页码     | `page=1`  | 常规分页   |
| `size` | 每页数量 | `size=20` | 控制返回量 |

**使用示例：**

```
GET /api/v1/users?page=1&size=20
```

**响应格式：**

```json
{
  "data": [...],
  "meta": {
    "pagination": {
      "total": 100,
      "page": 1,
      "size": 20,
      "pages": 5
    }
  }
}
```

### 2.2 游标分页

**参数说明：**

| 参数     | 说明     | 示例                      | 适用场景   |
| -------- | -------- | ------------------------- | ---------- |
| `cursor` | 游标     | `cursor=eyJpZCI6IjEyMyJ9` | 大数据集   |
| `limit`  | 限制数量 | `limit=20`                | 控制返回量 |

**使用示例：**

```
GET /api/v1/users?cursor=eyJpZCI6IjEyMyJ9&limit=20
```

**响应格式：**

```json
{
  "data": [...],
  "meta": {
    "pagination": {
      "next_cursor": "eyJpZCI6IjEyNCJ9",
      "has_more": true
    }
  }
}
```

### 2.3 性能优化

**优化策略：**

- 大数据集优先使用游标分页
- 控制单页数据量
- 避免深页查询
- 使用索引优化

**使用限制：**

- 最大页大小：100 条
- 最大页码：1000
- 游标有效期：5 分钟
- 分页深度：建议不超过 100 页

### 2.4 最佳实践

**选择建议：**

- 常规列表：使用偏移分页
- 实时数据：使用游标分页
- 大数据集：使用游标分页
- 需要总数：使用偏移分页

**实现建议：**

- 保持分页参数一致性
- 返回完整的分页信息
- 处理边界情况
- 考虑并发更新

## 3. IM 业务场景示例

### 3.1 用户列表

**偏移分页：**

```
GET /api/v1/users?page=1&size=20
```

**游标分页：**

```
GET /api/v1/users?cursor=eyJpZCI6IjEyMyJ9&limit=20
```

### 3.2 群组列表

**偏移分页：**

```
GET /api/v1/groups?page=1&size=20
```

**游标分页：**

```
GET /api/v1/groups?cursor=eyJpZCI6IjEyMyJ9&limit=20
```

### 3.3 消息列表

**偏移分页：**

```
GET /api/v1/messages?page=1&size=20
```

**游标分页：**

```
GET /api/v1/messages?cursor=eyJpZCI6IjEyMyJ9&limit=20
```
