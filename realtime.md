# 实时通信服务开发指南

除了实时的消息推送服务外，LeanCloud 从 2.5.9 版本开始提供实时的点对点消息服务，这意味着，你将可以通过我们的服务开发实时的用户间聊天、游戏对战等互动功能。截至目前，我们提供 Android、JavaScript、iOS、Windows Phone 四个主要平台的客户端 SDK。

我们也提供了一些 Demo 帮助您快速入门，

* [Android Chat Demo](https://github.com/avoscloud/Android-SDK-demos/tree/master/keepalive)
* [iOS Chat Demo](https://github.com/avoscloud/iOS-SDK-demos/tree/master/KeepAlive)
* [JavaScript Demo](https://github.com/leancloud/leanmessage-javascript-sdk/tree/master/demo)
* 一个完整的社交应用 [LeanChat](https://github.com/leancloud/leanchat-android)，类似微信。

## 功能和特性

在进入开发之前，请允许我们先介绍一下实时通信服务的功能和特性，加粗的条目是最新添加的：

* 登录，通过签名与你的用户系统集成
* 单个设备多个帐号，单个帐号多个设备，实时消息同步到所有设备
* 单聊（发给一个人），群发（发给多个人），群聊（发给一个群）
* 自定义消息解析；基于 AVFile 可以实现图片、音频和视频等丰富格式
* 通过签名控制关注权限和参与对话权限
* 上下线通知
* 群组管理，创建、加入、离开、邀请、踢出、查询、成员变动通知。
* 消息时间戳
* 离线消息
* 离线推送通知：iOS, Windows Phone
* 支持平台（排名不分先后）：
  * iOS
  * Android
  * Browser JavaScript
  * Windows Phone
  * Server-side Nodejs
* 消息记录 REST API
* 未读消息数 REST API
* 敏感词过滤
* 异常数据报警
* **消息到达回执**

## 核心概念

### Peer

实时通信服务中的每一个终端称为 Peer。Peer 拥有一个在应用内唯一标识自己
的ID。这个 ID 由应用自己定义，只要求是少于 50 个字符长度的字符串。系统中的每一条消息都来自于一个 Peer，发送到一个或多个 Peer。

简单来讲，你可以将 Peer 认为是参与通信的一个节点，通常就是你的应用上的某个用户。

LeanCloud 的通信服务允许一个 Peer ID 在多个不同的设备上登录，也允许一个设备上有多个 Peer ID 同时登录。开发者可以根据自己的应用场景选择ID。

为了做到细粒度的权限控制，Peer 需要先 watch 对方方可给对方发送消息，你可以在 watch 动作上增加签名认证来控制权限，防止骚扰。

Super Peer（超级用户）可以在不 watch 的状态下给任意 Peer 发送消息，不过 Super Peer 的登录需要服务器端签名控制，目前仅服务器端的 NodeJS SDK 支持 Super Peer。通常作为管理员角色来使用。

### Session

Peer 通过开启(open)一个 Session 加入实时通信服务，Peer 可以在 Session 中关注(watch)一组 Peer ID，当被关注者上下线时，会收到通知。Peer 在开启 Session 后会收到其他 Peer 的消息，关注(watch)其他 Peer 后也可以向其发送消息。Peer 只能向自己关注的其他 Peers 发送消息，但可以接收到其他 Peer 的消息。

Session 的几种状态：

* **opened** Session 被打开
* **pause** 网络异常，Session 进入暂停状态，当网络恢复时 Session 会自动重新打开
* **closed** Session 结束，仅在显示调用 `Session.close` 方法时发生，用户注销实时通信服务，不再能够接收到消息或推送通知

Session 中的几个动词：

* **open** 以一个 Peer ID 打开 Session
* **watch** 关注一组 Peer ID，关注后可以收到这个 Peer 的上下线通知，发送消息
* **unwatch** 取消对一组 Peer ID 的关注
* **sendMessage** 给一组 Peer ID 发送消息
* **close** 注销服务，关闭 Session

在现代移动应用里，我们建议仅在用户进入互动环节（例如打开聊天对话界面，游戏对战界面）时`watch`目标用户，这样可以有效减少对方由于网络不稳定频繁上下线发送的通知，节约流量。

### Message

实时通信服务的消息。我们的消息体允许用户一次传输不超过 **5 KB**的文本数据。开发者可以在文本协议基础上自定义自己的应用层协议。

消息分为暂态(transient)和持久消息。LeanCloud 为后者提供 7 天内最多 50 条的离线消息。暂态消息并不保存离线，适合开发者的控制协议。

我们现在还为通信消息提供存储和获取功能，你可以通过 [REST API](rest_api.html#实时通信-api) 或 SDK（即将加入）获取整个应用或特定对话的消息记录。

### Group

聊天群组，用户加入群后向群发送的消息可以被所有群成员收到。当有群成员退出，或有新的群成员加入时，所有群成员会收到相应的通知。用户可以对群做以下几个动作：

* 创建并加入
* 加入已有群
* 离开已有群
* 将其他 peer 加入已有的群
* 将其他 peer 从已有的群踢出

如果你对实时通信服务启用签名认证（从安全角度推荐你这么做），除了退出群以外的其他操作都需要签名，签名见下文。

应用所有的群组数据存储在数据管理中的 `AVOSRealtimeGroups` 表中，成员数据以数组形式存储在 `m` 列，应用可以通过 API 调用获得某个群组的所有成员，和某个用户加入的所有群组。

## 权限和认证

为了满足开发者对权限和认证的需求，我们设计了签名的概念。你可以在
LeanCloud 应用控制台 -> 设置 -> 应用选项中强制启用签名。启用后，所有的
Session open 和 watch 行为都需要包含签名，这样你可以对用户的登录以及他
可以关注哪些用户，进而可以给哪些用户发消息进行充分的控制。

![image](images/signature.png)

1. 客户端发起 session open 或 watch 等操作，SDK 会调用
SignatureFactory 的实现，并携带用户信息和用户行为（登录、关注或群组操
作）请求签名；
2. 应用自有的权限系统，或应用在云代码上的签名程序收到请求，进行权限验
证，如果通过则利用**下文所述的签名算法**生成时间戳、随机字符串和签名返回给
客户端；
3. 客户端获得签名后，编码到请求中，发给实时通信服务器；
4. 实时通信服务器通过请求的内容和签名做一遍验证，确认这个操作是经由服
务器允许的，进而执行后续的实际操作。

### 云代码签名范例

我们提供了一个运行在 LeanCloud [云代码](https://cn.avoscloud.com/docs/cloud_code_guide.html)上的
[签名范例程序](https://github.com/leancloud/realtime-messaging-signature-cloudcode)
，他提供了基于 Web Hosting 和 Cloud Function 两种方式的签名实现，你可以根据实际情况选
择自己的实现。

### 签名方法

签名采用**Hmac-sha1**算法，输出字节流的十六进制字符串(hex dump)，签名的消息格式如下

```
app_id:peer_id:watch_peer_ids:timestamp:nonce
```

其中：

* `app_id` 是你的应用 ID
* `peer_id` 是打开此 Session 的 Peer ID
* `watch_peer_ids` 是 open 或 watch 请求中关注的peer ids，**升序排序**后以`:`分隔
* `timestamp` 是当前的UTC时间距离unix epoch的**秒数**
* `nonce` 为随机字符串

签名的 key 必须是应用的 **master key**，您可以在应用设置的应用 Key 里找到，请保护好 Master Key ，不要泄露给任何无关人员。

开发者可以实现自己的 SignatureFactory，调用远程的服务器的签名接口获得签名。如果你没有自己的服务器，可以直接在我们的云代码上通过 Web Hosting 动态接口实现自己的签名接口。在移动应用中直接做签名是**非常危险**的，它可能导致你的**master key**泄漏。

使用蟒蛇(Python)大法的签名范例：

```python
import hmac, hashlib

### 签名函数 hmac-sha1 hex dump
def sign(msg, k):
    return hmac.new(k, msg, hashlib.sha1).digest().encode('hex')

### 签名的消息和 key
sign("app_id:peer_id:watch_peer_ids:timestamp:nonce", "master key")
```

### 群组功能的签名

在群组功能中，我们对**加群**，**邀请**和**踢出群**这三个动作也允许加入签名，他的签名格式是：

```
app_id:peer_id:group_id:group_peer_ids:timestamp:nonce:action
```

其中：

* `app_id`, `peer_id`, `timestamp` 和 `nonce` 的含义同上
* `group_id` 是此次行为关联的群组 ID，对于创建群尚没有id的情况，`group_id`是空字符串
* `group_peer_ids` 是`:`分隔的**升序排序**的 peer id，即邀请和踢出的 peer_id，对加入群的情况，这里是空字符串
* `action` 是此次行为的动作，三种行为分别对应常量 `join`, `invite` 和 `kick`

### Super Peer

为了方便用户的特殊场景，我们设计了超级用户（Super Peer）的概念。超级用户可以无需 watch 某一个用户就给对方发送消息。超级用户的使用需要强制签名认证。

签名格式是在普通用户的签名消息后加常量 `su`。


```
app_id:peer_id:watch_peer_ids:timestamp:nonce:su
```
## Android 开发指南

参考 [Android 实时通信开发指南](./android_realtime.html)

## iOS 开发指南

参考 [iOS 实时通信开发指南](./ios_realtime.html)

## Windows Phone 8.0 开发指南

参考 [Windows Phone 8.0 开发指南](dotnet_realtime.html)

##  JavaScript 开发指南

我们已经开源 JS Messaging SDK 了， 见 [leancloud/realtime-messaging-jssdk](https://github.com/leancloud/realtime-messaging-jssdk) 。

## LeanChat Demo 

为了帮助大家更容易上手实时通信组件，我们开发了多平台应用 LeanChat，像一个简易版的微信，可点击[这里](http://fir.im/Lean)下载。项目代码放在了 Github上，[LeanChat-Android](https://github.com/leancloud/leanchat-android) 和 [LeanChat-iOS](https://github.com/leancloud/leanchat-ios)。先上图，

![image](images/leanchat.png)

LeanChat 用到了大多数实时通信组件的提供的接口与功能，通过阅读它的源码，相信您可以很快学会使用通信组件。当然，首要的是能编译运行 LeanChat，项目Readme 上都有说明，仍然遇到问题的话请[联系我们](https://ticket.avosapps.com)。

代码实现上有三点比较重要，

* `Msg` 对象，它代表一个具体的消息对象，`Msg`对象可转换成 `Json`文本，发送给对方，对方接收到后可转换成 `Msg` 对象。可参考 [Msg.java](https://github.com/leancloud/leanchat-android/blob/master/src/com/avoscloud/chat/entity/Msg.java)。
* `messages` 表，用来保存消息，字段基本和 `Msg`对象的成员一一对应。可参考 [DBMsg.java](https://github.com/leancloud/leanchat-android/blob/master/src/com/avoscloud/chat/db/DBMsg.java)。
* 音频、图片消息的发送，用到了[AVFile](./android_guide.html#文件)，通过相应的函数创建、上传`AVFile`，得到 `url`之后作为 `Msg` 对象的一部分发送给对方。

除了上述代码，Android 项目中，推荐阅读 [MsgReceiver.java](https://github.com/leancloud/leanchat-android/blob/master/src/com/avoscloud/chat/service/receiver/MsgReceiver.java)与 [ChatService.java](https://github.com/leancloud/leanchat-android/blob/master/src/com/avoscloud/chat/service/ChatService.java)。iOS 项目中，推荐阅读 [CDSessionManager.m](https://github.com/leancloud/leanchat-ios/blob/master/AVOSChatDemo/service/CDSessionManager.m)与 [CDDatabaseService.m](https://github.com/leancloud/leanchat-ios/blob/master/AVOSChatDemo/service/CDDatabaseService.m)。


至于其它技术细节，请参考 [项目wiki](https://github.com/leancloud/leanchat-android/wiki) 。


## FAQ

### 我有自己的用户系统

我们并不强制接入实时消息服务的应用使用 LeanCloud 的用户系统。实时消息服务中的 PeerId 可以由用户任意指定，只要在用户系统中保证一致即可（在匿名聊天 Demo KeepAlive 里我们用的是 installationId）。对已有用户系统的应用来说，你可以使用自己的用户 ID 作为 PeerId，并通过[签名](realtime.html#权限和认证)做权限认证。

### 聊天支持图片、语音吗？

当然支持。你可以把图片、语言等 blob 上传到我们的文件存储服务，再传输 URL，并在 UI 上进行适当地展示。这么做的好处：

* 可以传输大文件，避免超过 5K 的消息大小限制
* 节省流量：聊天的对方只要接收一个 URL 文本即可，图片和语音是可选下载
* 加速：我们的 CDN 提供更快的下载速度，对图片还支持自动缩略图

### 聊天离线时可以推送吗？

当然可以。我们的Android聊天服务是和后台的推送服务共享连接的，所以只要有网络就永远在线，不需要专门做推送。消息达到后，你可以根据用户的设置来 判断是否需要弹出通知。网络断开时，我们为用户保存50条的离线消息。

iOS在应用退出前台后即离线，这时收到消息会触发一个APNS的推送。因为APNS
有消息长度限制，且你们的消息正文可能还包含上层协议，所以 我们现在APNS
的推送内容是让应用在控制台设置一个静态的APNS json，如“你有新的未读消
息” 。

![image](images/rtm-push.png)

### 聊天记录

聊天记录的查询我们支持 4 种方式：

* 按对话查询，对话 id 即所有对话参与者的 peerId 排序后以`:`分隔`md5`所得的字符串；
* 按群组查询，对话 id 即群组 id；
* 按用户查询，你可以查到某一个用户的所有消息，以时间排序；
* 按应用查询，你可以查到自己应用中所有的消息，以时间排序。

参考消息记录的 [REST API](rest_api.html#实时通信-api)。

### 未读消息数

你可以调用 [REST API](rest_api.html#实时通信-api) 获得某个用户的未读消息数。

### 黑名单

我们目前的设计里，黑名单等权限控制是通过签名来实现的。如果你是自己比较成熟的应用，你可能已经有了一定的用户关系、屏蔽关系，我们把这个步骤通过签名来实现，避免你把所有的关系数据都同步到我们的服务器。

如果你启用了签名，用户间发起对话、用户进入聊天群组，都需要通过服务器签名认证，在这一步应用可以实施自己的黑名单功能。

### 如何实现自动回复的客服

我们的 nodejs 客户端正在开发中，届时你可以利用 nodejs 客户端编写一些机器人，或是和自有的系统集成。


### 我希望给群组增加一些自定义数据，如名字

我们的群组信息实际上是 LeanCloud 的一个标准的数据表 `AVOSRealtimeGroups`。对于群组的元信息，你可以在关联的表里设置，也可以在这个表里添加新的列。请对这个表设置合理的 ACL 来保证内容不会被恶意篡改。

**注意请不要通过修改 `m` 列来改变群组成员**，这样目标用户无法收到通知，会造成数据不一致的情况。
