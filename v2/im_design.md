根据 [基本规则](./basic_design.md) 我的设计:

我认为 IM 合理的类别是：

- auth: 调用其他类别的前提。
- users: 用户管理。
- groups: 群组管理。
- rooms: 聊天室管理。
- messages: 消息管理。
- push: 推送通知管理。

我设计的 URL：

```
https://{host}/{org_name}/{app_name}/{version}/auth
https://{host}/{org_name}/{app_name}/{version}/users
https://{host}/{org_name}/{app_name}/{version}/groups
https://{host}/{org_name}/{app_name}/{version}/rooms
https://{host}/{org_name}/{app_name}/{version}/messages
https://{host}/{org_name}/{app_name}/{version}/push
```
