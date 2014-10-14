
# LeanCloud FAQ

## 帐户和平台常见问题

### LeanCloud 部署在哪个云平台上

我们部署国内多个云计算平台提供商上，采用混合部署策略（虚拟机和实体机混合部署），并且都在双线机房内，保证您的应用的访问体验和可靠性。

### 获取客服支持有哪些途径

* 访问技术问答社区，提出问题
* 登录，进入控制台，选择上面菜单栏中的「帮助」，选择技术支持菜单，进入工单（ticket）系统，提交工单。
* 发送邮件到 support@avoscloud.com，获取帮助
* 紧急情况拨打客服电话： 010-82800646

### 计费是基于账号还是应用

我们的计费都是基于用户帐号，详细信息请看我们的[价格](/pricing.html)说明页面。

### 如果没有缴费会怎么样

我们会给每个账单的缴费一个月的缓冲期，请在收到账单的一个月内缴费即可。在这段时间内，我们会通过邮件或者电话等方式联系您，因此强烈建议你填写[开发者详细信息](/settings.html#/setting/info)。

### 如何付费

请使用[支付宝](http://alipay.com)充值到 LeanCloud 账户，我们将每个月自动从您的账户余额里扣除账单费用。每次扣费优先使用充值金额，其次是赠送金额。

### 付费可以开发票吗

可以的，请在`技术支持`里提出申请，并详细填写[开发者详细信息](/settings.html#/setting/info)里的地址一栏，我们将在扣费后一到两周内邮寄发票到您地址。发票抬头(Title)将使用您的公司名称（或者个人名称，如果没有提供公司名称的话）。

邮寄费用：

* 超过100元的免邮费。
* 100元以下到付。


### 哪里获取平台的更新信息

通常情况下，我们以一到两周的周期发布新版本更新，获取更新信息可以通过：

* 我们的[博客](http://blog.avoscloud.com/)，每次更新的详细信息都会发布在那里。
* [官方微博](http://weibo.com/avoscloud)
* 每个月初我们还会发送一个每月的更新摘要邮件到您的注册邮箱。
* 控制台右上方有一个`消息中心`功能，请注意查看新通知。

### API开放吗

我们的API完全开放，我们提供的SDK也都是基于开放API实现的，详细API请访问[REST API文档](https://leancloud.cn/docs/rest_api.html)。

### 提供哪些平台的SDK

目前官方提供的包括：

* iOS和OS X
* Android
* JavaScript

开源项目，来自第三方：

* [C#](https://github.com/freewing99/BaasReponsitory)
* [PHP](https://github.com/killme2008/avoscloud-php-library)

详情请到[SDK下载页面](https://leancloud.cn/docs/sdk_down.html)。

### iOS和Android是否可以使用同一个App

当然可以，同一个应用，您可以使用我们的sdk开发多个平台的版本，共享数据。


### Unity 3D支持如何

请在我们的 [SDK 下载页面](https://avoscloud.com/docs/sdk_down.html) 下载 Unity SDK。

### 开发文档有提供搜索功能吗

文档首页右上角就有搜索框，您也可以直接访问[搜索页面](https://leancloud.cn/search.html)。

## API相关

### 可以在线测试API吗

请访问 [API 在线测试工具](https://leancloud.cn/apionline/)。

### 403错误

403错误分为两类

* 错误信息为 `The user cannot be altered by a client without the session.`，表示用户没有登录，无法修改用户信息。
* 错误信息为 `Forbidden to write by class permissions.` 或者 `Forbidden to read by class permissions.`，表示想要修改的class表没有打开读或者写的权限。在数据管理平台，点击相应class，在右侧选择「其他」菜单进入权限管理可以设置class权限。

![image](images/permission.png)

### Unauthorized错误

表示应用API授权失败，请检查是否初始化了App Id和App Key。

* 如何初始化，请查看[快速入门](https://leancloud.cn/start.html)
* App Id 和 App key 在应用设置菜单里可以找到。

### 错误信息代码和详细解释在哪里

* [错误代码详解](./error_code.html)。
* iOS SDK 请看 [https://leancloud.cn/docs/api/iOS/docs/AVConstants.html](https://leancloud.cn/docs/api/iOS/docs/AVConstants.html)
* Android SDK 请看[https://leancloud.cn/docs/api/android/doc/index.html](https://leancloud.cn/docs/api/android/doc/index.html)

REST API 返回的错误信息跟 SDK 保持一致。

### 其他语言调用REST API如何编码参数

REST API 文档使用 curl 作为示范，其中 `--data-urlencode` 表示要对参数做 URL encode 编码，如果是 GET 请求，直接将 URL encode 过的参数通过`&`连接起来，放到 URL 的问号后。类似`https://leancloud.cn/1.1/login?username=xxxx&password=xxxxx`。

### 如何实现大小写不敏感的查询

目前不提供直接的支持，请走正则表达式查询的办法，可以参考[StackOverflow的帖子](http://stackoverflow.com/questions/1863399/mongodb-is-it-possible-to-make-a-case-insensitive-query)。

使用各平台 SDK 的 AVQuery 对象提供的 matchesRegex 功能（android SDK 里就是 `whereMatches` 方法）。


### 应用内用户的密码需要加密吗

您不需要加密密码，我们服务端自动帮你做了密码加密，基于随机生成的 salt。 用户如果忘记密码，我们提供了 requestResetPassword 功能（具体看 SDK 的 `AVUser` 方法），会发送邮箱到用户注册的邮箱，用户可以自行重设密码。 在整个过程中，密码都不会有明文保存的问题。密码也不会在客户端保存，只是会保存 sessionToken 标示用户登录状态。

### API调用次数有什么限制吗

我们每个月提供 500 万次的免费额度，超过的才收费。免费额度不会将完全免费的推送服务和统计服务计算进去。另外，对于免费版和专业版，默认情况下每个应用都是 1000 每秒的并发访问上限，如果需要更高上限，请跟我们联系。

对于从控制台或者 JavaScript SDK 上传文件（包括云代码内），我们限制每秒最多上传一个文件，一分钟内最多上传 30 个文件。


## 控制台相关

### 如何在App邮件内完全使用自己的品牌

请参考博客文章[自定义应用内用户重设密码和邮箱验证页面](http://blog.avoscloud.com/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

### 创建唯一索引失败

请确认想要创建索引的列没有已经存在的重复值。

### 如何上传文件

任何一个 class 如果有 File 类型的列，就可以直接在数据管理平台里上传文件到该列。如果没有，请自行创建列，指定类型为文件。

### 如何在应用之间共享数据

参见这篇[博客](http://blog.avoscloud.com/blog/2014/03/31/2014nian-3yue-di-4zhou-avos-cloud-geng-xin/)。

## iOS/OS X SDK

### 安装 Cocopods 失败怎么解决

推荐使用淘宝提供的 Gem 源，访问 [http://ruby.taobao.org/](http://ruby.taobao.org/)。

```
$ gem sources --remove https://rubygems.org/
$ gem sources -a http://ruby.taobao.org/
$ gem sources -l
*** CURRENT SOURCES ***
http://ruby.taobao.org
# 请确保只有 ruby.taobao.org
$ gem install cocoapods
```

### 编译失败

#### Symbol(s) not found x86_64

请使用 32 位模拟器进行编译和调试.

#### Undefined symbols for architecture

 一般是由于 Framework 的链接找不到造成的。建议用 CocoaPods 更新一下, 没有用 CocoaPods 的项目可以删除对 Framework 的引用,然后再重新加进来。

LeanCloud 依赖的 framework 包括：

* SystemConfiguration.framework
* MobileCoreServices.framework
* CoreTelephony.framework
* CoreLocation.framework

如果是 `for architecture arm64`，这是因为 Xcode 更新到 5.1 后, CocoaPods 没有及时更新对 64 位 CPU 的支持, 请参考[这里](http://stackoverflow.com/questions/19213782/undefined-symbols-for-architecture-arm64)

### 请求报错

请求返回的错误码的详细说明 请查阅[这个文档](https://leancloud.cn/docs/error_code.html)

### 地理位置查询错误

如果错误信息是类似 `can't find any special indices: 2d (needs index), 2dsphere (needs index), for 字段名`，这是表示用于查询的字段没有建立 2D 索引，可以在 Class 管理的「其他」菜单里找到「索引」管理，点击进入，找到字段名称，选择索引类型为「2dsphere」并创建。

![image](images/geopoint_faq.png)


## Android SDK

### 对 ·AVObject· 对象使用 ·getDate("createdAt")· 方法读取创建时间为什么会返回null

请用 `AVObject` 的 `getCreatedAt` 方法，获取 updatedAt 也类似。



## JavaScript SDK

### 有没有同步 API

JavaScript SDK 由于平台的特殊性（运行在单线程运行的浏览器或者 Node.js 环境中），不提供同步 API，所有需要网络交互的 API 都需要以 callback 的形式调用，我们提供了[Promise模式](https://leancloud.cn/docs/js_guide.html#promise)来减少 callback 嵌套过多的问题。

## 消息推送

### 推送的到达率如何

关于到达率这个概念，业界并没没有统一的标准，我们测试过，在线用户消息的到达率基本达到 100%。我们的SDK做了心跳和重连等功能，尽量维持对推送服务器的长连接存活，提升消息到达用户手机的实时性和可靠性。

### 推送是基于 XMPP 还是其他协议

老版本推送基于 XMPP 协议，v2.4.1 版本开始推送我们采用了 WebSocket 协议，方便支持多平台，包括将要推出的 Web 端消息推送功能。

### iOS 推送如何区分测试证书和生产证书

暂不提供在同一个 App 里同时上传测试证书和生产证书，推荐创建单独的测试 App，可以利用数据导出和导入来快速模拟生产环境。

### Android 消息接收能不能自定义 Receiver 不弹出通知

可以，请看[消息推送开发指南](https://leancloud.cn/docs/push_guide.html#%E8%87%AA%E5%AE%9A%E4%B9%89-receiver)。

如果要自定义 receiver，必须在消息的data里带上自定义的 action，我们在接收到消息后，将广播 action 为您定义的值的 intent 事件，您的 receiver 里也必须带上 `intent-filter` 来捕获该 action 值的 intent 事件。

## 统计

### 统计服务免费吗

统计服务完全免费，不占用每月的 API 免费额度。

### 统计服务支持哪些平台

目前支持

* iOS
* OS X
* Android

更多平台 SDK 正在开发中。

### 统计支持哪些发送策略

* 启动时发送（推荐使用），也是默认策略
* 批量发送
* 按最小间隔发送

可以在 `分析 -> Android(或者iOS)统计 -> 统计设置 -> 数据发送策略` 的菜单里实时修改这些策略。

## 云代码


### 定时器 crontab 的语法

请参考 [Quartz文档](http://www.quartz-scheduler.org/documentation/quartz-1.x/tutorials/crontrigger)。

### 云端代码为什么只支持 Node.js

因为我们官方目前只提供了 JavaScript SDK 和运行沙箱环境，暂时只支持 Node.js。未来可能会引入 PHP 等其他语言。

### 云代码如何上传文件

参考这篇[博客](http://blog.avoscloud.com/blog/2013/11/23/zai-yun-dai-ma-zhong-chu-li-shang-chuan-wen-jian/)，或者[云代码开发指南这部分](https://leancloud.cn/docs/cloud_code_guide.html#%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6)。

### 云代码中如何处理用户登录和 Cookie

参考这篇[博客](http://blog.avoscloud.com/blog/2013/12/16/zai-yun-dai-ma-zhong-chu-li-yong-hu-deng-lu/)，或者[云代码指南这部分内容](https://leancloud.cn/docs/cloud_code_guide.html#%E5%A4%84%E7%90%86%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E5%92%8C%E7%99%BB%E5%87%BA)。


### 云代码的二级域名多久生效

我们设置的 TTL 是 10 秒，但是因为使用 Amazon 的 DNS 服务，因此可能国内部分地区会有一定延迟，最迟应该在 24 小时内生效。如果没有，请及时联系我们处理。

### 云代码二级域名可以启用 HTTPS 吗

参考[博客](http://blog.avoscloud.com/blog/2013/12/20/wei-yun-dai-ma-tuo-guan-wang-zhan-qi-yong-https/)。

## 文件

### 文件存储有 CDN 加速吗？

有的，我们的文件存储目前由[七牛](http://qiniu.com)提供，都有 CDN 加速访问。

### 文件存储有大小限制吗？

没有。除了在浏览器里通过 JavaScript SDK 上传文件，或者通过我们网站直接上传文件的有 10M 的大小限制之外，其他 sdk 都没有限制。 JavaScript SDK 在 Node.js 环境也没有大小限制。

### 存储图片可以做缩略图等处理吗？

可以的，默认我们的 AVFile 类提供了缩略图获取方法，可以参见各个 SDK 的开发指南。如果您要自己处理，可以通过获取 AVFile 的 URL 属性，使用[七牛图片处理 API](http://docs.qiniu.com/api/v6/image-process.html)执行处理，例如添加水印、裁剪等。




