# 文件介绍

1. 简单文件上传:
   1. 采用 json 格式，一次性发送到服务器
   2. 采用二进制格式，一次性发送到服务器
2. 分片上传: 采用二进制格式，多次发送到服务器
3. 简单文件下载:
   1. 使用 GET 方法，一次性下载到本地（指定保存地址）
   2. 使用 POST 方法，采用 json 格式（指定下载地址），一次性下载到本地
4. 断点续传下载: 采用多次请求，分片下载到本地

# 个人整理

1. telegram
   1. 大文件上传: 分片流式上传（客户端）: https://github.com/wildfirechat/android-chat/blob/master/client/src/main/java/cn/wildfirechat/client/ClientService.java#L4987
   2. 大文件下载: 断点续传下载（客户端）
   3. 没有 restapi
2. getstream
   1. 简单 json 上传: https://github.com/GetStream/stream-node/blob/91e1586590e54096ed94b566e5745ce493609ddf/src/gen/common/CommonApi.ts#L781
   2. 没有找到下载相关内容
3. rondcloud
   1. 源码太老，没有参考性，同时，也没有找到
4. wildfirechat
   1. 没有找到 restapi 相关内容
5. tencent
   1. 上传文件需要 url？ https://cloud.tencent.com/document/product/269/2720#.E6.96.87.E4.BB.B6.E6.B6.88.E6.81.AF.E5.85.83.E7.B4.A0
   2. ```原文
      通过服务端集成的 Rest API 接口发送文件消息时，需要填入文件的 Url、UUID、Download_Flag 字段。需保证通过该 Url 能下载到对应文件。UUID 字段需填写全局唯一的 String 值，一般填入文件的 MD5 值。消息接收者可以通过调用 V2TIMFileElem.getUUID()  拿到设置的 UUID 字段，业务 App 可以用这个字段做文件的区分。Download_Flag字段必须填2。
      ```
6. sendbird
   1. 文件上传: 分片上传（restapi）https://sendbird.com/docs/chat/platform-api/v3/message/messaging-basics/send-a-message
   2. 文件下载：普通下载

# 推荐

1. 发送文件
   1. ✅ 采用二进制格式，一次性发送到服务器
2. 下载文件
   1. ✅ 使用 POST 方法，采用 json 格式（指定下载地址），一次性下载到本地

# 参考资料

[telegram_github](https://github.com/telegramdesktop/tdesktop)
[telegram_github_docs](https://github.com/telegramdesktop/tdesktop/docs)
[telegram_files_docs](https://core.telegram.org/api/files)

[getstream_github](https://github.com/GetStream/stream-chat-js)
[getstream_github_docs](https://github.com/GetStream/stream-chat-js/docs)

[rondcloud_github](https://github.com/rongcloud/server-sdk-nodejs)
[rondcloud_github_docs](https://github.com/rongcloud/server-sdk-nodejs)

[wildfirechat_github](https://github.com/wildfirechat/im-server)
[wildfirechat_files_cn_not_found](https://docs.wildfirechat.cn/server/admin_api/message_api.html)

[tencent](https://cloud.tencent.com/document/product/269/2282)
