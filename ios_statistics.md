# iOS 统计 SDK 开发指南

## 建立 app，下载 SDK

您可以在leancloud.cn上创建新的应用，然后下载LeanCloud iOS SDK以及相应的demo。


## 创建 Xcode 工程，使用基本的统计功能


### 导入 SDK

您可以从 [SDK 下载页面](https://leancloud.cn/docs/sdk_down.html) 下载iOS的SDK。您也可以使用Podfile通过cocoapods下载我们的SDK，如

```sh
pod 'AVOSCloud'
```

### 使用统计功能

经过我们的一系列更新升级, 使用最新的SDK您不需要任何代码上的操作就可以使用统计的基本功能.
**统计功能为默认打开, 并且可以在线配置.** 您可以进入应用的 [分析 -> 统计设置（左下角） -> 数据发送策略](/stat.html?appid={{appid}}#/statconfig/trans_strategoy) 在线更改SDK端的数据报告发送策略.

* 建议使用AV_BATCH形式，减少App与网络的交互，为用户节约流量. 如果不设置channelId,默认会是@"App Store"渠道)


默认是`启动时发送`策略: 应用程序每次会在启动时会向服务器发送一次消息，在应用程序过程中产生的所有统计信息(包括自定义事件和本次使用时长)都会在下次启动时候发送。 如果应用程序启动时处在不联网状态，那么消息将会缓存在本地，下次再尝试发送。

如果需要修改渠道名称(channelId),请使用下面的方法：

```objc
[AVAnalytics setChannel:@"越狱市场"];
```


到此，您已经可以使用基本的统计功能。

###  页面访问路径统计

您可以统计每个View停留时长，请确保配对使用

```objc
- (void)viewWillAppear:(BOOL)animated
{
	[super viewWillAppear:animated];
  [AVAnalytics beginLogPageView:@"PageOne"];
}


- (void)viewWillDisappear:(BOOL)animated {
   [super viewWillDisappear:animated];
   [AVAnalytics endLogPageView:@"PageOne"];
}
```

如果您想自己传递时长，我们也提供了方法。 单位为秒.

```objc
+ (void)logPageView:(NSString *)pageName seconds:(int)seconds;
```


### 使用自定义事件
自定义事件方便您更加灵活的了解用户的行为,并且可以在控制台看到实时的数据.

但是要特别强调的是, 自定义事件名(`event_id`)请尽量保持其为静态值, 否则可能出现数目庞大的自定义事件列表,而无法达到了解与分析用户行为的目的.

例如, 如果要统计用户喜欢在每周几喜欢看做某件事情(比如查看彩票开奖), 我们应该实现比较合理, 统一用一个事件名(比如`check`), 然后给这个事件设置周几的`label`. 这样在控制台打开check的自定义事件统计中,就可以看到一周每天的分布情况了. 使用方法见下面的`多标签事件`.


您可以通过以下几个接口来调用事件记录：

#### 简单事件

```objc
+(void)event:(NSString *)event_id;
```

将统计event_id对应事件的发生次数，变化趋势，例如广告点击，短信数量等等。event_id为当前统计的事件ID，您可以在程序里直接定义，无需配置。
例如，监测应用程序里like的点击次数，事件ID为“like_click”。那么需要在程序里每次like点击后调用
```objc
[AVAnalytics event:@"like_click"]; // 通知服务器一个like点击事件。
```


#### 多标签事件

```objc
+ (void)event:(NSString *)event_id label:(NSString *)label;
```


除了能够统计event_id所对应事件的发生次数，变化趋势外，还能统计此事件中具体标签所占的比例，label为当前标签。
例如：在应用程序中，您可以定义一个事件，其ID为“like”，针对每个like事件，您可以根据用户的不同，定义不同的label，比如对于女性用户，您可以定义一个label_female ，对于男性用户您可以定义另一个label_male，通过我们的后台页面，您可以观察到不同性别的用户对于特定内容的like比例。

```objc
[AVAnalytics event:[NSString stringWithFormat:@"like"] label:@"label_female"]; //type是变量,表示用户类型
```


####  事件累计

在应用程序中某些自定义事件可能会被频繁触发，例如用户点击某个按钮，但是还是希望统计为一次事件。开发者可以在程序中维护一个计数器，这样某个事件被多次触发但只需要生成一个消息，这个消息中包括该事件被触发的次数。为了支持这个功能，这里我们定义了两个接口：

```objc
+ (void)event:(NSString *)event_id acc:(NSInteger)acc;
+ (void)event:(NSString *)event_id label:(NSString *)label acc:(NSInteger)acc;
```

参数acc是对应事件 （和对应标签）被触发的次数。


#### 使用多渠道统计

您可以选择多种程序分发渠道, 例如某Cydia源，或其他网站，可以为不同渠道(提供不同)的软件包进行分发渠道的设定，以便统计终端用户得到程序的来源。 如果您只有苹果官方App Store一个分发渠道，则不再需要做设定， LeanCloud SDK默认渠道标记为`App Store`。

例如您在360发布,需要统计360渠道:
```objc
[AVAnalytics startWithReportPolicy:AV_BATCH channelId:@"360"];
```

例如您在小米发布,需要统计同步推渠道:
```objc
[AVAnalytics startWithReportPolicy:AV_BATCH channelId:@"xiaomi"];
```

### 获取在线参数

从1.4.2 LeanCloud SDK开始，您可以在您的应用中动态添加您的在线参数。通过在线参数，您可以控制您的应用行为，而不需要再次发布您的应用。 您可以控制台某个应用的`组件 -> 自定义参数设置`中配置你的自定义在线参数，在线配置参数会在SDK启动后，自动从后台定义的数据表格中获取。从1.4.2开始，您可以使用带回调版本的在线查询API，比如如果您在后台定义了在线参数<k1, v1>，您可以通过以下方法来获取

```objc
    [AVAnalytics updateOnlineConfigWithBlock:^(NSDictionary *dict, NSError *error) {
         if (error == nil) {
             // read your online config from dict. value of dict["k1"] should be v1
         }
    }];
```

如果想要测试时发送实时的数据, 也可以在上面这个block中设置

### 提供开发者选项，控制是否启用统计

从2.5.5 LeanCloud SDK开始，您可以添加如下代码禁止统计功能（默认启用统计功能），这样可以避免测试数据污染线上数据。

```objc
[AVAnalytics setAnalyticsEnabled:NO];
```

