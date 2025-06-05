# 列举实际问题

1. 命名不一致
   messages 和 chatmessages

   ```
   POST https://{host}/{org_name}/{app_name}/messages/chatgroups
   GET https://{host}/{org_name}/{app_name}/chatmessages/{time}
   ```

2. URL结构不统一
   有些使用复数，有些使用单数

   ```
   GET https://{host}/{org_name}/{app_name}/user/{user_id}        # 单数
   GET https://{host}/{org_name}/{app_name}/users/{user_id}       # 复数
   ```

3. 动词形式的URL
   ```
   POST https://{host}/{org_name}/{app_name}/sendMessage
   GET https://{host}/{org_name}/{app_name}/getUsers
   ```
