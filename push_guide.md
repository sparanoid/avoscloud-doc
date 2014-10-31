# 消息推送开发指南

消息推送，使得开发者可以即时地向其应用程序的用户推送通知或者消息，与用户保持互动，从而有效地提高留存率，提升用户体验。平台提供整合了 Android推送、iOS推送的统一推送服务。

除了iOS,Android SDK做推送服务之外，你还可以通过我们的REST API来发送推送请求。

## 基本概念

### Installation

Installation 表示一个允许推送的设备的唯一标示，对应[数据管理](/data.html?appid={{appid}})平台中的 `_Installation` 表。它就是一个普通的对象，主要属性包括:

* deviceType  设备类型，目前只支持"ios"和"android"
* deviceToken iOS设备才有的用于 APNS 推送的唯一标识符，只对 iOS 有效。
* installationId LeanCloud 为每个Android设备产生的唯一标识符，只对android有效。
* badge iOS设备呈现在应用程序图标右上角的红色圆形数字提示,用于提示一些无需即时处置的音讯,比方程序更新数、未读数等。
* timeZone 设备设定的时区
* channels 设备订阅的频道
* subscriptionUri Windows Phone 设备才有的 MPNS（微软推送服务）推送的通道 ID，只针对微软平台的设备（微软的平板以及手机）有效，目前仅支持 Windows Phone。

增删改查设备，请看后面的 SDK 说明和 REST API 一节。


### Notification

对应 `_Notification` 表，表示一条推送消息，它包括下列属性：

* subscribers 本条消息成功推送到的设备数
* status 状态，可能是"in queue","done"或者错误信息

如何发送消息也请看下面的详细指南。

## iOS消息推送

### 如何使用 LeanCloud 的 Push 功能

本节将向您简单介绍如何在iOS设备中使用LeanCloud的推送功能。

### 配置 iOS 推送证书

配置iOS证书相对麻烦，但是却是必须的步骤，请仔细看[iOS推送证书设置指南](./ios_push_cert.html)。


### 保存 Installation

在保存installation前，要先通过下列代码获取用户推送权限：

```
// Before iOS 8:
- (BOOL)application:(UIApplication *)application
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ...
    // Register for push notifications
    [application registerForRemoteNotificationTypes:
                                UIRemoteNotificationTypeBadge |
                                UIRemoteNotificationTypeAlert |
                                UIRemoteNotificationTypeSound];
    ...
}
```
```
//For iOS 8:
- (BOOL)application:(UIApplication *)application
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ...
    UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert
                                            | UIUserNotificationTypeBadge
                                            | UIUserNotificationTypeSound
                                                                             categories:nil];
    [application registerUserNotificationSettings:settings];
    [application registerForRemoteNotifications];
    ...
}
```

在iOS设备中，Installation的类是AVInstallation，并且是AVObject的子类，使用同样的API存储和查询。如果要访问当前应用的Installation对象，可以通过`[AVInstallation currentInstallation]`方法。当你第一次保存AVInstallation的时候，它会插入`_Installation`表，你可以在[数据管理](/data.html?appid={{appid}})平台看到和查询。当deviceToken一被保存，你就可以向这台设备推送消息了。

```
- (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    AVInstallation *currentInstallation = [AVInstallation currentInstallation];
    [currentInstallation setDeviceTokenFromData:deviceToken];
    [currentInstallation saveInBackground];
}
```

可以像修改AVObject那样去修改AVInstallation，但是有一些特殊字段可以帮你管理目标设备：

* badge 应用icon旁边的红色数字，修改AVInstallation的这个值将修改应用的badge。修改应该保存到服务器，以便为以后做badge增量式的推送做准备。
* channels 当前设备订阅的频道数组。
* appName 应用名称，只读
* appVersion 应版本，只读

### 发送推送消息

发送iOS推送消息，可以通过REST API，或者我们的消息推送web平台，请进入你的应用管理界面查看。

### 使用频道

使用频道可以实现发布——订阅的模型。设备订阅某个频道，然后发送消息的时候指定要发送的频道即可。

#### 订阅和退订

订阅"Giants"频道：

```
// When users indicate they are Giants fans, we subscribe them to that channel.
AVInstallation *currentInstallation = [AVInstallation currentInstallation];
[currentInstallation addUniqueObject:@"Giants" forKey:@"channels"];
[currentInstallation saveInBackground];
```

订阅后要记得保存，即可在[数据管理](/data.html?appid={{appid}})平台看到该installation的channels字段多了"Giants"。

退订：

```
// When users indicate they are no longer Giants fans, we unsubscribe them.
AVInstallation *currentInstallation = [AVInstallation currentInstallation];
[currentInstallation removeObject:@"Giants" forKey:@"channels"];
[currentInstallation saveInBackground];
```

获取所有订阅的频道：

```
NSArray *subscribedChannels = [AVInstallation currentInstallation].channels;
```

#### 发送消息到频道

例如，发送消息到刚才订阅的"Giants"频道：

```
// Send a notification to all devices subscribed to the "Giants" channel.
AVPush *push = [[AVPush alloc] init];
[push setChannel:@"Giants"];
[push setMessage:@"The Giants just scored!"];
[push sendPushInBackground];
```

如果你想发送到多个频道，可以指定channels数组:

```
NSArray *channels = [NSArray arrayWithObjects:@"Giants", @"Mets", nil];
AVPush *push = [[AVPush alloc] init];

// Be sure to use the plural 'setChannels'.
[push setChannels:channels];
[push setMessage:@"The Giants won against the Mets 2-3."];
[push sendPushInBackground];
```

### 高级定向发送

频道对于大多数应用来说可能就足够了。但是某些情况下，你可能需要更高精度的定向推送。LeanCloud 允许你通过AVQuery API查询Installation列表，并向指定条件的query推送消息。

因为AVInstallation同时是AVObject的子类，因此你可以保存任何数据类型到AVInstallation，并将它和你的其他应用数据对象关联起来，这样以来，你可以非常灵活地向你用户群做定制化、动态的推送。

#### 保存 Installation 数据

为AVInstallation添加三个新字段：

```
// Store app language and version
AVInstallation *installation = [AVInstallation currentInstallation];
[installation setObject:@(YES) forKey:@"scores"];
[installation setObject:@(YES) forKey:@"gameResults"];
[installation setObject:@(YES) forKey:@"injuryReports"];
[installation saveInBackground];
```

表示比赛的分数，比赛结果和受伤报告。

设置，你可以给Installation添加owner属性，比如当前的登陆用户：

```
// Saving the device's owner
AVInstallation *installation = [AVInstallation currentInstallation];
[installation setObject:[AVUser currentUser] forKey:@"owner"];
[installation saveInBackground];
```

#### 根据查询来推送消息

一旦Installation保存了你的应用数据，你可以使用`AVQuery`来查询出设备的一个子集做推送。Installation的查询跟其他对象的查询没有什么不同，只是使用特殊的静态方法
`[AVInstallation query]`创建查询对象：

```
// Create our Installation query
AVQuery *pushQuery = [AVInstallation query];
[pushQuery whereKey:@"injuryReports" equalTo:@(YES)];

// Send push notification to query
AVPush *push = [[AVPush alloc] init];
[push setQuery:pushQuery]; // Set our Installation query
[push setMessage:@"Willie Hayes injured by own pop fly."];
[push sendPushInBackground];
```

你也可以在查询中添加channels的条件：

```
// Create our Installation query
AVQuery *pushQuery = [AVInstallation query];
[pushQuery whereKey:@"channels" equalTo:@"Giants"]; // Set channel
[pushQuery whereKey:@"scores" equalTo:@(YES)];

// Send push notification to query
AVPush *push = [[AVPush alloc] init];
[push setQuery:pushQuery];
[push setMessage:@"Giants scored against the A's! It's now 2-2."];
[push sendPushInBackground];
```

如果你在Installation还保存了其他对象的关系，我们同样可以在查询条件中使用这些数据，例如，向靠近北京大学的设备推送消息：

```
// Find users near a given location
AVQuery *userQuery = [AVUser query];
[userQuery whereKey:@"location"
        nearGeoPoint:beijingUniversityLocation,
         withinMiles:[NSNumber numberWithInt:1]]

// Find devices associated with these users
AVQuery *pushQuery = [AVInstallation query];
[pushQuery whereKey:@"user" matchesQuery:userQuery];

// Send push notification to query
AVPush *push = [[AVPush alloc] init];
[push setQuery:pushQuery]; // Set our Installation query
[push setMessage:@"Free hotdogs at the AVOSCloud concession stand!"];
[push sendPushInBackground];
```

### 发送选项

除了发送一个文本信息之外，你还可以播放一个声音，设置badge数字或者其他想自定义的数据。你还可以设置一个消息的过期时间，如果对消息的时效性特别敏感的话。

#### 定制通知

如果你不仅想发送一条文本消息，你需要一个NSDictionary来打包想发送的数据。这里有一些保留字段有特殊含义需要说明下：

* alert: 推送消息的文本内容
* badge: (iOS only) app icon右上角的数字。可以设置一个值或者递增当前值。
* sound: (iOS only) 应用bundle里的声音文件名称。
* content-available: (iOS only) 如果你在使用Newsstand, 设置为1来开始一次后台下载。
* action: (Android only) 当消息收到的时候，触发的Intent名称。如果没有设置title或者alert，Intent将触发，但是不会显示通知给用户。
* title: (Android only) 显示在系统通知栏的标题。


例如，递增badge数字，并播放声音可以这样做:

```
NSDictionary *data = [NSDictionary dictionaryWithObjectsAndKeys:
    @"The Mets scored! The game is now tied 1-1!", @"alert",
    @"Increment", @"badge",
    @"cheering.caf", @"sound",
    nil];
AVPush *push = [[AVPush alloc] init];
[push setChannels:[NSArray arrayWithObjects:@"Mets", nil]];
[push setData:data];
[push sendPushInBackground];
```

当然，你还可以添加其他自定义的数据。你会在接收推送一节看到，当应用通过推送打开你的App的时候，你就可以访问这些数据。当你要在用户打开通知的时候显示一个不同的view controller的时候，这特别有用。

```
NSDictionary *data = [NSDictionary dictionaryWithObjectsAndKeys:
    @"Ricky Vaughn was injured in last night's game!", @"alert",
    @"Vaughn", @"name",
    @"Man bites dog", @"newsItem",
    nil];
AVPush *push = [[AVPush alloc] init];
[push setQuery:injuryReportsQuery];
[push setChannel:@"Indians"];
[push setData:data];
[push sendPushInBackground];
```


#### 设置过期日期

当设备关闭或者无法连接到网络的时候，推送通知就无法被送达。如果你有一条时间敏感的推送通知，不希望在太长时间后被用户读到，那么可以设置一个过期时间来避免打扰用户。

AVPush提供了两个方法来设置通知的过期日期，首先是expireAtDate：接收NSDate来告诉LeanCloud 不要再去发送通知。

```
NSDateComponents *comps = [[NSDateComponents alloc] init];
[comps setYear:2013];
[comps setMonth:10];
[comps setDay:12];
NSCalendar *gregorian =
  [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
NSDate *date = [gregorian dateFromComponents:comps];

// Send push notification with expiration date
AVPush *push = [[AVPush alloc] init];
[push expireAtDate:date];
[push setQuery:everyoneQuery];
[push setMessage:@"Season tickets on sale until October 12th"];
[push sendPushInBackground];
```

这个方法有个隐患，因为设备的时钟是无法保证精确的，你可能得到错误的结果。因此，AVPush还提供了expireAfterTimeInterval方法，接收NSTimeInterval对象。通知将在指定间隔时间后失效：

```
// Create time interval
NSTimeInterval interval = 60*60*24*7; // 1 week

// Send push notification with expiration interval
AVPush *push = [[AVPush alloc] init];
[push expireAfterTimeInterval:interval];
[push setQuery:everyoneQuery];
[push setMessage:@"Season tickets on sale until October 18th"];
[push sendPushInBackground];
```



#### 指定设备平台

跨平台的应用，可能想指定发送的平台，比如ios或者android:

```
AVQuery *query = [AVInstallation query];
[query whereKey:@"channels" equalTo:@"suitcaseOwners"];

// Notification for Android users
[query whereKey:@"deviceType" equalTo:@"android"];
AVPush *androidPush = [[AVPush alloc] init];
[androidPush setMessage:@"Your suitcase has been filled with tiny robots!"];
[androidPush setQuery:query];
[androidPush sendPushInBackground];

// Notification for iOS users
[query whereKey:@"deviceType" equalTo:@"ios"];
AVPush *iOSPush = [[AVPush alloc] init];
[iOSPush setMessage:@"Your suitcase has been filled with tiny apples!"];
[iOSPush setChannel:@"suitcaseOwners"];
[iOSPush setQuery:query];
[iOSPush sendPushInBackground];
```

### 定时推送

请进入消息推送的web管理平台，可以做到定时推送（延迟或者指定时间）。


### 接收推送通知

正如前面定制通知一节提到，你可以随通知发送任意的数据。我们使用这些数据修改应用的行为，当应用是通过通知打开的时候。例如，当打开一条通知告诉你有一个新朋友的时候，这时候如果显示一张图片会非常好。

由于Apple的对消息大小的限制，请尽量缩小要发送的数据大小，否则可能被截断：

```
NSDictionary *data = @{
  @"alert": @"James commented on your photo!",
  @"p": @"vmRZXZ1Dvo" // Photo's object id
};
AVPush *push = [[AVPush alloc] init];
[push setQuery:photoOwnerQuery];
[push setData:data];
[push sendPushInBackground];
```

### 响应通知数据

当应用是被通知打开的时候，, 你可以通过`application:didFinishLaunchingWithOptions: `方法的launchOptions dictionary访问到数据：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  . . .
  // Extract the notification data
  NSDictionary *notificationPayload = launchOptions[UIApplicationLaunchOptionsRemoteNotificationKey];

  // Create a pointer to the Photo object
  NSString *photoId = [notificationPayload objectForKey:@"p"];
  AVObject *targetPhoto = [AVObject objectWithoutDataWithClassName:@"Photo"
                                                          objectId:photoId];

  // Fetch photo object
  [targetPhoto fetchIfNeededInBackgroundWithBlock:^(AVObject *object, NSError *error) {
    // Show photo view controller
    if (!error && [AVUser currentUser]) {
      PhotoVC *viewController = [[PhotoVC alloc] initWithPhoto:object];
      [self.navController pushViewController:viewController animated:YES];
    }
  }];
}
```

如果当通知到达的时候，你的应用已经在运行，那么你可以通过`application:didReceiveRemoteNotification:fetchCompletionHandler:`方法的userInfo dictionary访问到数据：

```
- (void)application:(UIApplication *)application
      didReceiveRemoteNotification:(NSDictionary *)userInfo
            fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))handler {
  // Create empty photo object
  NSString *photoId = [userInfo objectForKey:@"p"];
  AVObject *targetPhoto = [AVObject objectWithoutDataWithClassName:@"Photo"
                                                          objectId:photoId];

  // Fetch photo object
  [targetPhoto fetchIfNeededInBackgroundWithBlock:^(AVObject *object, NSError *error) {
    // Show photo view controller
    if (error) {
      handler(UIBackgroundFetchResultFailed);
    } else if ([AVUser currentUser]) {
      PhotoVC *viewController = [[PhotoVC alloc] initWithPhoto:object];
      [self.navController pushViewController:viewController animated:YES];
    } else {
      handler(UIBackgroundModeNoData);
    }
  }];
}
```


你可以阅读[Apple本地化和推送的文档](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Introduction.html#//apple_ref/doc/uid/TP40008194-CH1-SW1)来更多地了解推送通知。

### 跟踪推送和 app 的打开情况

通过AVAnalytics你可以跟踪通知和应用的打开情况。添加下列代码到上面例子中的`application:didFinishLaunchingWithOptions: `方法来收集打开信息：

```
if (application.applicationState != UIApplicationStateBackground) {
  // Track an app open here if we launch with a push, unless
  // "content_available" was used to trigger a background push (introduced
  // in iOS 7). In that case, we skip tracking here to avoid double
  // counting the app-open.
  BOOL preBackgroundPush = ![application respondsToSelector:@selector(backgroundRefreshStatus)];
  BOOL oldPushHandlerOnly = ![self respondsToSelector:@selector(application:didReceiveRemoteNotification:fetchCompletionHandler:)];
  BOOL noPushPayload = ![launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
  if (preBackgroundPush || oldPushHandlerOnly || noPushPayload) {
    [AVAnalytics trackAppOpenedWithLaunchOptions:launchOptions];
  }
}
```


传递nil或者空白的参数给`trackAppOpenedWithLaunchOptions:`方法只是统计一次标准的应用打开事件 (比如不是通过通知打开的应用）。

你可以在[请求分析](/apistat.html?appid={{appid}}#/_apiRequest)菜单里看到通知和App的打开情况。


Application opens and push-related open rates will be available in your application's dashboard.

请注意，如果你的应用正在运行或者在后台，`application:didReceiveRemoteNotification:`方法将会处理收到的推送通知。

***如果您的app处于运行状态，iOS系统将不会在系统的通知中心显示推送消息，您可以使用`UILocalNotification`展示一个通知给用户。***

如果应用在后台，并且用户点击了通知，那么应用将被带到前台可视，为了跟踪这种通过通知打开应用的情况，你需要在跟踪代码里多作一个检查：

```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  if (application.applicationState == UIApplicationStateActive) {
    // 此处可以写上应用激活状态下接收到通知的处理代码，如无需处理可忽略
  } else {
    // The application was just brought from the background to the foreground,
    // so we consider the app as having been "opened by a push notification."
    [AVAnalytics trackAppOpenedWithRemoteNotificationPayload:userInfo];
  }
}
```
如果使用iOS7 push的新特性（包括新的"content-available" 功能），你需要实现iOS7新加的方法：
```
- (void)application:(UIApplication *)application
        didReceiveRemoteNotification:(NSDictionary *)userInfo
        fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
  if (application.applicationState == UIApplicationStateActive) {
    // 此处可以写上应用激活状态下接收到通知的处理代码，如无需处理可忽略
  } else {
    [AVAnalytics trackAppOpenedWithRemoteNotificationPayload:userInfo];
  }
}
```

#### 跟踪本地通知 (iOS only)

为了统计跟踪本地通知消息，需要注意`application:didFinishLaunchingWithOptions:`和`application:didReceiveLocalNotification:`都会调用到，如果你实现了`application:didReceiveLocalNotification:`这个方法,要注意避免重复统计。

#### 清除 Badge

清除Badge数字的最好时机是打开App的时候。 设置当前installation的badge属性并保存到服务器:

```
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AVInstallation *currentInstallation = [AVInstallation currentInstallation];
  if (currentInstallation.badge != 0) {
    currentInstallation.badge = 0;
    [currentInstallation saveEventually];
  }
  // ...
}
```

清除Badge数字最相关的三个方法是`applicationDidBecomeActive:`, `application:didFinishLaunchingWithOptions:`和`application:didReceiveRemoteNotification:`。请阅读[ UIApplicationDelegate文档](http://developer.apple.com/library/ios/#DOCUMENTATION/UIKit/Reference/UIApplicationDelegate_Protocol/Reference/Reference.html)。


### Installation 自动过期和清理

我们根据Apple推送服务的反馈，将Installation设置为失效，失效后推送到该设备的消息就被忽略。当失效时间超过60天，并且用户没有再次使用这个installation，我们会删除该installation；在60天内，用户如果再次使用这个Installation，将自动启用Installation并设置为有效状态，并继续推送消息给该设备。

## Android 消息推送

Android 推送功能除了需要必须的avoscloud.jar以外，还需要额外的avospush.jar。

Android 消息推送有专门的Demo，请见[AVOSCloud-Push](https://github.com/avoscloud/Android-SDK-demos/tree/master/AVOSCloud-Push)项目。

### Installation

当您的app安装在用户设备后，如果要使用消息推送功能，LeanCloud SDK会自动生成一个Installation对象。Installation对象包含了推送所需要的所有信息。您可以使用Android SDK，通过installation对象进行消息推送。Installation对象本质上代表了设备安装您的App的一个安装信息。

#### 保存 installation

您可以通过以下代码保存您的installation id。如果您的系统之前还没有installation id, 系统会为您自动生成一个。如果您的app卸载后，installation id也将会被删除。


```
AVInstallation.getCurrentInstallation().saveInBackground();

```

### 订阅频道

你的App可以订阅某个频道的消息，只要在保存Installation之前调用`PushService.subscribe`方法：

```
        // set a default callback. It's necessary for current SDK.
        // 在v2.0以后的版本请务必添加这段代码，以避免推送无法成功达到客户端的问题
        PushService.setDefaultPushCallback(this, PushDemo.class);
        PushService.subscribe(this, "public", PushDemo.class);
        PushService.subscribe(this, "private", Callback1.class);
        PushService.subscribe(this, "protected", Callback2.class);
```

第一个参数是当前的context，第二个参数是频道名称，第三个参数是回调对象的类，回调对象是指用户点击通知栏的通知进入的Activity页面。

退订频道也很简单：

```
  PushService.unsubscribe(context, "protected");
  //退订之后需要重新保存Installation
  AVInstallation.getCurrentInstallation().saveInBackground();
```


### 推送消息

#### 配置

请确保您的AndroidManifest.xml 包含如下内容
```
<service android:name="com.avos.avoscloud.PushService"/>

```

同时设置了必要的权限

```
  <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

为了让App能在关闭的情况下也可以收到push，你需要在`<application>`中加入：

```
        <receiver android:name="com.avos.avoscloud.AVBroadcastReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.USER_PRESENT" />
            </intent-filter>
        </receiver>
```

#### 推送给所有的设备

```
AVPush push = new AVPush();
JSONObject object = new JSONObject();
object.put("alert", "push message to android device directly");
push.setPushToAndroid(true);
push.setData(object);
push.sendInBackground(new SendCallback() {
    @Override
    public void done(AVException e) {
        if (e == null) {
            // push successfully.
        } else {
            // something wrong.
        }
    });

```

#### 发送给特定的用户

* 发送给public频道的用户

```
AVQuery pushQuery = AVInstallation.getQuery();
pushQuery.whereEqualTo("channels", "public");
AVPush push = new AVPush();
push.setQuery(pushQuery);
push.setMessage("Push to channel.");
push.setPushToAndroid(true);
push.sendInBackground(new SendCallback() {
    @Override
    public void done(AVException e) {
        if (e == null) {

        }   else {

        }
    }
});

```


* 发送给某个installation id的用户，通常来说，你会将AVInstallation关联到设备的登陆用户AVUser上作为一个属性，然后就可以通过下列代码查询installationId的方式来发送消息给特定用户，实现类似私信的功能：

```

AVQuery pushQuery = AVInstallation.getQuery();
pushQuery.whereEqualTo("installationId", THE_INSTALLATION_ID);
AVPush.sendMessageInBackground("message to installation",  pushQuery, new SendCallback() {
    @Override
    public void done(AVException e) {

    }
});

```

在2.6.7以后，我们加入了通过CQL来筛选推送目标的功能，主要代码如下：
```
    AVPush push = new AVPush();
    JSONObject data =
        new JSONObject(
            "{\"action\": \"com.avos.UPDATE_STATUS\", \"name\": \"Vaughn\", \"newsItem\": \"Man bites dog\"  }");
    push.setData(data);
    String installationId = AVInstallation.getCurrentInstallation().getInstallationId();
    push.setCloudQuery("select * from _Installation where installationId ='" + installationId
        + "'");
    push.sendInBackground(new SendCallback() {

      @Override
      public void done(AVException e) {

      }
    });
  
```
*注：CQL与AVQuery同时只能设置一个，并且在设置CQL时，请通过CQL来设置目标机器的类型(ios,android,wp)*
#### 自定义 Receiver

如果您想推送消息，但不显示在Andoid系统的通知栏中，而是执行应用程序预定义的逻辑，则可以发送类似下列这样的请求

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels":[ "public"],
        "data": {
          "action": "com.avos.UPDATE_STATUS"
          "name": "LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push
```

请注意：**如果您使用自定义的Receiver，发送的消息必须带action，并且其值在receiver配置的<intent-filter>列表里存在，比如这里的'com.avos.UPDATE_STATUS'**

您需要在您的Android项目中添加如下功能

AndroidManifest.xml中声明您的receiver

```
        <receiver android:name="com.avos.avoscloud.PushDemo.MyCustomReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.USER_PRESENT" />
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
                <action android:name="com.avos.UPDATE_STATUS" />
            </intent-filter>
        </receiver>

```

其中 com.avos.avoscloud.PushDemo.MyCustomReceiver 是您的android的receiver类。

而<action android:name="com.avos.UPDATE_STATUS" /> 需要与push的data中指定的action相对应。


您的receiver可以按照如下方式实现


```
public class MyCustomReceiver extends BroadcastReceiver {
    private static final String TAG = "MyCustomReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        LogUtil.log.d(TAG, "Get Broadcat");
        try {
            String action = intent.getAction();
            String channel = intent.getExtras().getString("com.avos.avoscloud.Channel");
            JSONObject json = new JSONObject(intent.getExtras().getString("com.avos.avoscloud.Data"));

            Log.d(TAG, "got action " + action + " on channel " + channel + " with:");
            Iterator itr = json.keys();
            while (itr.hasNext()) {
                String key = (String) itr.next();
                Log.d(TAG, "..." + key + " => " + json.getString(key));
            }
        } catch (JSONException e) {
            Log.d(TAG, "JSONException: " + e.getMessage());
        }
    }
}
```
#### 跟踪 Android 推送和 app 的打开情况
您可以在订阅频道对应的 activity 中添加跟踪 app 打开情况的统计代码，您的 activity 可以按照如下方式实现 `onStart` 方法：

```
public class MyActivity extends Activity {
	@Override
	protected void onStart() {
		super.onStart();

		Intent intent = getIntent();
		AVAnalytics.trackAppOpened(intent);
	}
}
```

您可以在 [请求分析](/apistat.html?appid={{appid}}#/_appOpenWithPush) 菜单里看到通知和App的打开情况。


#### Installation自动过期和清理

每当用户打开App，我们都会更新该设备的Installation的updatedAt时间戳。当用户长期没有更新Installation的updatedAt时间戳，换句话说，就是用户长期没有打开App（默认是超过60天没有打开），这个Installation的valid将被设置为false，往这个Installation发送的消息将被忽略，直到用户以后某天打开App更新了updatedAt，valid将再次设置为true。

如果超过120天，用户仍然没有打开过应用，那么该Installation将被删除。不过您不需要担心，当用户再次打开App的时候，仍然会自动创建一个新的Installation用于推送。

## Windows Phone 8 消息推送

Windows Phone 8 的推送较为特殊，因为微软在设计的时候把推送消息定义为一个包含跳转页面信息的载体，比如微信推送：你单击微信发送的 Windows Phone 的 Toast 推送消息，单击进去之后，它不是打开微信的默认首页（假如叫做Main.xaml），而是进入某一个聊天的具体的页面（假如叫做chat.xaml）。这种场景微软是通过在推送消息里面包含了代码逻辑来实现的，比如要实现刚才这一套流程，微信服务端必须向微软的 MPNS 发送一个如下类似的消息（Http或者Https Post 请求）：

```
<?xml version="1.0" encoding="utf-8"?>
    <wp:Notification xmlns:wp="WPNotification">
        <wp:Toast>
            <wp:Text1>微信</wp:Text1>
            <wp:Text2>您有一条聊天消息</wp:Text2>
            <wp:Param>/chat.xaml?NavigatedFrom=Toast Notification</wp:Param>
        </wp:Toast>
    </wp:Notification>
```
所以在使用 LeanCloud 推送服务向 Windows Phone 8 平台推送的时候一定要对微软官方的推送有所了解，如果想深入了解，可以点击详细查看微软官方关于 [Windows Phone 8 推送的官方教程](http://msdn.microsoft.com/en-us/library/windows/apps/hh202967\(v=vs.105\).aspx)。

针对 Windows Phone 8 的特殊性，LeanCloud 采用了统一接口去处理，如下 C# 代码可以实现以上所说的功能：
在 LeanCloud 所有 .NET 语言 SDK 均可如下进行操作。（注：Unity 暂时不支持.Wait（）方法 和 await 关键字，所以它需要使用任务的链式表达，详情请查看 Unity 的文档。）

```
  AVPush avPush = new AVPush();
  avPush.Data = new Dictionary<string, object>();
  avPush.Data.Add("title", "微信");
  avPush.Data.Add("alert", "您有一条聊天消息");
  avPush.Data.Add("wp-param", "/chat.xaml?NavigatedFrom=Toast Notification");
  await avPush.SendAsync()；
```
### 推送给所有的设备

```
AVPush push = new AVPush();
push.Alert = "message to all devices.";
var task = push.SendAsync();
await task;
```
以上这段代码就可以实现向所有安装了当前App的设备推送消息。

### 发送给特定的用户
发送给public频道的用户：

```
 AVPush push = new AVPush();
 push.Alert = "message to public channel.";
 push.Query = new AVQuery<AVInstallation>().WhereEqualTo("channels", "public");
 var task = push.SendAsync();
 await task;
```


## 使用REST API 推送消息


### Installation

当您的app安装在用户设备后，如果要使用消息推送功能，LeanCloud SDK会自动生成一个installation对象。installation对象包含了推送所需要的所有信息。您可以使用REST API，通过installation对象进行消息推送。

#### 保存 installation

##### 保存 iOS 设备的 device token

iOS设备通常使用DeviceToken来惟一标识一台设备。

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "deviceType": "ios",
        "deviceToken": "abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789",
        "channels": [
          "public", "protected", "private"
        ]
      }' \
  https://leancloud.cn/1.1/installations
```

##### 保存 Android 设备的 installId

对于Android设备，AVOS SDK会自动生成uuid作为installId保存到LeanCloud. 您可以使用以下REST API保存Android设备的installation ID.

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "deviceType": "android",
        "installationId": "12345678-4312-1234-1234-1234567890ab",
        "channels": [
          "public", "protected", "private"
        ]
      }' \
  https://leancloud.cn/1.1/installations
```

##### 订阅频道

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels": [
          "Giants"
        ]
      }' \
  https://leancloud.cn/1.1/installations/mrmBZvsErB
```

### 推送消息

#### 推送给所有的设备
```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "data": {
          "alert": "Hello From LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push
```

#### 发送给特定的用户

* 发送给public频道的用户

```
  curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "where":{
          "channels":
            {"$regex":"\\Qpublic\\E"}
        },
        "data": {
          "alert": "Hello From LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push
```

或者更简便的方式

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels":[ "public"],
        "data": {
          "alert": "Hello From LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push

```

* 发送给某个installation id的用户

```
  curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "where":{
            "installationId":"57234d4c-752f-4e78-81ad-a6d14048020d"
            },
        "data": {
          "alert": "Hello From LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push
```

* 推送给不活跃的用户

```
  curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "where":{
            "updatedAt":{
                "$lt":{"__type":"Date","iso":"2013-06-29T11:33:53.323Z"}
              }
        },
        "data": {
            "alert": "Hello From LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push
```

* 根据查询条件做推送：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "where": {
          "injuryReports": true
        },
        "data": {
          "alert": "Willie Hayes injured by own pop fly."
        }
      }' \
  https://leancloud.cn/1.1/push
```

**请注意，where条件查询的都是installations表。这里是假设installations表存储了injuryReports的布尔属性**

* 根据地理信息位置做推送：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "where": {
          "owner": {
            "$inQuery": {
              "location": {
                "$nearSphere": {
                  "__type": "GeoPoint",
                  "latitude": 30.0,
                  "longitude": -20.0
                },
                "$maxDistanceInMiles": 1.0
              }
            }
          }
        },
        "data": {
          "alert": "Free hotdogs at the avoscloud concession stand!"
        }
      }' \
  https://leancloud.cn/1.1/push
```

上面的例子假设installation有个owner属性指向_User表的记录，并且用户有个location属性是GeoPoint类型，我们就可以根据地理信息位置做推送。

#### 使用 CQL 查询推送

上述`where`的查询条件都可以使用 [CQL](./cql_guide.html) 查询替代，例如查询某个设备推送：

```
  curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "cql":"select * from _Installation where installationId='xxxxxxxxxxxxx'",
        "data": {
          "alert": "Hello From LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push
```

#### 推送消息属性

##### 消息过期

 过期时间，可以是绝对时间：
```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "expiration_time": "2013-12-04T00:51:13Z",
        "data": {
          "alert": "Season tickets on sale until December  4, 2013"
        }
      }' \
  https://leancloud.cn/1.1/push
```

也可以是相对时间（从推送API调用开始算起，结合push_time做定期推送）:
```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "push_time": "2013-11-28T00:51:13Z",
        "expiration_interval": 518400,
        "data": {
          "alert": "Season tickets on sale until December  4, 2013"
        }
      }' \
  https://leancloud.cn/1.1/push
```

##### 定制消息属性：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels": [
          "Mets"
        ],
        "data": {
          "alert": "The Mets scored! The game is now tied 1-1.",
          "badge": "Increment",
          "sound": "cheering.caf",
          "title": "Mets Score!"
        }
      }' \
  https://leancloud.cn/1.1/push
```


#### Android 自定义 receiver

* 推送消息，但不显示在Andoid系统的通知栏中，而是执行应用程序预定义的逻辑

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}"          \
  -H "X-AVOSCloud-Application-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels":[ "public"],
        "data": {
          "action": "com.avos.UPDATE_STATUS"
          "name": "LeanCloud."
        }
      }' \
  https://leancloud.cn/1.1/push
```

请注意：**如果您使用自定义的Receiver，发送的消息必须带action，并且值为'com.avos.UPDATE_STATUS'**

您需要在您的Android项目中添加如下功能

AndroidManifest.xml中声明您的receiver

```
        <receiver android:name="com.avos.avoscloud.PushDemo.MyCustomReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.USER_PRESENT" />
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
                <action android:name="com.avos.UPDATE_STATUS" />
            </intent-filter>
        </receiver>

```

其中 com.avos.avoscloud.PushDemo.MyCustomReceiver 是您的android的receiver类。

而<action android:name="com.avos.UPDATE_STATUS" /> 需要与push的data中指定的action相对应。


您的receiver可以按照如下方式实现


```
public class MyCustomReceiver extends BroadcastReceiver {
    private static final String TAG = "MyCustomReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        LogUtil.log.d(TAG, "Get Broadcat");
        try {
            String action = intent.getAction();
            String channel = intent.getExtras().getString("com.avos.avoscloud.Channel");
            JSONObject json = new JSONObject(intent.getExtras().getString("com.avos.avoscloud.Data"));

            Log.d(TAG, "got action " + action + " on channel " + channel + " with:");
            Iterator itr = json.keys();
            while (itr.hasNext()) {
                String key = (String) itr.next();
                Log.d(TAG, "..." + key + " => " + json.getString(key));
            }
        } catch (JSONException e) {
            Log.d(TAG, "JSONException: " + e.getMessage());
        }
    }
}
```












