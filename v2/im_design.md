根据 [基本规则](./basic_design.md) 我的设计:

我认为 IM 合理的类别是：

- auth: 调用其他类别的前提。
- users
- groups
- rooms
- messages
- push

我设计的 URL：

```
https://{host}/{org_name}/{app_name}/{version}/auth
https://{host}/{org_name}/{app_name}/{version}/users
https://{host}/{org_name}/{app_name}/{version}/groups
https://{host}/{org_name}/{app_name}/{version}/rooms
https://{host}/{org_name}/{app_name}/{version}/messages
https://{host}/{org_name}/{app_name}/{version}/push
```
