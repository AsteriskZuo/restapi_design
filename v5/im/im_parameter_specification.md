# IM 参数规范

## 1. 参数规范

1. **路径参数**

   - 用于标识资源
   - 示例：`/api/v1/users/{userId}`

2. **查询参数**

   - 用于过滤、排序、分页
   - 示例：`?status=active&sort=createdAt:desc`

3. **请求体参数**

   - 用于复杂数据传递
   - 示例：JSON 格式的请求体

4. **请求头参数**

   - 用于认证、控制
   - 示例：`authorization: Bearer token`
