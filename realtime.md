# 实时通信服务开发指南

除了实时的消息推送服务外，LeanCloud 从 2.5.9 版本开始提供实时的点对点消息服务，这意味着，你将可以通过我们的服务开发实时的用户间聊天、游戏对战等互动功能。截至目前，我们提供 Android 和 iOS 两个主要平台的客户端SDK。

我们也提供了两个平台下的 Demo， [Android Chat Demo ](https://github.com/avoscloud/Android-SDK-demos/tree/master/keepalive)， [iOS Chat Demo](https://github.com/avoscloud/iOS-SDK-demos/tree/master/KeepAlive) 。

## 核心概念

### Peer

实时通信服务中的每一个终端称为 Peer。Peer 拥有一个在应用内唯一标识自己的ID。系统中的每一条消息都来自于一个 Peer，发送到一个或多个 Peer。

LeanCloud 的通信服务允许一个 Peer ID 在多个不同的设备上登录，也允许一个设备上有多个 Peer ID 同时登录。开发者可以根据自己的应用场景选择ID。

### Session

Peer 通过开启(open)一个 Session 加入实时通信服务，Peer 可以在 Session 中关注(watch)一组 Peer ID，当被关注者上下线时，会收到通知。Peer 在开启 Session 后会收到其他 Peer 的消息，关注(watch)其他 Peer 后也可以向其发送消息。Peer 只能向自己关注的其他 Peers 发送消息，但可以接收到其他peers的消息。

Session 的几种状态：

* **opened** Session 被打开
* **pause** 网络异常，Session 进入暂停状态，当网络恢复时 Session 会自动重开
* **closed** Session 结束，仅在显示调用 `Session.close` 方法时发生，用户注销实时通信服务，不再能够接收到消息或推送通知

Session 中的几个动词：

* **open** 以一个 Peer ID 打开 Session
* **watch** 关注一组 Peer ID，关注后可以收到这个 Peer 的上下线通知，发送消息
* **unwatch** 取消对一组 Peer ID 的关注
* **sendMessage** 给一组 Peer ID 发送消息
* **close** 注销服务，关闭 Session

在现代移动应用里，我建议仅在用户进入互动环节（打开聊天对话界面，游戏对战界面）时`watch`目标用户，这样可以有效减少对方由于网络不稳定频繁上下线发送的通知，节约流量。

### Message

实时通信服务的消息。我们的消息体允许用户一次传输不超过**5KB**的文本数据。开发者可以在文本协议基础上自定义自己的应用层协议。

消息分为暂态(transient)和持久消息。LeanCloud 为后者提供至多7天50条的离线消息。暂态消息并不保存离线，适合开发者的控制协议。

我们现在还为通信消息提供存储和获取功能，你可以通过 [REST API](rest_api.html#实时通信-api) 或 SDK（即将加入）获取整个应用或特定对话的消息记录。

### Group

聊天群组，用户加入群后向群发送的消息可以被所有群成员收到。当有群成员退出，或有新的群成员加入时，所有群成员会收到相应的通知。用户可以对群做以下几个动作：

* 创建并加入
* 加入已有群
* 离开已有群
* 将其他 peer 加入已有的群
* 将其他 peer 从已有的群踢出

如果你对实时通信服务启用签名认证（推荐），除了退出群其他操作都需要签名，签名见下文。

应用所有的群组数据存储在 `AVOSRealtimeGroups` 表中，成员数据以数组形式存储在 `m` 列，应用可以通过 API 调用获得某个群组的所有成员，和某个用户加入的所有群组。

## 权限和认证

为了满足开发者对权限和认证的需求，我们设计了签名的概念。你可以在 LeanCloud 应用控制台、设置、应用选项中强制启用签名。启用后，所有的 Session open 和 watch 行为都需要包含签名，这样你可以对用户的登录以及他可以关注哪些用户进行充分的控制。

签名采用**Hmac-sha1**算法，输出字节流的十六进制字符串(hex dump)，签名的消息各式如下

```
app_id:peer_id:watch_peer_ids:timestamp:nonce
```

其中：

* `app_id` 是你的应用 ID
* `peer_id` 是打开此 Session 的 Peer ID
* `watch_peer_ids` 是 open 或 watch 请求中关注的peer ids，**升序排序**后以`:`分隔
* `timestamp` 是当前的UTC时间距离unix epoch的**秒数**
* `nonce` 为随机字符串

签名的key是应用的 **master key**

开发者可以实现自己的SignatureFactory，调用远程的服务器的签名接口获得签名。如果你没有自己的服务器，可以直接在我们的云代码上通过 Web Hosting 动态接口实现自己的签名接口。在移动应用中直接做签名是**非常危险**的，它可能导致你的**master key**泄漏。

### 群组功能的签名

在群组功能中，我们对**加群**，**邀请**和**踢出群**这三个动作也允许加入签名，他的签名格式是：

```
app_id:peer_id:group_id:group_peer_ids:timestamp:nonce:action
```

其中：

* `app_id`, `peer_id`, `timestamp` 和 `nonce` 同上
* `group_id` 是此次行为关联的群组 ID，对于创建群尚没有id的情况，`group_id`是空字符串
* `group_peer_ids` 是`:`分隔的**升序排序**的 peer id，即邀请和踢出的 peer_id，对加入群的情况，这里是空字符串
* `action` 是此次行为的动作，三种行为分别对应常量 `join`, `invite` 和 `kick`

## Android 实时通信服务

###初始化
和其他的LeanCloud服务一样，实时聊天系统的初始化也是在Application的onCreate方法中进行的：

```
 public class MyApplication extends Application{
 
 public void onCreate(){
 
 AVOSCloud.initialize(this,"YourAppId","YourAppKey");
 }
}

```

并且在AndroidManifest.xml中间声明：

```
<manifest ...

 <application
        android:name=".MyApplication"
        ....>
</application>
</manifest>
```

###登录
以一个最简单的聊天系统的原型来分析实现Android的实时通信，这样的一个聊天系统最主要的有两个模块：用户系统，对话的发送接收。
在每一个用户系统中间，用户一定有一个唯一表示的符号来标识他们与别人的区别，比如：userId或者email或者手机号码或者AVUser的objectId；同时这个符号也需要能够通过某种方式（登录）而正确获取。
由于考虑到很多开发者在接入实时通信系统时，可能已经有现成的用户系统，所以我们在设计实时通信模块的时候，并没有强制将用户系统的登录状态与实时通信的登录状态绑定到一起，而是通过一种更为开放的方式去控制实时通信的登录状态。当一个用户需要登录实时通信模块的时候，我们需要:
```
   AVUser.logInInBackground("用户名","password",new LogInCallback<AVUser>(){
      @Override
      public void done(AVUser user, AVException e){
            String selfId = user.getObjectId();//此处的selfId就是之前提到的用户的唯一标识符,也可以替换成你现有用户系统中的唯一标识符
            Session session = SessionManager.getInstance(selfId);
            List<String> yourFriends = new List<String>();
            .... //add your friends' peerIds 
            session.open(yourFriends);
      }
   });
```

这样你就向服务器发起了一个实时通信的登录请求。但是至今为止还不能发送消息，因为实时通信的所有请求都是异步的，只有当你接收到异步请求对应的成功回调时，你才能进行下一步操作。
要接收异步请求对应的回调，你需要实现继承AVMessageReceiver的Receiver，并且注册到AndroidManifest.xml。

```
public class ChatDemoMessageReceiver extends AVMessageReceiver{
  ...实现抽象方法,比如：
  @Override
  public void onSessionOpen(Context context, Session session) {
    System.out.println("用户成功登录上实时聊天服务器了");
  }
}
```

并且在AndroidManifest.xml中间声明:

```
        <receiver android:name=".ChatDemoMessageReceiver" >
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="com.avoscloud.session.action" />
            </intent-filter>
        </receiver>
```
至此，就完成了用户的登录环节。
不管你接下来的操作是单聊还是群聊，你都需要实现之前的所有步骤才能进行下一步的操作。

###单聊
单聊的情景相对也是比较简单的，用户可也选择向已经watch过的人发送相应的消息

####添加好友
正如上文提到了，实时聊天系统在发送消息前需要保证发送的对象是被watch过的。对于已有的好友列表，你可以如上文提到的方法，在登录实时通信系统的时候放在参数中间；对于新的好友，你可以通过如下代码进行添加：

```
  Session session = SessionManager.getInstance(selfId);
  session.watch(Arrays.asList("friend1","friend2"));
```

之后添加是否成功则可以通过Receiver中的回调的方式来获悉：

```
public class ChatDemoMessageReceiver extends AVMessageReceiver{

  @Override
  public void onPeersWatched(Context context, Session session, List<String> peerIds) {
     ...//已经watch
  }
}
```

在任何一个时候你也可以通过以下代码来判断是否已经watch过某个用户：

```
  Session session = SessionManager.getInstance(selfId);
  session.isWatching("friend1");
```

####发送消息
在用户成功登录实时消息系统以后，用户就可以进行消息的发送接收等。

```
   Session session = SessionManager.getInstance(selfId);
   AVMessage msg = new AVMessage();
   msg.setMessage("这是一个普通的消息");
   msg.setToPeerIds(Arrays.asList(friendId));//friendId是指想要发送消息的目标用户在你选用的用户系统中的唯一标识符
   session.sendMessage(msg);
```

正如上文提到的，实时通信中所有的操作都是异步操作，发送消息也是一样，针对于消息发送的结果，我们需要在之前提到的Receiver中实现对应的方法：

```
public class ChatDemoMessageReceiver extends AVMessageReceiver{

  @Override
  public void onMessageSent(Context context, Session session, AVMessage msg) {
     System.out.println("消息发送成功了，发送成功时间是"+msg.getTimestamp());//这个时间是来自服务器端的时间，这样即便是多台设备中间也不会出现时间的混乱
  }

  @Override
  public void onMessageFailure(Context context, Session session, AVMessage msg) {
     System.out.println("消息发送失败了，可能需要在app端进行重试等");
  }
}

```

##### 在线消息
有些特别的应用可能会有指定消息是否是只有用户在线才能收到消息的需求，我们在系统中间也进行了支持。

```
   Session session = SessionManager.getInstance(selfId);
   AVMessage transientMsg = new AVMessage();
   transientMsg.setMessage("这是一个在线消息，只有对方正在线才能收到");
   transientMsg.setTransient(true);
   transientMsg.setToPeerIds(Arrays.asList(friendId));
   session.sendMessage(transientMsg);

   AVMessage msg = new AVMessage();
   msg.setMessage("这是一个离线消息，对方当时不在线，重新上线也能够收到");
   msg.setTransient(false);//如果不设置，默认是false
   msg.setToPeerIds(Arrays.asList(friendId));
   session.sendMessage(msg);
```

##### 消息回执
由于离线消息的存在，消息的发送成功与真正对方收到消息，可能在时间上存在一定的先后消息。有一些App需要明确知道消息的到达，我们也通过消息回执的形势来支持这样的操作：

```
   Session session = Session.getInstance(selfId);
   AVMessage msg = new AVMessage();
   msg.setMessage("这是一个带有消息回执的消息");
   msg.setRequestReceipt(true);
   msg.setToPeerIds(Arrays.asList(friendId));
```

针对消息回执，我们会产生额外的回调：

```
public class ChatDemoMessageReceiver extends AVMessageReceiver{

  @Override
  public void onMessageDelivered(Context context, Session session, AVMessage msg) {
    System.out.println(msg.getMessage() + "delivered at " + msg.getReceiptTimestamp());
  } 
}
```
**注:消息回执的功能仅仅能够在单聊中使用，消息接收者不能多于一人，并且要求消息不能是在线消息。**

##### 多媒体消息
实时聊天系统已经不在是多年以前的聊天室，用户往往会通过更多更丰富的多媒体内容来进行有效的交互，比如：图片，短视频，语音，地理位置等等。开发者可以通过将AVMessage中的message当做一个相对复杂的数据结构的形势来实现这样的消息内容。

```
    //示范一个简单的带图片的消息{"type":"file","content":"https://cn.avoscloud.com/images/static/partner-iw.png"}
    HashMap<String, Object> params = new HashMap<String, Object>();
    params.put("type", "file");
    params.put("content", "https://cn.avoscloud.com/images/static/partner-iw.png");
    AVMessage msg = new AVMessage(JSON.toJSONString(params));    
```

#### 接收消息
一个客户端在实时通信系统中间不仅仅会扮演简单的发送者的概念，同时也会需要扮演接收者的角色。和之前的所有回调一样，消息的接收也是通过继承的Receiver来接收的：

```
public class ChatDemoMessageReceiver extends AVMessageReceiver{

  @Override
  public void onMessage(Context context, Session session, AVMessage msg) {
    ...处理接收到的消息
  }
}
```

###群聊




## iOS 实时通信服务

与 Android 不同，iOS 并没有提供类似于 `service` 这样的组件。当应用进入后台，聊天组件将会关闭连接，session 进入 `paused` 状态。而当应用转入前台，聊天组件将会重新建立连接，session 进入 `resume` 状态。你可以实现 `AVSessionDelegate`相关方法，以完成实时通信应用的开发。

**注意** 请首先确保你添加了如下依赖库

* SystemConfiguration.framework
* MobileCoreServices.framework
* CoreTelephony.framework
* CoreLocation.framework
* libicucore.dylib

### 实现你的 AVSessionDelegate

同 Android 版 SDK 类似，`AVsessionDelegate` 包含了与 session 相关的各种事件。你可以实现 `AVSessionDelegate`，对这些事件做出响应。

#### sessionOpened

session 成功打开

```
- (void)sessionOpened:(AVSession *)session;
```

#### sessionPaused

已经打开的 session 由于网络原因、或者应用转入后台，因而进入暂停状态。此时无法成功地发送消息。

```
- (void)sessionPaused:(AVSession *)session;
```

#### sessionResumed

之前暂停的 session 重新恢复连接

```
- (void)sessionResumed:(AVSession *)session;
```

#### session:didReceiveMessage:

收到消息

```
- (void)session:(AVSession *)session didReceiveMessage:(AVMessage *)message;
```

#### session:messageSendFinished:

服务器确认之前发送的消息已发出

```
- (void)session:(AVSession *)session messageSendFinished:(AVMessage *)message;
```

#### session:messageSendFailed:

这些消息发出后没有及时收到服务器确认，客户端会启动重连流程，这些消息被认为发送失败。**注意**：此时连接处在已断开，不能立即重发。

```
- (void)session:(AVSession *)session messageSendFailed:(AVMessage *)message error:(NSError *)error;
```

#### session:didReceiveStatus:peerIds:

你关注的用户状态改变了

```
- (void)session:(AVSession *)session didReceiveStatus:(AVPeerStatus)status peerIds:(NSArray *)peerIds;
```

#### sessionFailed:error:

所有 `AVSession` 中操作失败会触发此回调

```
- (void)sessionFailed:(AVSession *)session error:(NSError *)error;
```

### 打开 session

```
AVSession *session = [[AVSession alloc] init];
session.sessionDelegate = self;
NSString *selfId = [self getMyUserId];

// 打开 session，并关注一些 id
[session openWithPeerId:selfId watchedPeerIds:watchedIds];
```

### 实现签名（可选）

如果使用了签名认证，你需要实现 `AVSignatureDelegate`，并在调用 `[session open]`之前为 session 设置 `signatureDelegate`：

```
session.signatureDlegate = self;
```

需要说明的是，你需要为 AVSignatureDelegate 实现的方法是：

```
- (AVSignature *)signatureForPeerWithPeerId:(NSString *)peerId watchedPeerIds:(NSArray *)watchedPeerIds action:(NSString *)action;
```

你需要做的就是按照前文所述的签名算法实现签名，其中 `AVSignature` 声明如下：

```
@interface AVSignature : NSObject

@property (nonatomic, retain) NSString *signature;
@property (nonatomic, assign) long timestamp;
@property (nonatomic, retain) NSString *nonce;
@property (nonatomic, retain) NSArray *signedPeerIds;

@end
```

其中四个属性分别是:

* 签名
* 时间戳
* 随机字符串 nonce
* 签名通过的可关注的 signedPeerIds

### 关注、取消关注

```
/*!
 *  增量关注一组 peerIds
 *  @param peerIds peer id 数组
 */
- (void)watchPeerIds:(NSArray *)peerIds;

/*!
 *  取消关注一组 peerIds
 *  @param peerIds peer id 数组
 */
- (void)unwatchPeerIds:(NSArray *)peerIds;

```

### 发送消息

使用如下方法构造一个AVMessage对象，注意：toPeerId必须是已经关注了的，否则发送消息时将无法送达。

```
/*!
 *  构造一个发送给 toPeerId 的message对象
 *  @param session 服务器会话
 *  @param toPeerId 要发往的 peerId
 *  @param payload 消息载体
 *  @return message 对象
 */
+ (AVMessage *)messageForPeerWithSession:(AVSession *)session
                                toPeerId:(NSString *)toPeerId
                                 payload:(NSString *)payload;
```

调用如下方法发送消息。

```
/*!
 *  发送消息
 *  @param message 消息对象
 */
- (void)sendMessage:(AVMessage *)message;

/*!
 *  发送消息
 *  @param message 消息对象
 *  @param transient 设置为 YES, 当且仅当某个 peer 在线才会收到该条消息，且该条消息既不会存为离线消息，也不会通过消息推送系统发出去.
 *         如果设置为 NO, 则该条消息会设法通过各种途径发到 peer 客户端，比如即时通信、推送、离线消息等。
 */
- (void)sendMessage:(AVMessage *)message transient:(BOOL)transient;
```

服务器端确认收到消息后，你会收到`session:messageSendFinished:`事件。

### 聊天室功能

iOS SDK从v2.6.1开始提供聊天室功能。客户端对一个聊天室对象发送消息，所有监听这个聊天室的其他客户端，都能够收到这个消息；聊天室成员状态发生变化，每一个客户端会收到状态变化的消息。

#### 基本组成

与 `AVSession` 相类似的，聊天组的实现构成非常简单，主要涉及的类只有3个：

`AVGroup` 来作为所有的聊天操作的接口类

`AVSignature`中多加入一个`-signatureForGroupWithPeerId:groupId:groupPeerIds:action:`方法来实现聊天室签名相关

`AVGroupDelegate` 协议用来实现接收到服务器反馈的聊天组消息后的回调处理

#### 加入一个聊天室
由于整个实时通信功能都是建立在Session的基础上，所以您要加入一个聊天室也需要建立在一个已经打开的Session上。
当您已经打开一个Session以后，可以通过一下操作来加入一个Group

```
    //新建并加入一个聊天室
    [AVGroup createGroupWithSession:session groupDelegate:self callback:^(AVGroup *group, NSError *error) {
        if (!error) {
             //聊天室创建并加入成功
        } else {
            NSLog(@"error:%@", error);
        }
    }];
    //加入一个已经存在的聊天室
    AVGroup *group = [AVGroup getGroupWithGroupId:groupId session:session];
    group.delegate = self;
    [group join];
```

加入一个已经存在的聊天室时，成功以后delegate的`-[group:didReceiveEvent:peerIds:]`方法会被调用。

#### 查询聊天室组员
在应用管理的数据中心的 `AVOSRealtimeGroups` 表中，记录所有聊天室基本信息。当你知道一个聊天室的groupId的时候，您就可以通过AVObject接口来查看这个聊天室的组员情况。

```
    AVObject *groupObject = [AVObject objectWithoutDataWithClassName:@"AVOSRealtimeGroups" objectId:groupId];
    [groupObject fetch];
    NSArray *groupMembers = [groupObject objectForKey:@"m"];
```

当然您也可以使用AVQuery来查询。

#### 管理聊天室组员
在查询到聊天室组员以后，您可以邀请一些您的朋友加入，或者踢出一些"可怕"的组员。

```
    [group invitePeerIds:@[@"peerId1",@"peerId2",@"peerId3"]];
    [group kickPeerIds:@[@"peerId1",@"peerId2",@"peerId3"]];
```

成功以后delegate的`-[group:didReceiveEvent:peerIds:]`方法会被调用。

而被邀请的人或者被踢的人的客户端中，相应的`-[group:didReceiveEvent:peerIds:]`会被调用。

#### 发送消息

通过如下代码您就可以向对应的聊天室发送代码:

```
    AVMessage *message = [AVMessage messageForGroup:group payload:@"hello world"];
    [group sendMessage:message];
```
发送成功后聊天室内的其他组员的客户端，`-[group:didReceiveMessage:]`方法会被调用，也就是接收到来自聊天室的消息。

#### 签名(可选)

在群组功能中，我们对**加群**，**邀请**和**踢出群**这三个动作也允许加入签名，他的签名格式是：

```
app_id:peer_id:group_id:group_peer_ids:timestamp:nonce:action
```

其中：

* `app_id`, `peer_id`, `timestamp` 和 `nonce` 同上
* `group_id` 是此次行为关联的群组 ID
* `group_peer_ids` 是`:`分隔的 peer id，即邀请和踢出的 peer_id，对加入群的情况，这里是空字符串
* `action` 是此次行为的动作，三种行为分别对应常量 `join`, `invite` 和 `kick`

你需要为 AVSignatureDelegate 实现的方法是：

```
- (AVSignature *)signatureForGroupWithPeerId:(NSString *)peerId groupId:(NSString *)groupId groupPeerIds:(NSArray *)groupPeerIds action:(NSString *)action
```

## JS 开发指南

###  方法
#### new AVChatClient(settings)
```
settings:{
  appId: 应用ID,
  peerId: 当前用户的PeerID,
  auth: 私聊签名函数(当平台设置启动签名后，需要传递),
  groupAuth: 群组聊天签名函数(当平台设置启动签名后，需要传递),
  watchingPeerIds: (非必须)
}
```
具体签名函数 需要类似下面的示例格式，基于 Promise 的异步操作。

```
function auth(peerId, watchingPeerIds){
  // 类似
  /*
  return new Promise(resolve,reject){

    //这里放ajax auth code
    resolve({
      watchingPeerIds: ajax返回值
    });

  }
  */
  //这里实现了一个空函数
  return Promise.resolve({
    watchingPeerIds: watchingPeerIds||[]
  });
}
function groupAuth(peerId, groupId, action, groupPeerIds){
  return Promise.resolve({
    groupPeerIds: groupPeerIds || []
  });
}

```
实例化一个 消息客户端
#### open()
打开链接，需要先执行上面的 new,

```
 open().then(function(data){
  //打开成功
})
```

所有方法都会返回promise then,因为都是异步执行，这样可以确认成功失败。
#### close()
关闭链接
#### send(msg, to, transient)
 发送私聊消息
 参数：msg:消息内容, to:发送目标 PeerId, transient（非必须):为true时代表无需离线，默认为支持离线发送。

```
 send().then(function(data){
  //success full send callback
  },function(err){
  //error callback
})
```
#### watch(peers)
参数：peers:单个peerId 或数组。
#### unwatch(peers)
参数：peers:单个peerId 或数组。
#### getStatus(peers)
查询 peer 在线或离线状态。适应于非 watch 情况下。
参数：peers:单个peerId 或数组。
#### on(name, func)
监听时间
参数：name:事件名称,func:事件处理函数

###  事件

#### close
 链接关闭
#### online
上线
当关注的人上线时触发
#### offline
下线
当关注的人下线时触发
#### message
收到消息时触发

### 群组方法
#### joinGroup(groupId)
创建或加入群组
groupId: 群组Id,创建时无需传递。
#### sendToGroup(msg, groupId, transient)
发送消息到指定群组
msg:消息内容,grouipId:群组ID, transient（非必须):为true时代表无需离线，默认为支持离线发送。
#### inviteToGroup(groupId, groupPeerIds)
邀请加入群组
groupId:群组ID,groupPeerIds:单个或数组群组ID
#### kickFromGroup(groupId, groupPeerIds)
踢出群组
groupId:群组ID,groupPeerIds:单个或数组群组ID
#### leaveGroup(groupId)
离开群组
groupId:群组ID

### 群组事件

#### membersJoined
有成员加入群
#### membersLeft
有成员离开群
#### joined
自己加入了群
#### left
自己离开了群

### 运行DEMO
直接启动一个 web 服务器 即可运行 demo。
对于不支持 websocket的浏览器 参考demo做法。
依赖   <a href="https://github.com/gimite/web-socket-js">web-socket-js</a> 可以用flash做 gateway


### 浏览器端环境依赖：
1. jQuery (非必须)  用于 jsonp 方式请求 (请求 socket 服务器信息)，主要是针对 ie9 以下浏览器的跨域支持。如果没有 jQuey 会根据 XMLHttpRequest 创建ajax跨域请求。
2. es6-promise (非必须) 当需要签名认证的时候需要，是一个 promise 接口。
3.  /lib/flash/swfobject.js web_socket.js (非必须) 用于跨浏览器支持 websocket.针对 不支持 websocket 的浏览器。 参照 <a href="https://github.com/gimite/web-socket-js">web-socket-js</a>

### 浏览器端 lib 生成

browserify chat.js -o  lib/chat.js --exclude xmlhttprequest --exclude ws -s AVChatClient

### node 环境
npm install lean-cloud-chat


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

iOS在应用退出前台后即离线，这时收到消息会触发一个APNS的推送。因为APNS有消息长度限制，且你们的消息正文可能还包含上层协议，所以 我们现在APNS的推送内容是让应用在控制台设置一个静态的APNS json，如“你有新的未读消息” 。

### 聊天记录

聊天记录的查询我们支持 4 种方式：

* 按对话查询，对话 id 即所有对话参与者的 peerId 排序后以`:`分隔`md5`所得的字符串；
* 按群组查询，对话 id 即群组 id；
* 按用户查询，你可以查到某一个用户的所有消息，以时间排序；
* 按应用查询，你可以查到自己应用中所有的消息，以时间排序。

参考消息记录的 [REST API](rest_api.html#实时通信-api)。

### 黑名单

我们目前的设计里，黑名单等权限控制是通过签名来实现的。如果你是自己比较成熟的应用，你可能已经有了一定的用户关系、屏蔽关系，我们把这个步骤通过签名来实现，避免你把所有的关系数据都同步到我们的服务器。

如果你启用了签名，用户间发起对话、用户进入聊天群组，都需要通过服务器签名认证，在这一步应用可以实施自己的黑名单功能。

### 自动回复的客服

我们的 nodejs 客户端正在开发中，届时你可以利用 nodejs 客户端编写一些机器人，或是和自有的系统集成。


### 我希望给群组增加一些自定义数据，如名字

我们的群组信息实际上是 LeanCloud 的一个标准的数据表 `AVOSRealtimeGroups`。对于群组的元信息，你可以在关联的表里设置，也可以在这个表里添加新的列。请对这个表设置合理的 ACL 来保证内容不会被恶意篡改。

**注意请不要通过修改 `m` 列来改变群组成员**，这样目标用户无法收到通知，会造成数据不一致的情况。
