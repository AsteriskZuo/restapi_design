根据 [基本规则](./basic_design.md) 我的设计:

我认为 IM 合理的类别是：

- auth: 调用其他类别的前提。
- users: 用户管理。
- groups: 群组管理。
- rooms: 聊天室管理。
- messages: 消息管理。
- push: 推送通知管理。

我推荐的设计：

```
https://{host}/{version}/{org_name}/{app_name}/auth
https://{host}/{version}/{org_name}/{app_name}/users
https://{host}/{version}/{org_name}/{app_name}/groups
https://{host}/{version}/{org_name}/{app_name}/rooms
https://{host}/{version}/{org_name}/{app_name}/messages
https://{host}/{version}/{org_name}/{app_name}/push
```
