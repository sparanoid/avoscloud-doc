# iOS / OS X 指南

如果您还没有安装 LeanCloud iOS SDK，请按照[快速入门引导](/start.html)来获得我们的 SDK，并在 Xcode 中熟悉和允许示例代码。我们的 SDK 支持 iOS 4.3 及更高版本。

如果您希望从项目中学习，请前往 [iOS-SDK-Demos](https://github.com/avoscloud/iOS-SDK-demos) 。

## 介绍

LeanCloud 是一个完整的平台解决方案，为您的应用提供全方位的后端服务。我们的目标是让你不需要进行后端开发及服务器运维等工作就可以开发和发布成熟的应用。

如果你熟悉像 Ruby on Rails 这样的 Web 框架，LeanCloud 将会十分容易上手。我们在设计 LeanCloud 时应用了许多与之相同的原则。如果你之前使用过 Parse 或类似的后端服务，会发现我们在设计 API 时尽可能与之保持兼容，让应用非常容易从其他服务迁移到 LeanCloud，开发者在使用我们的 SDK 时也会得心应手。

## 使用 Cocopods 安装SDK

在[快速入门](https://leancloud.cn/start.html)里可以看到怎么在你的项目里安装SDK。

[Cocopods](http://beta.cocoapods.org/)是一个很好的依赖管理工具，下面我们大概介绍下怎么安装：

* 首先确保您的机器安装了Ruby，一般来说，如果安装了XCode，都会自动安装了Ruby
* 我们建议使用淘宝提供的[Gem源](http://ruby.taobao.org/)，在终端执行下列命令：

```sh
$ gem sources --remove https://rubygems.org/
$ gem sources -a http://ruby.taobao.org/
$ gem sources -l
*** CURRENT SOURCES ***
http://ruby.taobao.org
#请确保下列命令的输出只有ruby.taobao.org
$ gem install rails
```

* 通过下列命令，安装(或者更新)cocopods（可能需要输入登录密码）：

```sh
sudo gem install cocoapods
```

* 在你的项目根目录创建一个`Podfile`文件，添加下列内容：

```sh
pod 'AVOSCloud'
```
如果 SNS 组件的相关功能，可以添加：

```sh
pod 'AVOSCloudSNS'
```

* 执行命令`pod install`安装SDK。

您还可以参考这篇文章 [《CocoaPods安装和使用教程》](http://code4app.com/article/cocoapods-install-usage)



## 应用

在 LeanCloud 的每个应用有自己的 ID 和客户端密钥，在客户端代码中应该用他们来初始化 SDK。

LeanCloud 的每一个账户都可以创建多个应用。同一个应用可以分别在测试环境和生产环境部署不同的版本。

## 对象

### AVObject

在 LeanCloud 上，数据存储是围绕 `AVObject` 进行的。每个 `AVObject` 都包含了与 JSON 兼容的 key-value 对应的数据。数据是 schema-free 的，你不需要在每个 AVObject 上提前指定存在哪些键，只要直接设定对应的 key-value 即可。

例如，您需要检测一个游戏中的分数对象。建立一个独立的 `AVObject` 即可 ：

```objc
score: 1337, playerName: "Steve", cheatMode: false
```

key 必须是字母数字或下划线组成的字符串，自定义的键不能以`__`开头。值可以是字符串，数字，布尔值，甚至数组和字典。

**注意: 在iOS SDK中, `uuid`也是保留字段, 不能作为key来使用.**

每个 `AVObject` 都必须有一个类（Class）名称，以便于您区分不同类型的数据。例如，我们可以将对应的分数称为 GameScore。我们建议的您将类和 key 按照 `NameYourClassesLikeThis` 以及 `nameYourKeysLikeThis` 这样的惯例命名。

### 保存对象

接下来，你需要将上文中的 `GameScore` 存储到 LeanCloud 的服务。LeanCloud 的相关接口和 `NSMutableDictionary` 类似，但只有调用 `save` 方法时才会实际保存到服务器：

```objc
AVObject *gameScore = [AVObject objectWithClassName:@"GameScore"];
[gameScore setObject:[NSNumber numberWithInt:1337] forKey:@"score"];
[gameScore setObject:@"Steve" forKey:@"playerName"];
[gameScore setObject:[NSNumber numberWithBool:NO] forKey:@"cheatMode"];
[gameScore save];
```

在运行此代码后，您应当了解保存动作是否已经生效 。为了确保数据被保存，您可以在 LeanCloud 上的[数据管理](/data.html?appid={{appid}})中查看您应用的数据。

您应该可以在 `GameScore` 数据列表中看到下面的对象：

```objc
objectId: "51a90302e4b0d034f61623b5", score: 1337, playerName: "Steve", cheatMode: false,
createdAt:"2013-06-01T04:07:30.32Z", updatedAt:"2013-06-01T04:07:30.32Z"
```

此处有两件事情需要特别注明。
首先，在运行此代码之前，您不必配置或设置一个称为「GameScore」的新类。LeanCloud 会自动创建这个类。

此外，为了更方便的使用 LeanCloud，还有其它几个字段您不需要事先指定。`objectId` 是为每个对象自动生成的唯一的标识符；`createdAt` 和 `updatedAt` 分别代表每个对象在 LeanCloud 中创建和最后修改的时间并会被自动填充。
在您执行保存操作之前，这些字段不会被自动保存到 `AVObject` 中。


### 检索对象

如果你觉得将数据保存到 LeanCloud 是简洁而优雅的，获取数据更是如此。如果已知 `objectId`，就可以使用 `AVQuery` 得到对应的 `AVObject`：

```objc
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
AVObject *gameScore = [query getObjectWithId:@"51a90302e4b0d034f61623b5"];
```

使用 `objectForKey` 来得到属性的值，方法如下：

```objc
int score = [[gameScore objectForKey:@"score"] intValue];
NSString *playerName = [gameScore objectForKey:@"playerName"];
BOOL cheatMode = [[gameScore objectForKey:@"cheatMode"] boolValue];
```

其中有三个特殊属性可以这样得到：

```objc
NSString *objectId = gameScore.objectId;
NSDate *updatedAt = gameScore.updatedAt;
NSDate *createdAt = gameScore.createdAt;
```

如果需要刷新特定对象的最新数据，可以调用refresh方法，如下 ：

```objc
[myObject refresh];
```

### 后台运行

在 iOS 或 OS X 中，大部分代码是在主线程中运行的。不过，在主线程中访问网络时，您的应用程序可能会常常遇到卡顿或者崩溃的现象。

由于 save 和 getObjectWithId 这两个方法会访问网络，所以不应当在主线程上运行。处理这种情况是件十分麻烦的事情。为此，LeanCloud 提供了辅助功能，能够覆盖绝大多数应用场景。

例如， 只需使用 saveInBackground，即可在后台线程中保存我们以前的 `AVObject`：

```objc
[gameScore saveInBackground];
```

这样，saveInBackground 的调用将立即返回，所以主线程不会被阻塞，应用会保持在响应状态。

通常情况下，我们都希望在操作完成后立即运行代码。这时您可以使用块（仅支持 iOS 4.0+ 或 OS X 10.6+）或回调方法。例如，如果您想在保存完成后运行一些代码：

```objc
[gameScore saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
  if (!error) {
    // The gameScore saved successfully.
  } else {
    // There was an error saving the gameScore.
  }
}];
```

或者您可以写成回调的方式

```objc
// First set up a callback.
- (void)saveCallback:(NSNumber *)result error:(NSError *)error {
  if (!error) {
    // The gameScore saved successfully.
  } else {
    // There was an error saving the gameScore.
  }
}

// Then, elsewhere in your code...
[gameScore saveInBackgroundWithTarget:self
                             selector:@selector(saveCallback:error:)];
```

LeanCloud 在网络接入时将不会阻塞调用线程，同时在主线程上块或回调将维持正常。这意味着，网络访问不会对 UI 产生不良影响，并且您仍然可以在回调中对 UI 进行操作。

AVQuery也遵循相同的模式。如果您想要从 GameScoreobject 获取并记录得分，同时确保不阻塞主线程：

```objc
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query getObjectInBackgroundWithId:@"51a90302e4b0d034f61623b5"
                             block:^(AVObject *gameScore, NSError *error) {
  if (!error) {
    // The get request succeeded. Log the score
    NSLog(@"The score was: %d", [[gameScore objectForKey:@"score"] intValue]);
  } else {
    // Log details of our failure
    NSLog(@"Error: %@ %@", error, [error userInfo]);
  }
}];
```

或者您可以写成回调的方式

```objc
// First set up a callback.
- (void)getCallback:(AVObject *)gameScore error:(NSError *)error {
  if (!error) {
    // The get request succeeded. Log the score
    NSLog(@"The score was: %d", [[gameScore objectForKey:@"score"] intValue]);
  } else {
    // Log details of our failure
    NSLog(@"Error: %@ %@", error, [error userInfo]);
  }
}

// Then, elsewhere in your code...
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query getObjectInBackgroundWithId:@"51a90302e4b0d034f61623b5"
                            target:self
                          selector:@selector(getCallback:error:)];
```

###离线存储对象

大多数保存功能可以立刻执行，并告知您的应用程序「保存完毕」。不过假设您不需要知道保存完成的时间，可以使用 saveEventually 作为替代品。

它的优点在于，如果用户目前尚未接入网络，saveEventually 将存储设备中的数据，并将在网络连接恢复后上传。如果您的应用在网络恢复之前就被关闭了，下一次打开应用程序 LeanCloud 会再次尝试连接。所有 saveEventually（deleteEventually）的相关调用将按照调用的顺序依次执行。因此，调用 saveEventually 的对象多次是安全的。

```objc
// Create the object.
AVObject *gameScore = [AVObject objectWithClassName:@"GameScore"];
[gameScore setObject:[NSNumber numberWithInt:1337] forKey:@"score"];
[gameScore setObject:@"Sean Plott" forKey:@"playerName"];
[gameScore setObject:[NSNumber numberWithBool:NO] forKey:@"cheatMode"];
[gameScore setObject:[NSArray arrayWithObjects:@"pwnage", @"flying", nil] forKey:@"skills"];
[gameScore saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {

    [gameScore setObject:[NSNumber numberWithBool:YES] forKey:@"cheatMode"];
    [gameScore setObject:[NSNumber numberWithInt:1338] forKey:@"score"];
    [gameScore saveEventually];
}];
```



### 更新对象

更新一个对象很简单。仅仅需要更新一些属性并调用一个保存方法。例如：

```objc
// Create the object.
AVObject *gameScore = [AVObject objectWithClassName:@"GameScore"];
[gameScore setObject:[NSNumber numberWithInt:1337] forKey:@"score"];
[gameScore setObject:@"Steve" forKey:@"playerName"];
[gameScore setObject:[NSNumber numberWithBool:NO] forKey:@"cheatMode"];
[gameScore setObject:[NSArray arrayWithObjects:@"pwnage", @"flying", nil] forKey:@"skills"];
[gameScore saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {

    // Now let's update it with some new data. In this case, only cheatMode and score
    // will get sent to the cloud. playerName hasn't changed.
    [gameScore setObject:[NSNumber numberWithBool:YES] forKey:@"cheatMode"];
    [gameScore setObject:[NSNumber numberWithInt:1338] forKey:@"score"];
    [gameScore saveInBackground];
}];
```

客户端会自动计算出哪些数据已经改变，并且将修改过的的字段发送到 LeanCloud。您不必担心未更新的数据产生变动。

### 计数器

上面是一个常见的使用案例。在这个例子中 `score` 字段是一个计数器，我们需要不断更新玩家的最新得分。使用上述方法之后，这个计数器运行良好，但如果有多个客户端试图更新同一个计数器，上面的方法就十分繁琐并且容易出现问题。
为了帮助计数器类的数据存储，LeanCloud 在任何数字字段中提供原子递增（或递减）的方法。故相同的更新可以改写为：

```objc
[gameScore incrementKey:@"score"];
[gameScore saveInBackground];
```

您也可以使用 incrementKey: byAmount 来增加字段中的数字。

对于计数器，在某些时候，您可能希望马上知道目前后端最新的数据，而不再使用一次fetch操作，LeanCloud为您提供了一个fetchWhenSave属性，您可以设置此属性为true，当您进行保存操作时，LeanCloud会自动返回目前最新的数值。


### 数组

为了更好的存储数组数据，LeanCloud 提供了三种不同的操作来自动变更一个数组字段：

* addObject：forKey： 和 addObjectsFromArray：forKey 将指定的对象附加到数组的末尾。
* addUniqueObject：forKey 和 addUniqueObjectsFromArray：forKey：如果您不确定某个对象是否已经包含在一个数组字段中，您可以使用此操作将对象添加到对应字段。插入的位置是随机的。
* removeObject：forKey：和 removeObjectsInArray：forKey：从一个数组字段中，将删除每个给定对象的所有实例。

例如，我们可以将对象添加到“技能”字段，像这样：

```objc
[gameScore addUniqueObjectsFromArray:[NSArray arrayWithObjects:@"flying", @"kungfu", nil] forKey:@"skills"];
[gameScore saveInBackground];
```

###删除对象

从 LeanCloud 中删除一个对象：

```objc
[myObject deleteInBackground];
```

如果您想通过一个回调来确认删除操作，您可以使用方法 `deleteInBackgroundWithBlock:` 或：`deleteInBackgroundWithTarget: selector:`。如果您想强制在当前线程执行，您可以使用方法 `delete`。

您可以使用方法 removeObjectForKey：从一个对象中删除单个属性。

```objc
// After this, the playerName field will be empty
[myObject removeObjectForKey:@"playerName"];

// Saves the field deletion to LeanCloud
[myObject saveInBackground];
```



### 关系型数据

对象可以与其他对象的产生关系。为了模拟这种行为，任何 AVObject 均可以作为另一个 `AVObject` 的属性，在其他AVObjects中使用。在内部，LeanCloud 框架会将引用到的对象储存到同一个地方，以保持一致性。

关系最主要的特性在于它可以非常容易动态扩展（对于数组而言），同时它具备很好的查询能力，数组在查询上的功能比较有限，而且使用起来不容易。数组和关系都可以用来存储一对多的映射。

例如在一个博客应用中，每个评论可能对应一个文章。

要创建一篇有一个评论的文章，您可以使用如下代码：

```objc
// Create the post
AVObject *myPost = [AVObject objectWithClassName:@"Post"];
[myPost setObject:@"I'm Smith" forKey:@"title"];
[myPost setObject:@"Where should we go for lunch?" forKey:@"content"];

// Create the comment
AVObject *myComment = [AVObject objectWithClassName:@"Comment"];
[myComment setObject:@"Let's do Sushirrito." forKey:@"content"];

// Add a one-one relation between the Post and Comment
[myComment setObject:myPost forKey:@"parent"];

// This will save both myPost and myComment
[myComment saveInBackground];
```

您还可以只使用 `ObjectID` 来关联对象，如下：

```objc
// Add a relation between the Post with objectId "51a902d3e4b0d034f6162367" and the comment
[myComment setObject:[AVObject objectWithoutDataWithClassName:@"Post" objectId:@"51a902d3e4b0d034f6162367"]
              forKey:@"parent"];
```

默认情况下，获取对象时，相关的 `AVObject` 并没有一起获取。在被获取之前，这些对象的属性不能被访问，像这样：

```objc
AVObject *post = [fetchedComment objectForKey:@"parent"];
[post fetchIfNeededInBackgroundWithBlock:^(AVObject *object, NSError *error) {
  NSString *title = [post objectForKey:@"title"];
}];
```

您也可以使用 `AVRelation` 对象来模拟多对多的关系。这里的工作原理类似于 `AVObject` 中的 `Array`。二者不同之处在于，您不需要即时下载所有关系中的对象。这意味着， 使用 `AVRelation` 可以比 `AVObject` 中的 `Array` 扩展更多的对象 。

例如，一个用户可能有很多喜欢的文章。在这个场景中，您可以使用 relationforKey 为一个用户的喜欢行为存储一组文章。按顺序添加一篇文章到列表中后，代码应当类似于：

```objc
AVUser *user = [AVUser currentUser];
AVRelation *relation = [user relationforKey:@"likes"];
[relation addObject:post];
[user saveInBackground];
```

您可以从 `AVRelation` 中移除一篇喜欢的「文章」：

```objc
[relation removeObject:post];
```

默认情况下，这个关系中的对象列表不会被下载。你可以调用查询返回的 `AVQuery` 的 `findObjectsInBackgroundWithBlock` 来获得文章列表，代码看起来像这样：

```objc
[[relation query] findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
  if (error) {
     // There was an error
  } else {
    // objects has all the Posts the current user liked.
  }
}];
```

如果您只想要文章中的一个子集，您可以对 AVQuery 添加额外的限制，像这样：

```objc
AVQuery *query = [relation query];
// Add other query constraints.
```

在某些时候，您可能会希望进行反向查询，比如，您想查询您的文章被哪些用户喜欢过，我们为您提供了反向查询的功能，如

```objc
AVUser * user = [AVUser currentUser];
AVRelation * relation = [user relationforKey:@"myLikes"];
AVObject * post = [AVObject objectWithClassName:@"post"];
[post setObject:@"article content" forKey:@"content"];
[post save];
[relation addObject:post];
[user save];


AVQuery * query = [AVRelation revreseQuery:user.className relationKey:@"myLikes" childObject:post];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
   // get user list
}];

```

对于更多 `AVQuery` 的细节，请看看本指南的「查询部分」。`AVRelation` 的行为接近于一个 `AVObject` 中的 `Array`，所以在对象数组上的任何操作都同样适用于 `AVRelation`。

### 数据类型

到目前为止，我们已经用过数据类型有 `NSString`，`NSNumber`， 以及 `AVObject`。LeanCloud 还支持  `NSDate`，`NSData`，和 `NSNull`。
你可以嵌套 `NSDictionary` 和 `NSArray` 这两类对象，这样就可在一个单独的 `AVObject` 中储存更多的结构化数据。

以下是一些例子：

```objc
NSNumber *number = [NSNumber numberWithInt:42];
NSString *string = [NSString stringWithFormat:@"the number is %i", number];
NSDate *date = [NSDate date];
NSData *data = [@"foo" dataUsingEncoding:NSUTF8StringEncoding];
NSArray *array = [NSArray arrayWithObjects:string, number, nil];
NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:number, @"number",
                                                                      string, @"string",
                                                                      nil];
NSNull *null = [NSNull null];

AVObject *bigObject = [AVObject objectWithClassName:@"BigObject"];
[bigObject setObject:number     forKey:@"myNumber"];
[bigObject setObject:string     forKey:@"myString"];
[bigObject setObject:date       forKey:@"myDate"];
[bigObject setObject:data       forKey:@"myData"];
[bigObject setObject:array      forKey:@"myArray"];
[bigObject setObject:dictionary forKey:@"myDictionary"];
[bigObject setObject:null       forKey:@"myNull"];
[bigObject saveInBackground];
```

我们不推荐在 `AVObject` 中使用 `NSData` 字段来储存大块的二进制数据，比如图片或者整个文件。每个 `AVObject` 的大小都不应超过128KB。如果你需要储存更多的数据，我们建议你使用 `AVFile`。更多细节可以查看相关指南。

如果你希望了解更多 LeanCloud 如何解析处理数据的信息，请查看我们的文档「数据与安全」一节。

## 查询

我们已经看到了，一个 `AVQuery` 如何通过 `getObjectWithId:` 从 LeanCloud 中检索单个 `AVObject`。 此外，还有许多种检索 `AVQuery` 数据的方法 —— 你可以一次检索许多对象，在你希望检索的对象上设定条件，自动缓存查询结果来避免你亲自写这部分的代码。当然除此之外，还有更多方法。

### 基本查询

在许多情况下，`getObjectInBackgroundWithId: block:` 并不足以找到目标对象。AVQuery 不仅仅可以检索单一对象，还允许以不同的方式来检索得到一个对象的列表。

一般的方式是创建一个 `AVQuery` 并设定相应的条件。然后可以用 `findObjectsInBackgroundWithBlock:` 或者 `findObjectsInBackgroundWithTarget: selector:` 来检索一个和响应 `AVObject` 匹配的 `NSArray`。

例如，如果你想要检索分数和特定的 `playername`，那么你可以使用方法 `whereKey: equalTo:` 来锁定一个键与其对应的值。

```objc
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query whereKey:@"playerName" equalTo:@"Smith"];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
  if (!error) {
    // The find succeeded.
    NSLog(@"Successfully retrieved %d scores.", objects.count);
  } else {
    // Log details of the failure
    NSLog(@"Error: %@ %@", error, [error userInfo]);
  }
}];
```

也可以写成回调的方式


```objc
// First set up a callback.
- (void)findCallback:(NSArray *)results error:(NSError *)error {
  if (!error) {
    // The find succeeded.
    NSLog(@"Successfully retrieved %d scores.", results.count);
  } else {
    // Log details of the failure
    NSLog(@"Error: %@ %@", error, [error userInfo]);
  }
}

// Then, elsewhere in your code...
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query whereKey:@"playerName" equalTo:@"Smith"];
[query findObjectsInBackgroundWithTarget:self
                                selector:@selector(findCallback:error:)];
```
`findObjectsInBackgroundWithBlock:` 和 `findObjectsInBackgroundWithTarget: selector:` 都可以保证在完成网络请求的同时不阻塞主线程中的 Block 和回调。

如果你已经在后台线程中，有一个相应的方法 `findObjects` 会阻塞调用进程：

```objc
// Only use this code if you are already running it in a background
// thread, or for testing purposes!
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query whereKey:@"playerName" equalTo:@"Smith"];
NSArray* scoreArray = [query findObjects];
```


### 查询约束

有几种方法可以为由 `AVQuery` 找到的对象添加约束。你可以通过 `whereKey: notEqualTo:` 来使用特定的键-值配对过滤对象。

```objc
[query whereKey:@"playerName" notEqualTo:@"Smith"];
```


你可以给出多个约束，这所有这些约束所匹配的对象会在结果中给出。
换句话说，它们就像是一个 AND 约束。

```objc
[query whereKey:@"playerName" notEqualTo:@"Smith"];
[query whereKey:@"playerAge" greaterThan:[NSNumber numberWithInt:18]];
```

你可以通过设置一个限制来控制获取结果的数量。默认情况下这个数值是100。从1到1000的限制都是被允许的。

```objc
query.limit = 10; // limit to at most 10 results
```
如果你想要获取一个的结果，更方便的选择是使用 `getFirstObject` 或者 `getFirstObjectInBackground`

```objc
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query whereKey:@"playerEmail" equalTo:@"dstemkoski@example.com"];
[query getFirstObjectInBackgroundWithBlock:^(AVObject *object, NSError *error) {
  if (!object) {
    NSLog(@"The getFirstObject request failed.");
  } else {
    // The find succeeded.
    NSLog(@"Successfully retrieved the object.");
  }
}];
```

也可以写成回调的方式

```objc
// First set up a callback.
- (void)getCallback:(AVObject *)object error:(NSError *)error {
  if (!object) {
    NSLog(@"The getFirstObject request failed.");
  } else {
    // The find succeeded.
    NSLog(@"Successfully retrieved the object.");
  }
}

// Then, elsewhere in your code...
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query whereKey:@"playerEmail" equalTo:@"dstemkoski@example.com"];
[query getFirstObjectInBackgroundWithTarget:self
                                   selector:@selector(getCallback:error:)];

```

你可以使用 `skip` 来跳过初始结果，这对于分页十分有用：

```objc
query.skip = 10; // skip the first 10 results
```
对于合适的类型,如数字和字符串,可以控制顺序返回结果:

```objc
// Sorts the results in ascending order by the score field
[query orderByAscending:@"score"];

// Sorts the results in descending order by the score field
[query orderByDescending:@"score"];
```
你可以添加更多关于排序键的查询,如下:

```objc
// Sorts the results in ascending order by the score field if the previous sort keys are equal.
[query addAscendingOrder:@"score"];

// Sorts the results in descending order by the score field if the previous sort keys are equal.
[query addDescendingOrder:@"score"];
```
对于合适的类型,你也可以在查询中使用比较:

```objc
AVQuery
// Restricts to wins < 50
[query whereKey:@"wins" lessThan:[NSNumber numberWithInt:50]];

// Restricts to wins <= 50
[query whereKey:@"wins" lessThanOrEqualTo:[NSNumber numberWithInt:50]];

// Restricts to wins > 50
[query whereKey:@"wins" greaterThan:[NSNumber numberWithInt:50]];

// Restricts to wins >= 50
[query whereKey:@"wins" greaterThanOrEqualTo:[NSNumber numberWithInt:50]];
```

如果你想检索对象匹配几个不同的值，你可以使用 `whereKey:containedIn:` ，这将会提供包含可接受值的数组。这在使用单一查询来替代多个查询中通常十分有用。例如，如果你需要检索在一个指定列表中由任何球员创造的分数：

```objc
// Finds scores from any of Jonathan, Dario, or Shawn
NSArray *names = [NSArray arrayWithObjects:@"Jonathan Walsh",
                                           @"Dario Wunsch",
                                           @"Shawn Simon",
                                           nil];
[query whereKey:@"playerName" containedIn:names];
```

如果你想检索一组不匹配任何几个值的对象，你可以使用 `whereKey:notContainedIn:` 来提供可接受值的数组。例如，如果你想检索一个列表之外球员的成绩。

```objc
// Finds scores from anyone who is neither Jonathan, Dario, nor Shawn
NSArray *names = [NSArray arrayWithObjects:@"Jonathan Walsh",
                                           @"Dario Wunsch",
                                           @"Shawn Simon",
                                           nil];
[query whereKey:@"playerName" notContainedIn:names];
```

如果你想检索对象有一个特殊的键集，您可以使用 `whereKeyExists`。相反，如果你想检索对象没有一个特定的键集，您可以使用 `whereKeyDoesNotExist`。

```objc
// Finds objects that have the score set
[query whereKeyExists:@"score"];

// Finds objects that don't have the score set
[query whereKeyDoesNotExist:@"score"];
```

您可以使用方法 `whereKey:matchesKey:inQuery:` 来获取对象，这个对象包含一个键，匹配另一个查询获取的一组对象中一个键的值，例如，如果你有一个类包含着运动员队伍和你在用户类中存储的用户出生地，你可以使用一个查询来找到用户的列表和其家乡球队的获胜记录。就像这样：

```objc
AVQuery *teamQuery = [AVQuery queryWithClassName:@"Team"];
[teamQuery whereKey:@"winPct" greaterThan:[NSNumber withDouble:0.5]];
AVQuery *userQuery = [AVQuery queryForUser];
[userQuery whereKey:@"hometown" matchesKey:@"city" inQuery:teamQuery];
[userQuery findObjectsInBackgroundWithBlock:^(NSArray *results, NSError *error) {
    // results will contain users with a hometown team with a winning record
}];
```
相反，获取一个对象包含一个键，这个键和另一个查询获取的一组对象中一个键的值不匹配，可以使用 `whereKey:doesNotMatchKey:inQuery:` 例如找到用户家乡球队的失败记录：

```objc
AVQuery *losingUserQuery = [AVQuery queryForUser];
[losingUserQuery whereKey:@"hometown" doesNotMatchKey:@"city" inQuery:teamQuery];
[losingUserQuery findObjectsInBackgroundWithBlock:^(NSArray *results, NSError *error) {
    // results will contain users with a hometown team with a losing record
}];
```
你可以通过调用 `selectKeys:` 与一个 `NSArray` 键来限制返回的字段，检索只包含 `score` 和 `playerName` 的文档（也可以是内置字段，如 `objectId`, `createdAt`, 和 `updatedAt`）：

```objc
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query selectKeys:@[@"playerName", @"score"]];
NSArray *results = [query findObjects];
```
其余字段可以稍后对返回的对象调用一个 `fetchIfNeeded` 的变体来获取：

```objc
AVObject *object = (AVObject*)[results objectAtIndex:0];
[object fetchIfNeededInBackgroundWithBlock:^(AVObject *object, NSError *error) {
  // all fields of the object will now be available here.
}];
```
### 查询数组值

当 key 是数组时，你可以这样找到 key 数组中包含 2 的对象:

```objc
// Find objects where the array in arrayKey contains 2.
[query whereKey:@"arrayKey" equalTo:[NSNumber numberWithInt:2]];
```

你也可以像下面的例子一样找到 key 数组中包含 2、3、4 的对象:

```objc
// Find objects where the array in arrayKey contains each of the
// elements 2, 3, and 4.
[query whereKey:@"arrayKey" containsAllObjectsInArray:@[@2, @3, @4]];
```

### 查询字符串值

使用 `whereKey: hasPrefix:` 来限定起始于一个特定字符串的值。这有点像 MySQL 的 `LIKE` 条件，索引使得这个操作即使对于大的数据集也是高效的。

```objc
// Finds barbecue sauces that start with "Big Daddy's".
AVQuery *query = [AVQuery queryWithClassName:@"BarbecueSauce"];
[query whereKey:@"name" hasPrefix:@"Big Daddy's"];
```

### 关系查询
有几种方法可以查询关系数据。如果你想在以某个属性匹配一个已知的 `AVObject` 的对象，您可以使用 `whereKey:equalTo:`，就像和其他数据类型一样。例如，如果每个 `Comment` 在 `Post` 字段都有一个 `Post` 对象,您可以获取特定帖子的评论:

```objc
// Assume AVObject *myPost was previously created.
AVQuery *query = [AVQuery queryWithClassName:@"Comment"];
[query whereKey:@"post" equalTo:myPost];

[query findObjectsInBackgroundWithBlock:^(NSArray *comments, NSError *error) {
    // comments now contains the comments for myPost
}];
```

你也可以通过 `ObjectId` 做关系查询:

```objc
[query whereKey:@"post"
        equalTo:[AVObject objectWithoutDataWithClassName:@"Post" objectId:@"51c912bee4b012f89e344ae9"];
```
如果你想查找的对象的某个属性是符合另一个查询的对象，可以使用 `whereKey:matchesQuery`。

值得注意的是，对结果数的默认100和最大1000的限制也适用与内嵌查询，所以在大型数据集中，您可能需要仔细构造查询来得到想要的行为。你可以这样找到带有图片的文章的评论：

```objc
AVQuery *innerQuery = [AVQuery queryWithClassName:@"Post"];
[innerQuery whereKeyExists:@"image"];
AVQuery *query = [AVQuery queryWithClassName:@"Comment"];
[query whereKey:@"post" matchesQuery:innerQuery];
[query findObjectsInBackgroundWithBlock:^(NSArray *comments, NSError *error) {
    // comments now contains the comments for posts with images
}];
```

如果你想查找的对象的某个属性是不匹配另一个查询的对象，可以使用 `whereKey:doesNotMatchQuery`。你可以这样找到不带图片的文章的评论：

```objc
AVQuery *innerQuery = [AVQuery queryWithClassName:@"Post"];
[innerQuery whereKeyExists:@"image"];
AVQuery *query = [AVQuery queryWithClassName:@"Comment"];
[query whereKey:@"post" doesNotMatchQuery:innerQuery];
[query findObjectsInBackgroundWithBlock:^(NSArray *comments, NSError *error) {
    // comments now contains the comments for posts without images
}];
```

在一些场景中，你需要在一个查询中返回多个类型的相关对象。这时可以使用方法 `includeKey`。例如，搜索最近的十条评论，并同时获得与之对应的文章：

```objc
AVQuery *query = [AVQuery queryWithClassName:@"Comment"];

// Retrieve the most recent ones
[query orderByDescending:@"createdAt"];

// Only retrieve the last ten
query.limit = [NSNumber numberWithInt:10];

// Include the post data with each comment
[query includeKey:@"post"];

[query findObjectsInBackgroundWithBlock:^(NSArray *comments, NSError *error) {
    // Comments now contains the last ten comments, and the "post" field
    // has been populated. For example:
    for (AVObject *comment in comments) {
         // This does not require a network access.
         AVObject *post = [comment objectForKey:@"post"];
         NSLog(@"retrieved related post: %@", post);
    }
}];
```

**你还可以用点（`.`）操作来查询多层的包含关系**，如果你想要的结果中包含评论所对应的文章以及该文章的作者，可以这样做：

```objc
[query includeKey:@"post.author"];
```
你可以多次使用 `includeKey:` 来在查询中包含多个属性。这个功能对 `AVQuery` 的 `getFirstObject` 和 `getObjectInBackground` 等辅助方法。

某些时候你可能不需要返回全部数据，而只希望返回特定 key 对应的数据，比如某些对象包括多个 key，某些 key 对应的 value 数据量比较大而你并不需要，可以使用类似下面代码

```objc
AVQuery * query = [AVQuery queryWithClassName:@"someClass"];
[query selectKeys:@[@"key"]];
AVObject * result = [query getFirstObject];
```

它将只返回指定 key 对应的 value，而不会返回所有数据，这样有助于节省网络带宽和计算资源。

### 缓存查询
在磁盘上缓存请求结果通常是很有用的，这允许你在设备离线时、应用刚刚开始时、网络请求尚未完成时显示数据。当它占用太多空间时，LeanCloud会自动刷新缓存。

默认的查询行为不使用缓存,但是您可以通过设置 `query.cachePolicy` 启用缓存。例如，当网络不可用时，尝试网络连接并同时取回缓存的数据:

```objc
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
query.cachePolicy = kPFCachePolicyNetworkElseCache;

//设置缓存有效期
query.maxCacheAge = 24*3600;

[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
  if (!error) {
    // Results were successfully found, looking first on the
    // network and then on disk.
  } else {
    // The network was inaccessible and we have no cached data for
    // this query.
  }
}];


// Then, elsewhere in your code...
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
query.cachePolicy = kPFCachePolicyNetworkElseCache;
[query findObjectsInBackgroundWithTarget:self
                                selector:@selector(findCallback:error:)];
```
LeanCloud 提供了几个不同的缓存策略：

* `kPFCachePolicyIgnoreCache`

查询行为不从缓存中加载，也不会将结果保存到缓存中。`kPFCachePolicyIgnoreCache` 是默认的缓存策略。

* `kPFCachePolicyCacheOnly`

查询行为会忽略网络状况，只从缓存中加载。如果没有缓存的结果，这个策略会导致一个 `AVError`。

* kPFCachePolicyCacheElseNetwork   --   查询行为首先尝试从缓存中加载，如果加载失败,它会通过网络加载结果。如果缓存和网络中获取的行为都失败了，会有一个 `AVError`。
* kPFCachePolicyNetworkElseCache   --   查询行为首先尝试从网络中加载，如果加载失败,它会从缓存中加载结果。如果缓存和网络中获取的行为都失败了，会有一个 `AVError`。
* kPFCachePolicyCacheThenNetwork   --   查询首先从缓存中加载，然后从网络加载。在这种情况下,回调函数会被调用两次，第一次是缓存中的结果，然后是从网络获取的结果。因为它在不同的时间返回两个结果，这个缓存策略不能和 `findObjects` 同时使用。

如果你需要控制缓存的行为，可以使用 `AVQuery` 提供的一些这方面的方法。比如:

* 检查是否存在缓存查询结果:

```objc
BOOL isInCache = [query hasCachedResult];
```


* 删除任何缓存查询结果:

```objc
[query clearCachedResult];
```

* 删除缓存查询结果:

```objc
[AVQuery clearAllCachedResults];
```
* 设定缓存结果最长时限:

```objc
query.maxCacheAge = 60 * 60 * 24;  // One day, in seconds.
```
查询缓存也适用于 `AVQuery` 的辅助方法，包括 `getFirstObject` 和 `getObjectInBackground`。

### 对象计数
如果你只需要知道匹配查询的对象数量，但不需要检索匹配的对象时，您可以使用 `countObjects` 代替 `findObjects`。例如，数数一个特定的球员参加了多少场比赛:

```objc
AVQuery *query = [AVQuery queryWithClassName:@"GameScore"];
[query whereKey:@"playername" equalTo:@"Sean Plott"];
[query countObjectsInBackgroundWithBlock:^(int count, NSError *error) {
  if (!error) {
    // The count request succeeded. Log the count
    NSLog(@"Sean has played %d games", count);
  } else {
    // The request failed
  }
}];
```

如果你想阻止调用线程，也可以使用同步方法 `countObjects`。

对于类，以及超过1000个的对象,计数操作很可能会导致响应超时。他们会经常产生响应超时，或者返回的结果只是一个大概值。因此，最好在构建程序时就尽量避免这样的操作。

### 复合查询
如果你想找和特定对象相匹配的几个查询,您可以使用方法 `orQueryWithSubqueries:`。例如,如果您想找赢得很多场比赛或者只赢得几场比赛的球员：

```objc
AVQuery *lotsOfWins = [AVQuery queryWithClassName:@"Player"];
[lotsOfWins whereKey:@"wins" greaterThan:[NSNumber numberWithInt:150]];

AVQuery *fewWins = [AVQuery queryWithClassName:@"Player"];
[fewWins whereKey:@"wins" lessThan:[NSNumber numberWithInt:5]];
AVQuery *query = [AVQuery orQueryWithSubqueries:[NSArray arrayWithObjects:fewWins,lotsOfWins,nil]];
[query findObjectsInBackgroundWithBlock:^(NSArray *results, NSError *error) {
  // results contains players with lots of wins or only a few wins.
  }];
```

您可以在新创建的AVQuery中添加额外的约束，以此作为一个 and 操作符。

值得注意的是，我们在复合查询的子查询中，不支持非过滤的约束（如：e.g. limit, skip, orderBy...:, includeKey:）

### Cloud Query Language（CQL）查询
我们同时也提供类似 SQL 语言的查询语言 CQL，如果你熟悉 SQL，你会觉得很相似。使用下面的方式使用 CQL：

```objc
    NSString *cql = [NSString stringWithFormat:@"select * from %@", @"ATestClass"];
    AVCloudQueryResult *result = [AVQuery doCloudQueryWithCQL:cql];
    NSLog(@"results:%@", result.results);

    cql = [NSString stringWithFormat:@"select count(*) from %@", @"ATestClass"];
    result = [AVQuery doCloudQueryWithCQL:cql];
    NSLog(@"count:%lu", (unsigned long)result.count);
```
在更多的时候，一个查询语句中间会有很多的值是可变值，为此，我们也提供了类似 Java JDBC 里的 PreparedStatement 使用占位符查询的语法结构。

```objc
    NSString *cql = [NSString stringWithFormat:@"select * from %@ where durability = ? and name = ?", @"ATestClass"];
    NSArray *pvalues =  @[@100,@"祈福"];
    [AVQuery doCloudQueryInBackgroundWithCQL:cql pvalues:pvalues callback:^(AVCloudQueryResult *result, NSError *error) {
        if (!error) {
            //do something
        } else {
            NSLog(@"%@", error);
        }
    }];
```
可变参数`100` 和 `"祈福"` 会自动替换查询语句中的问号位置（按照问号的先后出现顺序）。我们更推荐使用占位符语法，理论上会降低 CQL 转换的性能开销。
关于 CQL 的详细介绍，参考 [Cloud Query Language 详细指南](cql_guide.html)。

## 子类化

LeanCloud 设计的目标是让你的应用尽快的运行起来. 你可以通过`AVObject`访问到所有的数据还可以通过`objectForKey:`来获取任意字段. 在成熟的代码中,子类化有很多优势,包括减少代码,扩展性和支持自动补全. 子类化是可选的, 可以参照下面的例子:

    AVObject *student=[AVObject objectWithClassName:@"Student"];
    [student setObject:@"小明" forKey:@"name"];
    [student saveInBackground];

可以改成这样:

    Student *student=[Student object];
    student.name=@"小明";
    [student saveInBackground];

这样代码看起来是不是更优雅一些了?

### 子类化的实现

要实现子类化,需要下面几个步骤:

1. 导入 `AVObject+Subclass.h`
2. 继承 `AVObject` 并实现 `AVSubclassing` 协议
3. 实现类方法 `parseClassName`. 返回的字符串是原本要传给 `initWithClassName:` 的,并且以后就不需要再进行跟对象名称有关的配置了. 如果不实现,默认返回的是类的名字. **请注意: `AVUser`的子类化中 必须返回 `_User`**
4. 在实例化子类之前调用 `[YourClass registerSubclass]`, **在app当前生命周期中,只需要调用一次**,所以
建议在你的 `ApplicationDelegate` 中, 在 `[AVOSCloud setApplicationId:clientKey:]` 前后调用即可.

下面是实现 `Student` 子类化的例子:


```objc
  //Student.h
  #import <AVOSCloud/AVOSCloud.h>

  @interface Student : AVObject <AVSubclassing>

  @property(nonatomic,copy) NSString *name;

  @end


  //Student.m
  #import "Student.h"

  @implementation Student

  @dynamic name;

  + (NSString *)parseClassName {
      return @"Student";
  }

  @end


  // AppDelegate.m
  #import <AVOSCloud/AVOSCloud.h>
  #import "Student.h"

  - (BOOL)application:(UIApplication *)application
  didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [Student registerSubclass];
    [AVOSCloud setApplicationId:appid clientKey:appkey];
  }
```


### 属性

添加自定义的属性和方法到 `AVObject` 的子类可以更好的包含这个类的逻辑. 通过 `AVSubclassing`,你可以把所有的相关逻辑放到一个地方而不是用不同的用分开的类来表示业务逻辑和存储转换逻辑.

`AVObject` 支持动态 `synthesizer`, 就像 `NSManagedObject` 一样. 先正常的声明一个属性, 只是在 `.m` 文件中把 `@synthesize` 变成 `@dynamic`.

请看下面的例子 怎么添加一个年龄的属性


```objc
  //Student.h
  #import <AVOSCloud/AVOSCloud.h>

  @interface Student : AVObject <AVSubclassing>

  @property int age;

  @end


  //Student.m
  #import "Student.h"

  @implementation Student

  @dynamic age;

  ......
```


这样就可以通过 `student.age=19` 这样的方式来读写age这个字段了, 当然也可以这样 `[student setAge:19]`.

**注意: 属性名字保持首字母小写!** 比如, 不要写成 `student.Age`，而是 `student.age`

`NSNumber` 类型的属性可以被实现为 `NSNumber` 或者是它的原始数据类型(`int`,`BOOL` 等),在这个例子中, `[student objectForKey:@"age"]` 返回的是一个 `NSNumber` 类型,而直接取属性是 `int` 类型. 下面的这个属性同样适用:

    @property BOOL isTeamMember;


你可以根据自己的需求来选择用哪种类型. 原始类型更易用而 `NSNumber` 支持 `nil` 值可以让结果更清晰易懂.

值得一提的是，`AVRelation` 同样可以作为子类化的一个属性来使用,比如:

```objc
@interface Student : AVUser <AVSubclassing>
@property(retain) AVRelation * friends
  ......
```

如果你需要更复杂的逻辑而不是简单的属性访问,也可以自己来这样实现:

```objc
  @dynamic iconFile;

  - (UIImageView *)iconView {
    UIImageView *view = [[UIImageView alloc] initWithImage:kPlaceholderImage];
    view.image = [UIImage imageNamed:self.iconFile];
    return [view autorelease];
  }

```

### 针对 AVUser 子类化的特别说明

假如您现在已经有一个基于 `AVUser` 的子类，如上面提到的 `Student`:


```objc
@interface Student : AVUser<AVSubclassing>
@property (retain) NSString *displayName;
@end


@implementation Student
@dynamic displayName;
+ (NSString *)parseClassName {
    return @"_User";
}
@end
```

登录时需要调用 `Student` 的登录方法才能通过 `currentUser` 得到这个子类:

```objc
[Student logInWithUsernameInBackground:@"USER_NAME" password:@"PASSWORD" block:^(AVUser *user, NSError *error) {
        Student * student = [AVUser currentUser];
        studen.displayName = @"YOUR_DISPLAY_NAME";
    }];
```



### 初始化子类

创建一个子类实例,要使用 `object` 类方法. 要创建并关联到已有的对象请使用 `objectWithoutDataWithObjectId:` 类方法.

### 子类查询

可以通过类方法 `query` 来得到这个子类的查询对象. 下面的例子查询年龄小于21岁的学生:

```objc
  AVQuery *query = [Student query];
  [query whereKey:@"age" lessThanOrEqualTo:@"21"];
  [query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    if (!error) {
      Student *stu1 = [objects objectAtIndex:0];
      // ...
    }
  }];
```

## ACL权限控制
ACL(Access Control List)是最灵活和简单的应用数据安全管理方法。通俗的解释就是为每一个数据创建一个访问的白名单列表，只有在名单上的用户(AVUser)或者具有某种角色(AVRole)的用户才能被允许访问。为了更好地保证用户数据安全性，LeanCloud表中每一张都有一个ACL列。当然，LeanCloud还提供了进一步的读写权限控制。一个 User 必须拥有读权限（或者属于一个拥有读权限的 Role）才可以获取一个对象的数据，同时，一个 User 需要写权限（或者属于一个拥有写权限的 Role）才可以更改或者删除一个对象。
以下列举了几种在LeanCloud常见的ACL使用范例：

### 默认访问权限
在没有显式指定的情况下，LeanCloud中的每一个对象都会有一个默认的ACL值。这个值代表了，所有的用户，对这个对象都是可读可写的。此时你可以在数据管理的表中ACL属性中看到这样的值:

```objc
    {"*":{"read":true,"write":true}}
```

而在iOS代码中，这样的值对应的代码是：

```objc

    AVACL *acl = [AVACL ACL];
    [acl setPublicReadAccess:YES];
    [acl setPublicWriteAccess:YES];

```
当然正如上文提到的，默认的ACL并不需要显式的指定。

### 指定用户访问权限
当一个用户在实现一个网盘类应用时，征对不同文件的私密性，用户就需要不同的文件访问权限。
譬如公开的文件，每一个其他用户都有读的权限，然后仅仅只有创建者才拥有更改和删除的权限。

```objc

    AVACL *acl = [AVACL ACL];
    [acl setPublicReadAccess:YES]; //此处设置的是所有人的可读权限
    [acl setWriteAccess:YES forUser:[AVUser currentUser]]; //而这里设置了文件创建者的写权限

    AVObject * object = [AVObject objectWithClassName:@"iOSAclTest"];

    object.ACL=acl;
    [object save];

```

当然用户也会上传一些隐私文件,只有这些文件的创建者才对这些文件拥有读写权限

```objc
    [acl setWriteAccess:YES forUser:[AVUser currentUser]];
```
注：一旦显式设置ACL，默认的ACL就会被覆盖

### 指定角色访问权限

#### AVUser与AVRole的从属关系
指定用户访问权限虽然很方便，但是依然会有局限性。
以工资系统为例，一家公司的工资系统，工资最终的归属者和公司的出纳们只拥有工资的读权限，而公司的人事和老板才拥有全部的读写权限。当然你可以通过多次设置指定用户的访问权限来实现这一功能（多个用户的ACL设置是追加的而非覆盖）。

```objc
    AVObect *salary = [AVObject objectWithClassName:@"Salary"];
    [salary setObject:@(2000000) forKey:@"value"];

    //这里为了方便说明, 直接声明了变量, 但没有实现
    AVUser *boss;//假设此处为老板
    AVUser *hrWang;  //人事小王
    AVUser *me; //我们就在文档里爽一爽吧
    AVUser *cashierZhou; //出纳老周


    AVACL *acl = [AVACL ACL];

    //4个人都有可读权限
    [acl setReadAccess:YES forUser:boss];
    [acl setReadAccess:YES forUser:hrWang];
    [acl setReadAccess:YES forUser:cashierZhou];
    [acl setReadAccess:YES forUser:me];

    //只有2个人可写
    [acl setWriteAccess:YES forUser:boss];
    [acl setWriteAccess:YES forUser:hrWang];

    [salary setACL:acl];
    [salary save];


```

但是这些涉及其中的人可能不止一个，也有离职换岗新员工的问题存在。这样的代码既不优雅，也太啰嗦,同样会很难维护。
这个时候我们就引入了AVRole来解决这个问题。
公司的员工可以成百上千，然而一个公司组织里的角色却能够在很长一段时间时间内相对稳定。

```objc
    AVObect *salary = [AVObject objectWithClassName:@"Salary"];
    [salary setObject:@(2000000) forKey:@"value"];

    //这里为了方便说明, 直接声明了变量, 但没有实现
    AVUser *boss;//假设此处为老板
    AVUser *hrWang;  //人事小王
    AVUser *me; //我们就在文档里爽一爽吧
    AVUser *cashierZhou; //出纳老周
    AVUser *cashierGe;//出纳小葛

    //这段代码可能放在员工管理界面更恰当，但是为了示意，我们就放在这里
    AVRole *hr =[AVRole roleWithName:@"hr"];
    AVRole *cashier = [AVRole roleWithName:@"cashier"];

    [[hr users] addObject:hrWang];
    [hr save];

    [[cashier users] addObject:cashierZhou];//此处对应的是AVRole里面有一个叫做users的Relation字段
    [[cashier users] addObject:cashierGe];
    [cashier save];

    AVACL *acl = [AVACL ACL];
    [acl setReadAccess:YES forUser:boss];//老板假设只有一个
    [acl setReadAccess:YES forUser:me];

    [acl setReadAccess:YES forRole:hr];
    [acl setReadAccess:YES forRole:cashier];

    [acl setWriteAccess:YES forUser:boss];
    [acl setWriteAccess:YES forRole:hr];

    [salary setACL:acl];
    [salary save];
```

当然如果考虑到一个角色(`AVRole`)里面有多少员工(`AVUser`)，编辑这些员工可需要做权限控制，`AVRole`同样也有`setACL`方法可以使用。

#### AVRole之间的从属关系

在讲清楚了用户与角色的关系后，我们还有一层角色与角色之间的关系。用下面的例子来理解可能会对我们有所帮助：

一家创业公司有移动部门，部门下面有不同的小组，Android和iOS。而每个小组只拥有自己组的代码的读写权限。但是他们同时拥有核心库代码的读取权限。

```objc
    AVRole *androidTeam = [AVRole roleWithName:@"AndroidTeam"];
    AVRole *iOSTeam = [AVRole roleWithName:@"IOSTeam"];
    AVRole *mobileDep = [AVRole roleWithName:@"MobileDep"];

    [androidTeam save];
    [iOSTeam save];

    [[mobileDep roles] addObject:androidTeam];
    [[mobileDep roles] addObject:iOSTeam];

    [mobileDep save];

    AVObject *androidCode = [AVObject objectWithClassName:@"Code"];
    AVObject *iOSCode = [AVObject objectWithClassName:@"Code"];
    AVObject *coreCode = [AVObject objectWithClassName:@"Code"];
    //.....此处省略一些具体的值设定

    AVACL *acl1=[AVACL ACL];
    [acl1 setReadAccess:YES forRole:androidTeam];
    [acl1 setWriteAccess:YES forRole:androidTeam];
    [androidCode setACL:acl1];

    AVACL *acl2=[AVACL ACL];
    [acl2 setReadAccess:YES forRole:iOSTeam];
    [acl2 setWriteAccess:YES forRole:iOSTeam];
    [iOSCode setACL:acl2];

    AVACL *acl3=[AVACL ACL];
    [acl3 setReadAccess:YES forRole:mobileDep];
    [coreCode setACL:acl3];

    [androidCode save];
    [iOSTeam save];
    [coreCode save];
```


## 文件


### AVFile

`AVFile` 可以让你的应用程序将文件存储到服务器中，比如常见的文件类型图像文件、影像文件、音乐文件和任何其他二进制数据都可以使用。
使用 `AVFile` 非常容易，首先你可以将文件数据存在 `NSData` 中，然后由 `NSData` 创建一个 `AVFile` 对象。 在下面的例子中，我们会使用一个字符串：

```objc
NSData *data = [@"Working with LeanCloud is great!" dataUsingEncoding:NSUTF8StringEncoding];
AVFile *file = [AVFile fileWithName:@"resume.txt" data:data];
```

请注意，在本例中，我们将文件名定为 `resume.txt`。这里有两件事情值得注意:

* 你不需要担心文件名的冲突。每一个上传的文件有惟一的ID，所即使上传多个文件名为 `resume.txt` 的文件也不会有问题。
* 给文件添加扩展名非常重要，通过扩展名，LeanCloud 可以获取文件类型以便可以正确处理文件。所以如果你将一个 `PNG` 图象存在 `AVFile` 中，要确保使用 `.png` 扩展名。

然后你会需要将文件存在LeanCloud中，你可以根据需要调用不同版本的save方法。

```objc
[file saveInBackground];
```

最终当文件存储完成后，你可以象其他的对象那样，将 `AVFile` 关联到 `AVObject`。

```objc
AVObject *jobApplication = [AVObject objectWithClassName:@"JobApplication"]
[jobApplication setObject:@"Joe Smith" forKey:@"applicantName"];
[jobApplication setObject:file         forKey:@"applicantResumeFile"];
[jobApplication saveInBackground];
```

重新获取只需要调用 `AVFile` 的 `getData`。


```objc
AVFile *applicantResume = [anotherApplication objectForKey:@"applicantResumeFile"];
NSData *resumeData = [applicantResume getData];
```

如 `AVObject` 那样，你也可以使用 `getData` 的异步版本。

**如果将文件存储到对象的一个数组类型的属性内，那么必须在查询该对象的时候加上include该属性，否则查询出来的数组将是AVObject数组。**

### 图象

你可以通过将图象转成 `NSData`，然后使用 `AVFile`，这样可以很容易地将图象存到LeanCloud上。比如你有一个叫"image"的 `UIImage` 对象，你希望将它存到 `AVFile` 中。

```objc
NSData *imageData = UIImagePNGRepresentation(image);
AVFile *imageFile = [AVFile fileWithName:@"image.png" data:imageData];
[imageFile save];

AVObject *userPhoto = [AVObject objectWithClassName:@"UserPhoto"];
[userPhoto setObject:@"My trip to Hawaii!" forKey:@"imageName"];
[userPhoto setObject:imageFile             forKey:@"imageFile"];
[userPhoto save];
```

### 进度提示

通过 `saveInBackgroundWithBlock:progressBlock:` 和 `getDataInBackgroundWithBlock:progressBlock:`  很容易可以得到 `AVFile` 的上传或者下载的进度。比如

```objc
NSData *data = [@"Working at AVOS is great!" dataUsingEncoding:NSUTF8StringEncoding];
AVFile *file = [AVFile fileWithName:@"resume.txt" data:data];
[file saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
  // Handle success or failure here ...
} progressBlock:^(int percentDone) {
  // Update your progress spinner here. percentDone will be between 0 and 100.
}];
```

### 得到图象的缩略图

当您保存了一个图象文件时，您可能希望在下载原图之前，得到缩略图，我们为您提供了便捷的API，您可以使用

```objc
AVFile * file = [AVFile fileWithURL:@"the-file-remote-url"];
[file getThumbnail:YES width:100 height:100 withBlock:^(UIImage * image, NSError *error) {
    }];
```
得到缩略图

### 文件元数据

某些时候，您会希望将一些元数据保存在文件对象中，您可以通过metadat属性来保存和获取这些数据

```objc
AVFile * file = [AVFile fileWithName:@"test.jpg" contentsAtPath:@"file-local-path"];
[file.metadata setObject:@(100) forKey:@"width"];
[file.metadata setObject:@(100) forKey:@"height"];
[file.metadata setObject:@”LeanCloud" forKey:@"author"];
NSError * error = nil;
[file save:&error];
```

### 删除

当您的文件比较多时，您可能希望将一些不需要的文件从LeanCloud上删除，您可以使用
```objc
[file deleteInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
}];
```
来删除您的文件

### 清除缓存
AVFile也提供了清除缓存的方法

```objc
//清除当前文件缓存
- (void)clearCachedFile;

//类方法, 清除所有缓存
+ (BOOL)clearAllCachedFiles;

//类方法, 清除多久以前的缓存
+ (BOOL)clearCacheMoreThanDays:(NSInteger)numberOfDays;

```


## 用户

用户是一个应用程序的核心。对于个人开发者来说，能够让自己的应用程序积累到多的用户，就能给自己带来更多的创作动力。因此LeanCloud提供了一个专门的用户类，AVUser来自动处理用户账户管理所需的功能。
有了这个类，你就可以在您的应用程序中添加用户账户功能。
AVUser是一个AVObject的子类，它继承了AVObject所有的方法具有AVObject相同的功能。不同的是，AVUser增加了一些特定的关于用户账户相关的功能。

### 属性
AVUser除了从AVObject继承的属性外，还有几个特定的属性：
username: 用户的用户名（必需）。
password: 用户的密码（必需）。
email: 用户的电子邮件地址（可选）。
和其他AVObject对象不同的是，在设置AVUser这些属性的时候不是使用的put方法，而是专门的setXXX方法。


### 注册

你的应用程序会做的第一件事可能是要求用户注册。下面的代码是一个典型的注册过程：

```objc
AVUser * user = [AVUser user];
user.username = @"steve";
user.password =  @"f32@ds*@&dsa";
user.email = @"steve@company.com";
[user setObject:@"213-253-0000" forKey:@"phone"];

[user signUpInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    if (succeeded) {

    } else {

    }
}];
```

在注册过程中，服务器会进行注册用户信息的检查，以确保注册的用户名和电子邮件地址是惟一的。**服务端还会对用户密码进行不可逆的加密处理，不会明文保存任何密码，应用切勿再次在客户端加密密码，这会导致重置密码等功能不可用**。请注意，我们使用的是signUpInBackgroundWithBlock方法，而不是saveInBackground方法。另外还有各种不同的signUp方法。像往常一样，我们建议在可能的情况下尽量使用异步版本的signUp方法，这样就不会影响到应用程序主UI线程的响应。你可以阅读API中更多的有关这些具体方法的使用。
如果注册不成功，你可以查看返回的错误对象。最有可能的情况是，用户名或电子邮件已经被另一个用户注册。这种情况您可以提示用户，要求他们尝试使用不同的用户名进行注册。
你也可以要求用户使用Email做为用户名注册，这样做的好处是，你在提交信息的时候可以将输入的“用户名“默认设置为用户的Email地址，以后在用户忘记密码的情况下可以使用LeanCloud提供重置密码功能。

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

### 登录
当用户注册成功后，您需要让他们以后能够登录到他们的账户后使用应用。要做到这样一点，你可以使用
AVUser类的loginInBackground方法。

```objc
[AVUser logInWithUsernameInBackground:@"username" password:@"password" block:^(AVUser *user, NSError *error) {
    if (user != nil) {

    } else {

    }
}];
```

### 当前用户

如果用户在每次打开你的应用程序时都要登录，这将会直接影响到你应用的用户体验。为了避免这种情况，你可以使用缓存的currentUser对象。
每当你注册成功或是第一次登录成功，都会在本地磁盘中又一个缓存的用户对象，你可以这样来获取这个缓存的用户对象来进行登录：


```objc
AVUser * currentUser = [AVUser currentUser];
if (currentUser != nil) {
    // 允许用户使用应用
} else {
    //缓存用户对象为空时， 可打开用户注册界面…
}
```

当然，你也可以使用如下方法清除缓存用户对象：

```objc
[AVUser logOut];  //清除缓存用户对象
AVUser * currentUser = [AVUser currentUser]; // 现在的currentUser是nil了
```

### 重置密码
这是一个事实，一旦你引入了一个密码系统，那么肯定会有用户忘记密码的情况。对于这种情况，我们提供了一种方法，让用户安全地重置起密码。
重置密码的流程很简单，开发者只要求用户输入注册的电子邮件地址即可：

```objc
[AVUser requestPasswordResetForEmailInBackground:@"myemail@example.com" block:^(BOOL succeeded, NSError *error) {
    if (succeeded) {

    } else {

    }
}];
```

密码重置流程如下：

 * 用户输入他们的电子邮件，请求重置自己的密码。
 * LeanCloud向他们的邮箱发送一封包含特殊的密码重置连接的电子邮件。
 * 用户根据向导点击重置密码连接，打开一个特殊的页面，让他们输入一个新的密码。
 * 用户的密码已被重置为新输入的密码。

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

### 修改密码

当用户系统中存在密码的时候，就会存在用户更改密码的需求，对于这种情况，我们提供了一种方法，能够同时验证老密码和修改新密码:

```objc
[AVUser logInWithUsername:@"username" password:@"111111"]; //请确保用户当前的有效登录状态
[[AVUser currentUser] updatePassword:@"111111" newPassword:@"123456" withTarget:self selector:@selector(passwordUpdated:error:)];
```
如果要求更改密码的用户不再登录状态、原密码错误和用户不存在等情况都会通过callback返回。

###  手机号码验证

在应用设置中打开`注册手机号码验证`选项后。当你在注册用户时，填写用户手机字段后，LeanCloud 会自动向该手机号码发送一个验证短信，用户在输入验证码以后，该用户就被表示为已经验证过手机。

以下代码就可发送注册验证码到用户手机:

```objc
	AVUser * user = [AVUser user];
	user.username = @"steve";
	user.password =  @"f32@ds*@&dsa";
	user.email = @"steve@company.com";
	user.mobilePhoneNumber = @"13613613613";
	[user signUp];
```

调用以下代码即可验证验证码:

```objc
	[AVUser verifyMobilePhone:@"123456" withBlock:^(BOOL succeeded, NSError *error) {
        //验证结果
    }];
```

验证成功后，用户的`mobilePhoneVerified`属性变为true，并且调用云代码的`AV.Cloud.onVerifed('sms', function)`方法。

### 手机号码登录

在手机号码被验证后，用户可以使用手机号码进行登录。手机号码包括两种方式：手机号码＋密码方式，手机号码＋短信验证码方式。

以下为手机号码＋密码来登录的方式：

```objc
    [AVUser logInWithMobilePhoneNumberInBackground:@"13613613613" password:@"yourpassword" block:^(AVUser *user, NSError *error) {

    }];
```

以下为发送登录短信验证码：

```objc
    [AVUser requestLoginSmsCode:@"123456" withBlock:^(BOOL succeeded, NSError *error) {

    }];
```

最后使用短信验证码＋手机号码进行登录:

```objc
    [AVUser logInWithMobilePhoneNumberInBackground:@"13613613613" smsCode:smsCode block:^(AVUser *user, NSError *error) {

    }];
```

### 手机号码重置密码
和使用电子邮件地址重置密码类似，使用手机号码重置密码使用下面的方法获取短信验证码：

```objc
[AVUser requestPasswordResetWithPhoneNumber:@"18812345678" block:^(BOOL succeeded, NSError *error) {
    if (succeeded) {

    } else {

    }
}];
```

注意用户需要先绑定手机号码。
然后使用短信验证码重置密码：

```objc
[AVUser resetPasswordWithSmsCode:@"123456" newPassword:@"password" block:^(BOOL succeeded, NSError *error) {
    if (succeeded) {

    } else {

    }
}];
```

### 查询

查询用户，你需要使用特殊的用户查询对象来完成：

```objc
AVQuery * query = [AVUser query];
[query whereKey:@"gender" equalTo:@"female"];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    if (error == nil) {

    } else {

    }
}];

```

浏览器中查看用户表
User表是一个特殊的表，专门存储AVUser对象。在浏览器端，你会看到一个_user表。

### 匿名用户
如果你需要创建匿名用户，可以使用 `AVAnonymousUtils` 来完成。通过如下代码，服务端会为你自动创建一个 `AVUser` 对象，其用户名为随机字符串。
完成之后，`currentUser` 会被置为此用户对象。之后的修改、保存、登出等操作都可以使用 `currentUser` 来完成。

```objc
    [AVAnonymousUtils logInWithBlock:^(AVUser *user, NSError *error) {
        if (user) {

        } else {

        }
    }];
```

## 地理位置
LeanCloud允许用户根据地球的经度和纬度坐标进行基于地理位置的信息查询。你可以在AVObject的查询中添加一个AVGeoPoint的对象查询。您可以实现轻松查找出离当前用户最接近的信息或地点的功能。

### 地理位置对象
首先需要创建一个AVGeoPoint对象。例如，创建一个北纬40.0度-东经-30.0度的AVGeoPoint对象：

```objc
AVGeoPoint * point = [AVGeoPoint geoPointWithLatitude:40.0 longitude:-30.0];new AVGeoPoint(40.0, -30.0);
```

添加地理位置信息

```objc
[placeObject setObject:point forKey:@"location"];
```

### 地理查询

现在，你的数据表中有了一定的地理坐标对象的数据，这样可以测试找出最接近某个点的信息了。你可以使用AVQuery对象的whereNear方法来这样做：

```objc
AVObject * userObject = nil;
AVGeoPoint * userLocation =  (AVGeoPoint *) [userObject objectForKey:@"location"];
AVQuery * query = [AVQuery queryWithClassName:@"PlaceObject"];
[query whereKey:@"locaton" nearGeoPoint:userLocation];
query.limit = 10;      //获取最接近用户地点的10条数据
NSArray<AVObject *> nearPlaces = [query findObjects];
```

在以上代码中，nearPlaces是一个返回的距离userLocation点（最近到最远）的对象数组。
要限制查询指定距离范围的数据可以使用whereWithinKilometers、whereWithinMiles或whereWithinRadians方法。
要查询一个矩形范围内的信息可以使用whereWithinGeoBox来实现：

```objc
AVGeoPoint * northeastOfSF = [AVGeoPoint geoPointWithLatitude:37.9 longitude:40.1];
AVGeoPoint * southwestOfSF = [AVGeoPoint geoPointWithLatitude:37.8 longitude:40.04];
AVQuery * query = [AVQuery queryWithClassName:@"PizzaPlaceObject"];
[query whereKey:@"location" withinGeoBoxFromSouthwest:southwestOfSF toNortheast:northeastOfSF];
NSArray<AVObject *> * pizzaPlacesInSF = [query findObjects];
```

注意事项
目前有几个需要注意的地方：

 * 每个AVObject数据对象中只能有一个AVGeoPoint对象。
 * 地理位置的点不能超过规定的范围。纬度的范围应该是在-90.0到90.0之间。经度的范围应该是在-180.0到180.0之间。如果您添加的经纬度超出了以上范围，将导致程序错误。
 * iOS 8.0 之后使用定位服务前需要调用 [locationManager requestWhenInUseAuthorization] 或者 [locationManager requestAlwaysAuthorization] 获取用户使用期授权或永久授权，而这两个请求授权需要在 info.plist 里面对应添加 NSLocationWhenInUseUsageDescription 或 NSLocationWhenInUseUsageDescription 的 key/value，value 为开启定位服务原因的描述，SDK 内部默认使用的是使用期授权。

## 调用云代码

### 调用云代码函数

使用`AVCloud`类的静态方法来调用云代码中定义的函数：


  NSDictionary *parameters=@{...};

    [AVCloud callFunctionInBackground:@"aFunctionName" withParameters:parameters block:^(id object, NSError *error) {
        // 执行结果
    }];


`aFunctionName`是函数的名称，`parameters`是传入的函数参数，`block`对象作为调用结果的回调传入。

### 区分生产环境调用

云代码区分测试和生成环境, 所以可以通过设置AVCloud来调用不同生成环境

  [AVCloud setProductionMode:NO];

其中`NO`表示测试环境. 默认是调用生产环境云代码.

## 短信验证码服务
除了用户相关的包括注册，登录等操作以外，LeanCloud还支持额外的短信验证码服务。
在实际的应用中，假如有一些相对比较敏感的操作，比如付费、删除重要资源等操作，您希望能够通过短信验证的方式来与用户进行确认，您就可以在用户验证过手机号码，应用管理平台打开了`启用手机号码短信认证`选项的前提下，使用LeanCloud提供的短信验证码服务。

### 请求短信验证码
以下操作为给某个操作发送验证短信

```objc
    [AVOSCloud requestSmsCodeWithPhoneNumber:@"13613613613"
                                     appName:@"某应用"
                                   operation:@"具体操作名称"
                                  timeToLive:10
                                    callback:^(BOOL succeeded, NSError *error) {
        // 执行结果
    }];
   //短信格式类似于：
   //您正在{某应用}中进行{具体操作名称}，您的验证码是:{123456}，请输入完整验证，有效期为:{10}分钟

```

### 自定义短信模板

如果您想完全自定义短信的内容，可以在应用设置的短信模板创建自定义的短信模板，但是需要**审核**。

在提交了短信模板并且得到审核以后，你可以通过SDK来发送符合短信模板的短信给你的用户。

假设您提交了如下的短信模板，并且将这个模板的名称保存为"Register_Template"：

<pre ng-non-bindable ><code>
Hi {{username}},
欢迎注册{{name}}应用，您可以通过验证码:{{code}}，进行注册。本条短信将在{{ttl}}分钟后自行销毁。请尽快使用。
以上。
{{appname}}
</code></pre>

**注：其中的name,code,ttl是预留的字段，分别代表应用名、验证码、过期时间。不需要填充内容，会自动填充。**

您可以通过如下代码进行短信发送：

```objc
    NSMutableDictionary *dict = [[NSMutableDictionary alloc] init];
    [dict setObject:@"MyName" forKey:@"username"];
    [dict setObject:@"MyApplication" forKey:@"appname"];
    [AVOSCloud requestSmsCodeWithPhoneNumber:@"12312312312" templateName:@"Register_Template" variables:dict callback:^(BOOL succeeded, NSError *error) {
        if (succeeded) {
            //do something
        } else {
            NSLog(@"%@", error);
        }
    }];
```

### 验证短信验证码
您可以通过以下代码来验证短信验证码：

```objc
    [AVOSCloud verifySmsCode:@"123456" callback:^(BOOL succeeded, NSError *error) {
        //code
    }];
```

## FAQ 常见问题和解答

### 怎么使用 LeanCloud iOS SDK
最简单的方式，使用CocoaPods，如以下的PodFile

```sh
pod 'AVOSCloud'
```

AVOSCloudSNS SDK:

```sh
pod 'AVOSCloudSNS'
```

### 如何使用用户登录功能

```objc
    [AVUser logInWithUsernameInBackground:@"zeng" password:@"123456" block:^(AVUser *user, NSError *error) {
        if (user != null) {
            NSLog(@"login success");
        } else {
            NSLog(@"signin failed");
        }
    }];

```


### 如何登出

```objc
[AVUser logOut];

```


### 如何使用新浪微博登录


```objc
[AVOSCloudSNS loginWithCallback:^(id object, NSError *error) {

  //callback code here

} toPlatform:AVOSCloudSNSSinaWeibo];

```

### 使用 AVOSCloudSNS，运行时报错：+[AVUser loginWithAuthData:block:]: unrecognized selector sent to class

请将 `Build Settings -> Linking -> Other Linker Flags` 设置为 `-ObjC`。具体原因可以参考苹果官方的 Technical Q&A QA1490 [Building Objective-C static libraries with categories](https://developer.apple.com/library/mac/qa/qa1490/_index.html)。此外，stackoverfow 也有一个比较详细的答案 [Objective-C categories in static library](http://stackoverflow.com/questions/2567498/objective-c-categories-in-static-library)。









