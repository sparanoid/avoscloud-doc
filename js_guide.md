
# JavaScript 指南

如果你还没有设置你的项目,请查看我们的QuickStart.

## 简介

LeanCloud平台提供了一个移动App的完整后端解决方案,我们的目标是完全消除写
后端代码和维护服务器的必要性.

我们的JavaScript SDK基于流行的Backbone.js框架.它与已经存在的
Backbone程序是兼容的,只需要在你的代码中做出一点点改变,我们的最小化
配置,让你很快地用在LeanCloud上使用JavaScript和HTML5.

请在阅读本文档的同时，对照查看 [JavaScript API文档](./api/javascript/)。本指南并没有完全覆盖所有的 API 调用。

### Apps

在LeanCloud上你可以为你的每一个移动应用创建一个App,每一个App都有它专属
的App id和App key.你在LeanCloud上的账户可以容纳多个App.对每一个应用你都
可以部署不同的版本到测试或生产环境.

### 依赖

我们的JavaScript SDK不需要引入其他的库,唯一的一个例外是AV.view
类,需要你提供jQuery或者一个jQuery兼容的$方法.


## 对象

### AV.Object

在LeanCloud上保存数据应该使用AV.Object.每一个AV.Object都包含
键-值对,对应一些兼容JSON的数据.这些数据是无模式的,这意味着你不
需提前指定好一个AV.Object中应该包含哪些key.你只需简单地设定你
需要的key-value对,我们的后端就会存储它.

例如，假设你在记录一局游戏的分数.一个简单的AV.Object可能包含:

```
score: 1337, playerName: "Sean Plott", cheatMode: false
```

Object的键必须是字母或者数字的字符串.值可以是字符串,数字,布尔值或者数
组和字典,只要是能被JSON编码的东西就行.

每一个AV.Object都是一个特定子类的实例,子类名可以来区分各种数据.比如
我们可以把游戏分数称之为GameScore.我们推荐你使用类似于
NameYourClassesLikeThis或者nameYourKeysLikeThis的命名,可以使你的代码看
上去更优雅.

为了建立一个新的子类,你可以使用AV.Object.extend方法.每次调用AV.Query都
会用一样的类名返回一个新的类的实例.如果你熟悉Backbone.Model的话,你
已经明白如何使用AV.Object了.它本身就是设计来让两者可以相互替换的.

```
// 创建AV.Object子类.
var GameScore = AV.Object.extend("GameScore");

// 创建该类的一个实例
var gameScore = new GameScore();

// 你可以用Backbone的语法.
var Achievement = AV.Object.extend({
  className: "Achievement"
});
```

你可以为AV.Object的子类添加任意方法.

```
// A complex subclass of AV.Object
var Monster = AV.Object.extend("Monster", {
  //实例方法
  hasSuperHumanStrength: function() {
    return this.get("strength") > 18;
  },
  strength: function(){
    return this.get('strength');
  }
}, {
  //类方法
  spawn: function(strength) {
    var monster = new Monster();
    monster.set("strength", strength);
    return monster;
  }
});

var monster = Monster.spawn(200);
alert(monster.strength());  // Displays 200.
```

创建对象还可以通过 `AV.Object.new` 方法，子类对象也可以：

```
//AV.Object
var gameScore = AV.Object.new('GameScore');
//子类
var monster = Monster.new({strength: 20});
```

在使用 [uglify](https://github.com/mishoo/UglifyJS) 做代码压缩的时候，推荐采用上述方式创建对象，否则可能遇到压缩后语法错误。

### 保存对象

假如你想要在LeanCloud上保存GameScore，方法和Backbone.Model差不多,就用
save就可以了.

```
var GameScore = AV.Object.extend("GameScore");
var gameScore = new GameScore();
gameScore.set("score", 1337);
gameScore.set("playerName", "Sean Plott");
gameScore.set("cheatMode", false);
gameScore.save(null, {
  success: function(gameScore) {
    // Execute any logic that should take place after the object is saved.
    alert('New object created with objectId: ' + gameScore.id);
  },
  error: function(gameScore, error) {
    // Execute any logic that should take place if the save fails.
    // error is a AV.Error with an error code and description.
    alert('Failed to create new object, with error code: ' + error.description);
  }
});
```

在代码运行以后,你可能会对后端发生了什么有兴趣,为了确认数据被保存了,你可以
在LeanCloud的[数据管理](/data.html?appid={{appid}})上查看你的数据.你大致可以看到如下的内容:

```
objectId: "520ca0bbe4b07e8e0e847e31", score: 1337, playerName: "Sean Plott", cheatMode: false,
createdAt:"2011-06-10T18:33:42Z", updatedAt:"2011-06-10T18:33:42Z"
```

这里有2点需要注意的地方:

* 你不需要设定一个叫GameScore的新类,LeanCloud会自动地在你第一次使用它的时候为你创建这个类.
* 每个对象有几个默认的键是不需要开发者额外指定的： objectId是一个对于每一个
保存的对象为一个标志； createdAt和updatedAt表示对象在cloud中创建和最后一
次更改的时间.这样的字段都是由LeanCloud来填充的,所以他们在保存之前
AV.Object中都不会有这些字段.

如果愿意,你也可以在调用save时直接设定属性.

```
var GameScore = AV.Object.extend("GameScore");
var gameScore = new GameScore();

gameScore.save({
  score: 1337,
  playerName: "Sean Plott",
  cheatMode: false
}, {
  success: function(gameScore) {
    // The object was saved successfully.
  },
  error: function(gameScore, error) {
    // The save failed.
    // error is a AV.Error with an error code and description.
  }
});
```

保存后，可以通过`gameScore.createdAt`获取对象的创建时间，通过`gameScore.updatedAt`获取对象的更新时间，两者都是Date对象。

### 检索对象

在LeanCloud中保存数据是很简单的,获取数据也非常容易。如果事先知道
objectId的话,你可以用一个AV.Query提取出整个AV.Object:

```
var GameScore = AV.Object.extend("GameScore");
var query = new AV.Query(GameScore);
query.get("520ca0bbe4b07e8e0e847e31", {
  success: function(gameScore) {
    // The object was retrieved successfully.
  },
  error: function(object, error) {
    // The object was not retrieved successfully.
    // error is a AV.Error with an error code and description.
  }
});
```

为了获得AV.Object的属性值, 应该使用get方法.

```
var score = gameScore.get("score");
var playerName = gameScore.get("playerName");
var cheatMode = gameScore.get("cheatMode");
```

### 更新对象

更新一个对象也是非常简单的。首先需要获取到要更新的 `AV.Object` 对象，然后进行修改值后保存数据。例如：

```
// Create the object.
var GameScore = AV.Object.extend("GameScore");
var gameScore = new GameScore();

gameScore.set("score", 1337);
gameScore.set("playerName", "Sean Plott");
gameScore.set("cheatMode", false);
gameScore.set("skills", ["pwnage", "flying"]);

gameScore.save(null, {
  success: function(gameScore) {
    // Now let's update it with some new data. In this case, only cheatMode and score
    // will get sent to the cloud. playerName hasn't changed.
    gameScore.set("cheatMode", true);
    gameScore.set("score", 1338);
    gameScore.save();
  }
});
```

LeanCloud自动查找哪些数据被改动了,所以只有"dirty"的字段会被发送到LeanCloud，
你不需要担心需要剔除你不想更新的数据.

#### fetchWhenSave

如果想更新成功后，获取更新后的最新的对象属性，请设置`fetchWhenSave`:

```
     //假设gamescore是已经存在的对象
    gameScore.fetchWhenSave(true);
    gameScore.set("cheatMode", true);
    gameScore.set("score", 1338);
    gameScore.save();
```

这个方法在对象被并发修改的时候特别有用，可以得到更新后对象的最新状态。例如维护一个计数器的场景，每次save后得到最新的计数。

#### 计数器

许多应用都需要维持一些计数器数据 -- 譬如用来跟踪游戏分数、金币甚至道具的数目等等。
LeanCloud提供了便捷的方式来对任何数字字段进行原子性的增加或者减少:

```
gameScore.increment("score");
gameScore.save();
```

你可以同样传入第二个参数到increment方法来指定增加多少,1就是默认值.

#### 数组

为了帮你存储数组类数据,有三种操作你可以原子性地改动一个数组,这需要一个
给定的key:

- add 在一个数组的末尾加入一个给定的对象
- addUnique 只会把原本不存在的对象加入数组，所以加入的位置没有保证
- remove 在一个数组中删除所有指定的所有实例

比如,我们想在数组"skills"中加入项目:

```
gameScore.addUnique("skills", "flying");
gameScore.addUnique("skills", "kungfu");
gameScore.save();
```

### 删除对象

为了在LeanCloud中删除一个对象:

```
myObject.destroy({
  success: function(myObject) {
    // The object was deleted from the LeanCloud.
  },
  error: function(myObject, error) {
    // The delete failed.
    // error is a AV.Error with an error code and description.
  }
});
```

你可以使用unset方法在一个对象中删除一个字段.

```
// After this, the playerName field will be empty
myObject.unset("playerName");

// Saves the field deletion to the LeanCloud
myObject.save();
```

批量删除一批对象可以这样：

```
AV.Object.destroyAll(objects);
```

其中objects是一个对象集合，其中的每个对象的className必须一样。


### 关系数据

对象可能与别的对象有关系,比如对于Weibo来说,一条微博信息(Post对象)可能有很
多评论(Comment对象).LeanCloud支持各种关系,包括一对一，一对多和多对多。

#### 一对一关系和一对多关系

一对一关系和一对多关系都可以通过在一个AV.Object内保存另一个对象来实现。
比如,每一个Comment都对应了一个Post.创建一个有一个Comment的Post,
你可以这样写:

```
// Declare the types.
var Post = AV.Object.extend("Post");
var Comment = AV.Object.extend("Comment");

// Create the post
var myPost = new Post();
myPost.set("title", "I'm Hungry");
myPost.set("content", "Where should we go for lunch?");

// Create the comment
var myComment = new Comment();
myComment.set("content", "Let's do Sushirrito.");

// Add the post as a value in the comment
myComment.set("parent", myPost);

// This will save both myPost and myComment
myComment.save();
```

LeanCloud内部会自动处理，调用Comment的save方法就可以同时保存两个新对象。

如果是现有对象想要关联到新对象，你同样可以通过**只用它们的objectId**来连接彼此。
请注意，不能直接像上面的例子那样将现有对象设置进去，而是必须new一个新对象并只设置id属性：

```
var post = AV.Object.createWithoutData("Post", "520c7e1ae4b0a3ac9ebe326a");
myComment.set("parent", post);
```

或者：

```
var post = new Post();
post.id = '520c7e1ae4b0a3ac9ebe326a';
myComment.set("parent", post);
```

默认情况下,当获取一个对象时,关联的AV.Object不会被获取到,这些对象的值不
能访问,除非像下面这样获取它们：

```
var post = fetchedComment.get("parent");
post.fetch({
  success: function(post) {
    var title = post.get("title");
  }
});
```

#### 多对多关系

多对多关系是通过AV.Relation来建模的.这样很像在一个key中存储一个
AV.Object数组,但是你不需要一次性下载关系中的所有对象.这使得
AV.Relation比数组可以更好地扩展到更多对象。例如,一个User可能喜欢很多Post,
在这种情况下,你可以把一个用户喜欢的所有Post存为一个Relation,
为了将一个Post加入一个User的like列表,你可以:

```
var user = AV.User.current();
var relation = user.relation("likes");
relation.add(post);
user.save();
```

你可以从一个AV.Relation中删除一个post:

```
relation.remove(post);
user.save();
```

你可以在用save方法保存前多次调用add和remove方法:

```
relation.remove(post1);
relation.remove(post2);
user.save();
```

你还可以传入一个AV.Object数组来做add和remove:

```
relation.add([post1, post2, post3]);
user.save();
```

默认情况下，relation关联的对象并不会被下载，你可以通过使用query方法返回的AV.Query
对象来获取AV.Object的列表，例如：

```
relation.query().find({
  success: function(list) {
    // list contains the posts that the current user likes.
  }
});
```

如果你仅仅要一个Post的子集,你可以在AV.Query中加入更多的条件:

```
var query = relation.query();
query.equalTo("title", "I'm Hungry");
query.find({
  success:function(list) {
    // list contains post liked by the current user which have the title "I'm Hungry".
  }
});
```

你可以在接下来关于AV.Query的章节中看到更详细的内容.一个
AV.Relation的行为很像一个AV.Object数组,所以任何
在数组可做的查询操作,也都可以作用在AV.Relation上.

如果你知道post，想反向查询user，可以通过`AV.Relation.reverseQuery`方法：

```
var query = AV.Relation.reverseQuery('_User', 'likes', post);
query.find({
  success:function(users) {
   //users是表示喜欢这个post的用户列表。
  }
});
```

### 数据类型

到现在为止我们使用了String,Number和AV.Object类型,LeanCloud同样支持
JavaScript的Date和null类型.

你可以用一个AV.Object中嵌套JavaScript对象和数组来表述更加结构化的数
据.

一些例子:

```
var number = 42;
var string = "the number is " + number;
var date = new Date();
var array = [string, number];
var object = { number: number, string: string };

var BigObject = AV.Object.extend("BigObject");
var bigObject = new BigObject();
bigObject.set("myNumber", number);
bigObject.set("myString", string);
bigObject.set("myDate", date);
bigObject.set("myArray", array);
bigObject.set("myObject", object);
bigObject.set("myNull", null);
bigObject.save();
```

AV.Objects的大小不应该超过128KB.

你可以参考文档中 数据与安全 的部分来了解更多数据处理的信息.

## 查询

### 基础查询

在很多情况下,get方法无法检索到符合你要求的数据，AV.Query提供一些
其他的方式来进行更复杂的查询。

通常的模式是创建一个AV.Query对象,加入一些条件,然后用find方法返
回一个满足条件的AV.Object数组.比如说,为了获取特定用户的分数,可
以使用equalTo方法来添加查询条件：

```
var GameScore = AV.Object.extend("GameScore");
var query = new AV.Query(GameScore);
query.equalTo("playerName", "Dan Stemkoski");
query.find({
  success: function(results) {
    alert("Successfully retrieved " + results.length + " scores.");
    // Do something with the returned AV.Object values
    for (var i = 0; i < results.length; i++) {
      var object = results[i];
      alert(object.id + ' - ' + object.get('playerName'));
    }
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
```

### 查询条件

有几种方式来设置查询条件。
你可以用notEqual方法和一个特定的值来过滤不符合要求的对象:

```
query.notEqualTo("playerName", "Michael Yabuti");
```

你可以给定更多的条件,只有满足所有条件的对象才会作为结果返回.换句话说,
就像是AND查询.

```
query.notEqualTo("playerName", "Michael Yabuti");
query.greaterThan("playerAge", 18);
```

你可以用设定limit的方法来限定返回的结果数,默认的返回结果数是100,但是任
何1到1000之间的数值都是合法的，在 0 到 1000 范围之外的都强制转成默认的 100。
.

```
query.limit(10); // limit to at most 10 results
```

如果你只想要一个结果,一个更加方便的方法可能是使用first,而不是find方法.

```
var GameScore = AV.Object.extend("GameScore");
var query = new AV.Query(GameScore);
query.equalTo("playerEmail", "dstemkoski@example.com");
query.first({
  success: function(object) {
    // Successfully retrieved the object.
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
```

你可以用skip跳过前面的结果,这可能对于分页很有用.

```
query.skip(10); // skip the first 10 results
```

对于可以排序的类型,比如number和string,你可以控制返回结果的顺序:

```
// Sorts the results in ascending order by the score field
query.ascending("score");

// Sorts the results in descending order by the score field
query.descending("score");
```

对于可以排序的类型,你同样可以在查询中进行比较.

```
// Restricts to wins < 50
query.lessThan("wins", 50);

// Restricts to wins <= 50
query.lessThanOrEqualTo("wins", 50);

// Restricts to wins > 50
query.greaterThan("wins", 50);

// Restricts to wins >= 50
query.greaterThanOrEqualTo("wins", 50);
```

如果想让返回的对象的某个属性匹配多个值,你可以使用containedIn,
提供一个数组就可以了.这样通常可以用单个的查询来获取多个结果.比如
你想获取某几个玩家的分数:

```
// Finds scores from any of Jonathan, Dario, or Shawn
query.containedIn("playerName",
                  ["Jonathan Walsh", "Dario Wunsch", "Shawn Simon"]);
```

相反地，你可以使用notContainedIn方法来查询在集合之外的目标对象。

如果你想要查询含有某一特定属性的对象,你可以使用exists.相对地,如果你想获
取没有这一特定属性的对象,你可以使用doesNotExist.

```
// Finds objects that have the score set
query.exists("score");

// Finds objects that don't have the score set
query.doesNotExist("score");
```

你可以使用matchesKeyInQuery方法来进行嵌套的子查询。
举例说,如果你有一个类包含了运动队,而你在用户的类中存储了用户
的家乡信息,你可以构造一个查询来查找某地的运动队有赢的记录的用户.查询应该看
起来像下面这样:

```
var Team = AV.Object.extend("Team");
var teamQuery = new AV.Query(Team);
teamQuery.greaterThan("winPct", 0.5);
var userQuery = new AV.Query(AV.User);
userQuery.matchesKeyInQuery("hometown", "city", teamQuery);
userQuery.find({
  success: function(results) {
    // results has the list of users with a hometown team with a winning record
  }
});
```

相对地,可以使用doesNotMatchKeyInQuery来获取属性不在子查询结果中的对象.比如为了获得用户的家乡队输了的情况:

```
var losingUserQuery = new AV.Query(AV.User);
losingUserQuery.doesNotMatchKeyInQuery("hometown", "city", teamQuery);
losingUserQuery.find({
  success: function(results) {
    // results has the list of users with a hometown team with a losing record
  }
});
```

你可以用select和一个keys的列表来限定返回的字段.为了获得只包含score和
playername字段的文档(包括build-in的字段,objectId,createdAt,
updatedAt):

```
var GameScore = AV.Object.extend("GameScore");
var query = new AV.Query(GameScore);
query.select("score", "playerName");
query.find().then(function(results) {
  // each of results will only have the selected fields available.
});
```

剩下的字段可以之后用返回的对象的fetch方法来获取:

```
query.first().then(function(result) {
  // only the selected fields of the object will now be available here.
  return result.fetch();
}).then(function(result) {
  // all fields of the object will now be available here.
});
```

###对数组值做查询

对于value是数组的情况,你可以这样查询数组中的值有2的情况的对象:

```
// Find objects where the array in arrayKey contains 2.
query.equalTo("arrayKey", 2);
```

你同样可以用下面的方式找到同时包含元素2,3,4的数组:

```
// Find objects where the array in arrayKey contains all of the elements 2, 3, and 4.
query.containsAll("arrayKey", [2, 3, 4]);
```

###对字符串类型做查询

使用startWith来限制属性值以一个特定的字符串开头，这和MySQL的LIKE操作
符很像,因为有索引所以对于大的数据集这个操作也是很高效的.

```
// Finds barbecue sauces that start with "Big Daddy's".
var query = new AV.Query(BarbecueSauce);
query.startsWith("name", "Big Daddy's");
```

###关系查询

对于查询关系型数据来说有几种不同的方式,如果你想要获取的对象中有某个属性
包含一个特定的AV.Object,你可以使用equalTo,就像对于别的数据类型一
样.举个例子,如果每一个Comment在它的post字段都有一个Post对象,你可以通过
如下的方式来获取一个Post的comment:

```
// Assume AV.Object myPost was previously created.
var query = new AV.Query(Comment);
query.equalTo("post", myPost);
query.find({
  success: function(comments) {
    // comments now contains the comments for myPost
  }
});
```

如果你想得到其字段中包含的子对象满足另一个查询的结果,你可以使用
matchesQuery操作.注意默认的结果条数限制100和最大limit 1000也同样适用于子查询,
所以对于大的数据集你可能需要小心构建你的查询,否则可能出现意料之外的状
况。例如，为了找到post中有图片的comment,你可以:

```
var Post = AV.Object.extend("Post");
var Comment = AV.Object.extend("Comment");
var innerQuery = new AV.Query(Post);
innerQuery.exists("image");
var query = new AV.Query(Comment);
query.matchesQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments now contains the comments for posts with images.
  }
});
```

如果你想要获取某字段中包含的子对象不满足指定查询的结果,你可以使用
doesNotMatchQuery.例如，为了找到针对不含图片的post的comment,你可以这样:

```
var Post = AV.Object.extend("Post");
var Comment = AV.Object.extend("Comment");
var innerQuery = new AV.Query(Post);
innerQuery.exists("image");
var query = new AV.Query(Comment);
query.doesNotMatchQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments now contains the comments for posts without images.
  }
});
```

你可以同样用objectId来做关系查询

```
var post = new Post();
post.id = "520c7e1ae4b0a3ac9ebe326a";
query.equalTo("post", post);
```

在某些情况下,你可能希望查询结果中包含多个相关联的其他数据类型.你可以使用include方
法.比如:假设你想获得最新的10个comment,你可能想同时获取它们相关的post数据:

```
var query = new AV.Query(Comment);

// Retrieve the most recent ones
query.descending("createdAt");

// Only retrieve the last ten
query.limit(10);

// Include the post data with each comment
query.include("post");

query.find({
  success: function(comments) {
    // Comments now contains the last ten comments, and the "post" field
    // has been populated. For example:
    for (var i = 0; i < comments.length; i++) {
      // This does not require a network access.
      var post = comments[i].get("post");
    }
  }
});
```

你同样可以用点操作符来做多级查询,如果你想同时找到comment的post和相应post
的author,你可以这样做:

```
query.include(["post.author"]);
```

你可以多次使用include来构建一个有多个字段的查询,这项功能同样适用于
AV.Query的helper函数例如first和get.

###对象计数

如果你只是想查询满足一个query的结果集到底有多少对象,但是你不需要得到它们,你
可以使用count来取代find. 比如,为了获得某个玩家到底玩过多少局游戏:

```
var GameScore = AV.Object.extend("GameScore");
var query = new AV.Query(GameScore);
query.equalTo("playerName", "Sean Plott");
query.count({
  success: function(count) {
    // The count request succeeded. Show the count
    alert("Sean has played " + count + " games");
  },
  error: function(error) {
    // The request failed
  }
});
```

对于超过1000个对象的类来说,count操作会被时间限制所约束.它们可能会一直
返回超时错误,或者只是返回一个近似正确的值.这样的话你应该更合理地规划你
程序的结构来避免这种情况.

###组合查询

如果你想要查找满足一系列查询的对象,你可以使用AV.Query.or方法来构建
查询,这样得到的结果是所有查询的并集。比如你想要找的玩家或者是有很多或者很
少的胜利的时候,你可以这样:

```
var lotsOfWins = new AV.Query("Player");
lotsOfWins.greaterThan("wins", 150);

var fewWins = new AV.Query("Player");
fewWins.lessThan("wins", 5);

var mainQuery = AV.Query.or(lotsOfWins, fewWins);
mainQuery.find({
  success: function(results) {
     // results contains a list of players that either have won a lot of games or won only a few games.
  },
  error: function(error) {
    // There was an error.
  }
});
```

你也可以对AV.Query加入更多的条件，如同AND查询一样，这样得到所有查询结果的交集.

请注意我们不会在组合查询的子查询中支持非过滤型的条件(比如:limit,skip,ascending/descending,include).

### 删除查询结果

如果你想将查询出来的对象都删除，或者删除符合查询条件的所有对象，可以调用`destroyAll`方法：

```
query.destroyAll({
   success: function(){
      //delete all objects by this query successfully.
   },
   error: function(err){
      //There was an error.
   }
   });
```

### CQL 查询语言

从 0.4.3 版本开始，我们允许使用类 SQL 语法的 CQL 查询语言来查询 LeanCloud 应用内的数据，例如：

```
AV.Query.doCloudQuery('select * from GameScore', {
  success: function(result){
    //results 是查询返回的结果，AV.Object 列表
    var results = result.results;
    //do something with results...
  },
  error: function(error){
    //查询失败，查看 error
    console.dir(error);
  }
});
//查询分数大于 50 的记录数并返回前100条。
AV.Query.doCloudQuery('select count(*),* from GameScore where score>50', {
  success: function(result){
    //results 是查询返回的结果，AV.Object 列表
    var results = result.results;
    //count 表示符合查询条件的总记录数
    var count = result.count;
    //do something with results...
  },
  error: function(error){
    //查询失败，查看 error
    console.dir(error);
  }
});
```

doCloudQuery 回调中的`result`包含三个属性：

* results - 查询结果的`AV.Object`列表
* count - 如果使用了`select count(*)`的查询语法，返回符合查询条件的记录数目。
* className - 查询的 class name

CQL 语法请参考 [CQL 详细指南](./cql_guide.html)。

针对查询条件，我们推荐使用占位符的 CQL 语句来提升性能，占位符对应的值按照顺序组合起来作为第二个参数 `pvalues` 数组传入：

```
//查询分数大于 50 的记录数并返回前10条。
AV.Query.doCloudQuery('select count(*),* from GameScore where score>? limit ?',[50,10],
 {
  success: function(result){
    //results 是查询返回的结果，AV.Object 列表
    var results = result.results;
    //count 表示符合查询条件的总记录数
    var count = result.count;
    //do something with results...
  },
  error: function(error){
    //查询失败，查看 error
    console.dir(error);
  }
});
```

`AV.Query.doCloudQuery` 返回的也是下面提到的 `AV.Promise` 对象。

##Promise

除了回调函数之外,每一个在LeanCloud JavaScript SDK中的异步方法都会返回一个
Promise.使用Promise，你的代码可以比原来的嵌套callback的方法看起来优雅得多.

###then 方法

每一个Promise都有一个叫then的方法,这个方法接受一对callback.第一个
callback在promise被解决的时候调用,第二个会在promise被拒绝的时候调用.

```
obj.save().then(function(obj) {
  // the object was saved successfully.
}, function(error) {
  // the save failed.
});
```

###将 Promise 组织在一起

Promise比较神奇，可以代替多层嵌套方式来解决发送异步请求代码的调用顺序问题。
如果一个Promise的回调会返回一个Promise，那么第二个then里的callback在第一个then
的callback没有解决前是不会解决的。

```
var query = new AV.Query("Student");
query.descending("gpa");
query.find().then(function(students) {
  students[0].set("valedictorian", true);
  return students[0].save();

}).then(function(valedictorian) {
  return query.find();

}).then(function(students) {
  students[1].set("salutatorian", true);
  return students[1].save();

}).then(function(salutatorian) {
  // Everything is done!

});
```

###错误处理

如果任意一个在链中的Promise返回一个错误的话,所有的成功的callback在接下
来都会被跳过直到遇到一个处理错误的callback.处理error的callback可以转换
eroor或者可以通过返回一个新的Promise的方式来处理它.你可以想象成拒绝的
promise有点像异常,而error callback则像是一个catch来处理这个异常或者抛
出异常.

```
var query = new AV.Query("Student");
query.descending("gpa");
query.find().then(function(students) {
  students[0].set("valedictorian", true);
  // Force this callback to fail.
  return AV.Promise.error("There was an error.");

}).then(function(valedictorian) {
  // Now this will be skipped.
  return query.find();

}).then(function(students) {
  // This will also be skipped.
  students[1].set("salutatorian", true);
  return students[1].save();
}, function(error) {
  // This error handler WILL be called. error will be "There was an error.".
  // Let's handle the error by returning a new promise.
  return AV.Promise.as("Hello!");

}).then(function(hello) {
  // Everything is done!
}, function(error) {
  // This isn't called because the error was already handled.
});
```

通常来说，在正常情况的回调函数链的末尾，加一个错误处理的回调函数，会是很
方便的做法.

###创建 Promise

在开始阶段,你可以只用系统（譬如find和save方法等）返回的promise.但是,在更高级
的场景下,你可能需要创建自己的promise.
在创建了Promise之后,你需要调用resolve或者reject来触发它的callback.

```
var successful = new AV.Promise();
successful.resolve("The good result.");

var failed = new AV.Promise();
failed.reject("An error message.");
```

如果你在创建promise的时候就知道它的结果,下面有两个很方便的方法可以使用:

```
var successful = AV.Promise.as("The good result.");

var failed = AV.Promise.error("An error message.");
```

###顺序的 Promise

在你想要某一行数据做一系列的任务的时候，Promise链是很方便的,每一个任务都等着前
一个任务结束.比如,假设你想要删除你的blog上的所有comment.

```
var query = new AV.Query("Comments");
query.equalTo("post", 123);

query.find().then(function(results) {
  // Create a trivial resolved promise as a base case.
  var promise = AV.Promise.as();
  _.each(results, function(result) {
    // For each item, extend the promise with a function to delete it.
    promise = promise.then(function() {
      // Return a promise that will be resolved when the delete is finished.
      return result.destroy();
    });
  });
  return promise;

}).then(function() {
  // Every comment was deleted.
});
```

###并行的 Promise

你也可以用Promise来并行的进行多个任务,这时需要使用when方法.你可以一次同
时开始几个操作.使用AV.Promise.when来创建一个新的promise，它会在所有输入
的Promise被解决之后才被解决.即便一些输入的promise失败了，新的Promise也会被
成功执行.你可以在callback的参数部分检查每一个promise的结果.并
行地进行操作会比顺序进行更快,但是也会消耗更多的系统资源和带宽.

```
var query = new AV.Query("Comments");
query.equalTo("post", 123);

query.find().then(function(results) {
  // Collect one promise for each delete into an array.
  var promises = [];
  _.each(results, function(result) {
    // Start this delete immediately and add its promise to the list.
    promises.push(result.destroy());
  });
  // Return a new promise that is resolved when all of the deletes are finished.
  return AV.Promise.when(promises);

}).then(function() {
  // Every comment was deleted.
});
```

###创建异步方法

有了上面这些工具以后,就很容易创建你自己的异步方法来返回promise了,举例
说,你可以创建一个有promise版本的setTimeout.

```
var delay = function(millis) {
  var promise = new AV.Promise();
  setTimeout(function() {
    promise.resolve();
  }, millis);
  return promise;
};

delay(100).then(function() {
  // This ran after 100ms!
});
```

##Collection

一个AV.Collection就是一个AV.Objects的有序集合.它和
Backbone.Collection是兼容的,有相同的特性和功能,你可以通过用一个模型类
或者一个特定的AV.Query来创建一个新的子类.

```
// A Collection containing all instances of TestObject.
var TestCollection = AV.Collection.extend({
  model: TestObject
});
var collection = new TestCollection();

// A Collection of TestObjects whose temperature is "hot".
var HotCollection = AV.Collection.extend({
  model: TestObject,
  query: (new AV.Query(TestObject)).equalTo("temperature", "hot")
});
var collection = new HotCollection();

// The Collection of TestObjects that match a complex query.
var query = new AV.Query(TestObject);
query.equalTo("temperature", "hot");
query.greaterThan("degreesF", 100);
var collection = query.collection();
```

###获取 Collection

使用fetch方法来获取一个collection里的所有元素:

```
var collection = new TestCollection();
collection.fetch({
  success: function(collection) {
    collection.each(function(object) {
      console.warn(object);
    });
  },
  error: function(collection, error) {
    // The collection could not be retrieved.
  }
});
```

###Collection 排序

你可以设定一个comparator来对collection中的元素进行排序:

```
var collection = new TestCollection();
collection.comparator = function(object) {
  return object.get("temperature");
};
```

###修改一个 Collection

Collection是可变的,你可以访问所有元素,增加或者删除元素.

```
var collection = new TestCollection();

collection.add([
  {"name": "Duke"},
  {"name": "Scarlett"}
]);

// Get the "Duke" AV.Object by its sorted position.
var model = collection.at(0);

// Or you can get it by LeanCloud objectId.
var modelAgain = collection.get(model.id);

// Remove "Duke" from the collection.
collection.remove(model);

// Completely replace all items in the collection.
collection.reset([
  {"name": "Hawk"},
  {"name": "Jane"}
]);
```

##文件

###新建一个 AV.File

AV.File让你可以在LeanCloud 中保存应用的文件,这样可以解决用一个
AV.Object存太大或者太难处理的问题.最常见的用例就是存储图片,但是你可
以随意用来存储文档,视频,音乐或者任何二进制数据。

在浏览器通过 javascript sdk 上传文件大小不能超过 10M，在 node.js 环境里没有这个限制。

开始使用AV.File是很容易的.有很多种不同的方式来新建一个file.第一个是
base64编码的字符串表示:

```
var base64 = "V29ya2luZyBhdCBQYXJzZSBpcyBncmVhdCE=";
var file = new AV.File("myfile.txt", { base64: base64 });
```

另外,也可以用一个byte数组来新建一个文件.

```
var bytes = [ 0xBE, 0xEF, 0xCA, 0xFE ];
var file = new AV.File("myfile.txt", bytes);
```

但是最经常的对于HTML5的应用来说,你可能需要用html表单和一个文件上传控制器.在
现代的浏览器中这很容易,只需要创建一个file input tag来允许用户选择他们磁盘
上的文件就可以了.

```
<input type="file" id="profilePhotoFileUpload">
```

然后,在一个处理click或其他的函数里,获取对那个文件的一个引用:

```
var fileUploadControl = $("#profilePhotoFileUpload")[0];
if (fileUploadControl.files.length > 0) {
  var file = fileUploadControl.files[0];
  var name = "photo.jpg";

  var avFile = new AV.File(name, file);
}
```

注意在这个例子里我们会给文件一个名字叫photo.jpg.这里有2点值得注意

- 你不需要担心文件名重复的问题,每一次上传都会有一个独一无二的标识符,所
  以上传多个文件都叫photo.jpg是没有问题的.
- 你应该给你的文件一个扩展名,这样会让LeanCloud明白文件的类型,并且会按文件
  类型来进行处理.所以如果你在储存PNG格式的文件的话,请保证你的文件名是
  以".png"为结尾的.

如果您是在Node.js里使用我们的SDK，从`0.3.1`版本开始，我们也让AV.File的构造函数接收[Buffer](http://nodejs.org/api/buffer.html)作为文件存储：

```
var file = new AV.File('test.txt', new Buffer('hello world'));
```

因为Node.js对IO的读写经常都是经过Buffer，通过支持Buffer，我们的SDK也能很好地工作在node.js环境。

从`0.3.2`版本开始，我们还支持保存一个现有存储在其他服务上的URL的文件对象：

```
var file = AV.File.withURL('test.jpg', 'https://leancloud.cn/docs/images/permission.png');
file.save();
```

下面你应该向LeanCloud 上传你的文件了.就像AV.Object一样,有很多不同的
save方法,你可以按你想用的callback和error处理的方式来使用它们.

```
avFile.save().then(function() {
  // The file has been saved to AV.
}, function(error) {
  // The file either could not be read, or could not be saved to AV.
});
```

最后,在保存完成之后,你可以将一个AV.File和一个AV.Object关联起来,
就像别的数据一样.

```
var jobApplication = new AV.Object("JobApplication");
jobApplication.set("applicantName", "Joe Smith");
jobApplication.set("applicantResumeFile", file);
jobApplication.save();
```

###获取文件的内容

怎样才能更好地获取你的应用数据取决于你的应用环境.因为跨域请求
的问题,最好你可以让浏览器代替你做这项事情.通常,这意味着在DOM中渲染这个
文件的url,下面就是我们如何用jquery将一个图片文件插入页面之中:

```
var profilePhoto = profile.get("photoFile");
$("profileImg")[0].src = profilePhoto.url();
```

如果你想在云代码中处理一个文件的数据,你可以用我们的http网络库来获
取这个文件.

```
AV.Cloud.httpRequest({ url: profilePhoto.url() }).then(function(response) {
  // The file contents are in response.buffer.
});
```

### 文件元信息

你还可以在上传文件之前设置文件的元信息，例如：

```
file.metaData().mimeType = 'text/plain';
```

获取元信息列表函数`metaDat`返回的是一个JSON对象。

默认会保存`size`和`ownerId`两个元信息，分别表示文件大小和文件上传者的object id:

```
var size = file.size();
var ownerId = file.ownerId();
```

`0.3.1`版本开始，我们增强了metaData方法，它同时是getter和setter方法:

```
//获取所有元信息组成的JSON对象
var metadata = file.metaData();
//设置format元信息
file.metaData('format','image/jpeg');
//获取format元信息
var format = file.metaData('format');
```

### 缩略图

如果保存的文件是图片，还可以通过`thumbnailURL`方法获得缩略图的URL：

```
//获得宽度为100像素，高度200像素的缩略图
var url = file.thumbnailURL(100, 200);
```

更多参数（格式、质量等）请看API文档。

### 删除文件

使用`destroy`方法来删除文件：

```
file.destroy().then(function(){
  //删除成功
}, function(error){
  //删除失败
  console.dir(error);
});
```

##用户

在许多应用中,都有一个用户账户的概念,用户账户让用户可以用安全的
方式访问他们自己的信息.我们提供了一个特殊的用户类叫AV.User来自动处
理有关用户的账户管理的功能.

使用这个类,你可以在应用中开启用户账户管理的功能.

AV.User是AV.Object的一个子类,而且有AV.Object一样的功能,比如可变的模式,自
动的持久化,还有键值对接口.所有对AV.Object有用的方法同样可以作
用于AV.User. AV.User的不同之处在于AV.User对于用户的账户有一些
特定的功能.

###属性

AV.User有一些与AV.Object不一样的字段:

- username : 用户的用户名(必须提供)
- password : 用户的密码(在注册的时候必须提供)
- email : 用户的email(可选)
- mobilePhoneNumber: 用户的手机号码（可选）

我们会在下面的用例中详细介绍细节

###注册

通常你的app第一件要做的事情就是让用户进行注册,下面的代码展示了怎样进行
通常的注册过程:

```
var user = new AV.User();
user.set("username", "my name");
user.set("password", "my pass");
user.set("email", "email@example.com");

// other fields can be set just like with AV.Object
user.set("phone", "415-392-0202");

user.signUp(null, {
  success: function(user) {
    // Hooray! Let them use the app now.
  },
  error: function(user, error) {
    // Show the error message somewhere and let the user try again.
    alert("Error: " + error.code + " " + error.message);
  }
});
```

这个调用会异步地在在你的应用中创建一个新的用户.在它这样做之前,它同
样会确认用户名和email在应用内都是唯一的.同样,为了安全我们会将密码散列
过后存储在LeanCloud 中.我们从不会将用户密码以明文存储,我们也不会用明文
向任何客户端发送密码.

注意我们使用了signUp方法而不是save方法.新的AV.User永远应该使用
signUp方法来新建.而随后的用户的信息更新可以调用save来做.

如果一个signup没有成功的话,你应该读取返回的错误对象.最常见的问题是
username或者email已经被其他用户所使用了.你应该清楚地反馈给你的用户,让
他们再次用一个不同的用户名来注册.

你也可以使用email来作为用户名,只要求你的用户输入他们的email但是同时自动
填充好username属性就可以了,AV.User会跟原来一样工作.我们会在下面的重设
密码环节再次说明这个细节.

###登录

在你要求你的用户注册之后,当然应该让他们在以后用自己的账户登录进来.你可
以使用logIn方法来进行登陆.

```
AV.User.logIn("myname", "mypass", {
  success: function(user) {
    // Do stuff after successful login.
  },
  error: function(user, error) {
    // The login failed. Check error to see why.
  }
});
```

###验证 Email

在应用设置的应用选项中中启用email验证可以让你的应用给最终用户一些更安全的使用体验，
譬如部分功能只开放给验证过邮箱的用户使用，等等。
Email验证会在AV.User上加入一个emailVerified字段.当一个
AV.User的email被设定或者修改后,emailVerfied会被设定为false.LeanCloud会
向用户的email来发送一个链接,点击这个链接会设置emailVerified为true.

有三种emailVerified状态可以供参考:

1. true 用户已经通过点击LeanCloud发过来的链接来确认邮箱地址.当用户账户新创
   建的时候这个值永远不应该是true.
2. false 在AV.User对象最后一次刷新的时候,用户还是没有确认他们的
   email地址,如果emailVerified是false的话,你应该考虑调用AV.User的
   fetch方法.
3. missing AV.User被创建了,但是当时的email验证功能还没有开启,
   或者说AV.User没有email地址.

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

用户邮箱验证后，会调用`AV.Cloud.onVerified('email',function)`的云代码回调函数，方便您做一些后处理。

### 短信验证手机号码

如果用户注册提供了 `mobilePhoneNumber`属性，并且你希望验证用户手机号码的真实性，你可能希望发送一条短信，并且让用户输入短信中的验证码来确认手机号码的真实性：

```
	        var user = new AV.User();
			user.set("username", "dennis");
			user.set("password", "123456");
			user.setMobilePhoneNumber('186xxxxxxxx');
	        user.signUp(null, ……)
```

为了发送短信，你需要在应用设置的应用选项里启用：`验证注册用户手机号码`。

如果用户注册没有收到短信，你可以通过`requestMobilePhoneVerify`方法强制重新发送：

```
	AV.User.requestMobilePhoneVerify('186xxxxxxxx').then(function(){
		//发送成功
	}, function(err){
	   //发送失败
	});
```

当用户收到验证短信后，会有 6 位数字的验证码，让用户输入，并调用`verifyMobilePhone`来确认是否正确：

```
	AV.User.verifyMobilePhone('6位数字验证码').then(function(){
		//验证成功
	}, function(err){
	   //验证失败
	});
```

验证成功后，用户的`mobilePhoneVerified`属性变为true，并且调用云代码的`AV.Cloud.onVerifed('sms', function)`方法。


### 手机号码和短信登录

当用户有填写`mobilePhoneNumber`的时候，可以使用手机和密码登录：

```
AV.User.logInWithMobilePhone('186xxxxxxxx', password).then(function(user){
   //登录成功
}, function(err){
   //登录失败
});
```

如果你在应用设置的应用选项里启用`允许用户使用手机短信登录 （需要先启用 验证注册用户手机号码）`，并且用户通过了手机号码认证，那么可以通过请求发送短信验证码来登录：

```
//请求登录验证码
 AV.User.requestLoginSmsCode('186xxxxxxxx').then(function(){
		//发送成功
	}, function(err){
	   //发送失败
	});
//用户收到6位登录验证码后，输入验证码登录
AV.User.logInWithMobilePhoneSmsCode('186xxxxxxxx', '6位登录验证码数字').then(function(user){
   //登录成功
}, function(err){
   //登录失败
});
```


###当前用户

如果用户每次打开App的时候都要求登录无疑是令人感到厌烦的,你可以通
过缓存当前的AV.User对象来避免这个问题.

无论你使用任何注册或者登录方法,用户都会在localStorage中储存,你可以把缓
存作为一个session对待,并且自动假设用户已经登录了.

```
var currentUser = AV.User.current();
if (currentUser) {
    // do stuff with the user
} else {
    // show the signup or login page
}
```

你可以通过logout来清除掉当前的用户:

```
AV.User.logOut();

var currentUser = AV.User.current();  // this will now be null
```

###用户对象的安全

AV.User类默认就是受保护的,在AV.User中保存的数据只能被那个用户所
修改.默认地,数据仍然可以被任意客户端所读取.这样就是说,有些AV.User对
象被认证后是可以修改的,其他的仍然是只读的.

特别的,你不能调用save或者delete方法除非AV.User经过了认证,就比如调用
过了logIn或者signUp方法.这样保证只有用户能改动他们自身的数据.

下面的代码展示了上面说的安全策略:

```
var user = AV.User.logIn("my_username", "my_password", {
  success: function(user) {
    user.set("username", "my_new_username");  // attempt to change username
    user.save(null, {
      success: function(user) {
        // This succeeds, since the user was authenticated on the device

        // Get the user from a non-authenticated method
        var query = new AV.Query(AV.User);
        query.get(user.objectId, {
          success: function(userAgain) {
            userAgain.set("username", "another_username");
            userAgain.save(null, {
              error: function(userAgain, error) {
                // This will error, since the AV.User is not authenticated
              }
            });
          }
        });
      }
    });
  }
});
```

从AV.User.current()获取的AV.User总是已经通过验证了的.

如果你需要查看一个AV.User是否已经认证过了,你可以调用authenticated方
法.你不需要查看一个认证方法中返回的AV.User对象是否已经通过验证了.

###其他对象的安全

和AV.User相同的安全模型也使用于其他对象.对于任何对象来说,你可以指定
哪些用户会被允许读取对象,哪些用户被允许修改对象.为了支持这种安全机制，
每一个对象都有一个允许访问列表(ACL),是被AV.ACL类所实现的.

使用一个AV.ACL最简单的方式是指定一个对象只能被一个单一的用户读或者
写.为了创建这样的对象,首先必须有一个已经登录的AV.User.然后,新的
AV.ACL(user)生成一个AV.ACL来限定user的访问.一个对象的ACL会在对象
保存的时候被存储起来,就像其他的属性一样.这样,为了创建一个当前user私有
的一个note:

```
var Note = AV.Object.extend("Note");
var privateNote = new Note();
privateNote.set("content", "This note is private!");
privateNote.setACL(new AV.ACL(AV.User.current()));
privateNote.save();
```

这个note只能由当前的用户所访问,但是可以在用户登录的任何设备上访问,只要
是相同的用户就可以了.这项功能对于你如果想让用户再任何其他的设备上保存
和访问数据十分有用.比如说一个私人的todo list应用.

权限也能在使用者的基础上授予,你可以通过setReadAccess和setWriteAccess方
法独立的向AV.ACL中添加权限.比如,假设你有一条消息想要发送给一个组里
的多个用户,他们中的每一个都有读和写的权限:

```
var Message = AV.Object.extend("Message");
var groupMessage = new Message();
var groupACL = new AV.ACL();

// userList is an array with the users we are sending this message to.
for (var i = 0; i < userList.length; i++) {
  groupACL.setReadAccess(userList[i], true);
  groupACL.setWriteAccess(userList[i], true);
}

groupMessage.setACL(groupACL);
groupMessage.save();
```

你同样可以对所有的用户授权,只要使用setPublicReadAccess和
setPublicWriteAccess就可以了.这样允许了在一个消息板上发评论的模式.比如,我
们要创建一个post只能被它的作者修改,但是可以被所有人读取:

```
var publicPost = new Post();
var postACL = new AV.ACL(AV.User.current());
postACL.setPublicReadAccess(true);
publicPost.setACL(postACL);
publicPost.save();
```

比如说删除一个对象,但是你没有写的权限这种操作是禁止的.这样会返回一个
AV.Error.OBJECT_NOT_FOUND的错误码,为了安全起见,这样防止了客户端区分
出到底有哪些对象被创建了但是无法读取还是根本不存在.

###重设密码

在现实中只要你引入了密码系统,总会有用户会忘掉他们的密码.在这种情形下,
我们的库提供一个让他们安全地重设密码的功能.

为了能让用户重设密码,应该要求用户提供他们的email地址,然后这样调用:

```
AV.User.requestPasswordReset("email@example.com", {
  success: function() {
    // Password reset request was sent successfully
  },
  error: function(error) {
    // Show the error message somewhere
    alert("Error: " + error.code + " " + error.message);
  }
});
```

这样会尝试匹配给定的email和用户的email或者username字段,然后会发送用户
的密码重设邮件.由于我们是这样做的,所以你可以选择用户是否拿email作为他
们的用户名,或者说用户把email作为用户的另一个信息保存.

密码重设的流程如下:

1. 用户输入email来请求重设他们的密码
2. LeanCloud向用户的email地址发送邮件,包含了一个重设密码的链接
3. 用户点击这个重设密码的链接,会重定向到一个LeanCloud页面来允许他们重设密
   码
4. 用户输入新的密码,他们的密码现在会更新为输入的新密码.

注意这个流程的信息会引用你的app的名字,这个名字是你初始在LeanCloud上创建的
app的名字.

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

###查询

为了查询用户,你可以简单的创建一个AV.Query针对AV.Users:

```
var query = new AV.Query(AV.User);
query.equalTo(gender, "female");  // find all the women
query.find({
  success: function(women) {
    // Do stuff
  }
});
```

###关联

关联一个AV.User的对象可以很快就见效.比如说,假设你有一个blog
程序,为了保存一个用户的新post还有读取他们所有的post.

```
var user = AV.User.current();

// Make a new post
var Post = AV.Object.extend("Post");
var post = new Post();
post.set("title", "My New Post");
post.set("body", "This is some great content.");
post.set("user", user);
post.save(null, {
  success: function(post) {
    // Find all posts by the current user
    var query = new AV.Query(Post);
    query.equalTo("user", user);
    query.find({
      success: function(usersPosts) {
        // userPosts contains all of the posts by the current user.
      }
    });
  }
});
```

###在后台查看 User

在后台的数据查看中,你可以看到User类保存了用户的信息.

## 短信验证服务

对于一些危险的操作，例如付费，删除数据等，你可能希望用户接收短信验证码并验证通过之后才允许进行，那么可以使用我们提供的短信验证服务。

首选需要在应用设置的应用选项里开启`启用手机号码短信认证 （针对 /1.1/verifySmsCode/:code 接口）`选项。

发送验证码通过：

```
	AV.Cloud.requestSmsCode('186xxxxxxxx').then(function(){
	    //发送成功
	}, function(err){
		//发送失败
	});
```

你还可以定制发送的内容，设置下列选项：

* name 应用名称，默认是你的应用在 LeanCloud 显示的名称。
* op 进行的操作字符串，例如`付费`。
* ttl 以分钟为单位的过期时间。

```
	AV.Cloud.requestSmsCode({
	   mobilePhoneNumber: '186xxxxxxxx',
	   name: 'PP打车',
	   op: '付费',
	   ttl: 5
	}).then(function(){
	    //发送成功
	}, function(err){
		//发送失败
	});
```

如果您在应用设置里创建了短信模板，并且通过了管理员审核，那就可以发送模板短信，假设模板名称为 `test`，模板内容为

```
欢迎您使用 {{name}} 服务，我们将在 {{date}} 举办庆祝活动，欢迎参加。
```

其中`name` 和 `date` 都是可替换的模板变量，那么可以通过下列方式来发送这条模板短信：

```
	AV.Cloud.requestSmsCode({
	   mobilePhoneNumber: '186xxxxxxxx',
	   template: "test"
	   name: 'PP打车',
	   date: '2014 年 10 月 22 号',
	   ttl: 5
	}).then(function(){
	    //发送成功
	}, function(err){
		//发送失败
	});
```

`template` 指定模板名称，`mobilePhoneNumber` 是接收短信的手机号码，其他变量都将作为模板变量渲染。发送的短信内容将渲染为 `欢迎您使用 pp打车 服务，我们将在 2014 年 10 月 22 号 举办庆祝活动，欢迎参加。`。

在用户收到验证码并输入后，通过下列代码来验证是否正确：

```
   AV.Cloud.verifySmsCode('6位数字验证码').then(function(){
	    //验证成功
	}, function(err){
		//验证失败
	});
```

##角色

随着你的App规模和用户基数的成长,你可能发现你需要比设定用户级的权限更加
宽泛的权限设置.LeanCloud提供一种基于角色的权限管理方案来满足这种需求.角色
提供了一种逻辑的方式来将用户分组并给与相同的权限.角色是一种有名字的对
象,包含了用户和其他的角色.任何授予一个角色的权限会被它包含的所有用户和
子角色所继承.

例如,在你的App中管理着一些内容,你可能有一些类似于"主持人"的角色可以修
改和删除其他用户创建的新的内容,你可能还有一些"管理员"有着与"主持人"相
同的权限,但是还可以修改App的其他全局性设置.通过给予用户这些角色,你可以
保证新的用户可以做主持人或者管理员,不需要手动地授予每个资源的权限给各
个用户.

我们提供一个特殊的类称为AV.Role在客户端代码中表示这种角色对
象,AV.Role是一个AV.Object的子类,而且有所有的特性,比如没有固定模
式,自动持久化和key value接口等.所有的在AV.Object上有用的方法在
AV.Role上依然有作用.不同之处是AV.Role有一些普通对象没有的特殊属
性和方法.

###属性

AV.Role有一些属性与普通的AV.Object不同:

- name 角色的名称,这个值是必须的,而且只能在角色创建的时候指定一次,名字
  必须由字母,数字,空格,减号或者下划线组成.名称会被用于表示角色名而不需
  要角色的objectId
- users 一个关系,包含了会继承角色权限的User
- roles 一个关系,包含了会继承角色权限的子角色

###角色对象的安全性

AV.Role使用和其他LeanCloud对象一样的ACL权限策略,除开它需要ACL被显式地设
置以外.通常来说,只有用户有极大的权限(比如管理员)才应该被允许创建或者更
改Role.所以你应该按这种标准来设定Role的ACL.请注意,如果你给了用户一个
AV.Role一个写权限,这个用户有可能会在这个权限中加入另一个user,或者甚
至直接把角色删除掉.

为了创建一个新的AV.Role,你应该如下写:

```
// By specifying no write privileges for the ACL, we can ensure the role cannot be altered.
var roleACL = new AV.ACL();
roleACL.setPublicReadAccess(true);
var role = new AV.Role("Administrator", roleACL);
role.save();
```

你可以通过增加"user"和"roles"关系的成员来在AV.Role中加入用户或者子
角色:

```
var role = new AV.Role(roleName, roleACL);
for (var i = 0; i < usersToAddToRole.length; i++) {
  role.getUsers().add(usersToAddToRole[i]);
}
for (var i = 0; i < rolesToAddToRole.length; i++) {
  role.getRoles().add(rolesToAddToRole[i]);
}
role.save();
```

请非常注意一点,注册角色的ACL的时候,它们只能被应该有权限修改它的人修改.

###其他对象的安全性

现在你应该已经创建了在你的程序中要使用的一系列的角色,你可以用ACL来定义
他们的用户可以拥有的权限.每一个AV.Object都可以指定一个AV.ACL,这
样提供了哪些用户或者角色应该有权限来读或者写这个对象.

将一个读或者写的权限授予一个角色是很直观的.你可以使用Rarse.Role:

```
var moderators = /* Query for some AV.Role */;
var wallPost = new AV.Object("WallPost");
var postACL = new AV.ACL();
postACL.setRoleWriteAccess(moderators, true);
wallPost.setACL(postACL);
wallPost.save();
```

你可以不需要查找这个Role,直接把名字提供给ACL:

```
var wallPost = new AV.Object("WallPost");
var postACL = new AV.ACL();
postACL.setRoleWriteAccess("Moderators", true);
wallPost.setACL(postACL);
wallPost.save();
```

###角色继承

就像上面所描述的一样,一个角色可能包含其他的角色,表示两个角色之间的父-
子关系,这样做的结果就是任何被授予一个角色的权限都会被隐式地授予这个角
色的所有子角色.

这样的关系很经常会在有用户管理内容的程序之中看到,比如论坛,有一个很少量
的用户称为管理员,有最高的权限,比如程序设定,创建新的论坛,设定所有人能看
到的内容等等.另一类有一部分类似于"版主"的用户,这些人有责任保持用户创建
的内容是合适的.任何一个"版主"有的权限"管理员"都应该有.为了启用这种关系,你
应该使"管理员"成为"版主"的一个子角色.

```
var administrators = /* Your "Administrators" role */;
var moderators = /* Your "Moderators" role */;
moderators.getRoles().add(administrators);
moderators.save();
```

##云代码 函数

云代码 函数应该用AV.Cloud.run函数来进行调用,比如,调用云代码中的
函数"hello"应该这样:

```
AV.Cloud.run('hello', {}, {
  success: function(result) {
    // result is 'Hello world!'
  },
  error: function(error) {
  }
});
```

你可以参考我们的Cloud Code指南来进一步了解这部分功能.

##Push 通知

通过JavaScript SDK也可以向移动设备发送推送消息，一个简单例子推送给所有订阅了`public`频道的设备：

```
AV.Push.send({
  channels: [ "Public" ],
  data: {
     alert: "Public message"
  }
});
```
这就向订阅了`public`频道的设备发送了一条内容为`public message`的消息。

如果希望按照某个 `_Installation` 表的查询条件来推送，例如推送给某个 `installationId` 的 Android 设备，可以传入一个 `AV.Query` 对象作为 `where` 条件：

```
var query = new AV.Query("_Installation");
query.equalTo("installationId", installationId);
AV.Push.send({
  where: query,
  data: {
     alert: "Public message"
  }
});
```

此外，如果你觉得 AV.Query 太繁琐，也可以写一句 [CQL](./cql_guide.html) 来搞定：

```
AV.Push.send({
  cql: "select * from _Installation where installationId='设备id'",
  data: {
     alert: "Public message"
  }
});
```

`AV.Push` 的更多使用信息参考 API 文档[AV.Push](https://leancloud.cn/docs/api/javascript/symbols/AV.Push.html)。

更多推送的查询条件和格式，请查阅我们的[Push Notification指南](./push_guide.html)来获取更详细的信息

##地理位置

LeanCloud允许你能够将真实世界的经度和纬度坐标放入对象之中.在AV.Object中
加入一个AV.GeoPoint可以让你查询一个Object离一个参考点的相对位置.这
允许你轻松的发现一个用户周围最近的用户,或者离一个用户最近的地点.

###AV.GeoPoint

为了将一个对象联系到一个点上,你需要先创建一个AV.GeoPoint.举例来说,
为了创建一个地理位置在纬度40度,经度在-30度的点:

```
var point = new AV.GeoPoint({latitude: 40.0, longitude: -30.0});
```

这个点接着就在对象中被指定了:

```
placeObject.set("location", point);
```

注意:现在我们只支持一个类中只能有一个key能对应AV.GeoPoint.

###地理位置查询

现在你可以有一系列的对象可以做空间坐标查询了,如果能轻松地发现有哪些对
象离一个点最近就好了.这样可以通过在AV.Query中加入一个near来做查询,
为了获得离用户最近的10个地点列表,可以这样:

```
// User's location
var userGeoPoint = userObject.get("location");
// Create a query for places
var query = new AV.Query(PlaceObject);
// Interested in locations near user.
query.near("location", userGeoPoint);
// Limit what could be a lot of points.
query.limit(10);
// Final list of objects
query.find({
  success: function(placesObjects) {
  }
});
```

在这时placeObjects会返回一个按离userGeoPoint的距离排序的列表,注意如果
一个ascending()/descending()给了查询的话,会取代按距离排序这项特性.

为了按距离限制返回的结果,你可以使用withinMiles,withinKilometers和
withinRadians.

同样,查询在特定地域的Object是可以的,为了找到用矩形表示的一块地域中的对
象,加入withinGeoBox来在AV.Query中加入约束.

```
var southwestOfSF = new AV.GeoPoint(37.708813, -122.526398);
var northeastOfSF = new AV.GeoPoint(37.822802, -122.373962);

var query = new AV.Query(PizzaPlaceObject);
query.withinGeoBox("location", southwestOfSF, northeastOfSF);
query.find({
  success: function(pizzaPlacesInSF) {
    ...
  }
});
```

###警告

在这里是有一些问题是值得留心的:

1. 每一个AV.Object只能有一个键指向一个AV.GeoPoint对象
2. Points不应该等于或者超出它的界. 纬度不应该是-90.0或者90.0,经度不应
   该是-180.0或者180.0. 试图在GeoPoint上使用超出范围内的经度和纬度会导
   致问题.


###View

我们引入的AV.View只是一个Backbone.View的简单复制,你可以随意拿它来建
立视图,你可以查看Backbone.View的API来查看详细信息,注意当你使用了
AV.View的时候,你需要包含一个jQuery库或者jQuery兼容的库(实现了$方法
的)

###转换 Backbone app

如果你已经有一个存在的Backbone程序,你可以用我们的JavaScript SDK轻松地
转换.在转换过后,你就有一些静态文件但是包含了你的app的所有功能.

我们的JavaScript SDK是Backbone兼容的,这意味着我们的AV.Object和
AV.Collection都可以用Backbone.Model和Backbone.Collection加上一点点
变化轻松地转换而来.下面就是怎样转换你的app:

1. 按照我们的说明来在你已经有的JavaScript程序中安装SDK.
2. 将所有的Backbone.Model都替换成AV.Object.这样做的时候,url和
   urlRoot应该用恰当的className替换,这些对象映射为LeanCloud的类

```
var BackboneTodo = Backbone.Model.extend({
  urlRoot: "/todos"
});

var AVOSCloudTodo = AV.Object.extend({
  className: "Todo"
});
```

3. 将所有的Backbone.Collection替换为AV.Collection然后指定
   AV.Object类作为model.你应该同时指定query所以collection知道怎样获
   取对象

```
var AVOSCloudTodoCollection = AV.Collection.extend({
  model: AVOSCloudTodo
});

var todos = new AVOSCloudTodoCollection();

// Construct a query to get the current user's todo items
var query = new AV.Query(AVOSCloudTodo);
query.equalTo("user", AV.User.current());
todos.query = query;
todos.fetch();
```

4. 在任何你建立model并从服务器获取的地方,你需要建立一个AV.Query来获
   取你感兴趣的对象,就像我们必须对AV.Collection的query属性做的一样
5. 加入或者将你的app更新为使用用户认证方式的,并且在需要的对象上应用ACL

这样就结束了,你的App应该已经就绪并使用LeanCloud作为后端.

##错误处理

大部分LeanCloud JavaScript函数会通过一个有callback的对象来报告它们是否成功
了,与Backbone的"options"对象类似.主要的2个callback是success和error.
在一个操作都没有错误发生的时候success会被调用.通常来说,它的参数在save
或者get的情况下可能是AV.Object, 或者在find的情形下是一个
AV.Object数组.

error会在任何一种在与LeanCloud的网络连接发生错误的时候调用.这些错误
信息一般会反映连接到cloud时出现的一些问题,或者处理请求的操作时遇到的一
些问题.我们可以看下另一个例子,在下面的代码中我们想要获取一个不存在的
objectId. LeanCloud会返回一个错误,所以这里就是我们怎样在你的callback
里处理错误.

```
var query = new AV.Query(Note);
query.get("aBcDeFgH", {
  success: function(results) {
    // This function will *not* be called.
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // This will be called.
    // error is an instance of AV.Error with details about the error.
    if (error.code === AV.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    }
  }
});
```

查询在无法连接到LeanCloud的时候同样有可能失败.下面是同样的
callback但是有一些其他的代码来处理这种情况:

```
var query = new AV.Query(Note);
query.get("thisObjectIdDoesntExist", {
  success: function(results) {
    // This function will *not* be called.
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // This will be called.
    // error is an instance of AV.Error with details about the error.
    if (error.code === AV.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    } else if (error.code === AV.Error.CONNECTION_FAILED) {
      alert("Uh oh, we couldn't even connect to the LeanCloud!");
    }
  }
});
```

对于像是save或者是signUp这种方法会对一个特定的AV.Object起作用的方法
来说,error函数的第一个参数是object本身,第二个是一个AV.Error对象,这
是为了与其他的Backbone类型的框架兼容而设计的.请查看JavaScript API来得
到所有的AV.Error的返回码.
