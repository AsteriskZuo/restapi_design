# 环信IM REST API 文档

本目录包含环信IM客户端SDK中所有REST API调用的详细文档。

## 文档结构

### 1. [用户管理](./user_management.md)
- 用户登录认证
- 用户加密信息获取
- 用户属性管理
- 已登录设备管理

### 2. [消息管理](./message_management.md)
- 消息发送与接收
- 消息历史记录
- 群组消息已读回执
- 消息翻译
- 消息删除

### 3. [群组管理](./group_management.md)
- 群组信息获取
- 群组成员管理
- 群组黑名单管理
- 群组禁言管理
- 群组白名单管理
- 群组公告管理
- 群组文件管理

### 4. [聊天室管理](./chatroom_management.md)
- 聊天室信息获取
- 聊天室成员管理
- 聊天室元数据管理

### 5. [线程管理](./thread_management.md)
- 子区（Thread）创建
- 子区成员管理
- 子区信息获取

### 6. [用户状态管理](./presence_management.md)
- 用户在线状态发布
- 用户在线状态订阅
- 订阅列表管理

### 7. [文件上传](./file_upload.md)
- 文件分片上传
- 文件上传任务管理

### 8. [联系人管理](./contact_management.md)
- 联系人列表获取
- 黑名单管理
- 好友请求处理

## API 调用规范

### 通用Headers
```
Authorization: Bearer {token}
Content-Type: application/json
```

### 基础URL
```
{rest_base_url}/{org_name}/{app_name}
```

### 错误码
- 200-299: 成功
- 401: 认证失败
- 404: 资源不存在
- 429: 请求频率超限
- 500: 服务器内部错误

### 分页参数
- pageNum: 页码（从1开始）
- pageSize: 页大小
- cursor: 游标（用于游标分页）
- limit: 限制数量 