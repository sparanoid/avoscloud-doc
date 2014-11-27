# REST API 详解

REST API可以让您用任何可以发送HTTP请求的设备来与AVOS Cloud进行交互,您可以使用REST API做很多事情,比如:

* 一个移动网站可以通过Javascript来获取AVOS Cloud上的数据.
* 一个网站可以展示来自AVOS Cloud的数据。
* 您可以上传大量的数据,之后可以被一个移动app读取。
* 您可以下载最近的数据来进行您自定义的分析统计。
* 使用任何语言写的程序都可以操作AVOS Cloud上的数据。
* 如果您不再需要使用AVOS Cloud，您可以导出您所有的数据。

## API 版本

* 1.1 版本： 2014 年 8 月 13 号发布，修复 Date 类型和 createdAt、updatedAt 的时区问题，返回标准 UTC 时间。
* 1 版本：  存在时间不准确的Bug，实际返回的Date 类型和 createdAt、updatedAt都是北京时间。不推荐再使用。

## 快速参考

所有的API访问都是通过HTTPS进行的.API访问需要在 __https://leancloud.cn__ 域名下,相对路径前缀 __/1.1/__ 表明现在使用的是第 1.1 版的API。

在线测试API，请打开[https://leancloud.cn/apionline/](https://leancloud.cn/apionline/)。

### 对象

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/classes/&lt;className&gt;</td>
      <td>POST</td>
      <td>创建对象</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>获取对象</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>更新对象</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;</td>
      <td>GET</td>
      <td>查询对象</td>
    </tr>
    <tr>
      <td>/1.1/cloudQuery</td>
      <td>GET</td>
      <td>使用 CQL 查询对象</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>删除对象</td>
    </tr>
  </tbody>
</table>

### 用户

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/users</td>
      <td>POST</td>
      <td>用户注册<br/>用户连接</td>
    </tr>
    <tr>
      <td>/1.1/login</td>
      <td>GET</td>
      <td>用户登录</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>获取用户</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;/updatePassword</td>
      <td>PUT</td>
      <td>更新密码，要求输入旧密码。</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>更新用户<br/>用户连接<br/>验证Email</td>
    </tr>
    <tr>
      <td>/1.1/users</td>
      <td>GET</td>
      <td>查询用户</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>删除用户</td>
    </tr>
    <tr>
      <td>/1.1/requestPasswordReset</td>
      <td>POST</td>
      <td>请求密码重设</td>
    </tr>
    <tr>
      <td>/1.1/requestEmailVerify</td>
      <td>POST</td>
      <td>请求验证用户邮箱</td>
    </tr>
    <tr>
      <td>/1.1/requestMobilePhoneVerify</td>
      <td>POST</td>
      <td>请求发送用户手机号码验证短信</td>
    </tr>
    <tr>
      <td>/1.1/verifyMobilePhone/&lt;code&gt;</td>
      <td>POST</td>
      <td>使用"验证码"验证用户手机号码</td>
    </tr>
    <tr>
      <td>/1.1/requestLoginSmsCode</td>
      <td>POST</td>
      <td>请求发送手机号码登录短信。</td>
    </tr>
    <tr>
      <td>/1.1/requestPasswordResetBySmsCode</td>
      <td>POST</td>
      <td>请求发送手机短信验证码重置用户密码。</td>
    </tr>
    <tr>
      <td>/1.1/resetPasswordBySmsCode/&lt;code&gt;</td>
      <td>PUT</td>
      <td>验证手机短信验证码并重置密码。</td>
    </tr>
  </tbody>
</table>

### 角色

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/roles</td>
      <td>POST</td>
      <td>创建角色</td>
    </tr>
    <tr>
      <td>/1.1/roles/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>获取角色</td>
    </tr>
    <tr>
      <td>/1.1/roles/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>更新角色</td>
    </tr>
    <tr>
      <td>/1.1/roles</td>
      <td>GET</td>
      <td>查询角色</td>
    </tr>
    <tr>
      <td>/1.1/roles/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>删除角色</td>
    </tr>
  </tbody>
</table>


### 推送通知

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/push</td>
      <td>POST</td>
      <td>推送通知</td>
    </tr>
  </tbody>
</table>

### 安装数据

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/installations</td>
      <td>POST</td>
      <td>上传安装数据</td>
    </tr>
    <tr>
      <td>/1.1/installations/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>获取安装数据</td>
    </tr>
    <tr>
      <td>/1.1/installations/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>更新安装数据</td>
    </tr>
    <tr>
      <td>/1.1/installations</td>
      <td>GET</td>
      <td>查询安装数据</td>
    </tr>
    <tr>
      <td>/1.1/installations/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>删除安装数据</td>
    </tr>
  </tbody>
</table>

### Cloud函数

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/functions</td>
      <td>POST</td>
      <td>调用Cloud Code函数</td>
    </tr>
  </tbody>
</table>

### 用户反馈组件

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/feedback</td>
      <td>POST</td>
      <td>提交新的用户反馈</td>
    </tr>
  </tbody>
</table>


### 短信验证 API

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/requestSmsCode</td>
      <td>POST</td>
      <td>请求发送短信验证码</td>
    </tr>
        <tr>
      <td>/1.1/verifySmsCode/&lt;code&gt;</td>
      <td>POST</td>
      <td>验证短信验证码</td>
    </tr>
  </tbody>
</table>


### 统计数据查询

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/stats/appinfo</td>
      <td>GET</td>
      <td>获取应用的基本信息</td>
    </tr>
    <tr>
      <td>/1.1/stats/appmetrics</td>
      <td>GET</td>
      <td>获取应用的具体统计数据</td>
    </tr>
  </tbody>
</table>

### 实时通信

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/rtm/messages/logs/</td>
      <td>GET</td>
      <td>获得应用聊天记录</td>
  </tbody>
</table>

### 请求格式

对于POST和PUT请求,请求的主体必须是JSON格式,而且HTTP header的Content-Type需要设置为application/json.

用户验证是通过HTTP header来进行的, __X-AVOSCloud-Application-Id__ 头标明正在运行的是哪个App程序, 而 __X-AVOSCloud-Application-Key__ 头用来授权鉴定endpoint.在下面的例子中,您的app的key被包含在命令中,您可以使用下拉框来显示其他app的示例代码.

对于Javascript使用,AVOS Cloud支持跨域资源共享,所以您可以将这些header同XMLHttpRequest一同使用.

#### 更安全的鉴权方式

我们服务端目前支持一种新的API鉴权方式，用户仍然需要传递`X-AVOSCloud-Application-Id`的http头表示App id，但是不需要再传递`X-AVOSCloud-Application-Key`。

替代地，增加了新HTTP头部——`X-AVOSCloud-Request-Sign`头，它的值要求是一个形如`sign,timestamp[,master]`的字符串，其中：

* timestamp（必须） - 客户端产生本次请求的unix时间戳，精确到毫秒。
* sign（必须）- 将timestamp加上app key(或者master key)组成的字符串做MD5签名。
* master （可选）- 字符串"master"，当使用master key签名请求的时候，必须加上这个后缀明确说明是使用master key。

我们举个例子来说明：假设

* 应用App id为`mdx1l0uh1p08tdpsk8ffn4uxjh2bbhl86rebrk3muph08qx7`,
* App key为`n35a5fdhawz56y24pjn3u9d5zp9r1nhpebrxyyu359cq0ddo`,
* Master key为`h2ln3ffyfzysxmkl4p3ja7ih0y6sq5knsa2j0qnm1blk2rn2`。

那么：

* x-avoscloud-request-sign: 28ad0513f8788d58bb0f7caa0af23400,1389085779854  -- 表示请求时间戳为`1389085779854`，签名为`28ad0513f8788d58bb0f7caa0af23400`，签名是通过对`1389085779854n35a5fdhawz56y24pjn3u9d5zp9r1nhpebrxyyu359cq0ddo`的字符串做md5sum得到，也就是时间戳加上app key组成的字符串做MD5签名。
* x-avoscloud-request-sign: c884fe684c17c972eb4e33bc8b29cb5b,1389085779854,master -- 表示使用master key产生签名，时间戳仍然是`1389085779854`，签名是通过对`1389085779854h2ln3ffyfzysxmkl4p3ja7ih0y6sq5knsa2j0qnm1blk2rn2`做md5sum得到，最后的master告诉服务器这个签名是使用master key产生的。

### 响应格式

对于所有的请求的响应格式都是一个JSON对象.

一个请求是否成功是由HTTP状态码标明的. 一个2XX的状态码表示成功,而一个4XX表示请求失败.当一个请求失败时响应的主体仍然是一个JSON对象,但是总是会包含code和error这两个字段,您可以用它们来进行debug.举个例子,如果尝试用不允许的key来保存一个对象会得到如下信息:

```
{
  "code": 105,
  "error": "invalid field name: bl!ng"
}
```

错误代码请看[错误代码详解](./error_code.html)。

##对象

###对象格式

通过REST API保存数据需要将对象的数据通过JSON来编码.这个数据是无模式化的（Schema Free）,这意味着您不需要提前标注每个对象上有那些key.您只需要随意设置key-value对就可以,后端会存储它的.
举个例子,假设您正在记录一局游戏的最高分.一个简单的对象可能包含:
```
{
  "score": 1337,
  "playerName": "Sean Plott",
  "cheatMode": false
}
```
Key必须是字母和数字组成的String,Value可以是任何可以JSON编码的东西.
每个对象都有一个类名,您可以通过类名来区分不同的数据.例如,我们可以把游戏高分对象称之为GameScore.我们推荐您使用 `NameYourClassesLikeThis` 和 `nameYourKeysLikeThis` 这样的格式为您的Key-Value命名,可以使您的代码看起来很漂亮.
当你从AVOS Cloud中获取对象时,一些字段会被自动加上: `createdAt`, `updatedAt` 和 `objectID`. 这些字段的名字是保留的,您不能自行设置它们.我们上面设置的对象在获取时应该是下面的样子.
```
{
  "score": 1337,
  "playerName": "Sean Plott",
  "cheatMode": false,
  "createdAt": "2011-08-20T02:06:57.931Z",
  "updatedAt": "2011-08-20T02:06:57.931Z",
  "objectId": "51e3a334e4b0b3eb44adbe1a"
}
```
createdAt和updatedAt都是UTC时间戳,以ISO 8601标准和毫秒级精度储存:`YYYY-MM-DDTHH:MM:SS.MMMMZ`. objectId 是一个string,在类中唯一标明了一个对象.
在REST API中class级的在一个资源上的操作只会根据类名来进行.例如,如果类名是GameScore,那么class的URL就是
```
https://leancloud.cn/1.1/classes/GameScore
```
用户有一个特殊的类级的url:
```
https://leancloud.cn/1.1/users
```
针对于一个特定的对象的操作可以通过组织一个URL来做.例如,对GameScore中的一个objectId为`51e3a334e4b0b3eb44adbe1a`的对象的操作应使用如下URL:
```
https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```
###创建对象
为了在AVOS Cloud上创建一个新的对象,应该向class的URL发送一个POST请求,其中应该包含对象本身.例如,要创建如上说的对象:
```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
  https://leancloud.cn/1.1/classes/GameScore
```
当创建成功时,HTTP的返回是201 Created,而header中的Location表示新的object的URL:
```
Status: 201 Created
Location: https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```
响应的主体是一个JSON对象,包含新的对象的objectId和createdAt时间戳.
```
{
  "createdAt": "2011-08-20T02:06:57.931Z",
  "objectId": "51e3a334e4b0b3eb44adbe1a"
}
```
###获取对象
当你创建了一个对象时,你可以通过发送一个GET请求到返回的header的Location以获取它的内容.例如,为了得到我们上面创建的对象:
```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```
返回的主体是一个JSON对象包含所有用户提供的field加上createdAt,updatedAt和objectId字段:
```
{
  "score": 1337,
  "playerName": "Sean Plott",
  "cheatMode": false,
  "skills": [
    "pwnage",
    "flying"
  ],
  "createdAt": "2011-08-20T02:06:57.931Z",
  "updatedAt": "2011-08-20T02:06:57.931Z",
  "objectId": "51e3a334e4b0b3eb44adbe1a"
}
```
当获取的对象有指向其子对象的指针时,您可以加入include选项来获取这些子对象.按上面的实例,通过`game`这个key来指向一个对象:
```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'include=game' \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

###更新对象

为了更改一个对象上已经有的数据,您可以发送一个PUT请求到对象相应的URL上,任何您未指定的key都不会更改,所以您可以只更新对象数据的一个子集.例如,我们来更改我们对象的一个score的字段:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"score":73453}' \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

返回的JSON对象只会包含一个updatedAt字段,表明更新发生的时间:

```
{
  "updatedAt": "2011-08-21T18:02:52.248Z"
}
```

####计数器

为了存储一个计数器类型的数据,AVOS Cloud提供对任何数字字段进行原子增加(或者减少)的功能,所以我们可以让score像下面一样增加:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"score":{"__op":"Increment","amount":1}}' \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

####数组

为了存储数组型数据,AVOS Cloud提供3种操作来原子性地更改一个数组字段:

* Add 在一个数组字段的后面添加一些指定的对象(包装在一个数组内)
* AddUnique 只会在数组内原本没有这个对象的情形下才会添加入数组,插入的位置不定.
* Remove 从一个数组内移除所有的指定的对象

每一种方法都会有一个key是`objects`即被添加或删除的对象列表.举个例子,我们可以在类似于`技能`的集合里加入一些对象:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"skills":{"__op":"AddUnique","objects":["flying","kungfu"]}}' \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

####关系

为了更新Relation的类型,AVOS Cloud提供特殊的操作来原子化地添加和删除一个关系,所以我们可以像这样添加一个关系:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"opponents":{"__op":"AddRelation","objects":[{"__type":"Pointer","className":"Player","objectId":"51c3ba67e4b0f0e851c16221"}]}}' \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

或者可以在一个对象中删除一个关系:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"opponents":{"__op":"RemoveRelation","objects":[{"__type":"Pointer","className":"Player","objectId":"51fa3f64e4b05df1766cfb90"}]}}' \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

###删除对象

为了在AVOS Cloud上删除一个对象,可以发送一个DELETE请求到指定的对象的URL,比如:

```
curl -X DELETE \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

您可以在一个对象中删除一个字段，通过Delete操作:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"opponents":{"__op":"Delete"}}' \
  https://leancloud.cn/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a
```

###批量操作

为了减少网络交互的次数太多带来的时间浪费,您可以在一个请求中对多个对象进行create/update/delete操作.

在一个批次中每一个操作都有相应的方法、路径和主体,这些参数可以代替您通常会使用的HTTP方法.这些操作会以发送过去的顺序来执行,比如我们要创建一系列的GameScore的对象:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "requests": [
          {
            "method": "POST",
            "path": "/1.1/classes/GameScore",
            "body": {
              "score": 1337,
              "playerName": "Sean Plott"
            }
          },
          {
            "method": "POST",
            "path": "/1.1/classes/GameScore",
            "body": {
              "score": 1338,
              "playerName": "ZeroCool"
            }
          }
        ]
      }' \
  https://leancloud.cn/1.1/batch
```

批量操作的响应会是一个列表,列表的元素数量和给定的操作数量是一致的.每一个在列表中的元素都有一个字段是"success"或者"error"."success"的值是通常是进行其他REST操作会返回的值:

```
{
  "success": {
    "createdAt": "2012-06-15T16:59:11.276Z",
    "objectId": "51c3ba67e4b0f0e851c16221"
  }
}
```

"error"的值会是一个对象有返回码和"error"字符串:

```
{
  "error": {
    "code": 101,
    "error": "object not found for delete"
  }
}
```

在batch操作中update和delete同样是有效的:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "requests": [
          {
            "method": "PUT",
            "path": "/1.1/classes/GameScore/51e3a334e4b0b3eb44adbe1a",
            "body": {
              "score": 999999
            }
          },
          {
            "method": "DELETE",
            "path": "/1.1/classes/GameScore/51a8a4d9e4b0d034f6159a35"
          }
        ]
      }' \
  https://leancloud.cn/1.1/batch
```

###数据类型

到现在为止我们只使用了可以被标准JSON编码的值,AVOS移动客户端SDK library同样支持日期,二进制数据和关系型数据.在REST API中,这些值都被编码了,同时有一个"__type"字段来标示出它们的类型,所以如果您采用正确的编码的话就可以读或者写这些字段.

Date类型包含了一个"iso"字段包含了一个UTC时间戳,以ISO 8601格式和毫秒级的精度来存储时间: `YYYY-MM-DDTHH:MM:SS.MMMZ`.

```
{
  "__type": "Date",
  "iso": "2011-08-21T18:02:52.249Z"
}
```

Date 和内置的createdAt字段和updatedAt字段相结合的时候特别有用,举个例子:为了找到在一个特殊时间创建的对象,只需要将Date编码在一个对比的query里面:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"createdAt":{"$gte":{"__type":"Date","iso":"2011-08-21T18:02:52.249Z"}}}' \
  https://leancloud.cn/1.1/classes/GameScore
```

Byte类型包含了一个"base64"字段,这个字段是一些二进制数据编码过的"base64"字符串,base64的标准是MIME使用的标准,不包含空白符.

```
{
  "__type": "Bytes",
  "base64": "VGhpcyBpcyBhbiBlbmNvZGVkIHN0cmluZw=="
}
```

Pointer类型是当移动代码设定 `AVObject` 作为另一个对象的值时使用的.它包含了className和objectId作为提及的值.

```
{
  "__type": "Pointer",
  "className": "GameScore",
  "objectId": "51e3a6b5e4b0169469540546"
}
```

指向用户对象的Pointer的className为`_User`,前面加一个下划线表示开发者不能定义的类名,而且所指的类是特别内置的.

Relation类型被用在多对多的类型上,移动端使用 `AVRelation` 作为值,它有一个className字段表示目标对象的类名.

```
{
  "__type": "Relation",
  "className": "GameScore"
}
```

当使用查询时,Relation对象的行为很像是Pointer的数组,任何操作针对于pointer的数组的(除了include)都可以对Relation起作用.

当更多的数据类型被加入的时候,它们都会采用hashmap加上一个__type字段的形式,所以您不应该使用__type作为您自己的JSON对象的key.

##查询

###基础查询

你可以一次获取多个对象通过发送一个GET请求到类的URL上,不需要任何URL参数,下面就是简单地获取所有在类之中的对象:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/classes/GameScore
```

返回的值就是一个JSON对象包含了results字段,它的值就是对象的列表:

```
{
  "results": [
    {
      "playerName": "Jang Min Chul",
      "updatedAt": "2011-08-19T02:24:17.787Z",
      "cheatMode": false,
      "createdAt": "2011-08-19T02:24:17.787Z",
      "objectId": "51c3ba67e4b0f0e851c16221",
      "score": 80075
    },
    {
      "playerName": "Sean Plott",
      "updatedAt": "2011-08-21T18:02:52.248Z",
      "cheatMode": false,
      "createdAt": "2011-08-20T02:06:57.931Z",
      "objectId": "51e3a334e4b0b3eb44adbe1a",
      "score": 73453
    }
  ]
}
```

###查询约束

通过where参数的形式可以对查询对象做出约束.`where`参数的值应该是JSON编码过的.就是说,如果您查看真正被发出的URL请求,它应该是先被JSON编码过,然后又被URL编码过.最简单的使用where参数的方式就是包含应有的key的value.举例说,如果我们想要得到Sean Plott的分数,而且他不在作弊模式下,我们应该这样构造查询:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"playerName":"Sean Plott","cheatMode":false}' \
  https://leancloud.cn/1.1/classes/GameScore
```

where的参数的值应该支持不光是匹配还有比较的方式,除了给定一个确定的值的方式,提供一个hash中有key用于比较也可以.where参数支持下面一些选项:

<table>
  <tr><th>Key</th><th>Operation</th></tr>
  <tr><td>$lt</td><td>小于</td></tr>
  <tr><td>$lte</td><td>小于等于</td></tr>
  <tr><td>$gt</td><td>大于</td></tr>
  <tr><td>$gte</td><td>大于等于</td></tr>
  <tr><td>$ne</td><td>不等于</td></tr>
  <tr><td>$in</td><td>包含</td></tr>
  <tr><td>$nin</td><td>不包含</td></tr>
  <tr><td>$exists</td><td>这个Key有值</td></tr>
  <tr><td>$select</td><td>匹配另一个查询的返回值</td></tr>
  <tr><td>$dontSelect</td><td>排除另一个查询的返回值</td></tr>
  <tr><td>$all</td><td>包括所有的给定的值</td></tr>
</table>

作为示例,为了获取在1000到3000之间的score,包含两个端点,我们应该这样请求:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"score":{"$gte":1000,"$lte":3000}}' \
  https://leancloud.cn/1.1/classes/GameScore
```

为了获得分数在10以下并且是一个奇数,我们需要这样做:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"score":{"$in":[1,3,5,7,9]}}' \
  https://leancloud.cn/1.1/classes/GameScore
```

为了获取一个不在列表中的player,我们可以:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"playerName":{"$nin":["Jonathan Walsh","Dario Wunsch","Shawn Simon"]}}' \
  https://leancloud.cn/1.1/classes/GameScore
```

为了获取有分数的对象,我们应该用:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"score":{"$exists":true}}' \
  https://leancloud.cn/1.1/classes/GameScore
```

为了获取没有分数的对象,用:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"score":{"$exists":false}}' \
  https://leancloud.cn/1.1/classes/GameScore
```

如果您有一个类包含运动队而您又储存了用户的家乡,您可以创建一个查询来寻找用户中的有故乡的运动队，并且赢得比赛的记录的人.查询看起来应该是这样:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"hometown":{"$select":{"query":{"className":"Team","where":{"winPct":{"$gt":0.5}}},"key":"city"}}}' \
  https://leancloud.cn/1.1/classes/_User
```

您可以用`order`参数来指定一个字段来排序.前面加一个负号的前缀表示逆序.这样返回的score会呈升序:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=score' \
  https://leancloud.cn/1.1/classes/GameScore
```

而这样会呈降序:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=-score' \
  https://leancloud.cn/1.1/classes/GameScore
```

您可以用多个字段进行排序,只要用一个逗号隔开的列表就可以.为了获取GameScore以score的升序和name的降序进行排序:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=score,-name' \
  https://leancloud.cn/1.1/classes/GameScore
```

您可以用`limit`和`skip`来做分页,limit的默认值是100,但是任何1到1000的值都是可选的，在 0 到 1000 范围之外的都强制转成默认的 100。
,就是说,为了获取在400到600之间的对象:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'limit=200' \
  --data-urlencode 'skip=400' \
  https://leancloud.cn/1.1/classes/GameScore
```

您可以限定返回的字段通过传入`keys`参数和一个逗号分隔列表.为了返回对象只包含score和playerName字段(还有特殊的内置字段比如objectId,createdAt和updatedAt):

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'keys=score,playerName' \
  https://leancloud.cn/1.1/classes/GameScore
```

所有以上这些参数都可以和其他的组合进行使用.

###对数组的查询

对于key的值是一个数组的情况,可以查找key的值中有2的对象:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"arrayKey":2}' \
  https://leancloud.cn/1.1/classes/RandomObject
```

您同样可以使用"$all"操作符来找到对象的key的值中有2,3和4的:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"arrayKey":{"$all":[2,3,4]}}' \
  https://leancloud.cn/1.1/classes/RandomObject
```

###关系查询

有几种方式来查询对象之间的关系数据,如果您想获取对象，而这个对象的一个字段对应了另一个对象,您可以用一个where查询,自己构造一个Pointer,和其他数据类型一样.举例说,如果每一个Comment有一个Post对象在它的post字段上,您可以对一个POST取得所有comment:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"post":{"__type":"Pointer","className":"Post","objectId":"51e3a359e4b015ead4d95ddc"}}' \
  https://leancloud.cn/1.1/classes/Comment
```

如果您想获取对象,这个对象的一个字段指向的对象是符合另一个查询的,您可以使用$inQuery操作符.注意默认的limit是100而且最大的limit是1000，这个限制同样适用于内部的查询,所以对于较大的数据集您可能需要细心地构建查询来获得期望的行为.举例说,假设您有一个Post类和一个Comment类,每个Comment都有一个指向它的Post的关系,您可以找到对于有图片的Post的Comment:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"post":{"$inQuery":{"where":{"image":{"$exists":true}},"className":"Post"}}}' \
  https://leancloud.cn/1.1/classes/Comment
```

如果您想获取作为其父对象的关系成员的对象,您可以使用$relatedTo操作符,假设您有一个Post类和一个User类,而每一个Post可以被不同的User所like.如果Post下面有一个key是Relation类型，并且叫做likes,存储了喜欢这个Post的User。您可以找到这些user,他们都like过同一个指定的post:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"$relatedTo":{"object":{"__type":"Pointer","className":"Post","objectId":"51e3a359e4b015ead4d95ddc"},"key":"likes"}}' \
  https://leancloud.cn/1.1/users
```

在某些情况之下,您可能需要在一个查询之中返回多种类型,您可以通过传入字段到include参数中.比如,我们想获得最近的10篇评论,而您想同时得到它们相关的post:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=-createdAt' \
  --data-urlencode 'limit=10' \
  --data-urlencode 'include=post' \
  https://leancloud.cn/1.1/classes/Comment
```

不是作为一个Pointer表示,post字段现在已经被展开为一个完整的对象. __type被设置为Object而className同样也被提供了.举例说,一个指向Post的Pointer可能被展示为:

```
{
  "__type": "Pointer",
  "className": "Post",
  "objectId": "51e3a359e4b015ead4d95ddc"
}
```

当一个查询使用include参数来包含进去来取代pointer之后,可以看到pointer被展开为:

```
{
  "__type": "Object",
  "className": "Post",
  "objectId": "51e3a359e4b015ead4d95ddc",
  "createdAt": "2011-12-06T20:59:34.428Z",
  "updatedAt": "2011-12-06T20:59:34.428Z",
  "otherFields": "willAlsoBeIncluded"
}
```

您可以同样做多层的include,这时要使用"."号.如果您要include一个comment对应的post对应的author:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=-createdAt' \
  --data-urlencode 'limit=10' \
  --data-urlencode 'include=post.author' \
  https://leancloud.cn/1.1/classes/Comment
```

如果您要构建一个查询,这个查询要include多个类,此时用逗号分隔列表即可.

###对象计数

如果您在使用limit,或者如果返回的结果很多,您可能想要知道到底有多少对象应该返回,而不用把它们全部获得以后再计数.此时您可以使用count参数.举个例子,如果您仅仅是关心一个特定的玩家玩过多少游戏:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"playerName":"Jonathan Walsh"}' \
  --data-urlencode 'count=1' \
  --data-urlencode 'limit=0' \
  https://leancloud.cn/1.1/classes/GameScore
```

因为这个request请求了count而且把limit设为了0,返回的值里面只有计数,没有results.

```
{
  "results": [

  ],
  "count": 1337
}
```

如果有一个非0的limit的话,既会返回results也会返回count.

###复合查询

如果您想查询对象符合几种查询之一,您可以使用$or操作符,带一个JSON数组作为它的值.例如,如果您想找到player赢了很多或者赢了很少,您可以用如下的方式:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"$or":[{"wins":{"$gt":150}},{"wins":{"$lt":5}}]}' \
  https://leancloud.cn/1.1/classes/Player
```

任何在查询上的其他的约束都会对返回的对象生效,所以您可以用$or对其他的查询添加约束.

注意我们不会在组合查询的子查询中支持非过滤型的约束(例如:limit skip sort include).

### 使用 CQL 查询

我们还提供类 SQL 语法的 CQL 查询语言，查询应用内数据，例如：

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'cql=select * from Player limit 0,100 order by name' \
  https://leancloud.cn/1.1/cloudQuery
```

更多请参考 [CQL 详细指南](./cql_guide.html)。

CQL 还支持占位符查询，where 和 limit 子句的条件参数可以使用问号替换，然后通过 `pvalues` 数组传入：

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'cql=select * from Player where name=? limit ?,? order by name' \
   --data-urlencode 'pvalues=["dennis", 0, 100]'
  https://leancloud.cn/1.1/cloudQuery
```

##用户

不仅在mobile app上,还在其他系统中,很多应用都有一个统一的登陆流程.通过REST API访问用户的账户让您可以通过AVOS Cloud使用这项功能.

通常来说,用户这个类的功能与其他的对象是相同的,比如都没有限制模式(Schema free).User对象和其他对象不同的是一个用户必须有用户名(username)和密码(password),密码会被自动地加密和存储.AVOS Cloud强制您username和email这两个字段必须是没有重复的.

###注册

注册一个新用户与创建一个新的普通对象之间的不同点在于username和password字段都是必要的.Password字段会以和其他的字段不一样的方式处理,它在储存时会被加密而且永远不会被返回给任何来自客户端的请求.

在您的app的设定菜单 ,您可以向AVOS Cloud来请求认证邮件地址.这项设置启用了的话,所有有email的用户的注册都会产生一个email验证地址.您可以在emailVerified字段上查看用户的email是否已经通过认证.

为了注册一个新的用户,需要向user路径发送一个POST请求,您可以加入一个新的字段,例如,创建一个新的用户有一个电话字段:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"username":"cooldude6","password":"p_n7!-e8","phone":"415-392-0202"}' \
  https://leancloud.cn/1.1/users
```

当创建成功时,HTTP返回为201 Created ,Location头包含了新用户的URL:

```
Status: 201 Created
Location: https://leancloud.cn/1.1/users/51fb1bf7e4b0cc0b5a3792f3
```

返回的主体是一个JSON对象,包含objectId, createdAt时间戳表示创建对象时间, sessionToken可以被用来认证这名用户随后的请求.

```
{
  "createdAt": "2011-11-07T20:58:34.448Z",
  "objectId": "51c3ba66e4b0f0e851c1621b",
  "sessionToken": "pnktnjyb996sj4p156gjtp4im"
}
```

###登陆

在您允许用户注册之后,在以后您需要让他们用自己的用户名和密码登陆.为了做到这一点,发送一个GET请求到/1.1/login,加上username和password作为URL编码后的参数.

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'username=cooldude6' \
  --data-urlencode 'password=p_n7!-e8' \
  https://leancloud.cn/1.1/login
```

返回的主体是一个JSON对象包括所有除了password以外的自定义字段.它同样包含了createdAt,updateAt,objectId和sessionToken字段.

```
{
  "username": "cooldude6",
  "phone": "415-392-0202",
  "createdAt": "2011-11-07T20:58:34.448Z",
  "updatedAt": "2011-11-07T20:58:34.448Z",
  "objectId": "51c3ba66e4b0f0e851c1621b",
  "sessionToken": "pnktnjyb996sj4p156gjtp4im"
}
```

###验证 Email

设置email验证是一个app选项的一个设置,这样可以对已经确认过的email的用户提供一部分保留的体验.Email验证会在User对象中加入emailVerified字段,当一个用户的email被新设置或者修改过的话,emailVerified会被设为false.AVOSCloud会对用户填写的邮箱发送一个链接,这个链接可以把emailVerified设置为true.

emailVerified字段有3种状态可以考虑

1. true : 用户可以点击email中的地址来连接AVOS Cloud来验证地址.一个用户永远不会在新创建这个值的时候emailVerified为true
2. false : User对象最后一次被刷新的时候,用户并没有确认过他的email地址,如果您看到emailVerified为false的话,您可以考虑刷新User对象
3. null : User对象在email验证没有打开的时候就已经创建了,或者User没有email

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

### 请求验证 Email

发送给用户的邮箱验证邮件在一周内失效，可以通过调用`/1.1/requestEmailVerify`来强制重新发送：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"email":"coolguy@iloveapps.com"}' \
  https://leancloud.cn/1.1/requestEmailVerify
```

###请求密码重设

您可以使用这项功能,前提是用户将email与他们的账户关联起来.如果要重设密码,发送一个POST请求到 `/1.1/requestPasswordReset` ,同时在request的body部分带上email字段.

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"email":"coolguy@iloveapps.com"}' \
  https://leancloud.cn/1.1/requestPasswordReset
```

如果成功的话,返回的值是一个JSON对象.

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

### 手机号码验证

在应用设置的应用选项里你还可以选择开启注册手机码号验证，当注册的时候用户填写`mobilePhoneNumber`字段， AVOS Cloud 将向该手机号码发送一条附带验证码的验证短信，用户在输入验证码后调用 AVOS Cloud 的 API 验证通过后，用户的`mobilePhoneNumberVerified`属性将设置为`true`。

**请注意，每个账户只有100条免费的短信额度，超过部分每一条短信都将实时扣费，请保证账户余额充足**

假设你在开启注册手机号码验证选项后，注册下列用户：


```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"username":"cooldude6","password":"p_n7!-e8","mobilePhoneNumber":"186xxxxxxxx"}' \
  https://leancloud.cn/1.1/users
```

那么在注册成功后，AVOS Cloud 将向`186xxxxxxxx`发送一条验证短信。开发者提供一个输入框让用户输入这个验证短信中附带的验证码，开发者调用下列 API 来确认验证码正确：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{}' \
  https://leancloud.cn/1.1/verifyMobilePhone/6位数字验证码
```

其中 URL 中的 `code` 就是6位验证数字。

验证成功后 ，用户的`mobilePhoneNumberVerified`将变为true，并调用云代码的`AV.Cloud.onVerified(type, function)`方法，type设置为`sms`。

### 请求手机号码验证

用户除了被动等待收到验证码短信之外，或者短信意外没有收到的情况下，开发者可以主动要求发送验证码短信：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://leancloud.cn/1.1/requestMobilePhoneVerify
```

### 手机号码短信登录

在验证号码后，用户可以采用短信验证码登录，来避免繁琐的输入密码的过程，请求发送登录验证码：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://leancloud.cn/1.1/requestLoginSmsCode
```

在用户收到短信验证码之后，可以输入该验证码加上手机号码来登录应用：

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'mobilePhoneNumber=186xxxxxxxx' \
  --data-urlencode 'smsCode=123456' \
  https://leancloud.cn/1.1/login
```

也可以采用手机号码和密码的方式登录：

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'mobilePhoneNumber=186xxxxxxxx' \
  --data-urlencode 'password=p_n7!-e8' \
  https://leancloud.cn/1.1/login
```

### 使用短信验证码重置用户密码

如果用户使用手机号码注册，您也许希望也能通过手机短信来实现`忘记密码`功能，通过：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://leancloud.cn/1.1/requestPasswordResetBySmsCode
```

发送一条重置密码的短信验证码到注册用户的手机上，需要传入注册时候的`mobilePhoneNumber`。

用户收到验证码后，调用`PUT /1.1/resetPasswordBySmsCode/:code`来设置新的密码：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"password": "new password"}' \
  https://leancloud.cn/1.1/resetPasswordBySmsCode/收到的6位验证码
```

修改成功后，就可以用新密码登陆了。

###获取用户

您可以发送一个GET请求到URL以获取用户的内容,返回的内容就是当创建用户时返回的内容.比如,为了获取上面创建的用户:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9
```

返回的body是一个JSON对象,包含所有用户提供的字段,除了密码以外.也包括了createdAt,updatedAt和objectId字段.

```
{
  "username": "cooldude6",
  "phone": "415-392-0202",
  "createdAt": "2011-11-07T20:58:34.448Z",
  "updatedAt": "2011-11-07T20:58:34.448Z",
  "objectId": "51fa6886e4b0cc0b5a3792e9"
}
```

###更新用户

在通常的情况下,没有人会指望一个用户被允许来改动他们自己的数据,为了让他们能认证做这件事,用户必须加入一个 `X-AVOSCloud-Session-Token` 头部来请求更新,这个session token在注册时和登录时会返回。

为了改动一个用户已经有的数据,需要对这个用户的URL发送一个PUT请求.任何您没有指定过的key会保持不动,所以您可以只改动用户数据中的一部分.username和password可以改动,但是新的username不能重复.

比如,如果我们想对 cooldude6 的电话做出一些改动:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "X-AVOSCloud-Session-Token: pnktnjyb996sj4p156gjtp4im" \
  -H "Content-Type: application/json" \
  -d '{"phone":"415-369-6201"}' \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9
```

返回的body是一个JSON对象,只有一个updatedAt字段表明更新发生的时间.

```
{
  "updatedAt": "2011-11-07T21:25:10.623Z"
}
```

###安全地修改用户密码

修改密码，可以直接使用上面的`PUT /1.1/users/:objectId`的API，但是很多开发者会希望让用户输入一次旧密码做一次认证，旧密码正确才可以修改为新密码，因此我们提供了一个单独的API `PUT /1.1/users/:objectId/updatePassword` 来安全地更新密码：

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "X-AVOSCloud-Session-Token: pnktnjyb996sj4p156gjtp4im" \
  -H "Content-Type: application/json" \
  -d '{"old_password":"the_old_pass", "new_password":"the_new_pass"}' \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/updatePassword
```

其中：

* old_password 就是用户的老密码。
* new_password 就是用户的新密码

注意，仍然需要传入`X-AVOSCloud-Session-Token`，也就是登陆用户才可以修改自己的密码。


###查询

您可以一次获取多个用户,只要向用户的根URL发送一个GET请求.没有任何URL参数的话,可以简单地列出所有用户:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/users
```

返回的值是一个JSON对象包括一个results字段, 值是包含了所有对象的一个JSON数组.

```
{
  "results": [
    {
      "username": "bigglesworth",
      "phone": "650-253-0000",
      "createdAt": "2011-11-07T20:58:06.445Z",
      "updatedAt": "2011-11-07T20:58:06.445Z",
      "objectId": "51fa0ff9e4b0cc0b5a3792e1"
    },
    {
      "username": "cooldude6",
      "phone": "415-369-6201",
      "createdAt": "2011-11-07T20:58:34.448Z",
      "updatedAt": "2011-11-07T21:25:10.623Z",
      "objectId": "51fa000be4b0cc0b5a3792e0"
    }
  ]
}
```

所有的对普通对象的查询选项都适用于对用户对象的查询,所以可以查看 查询 部分来获取详细信息.

###删除用户

为了在AVOS Cloud上删除一个用户,可以向它的URL上发送一个DELETE请求.您必须提供一个`X-AVOSCloud-Session-Token`在header上以便认证.例子:

```
curl -X DELETE \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "X-AVOSCloud-Session-Token: pnktnjyb996sj4p156gjtp4im" \
  https://leancloud.cn/1.1/users/g7y9tkhB7O
```

###用户账户连接

AVOSCloud允许你连接你的用户到其他服务，比如新浪微博和腾讯微博(未来我们还会加入更多的第三方服务)，这样就允许您的用户直接用他们现有的帐号id来登录您的App。通过siginup或者更新的endpoint，并使用`authData`字段来提供您希望连接的服务的授权信息就可以做到。一旦关联了某个服务，authData将被存储到您的用户信息里，并通过登录即可重新获取。

`authData`是一个普通的JSON对象，它所要求的key根据service不同而不同，具体要求见下面。每种情况下，你都需要自己负责完成整个授权过程(一般是通过OAuth协议，1.0或者2.0)来获取授权信息，提供给连接API。

[新浪微博](http://weibo.com/)的authData内容:

```
{
  "authData": {
    "weibo": {
      "uid": "123456789",
            "access_token": "2.00vs3XtCI5FevCff4981adb5jj1lXE",
            "expiration_in": "36000"
    }
  }
}
```

[腾讯微博](http://t.qq.com/)的authData内容:

```
{
  "authData": {
    "qq": {
      "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
      "access_token": "12345678-SaMpLeTuo3m2avZxh5cjJmIrAfx4ZYyamdofM7IjU",
      "expires_in": 1382686496
    }
  }
}
```

[微信](http://open.weixin.qq.com/)的authData内容:

```
{
  "authData": {
    "weixin": {
      "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
      "access_token": "12345678-SaMpLeTuo3m2avZxh5cjJmIrAfx4ZYyamdofM7IjU",
      "expires_in": 1382686496
    }
  }
}
```

匿名用户(Anonymous user)的authData内容:

```
{
  "anonymous": {
    "id": "random UUID with lowercase hexadecimal digits"
  }
}
```

#### 注册和登录

使用一个连接服务来注册用户并登录，同样使用POST请求users，只是需要提供authData字段。例如，使用新浪微博账户注册或者登录用户:


```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
     "authData": {
       "qq": {
         "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
         "access_token": "12345678-SaMpLeTuo3m2avZxh5cjJmIrAfx4ZYyamdofM7IjU",
         "expires_in": 1382686496
         }
    }
    }' \
  https://leancloud.cn/1.1/users
```

AVOSCloud会校验提供的authData是否有效，并检查是否已经有一个用户连接了这个authData服务。如果已经有用户存在并连接了同一个authData，那么返回200 OK和详细信息(包括用户的sessionToken):

```
Status: 200 OK
Location: https://leancloud.cn/1.1/users/51fb1bf7e4b0cc0b5a3792f3
```

应答的body类似:

```
{
  "username": "AVOSCloud",
  "createdAt": "2012-02-28T23:49:36.353Z",
  "updatedAt": "2012-02-28T23:49:36.353Z",
  "objectId": "51fb1bf7e4b0cc0b5a3792f3",
  "sessionToken": "samplei3l83eerhnln0ecxgy5",
  "authData": {
    "qq": {
      "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
      "access_token": "12345678-SaMpLeTuo3m2avZxh5cjJmIrAfx4ZYyamdofM7IjU",
      "expires_in": 1382686496
    }
  }
}
```

如果用户还没有连接到这个帐号，则你会收到201 Created的应答状态码，标识新的用户已经被创建:

```
Status: 201 Created
Location: https://leancloud.cn/1.1/users/51fb1bf7e4b0cc0b5a3792f3
```

应答内容包括objectId,createdAt,sessionToken以及一个自动生成的随机username，例如:

```
{
  "username": "iwz8sna7sug28v4eyu7t89fij",
  "createdAt": "2012-02-28T23:49:36.353Z",
  "objectId": "51fb1bf7e4b0cc0b5a3792f3",
  "sessionToken": "samplei3l83eerhnln0ecxgy5"
}
```

#### 连接

连接一个现有的用户到新浪微博或者腾讯微博帐号，可以通过发送一个PUT请求附带authData字段到user endpoint做到。例如，连接一个用户到新浪微博帐号发起的请求类似这样:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "X-AVOSCloud-Session-Token: samplei3l83eerhnln0ecxgy5" \
  -H "Content-Type: application/json" \
  -d '{
        "authData": {
          "weibo": {
            "uid": "123456789",
            "access_token": "2.00vs3XtCI5FevCff4981adb5jj1lXE",
            "expiration_in": "36000"
          }
        }
      }' \
  https://leancloud.cn/1.1/users/51fb1bf7e4b0cc0b5a3792f3
```

完成连接后，你可以使用匹配的authData来认证他们。

#### 断开连接

断开一个现有用户到某个服务，可以发送一个PUT请求设置authData中对应的服务为null来做到。例如，取消新浪微博关联:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "authData": {
      "weibo" : null
    }
      }' \
  https://leancloud.cn/1.1/users/51fb1bf7e4b0cc0b5a3792f3
```


###安全

当您用REST API key 来访问AVOS Cloud时,访问可能被ACL所限制,就像iOS和Android SDK上所做的一样.您仍然可以通过REST API来读和修改,只需要通过`ACL`的key来访问一个对象.

ACL按JSON对象格式来表示,JSON对象的key是objectId 或者一个特别的key——`*`(表示公共访问权限).ACL的值是"权限对象",这个JSON对象的key总是权限名,而这些key的值总是true.

举个例子,如果您想让一个id为"51f9d9c7e4b0cc0b5a3792da"的用户有读和写一个对象的权限,而且这个对象应该可以被公共读取,符合的ACL应该是:

```
{
  "51f9d9c7e4b0cc0b5a3792da": {
    "read": true,
    "write": true
  },
  "*": {
    "read": true
  }
}
```

##角色

当您的app的规模和用户基数成长时,您可能发现您需要比ACL模型(针对每个用户)更加粗粒度的访问控制您的数据的方法.为了适应这种需求,AVOSCloud支持一种基于角色的权限控制方式.角色系统提供一种逻辑方法让您通过权限的方式来访问您的AVOSCloud数据.角色是一种有名称的对象,包含了用户和其他角色.任何授予一个角色的权限隐含着授予它包含着的其他的角色相应的权限.

例如,在您的app中管理着一些内容,您可能有一些类似于"主持人"的角色可以修改和删除其他用户创建的新的内容,您可能还有一些"管理员"有着与"主持人"相同的权限,但是还可以修改app的其他全局性设置.通过给予用户这些角色,您可以保证新的用户可以做主持人或者管理员,不需要手动地授予每个资源的权限给各个用户.

我们提供一个特殊的角色（Role）类来表示这些用户组,为了设置权限用.角色有一些和其他对象不太一样的特殊字段.

* name : 角色的名字,这个值是必须的,而且只允许被设置一次,只要这个角色被创建了的话.角色的名字必须由字母,空格,减号或者下划线这些字符构成.这个名字可以用来标明角色而不需要它的objectId.
* users : 一个指向一系列用户的关系,这些用户会继承角色的权限.
* roles : 一个指向一系列子角色的关系,这些子关系会继承父角色所有的权限.

通常来说,为了保持这些角色安全,您的移动app不应该为角色的创建和管理负责.作为替代,角色应该是通过一个不同的网页上的界面来管理,或者手工被管理员所管理.我们的REST API允许您不需要一个移动设备就能管理您的角色.

###创建角色

创建一个新的角色与其他的对象不同的是name字段是必须的.角色必须指定一个ACL,这个ACL必须尽量的约束严格一些,这样可以防止错误的用户修改角色.

创建一个新角色,发送一个POST请求到roles根路径:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "name": "Moderators",
        "ACL": {
          "*": {
            "read": true
          }
        }
      }' \
  https://leancloud.cn/1.1/roles
```

您可以通过加入已有的对象到roles和users关系中来创建一个有子角色和用户的角色:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "name": "Moderators",
        "ACL": {
          "*": {
            "read": true
          }
        },
        "roles": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_Role",
              "objectId": "51fe4415e4b05df1766d0841"
            }
          ]
        },
        "users": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_User",
              "objectId": "51fc9241e4b074ac5c34cf0a"
            }
          ]
        }
      }' \
  https://leancloud.cn/1.1/roles
```

当创建成功时,HTTP返回是 201 Created而Location header包含了新的对象的URL地址:

```
Status: 201 Created
Location: https://leancloud.cn/1.1/roles/51e3812ee4b0b3eb44adbd44
```

###获取角色

您可以同样通过发送一个GET请求到Location header中返回的URL来获取这个对象,比如我们想要获取上面创建的对象:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/roles/51e3812ee4b0b3eb44adbd44
```

响应的body是一个JSON对象包含角色的所有字段:

```
{
  "createdAt": "2012-04-28T17:41:09.106Z",
  "objectId": "51e3812ee4b0b3eb44adbd44",
  "updatedAt": "2012-04-28T17:41:09.106Z",
  "ACL": {
    "*": {
      "read": true
    },
    "role:Administrators": {
      "write": true
    }
  },
  "name": "Moderators"
}
```

注意users和roles关系无法在JSON中见到,您需要相应地用$relatedTo操作符来查询角色中的子角色和用户.

###更新角色

更新一个角色通常可以像更新其他对象一样使用,但是name字段是不可以更改的.加入和删除users和roles可以通过使用`AddRelation` 和 `RemoveRelation`操作来进行.

举例来说,我们对"Moderators"角色加入2个用户:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "users": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_User",
              "objectId": "51dfb84ce4b0a918eba635d9"
            },
            {
              "__type": "Pointer",
              "className": "_User",
              "objectId": "51dfb8bde4b0a918eba635da"
            }
          ]
        }
      }' \
  https://leancloud.cn/1.1/roles/51e3812ee4b0b3eb44adbd44
```

相似的,我们可以删除一个"Moderrators"的子角色:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "roles": {
          "__op": "RemoveRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_Role",
              "objectId": "51dfb84ce4b0a918eba635d9"
            }
          ]
        }
      }' \
  https://leancloud.cn/1.1/roles/51e3812ee4b0b3eb44adbd44
```


###删除对象

为了从AVOS Cloud上删除一个角色,只需要发送DELETE请求到它的URL就可以了.

我们需要传入 X-AVOSCloud-Session-Token 来通过一个有权限的用户账号来访问这个角色对象.例如:

```
curl -X DELETE \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "X-AVOSCloud-Session-Token: pnktnjyb996sj4p156gjtp4im" \
  https://leancloud.cn/1.1/roles/51e3812ee4b0b3eb44adbd44
```

###安全性

当您通过REST API key访问AVOSCloud的时候,访问同样可能被ACL所限制,就像iOS和Android SDK上一样.您仍然可以通过REST API来读和修改ACL,只用通过访问"ACL"键就可以了.

除了用户级的权限设置以外,您可以通过设置角色级的权限来限制对AVOSCloud对象的访问.取代了指定一个objectId带一个权限的方式,您可以设定一个角色的权限为它的名字在前面加上`role:`前缀作为key.您可以同时使用用户级的权限和角色级的权限来提供精细的用户访问控制.

比如,为了限制一个对象可以被在"Members"里的任何人读到,而且可以被它的创建者和任何有"Moderators"角色的人所修改,您应该向下面这样设置ACL:

```
{
  "51ff181ae4b05df1766d0b42": {
    "write": true
  },
  "role:Members": {
    "read": true
  },
  "role:Moderators": {
    "write": true
  }
}
```

您不必为创建的用户和"Moderators"指定读的权限，如果这个用户和"Moderators"本身就是"Members"的子角色和用户.因为它们都会继承授予"Members"的权限.

###角色继承

就像上面所说的一样,一个角色可以包含另一个,可以为2个角色建立一个父-子关系.这个关系的结果就是任何被授予父角色的权限隐含地被授予子角色.

这样的关系类型通常在用户管理的内容类的app上比较常见,比如论坛.有一些少数的用户是"管理员",有最高级的权限来调整程序的设置,创建新的论坛,设定全局的消息等等.另一类用户是"版主",他们有责任保证用户生成的内容是合适的.任何有管理员权限的人都应该有版主的权利.为了建立这个关系,您应该把"Administartors"的角色设置为"Moderators"的子角色,具体来说就是把"Administrators"这个角色加入"Moderators"对象的roles关系之中:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "roles": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_Role",
              "objectId": "<AdministratorsRoleObjectId>"
            }
          ]
        }
      }' \
  https://leancloud.cn/1.1/roles/<ModeratorsRoleObjectId>
```

##文件

文件上传，我们推荐使用各个客户端的SDK进行上传，或者使用[命令行工具](./cloud_code_commandline.html)。

**通过 REST API 上传文件受到三个限制，而使用 sdk 或者命令行上传没有这些限制**：

* 上传最大文件大小有 10M 的限制
* 每个应用每秒最多上传 1 个文件
* 每个应用每分钟最多上传 30 个文件。


### 上传文件

上传文件到 AVOS Cloud 通过 POST 请求，注意必须指定文件的 `content-type`，例如上传一个文本文件 `hello.txt` 包含一行字符串:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: text/plain" \
  -d 'Hello, World!' \
  https://leancloud.cn/1.1/files/hello.txt
```

文件上传成功后，返回 `201 Created` 的应答和创建的文件对象（可以在 `_File` 表看到）：

```
{  "size":13,
   "bucket":"1qdney6b",
   "url":"http://ac-1qdney6b.qiniudn.com/3zLG4o0d27MsCQ0qHGRg4JUKbaXU2fiE35HdhC8j.txt",
   "name":"hello.txt",
   "createdAt":"2014-10-14T05:55:57.455Z",
   "objectId":"543cbaede4b07db196f50f3c"
}
```

其中 `url` 就是文件下载链接, `objectId` 是文件的对象 id。`name`就是上传的文件名称。

也可以尝试上传一张图片，假设当前目录有一个文件 `test.png`：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: image/png" \
  --data-binary '@test.png'  \
  https://leancloud.cn/1.1/files/test.png
```

### 关联文件到对象

一个文件上传后，您可以关联该文件到某个 AVObject 对象上：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
   -H "Content-Type: application/json" \
  -d '{
        "name": "Andrew",
        "picture": {
          "id": "543cbaede4b07db196f50f3c",
          "__type": "File"
        }
      }' \
  https://leancloud.cn/1.1/classes/Player
```

其中 `id` 就是文件对象的 objectId。


### 删除文件

知道文件对象 ObjectId 的情况下，可以通过 DELETE 删除文件：

```
curl -X DELETE \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/files/543cbaede4b07db196f50f3c
```

##Push 通知

请查看我们的[Push Notification文档](./push_guide.html#%E4%BD%BF%E7%94%A8rest-api%E6%8E%A8%E9%80%81%E6%B6%88%E6%81%AF)

##安装数据

###上传安装数据

一个安装对象表示了一个您的在手机上被安装的app,这些对象被用来保存订阅数据的,这些数据是一个或多个通知通道订阅的.安装数据除了一些特殊字段以外都可以是模式可变的.这些字段都有特殊的类型和验证需求.

* deviceType 是一个必须的字段,必须被设置为"ios"或者"android",而且自这个对象生成以后就不能变化.
* installationId 是一个AVOSCloud生成的字符串标志,而且如果deviceType是android的话是一个必填字段,如果是ios的话则可选.它只要对象被生成了就不能发生改变,而且对一个app来说是不可重复的.
* deviceToken 是一个Apple生成的字符串标志,在deviceType为ios上的设备是必须的,而且自对象生成开始就不能改动,对于一个app来说也是不可重复的.
* badge 是一个数字字段,表示最新的iOS的安装已知的application badge
* timeZone 是一个字符串字段表示安装的这个设备的系统时区.
* channels 是一个可选的数组,表示这个安装对象的订阅频道列表.

大部分时间,安装数据是被客户端中有关push的方法所修改的.举个例子,从客户端SDK中调用subsccribeToChannel或者unsubscribeFromChannel，如果现在还没有安装对象的或者没有更新安装对象的话会创建一个对象,而从客户端SDK中调用getSubscribedChanneles会从安装对象中读取订阅数据.REST的方法可以被用来模仿这些操作.比如,如果您有一个iOS的device token您可以注册它来向设备推送通知,只需要创建一个有需要的channels的安装对象就可以了.您同样可以做一些不能通过客户端SDK进行的操作,就比如说查询所有的安装来找到一个channel的订阅者的集合.

创建一个安装对象和普通的对象差不多,但是特殊的几个安装字段必须通过认证.举个例子,如果您有一个由Apple Push Notification提供的device token,而且想订阅一个广播频道,您可以如下发送请求:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "deviceType": "ios",
        "deviceToken": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
        "channels": [
          ""
        ]
      }' \
  https://leancloud.cn/1.1/installations
```

当创建成功后,HTTP的返回值为201 Created , Location header包括了新的安装的URL

```
Status: 201 Created
Location: https://leancloud.cn/1.1/installations/51ff1808e4b074ac5c34d7fd
```

返回的body是一个JSON对象,包括了objectId和createdAt这个创建对象的时间戳.

```
{
  "createdAt": "2012-04-28T17:41:09.106Z",
  "objectId": "51ff1808e4b074ac5c34d7fd"
}
```

###获取安装对象

您可以通过GET方法请求创建的时候Location表示的URL来获取Installation对象.比如,获取上面的被创建的对象:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/installations/51ff1808e4b074ac5c34d7fd
```

返回的JSON对象所有用户提供的字段,加上createdAt,updatedAt和objectId字段:

```
{
  "deviceType": "ios",
  "deviceToken": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
  "channels": [
    ""
  ],
  "createdAt": "2012-04-28T17:41:09.106Z",
  "updatedAt": "2012-04-28T17:41:09.106Z",
  "objectId": "51ff1808e4b074ac5c34d7fd"
}
```

###更新安装对象

安装对象可以向相应的URL发送PUT请求来更新.举个例子,为了让设备订阅一个"foo"的Push channel:

```
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "deviceType": "ios",
        "deviceToken": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
        "channels": [
          "",
          "foo"
        ]
      }' \
  https://leancloud.cn/1.1/installations/51ff1808e4b074ac5c34d7fd
```

###查询安装对象

您可以一次通过GET请求到installations的根URL来获取多个安装对象.这项功能在SDK中不可用。

没有任何URL参数的话,一个GET请求会列出所有安装:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/installations
```

返回的JSON对象的results字段包含了所有的结果:

```
{
  "results": [
    {
      "deviceType": "ios",
      "deviceToken": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
      "channels": [
        ""
      ],
      "createdAt": "2012-04-28T17:41:09.106Z",
      "updatedAt": "2012-04-28T17:41:09.106Z",
      "objectId": "51ff1808e4b074ac5c34d7fd"
    },
    {
      "deviceType": "ios",
      "deviceToken": "fedcba9876543210fedcba9876543210fedcba9876543210fedcba9876543210",
      "channels": [
        ""
      ],
      "createdAt": "2012-04-30T01:52:57.975Z",
      "updatedAt": "2012-04-30T01:52:57.975Z",
      "objectId": "51fcb74ee4b074ac5c34cf85"
    }
  ]
}
```

所有对普通的对象的查询都对installatin对象起作用,所以可以查看之前的查询部分以获取详细信息.通过做channels的数组查询,您可以查找一个订阅了给定的push channel的所有设备.

###删除安装对象

为了从AVOSCloud中删除一个安装对象,可以发送DELETE请求到相应的URL.这个功能在客户端SDK也不可用.举例:

```
curl -X DELETE \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/installations/51fcb74ee4b074ac5c34cf85
```

##Cloud 函数

Cloud 函数可以通过REST API来使用,比如,调用一个叫hello的Cloud函数:

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{}' \
  https://leancloud.cn/1.1/functions/hello
```

您可以查看Cloud Code Guide来查看更多的信息.

##地理查询

现在您有一系列的对象对应的地理坐标,如果能发现那些对象离指定的点近就好了.这可以通过GeoPoint数据类型加上在查询中使用`$nearSphere`做到.获取离用户最近的10个地点应该看起来像下面这个样子:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'limit=10' \
  --data-urlencode 'where={
        "location": {
          "$nearSphere": {
            "__type": "GeoPoint",
            "latitude": 30.0,
            "longitude": -20.0
          }
        }
      }' \
  https://leancloud.cn/1.1/classes/PlaceObject
```

这会按离纬度30.0,经度-20.0的距离排序返回一系列的结果.第一个就是最近的对象.(注意如果一个特定的order参数给了的话,它会覆盖按距离排序).例如,下面是两个上面的查询返回的结果:

```
{
  "results": [
    {
      "location": {
        "latitude": 40.0,
        "__type": "GeoPoint",
        "longitude": -30.0
      },
      "updatedAt": "2011-12-06T22:36:04.983Z",
      "createdAt": "2011-12-06T22:36:04.983Z",
      "objectId": "51e3a334e4b0b3eb44adbe1a"
    },
    {
      "location": {
        "latitude": 60.0,
        "__type": "GeoPoint",
        "longitude": -20.0
      },
      "updatedAt": "2011-12-06T22:36:26.143Z",
      "createdAt": "2011-12-06T22:36:26.143Z",
      "objectId": "51e3a2a8e4b015ead4d95dd9"
    }
  ]
}
```

为了限定搜素的最大举例,需要加入`$maxDistanceInMiles`和`$maxDistanceInKilometers`或者`$maxDistanceInRadians`参数来限定.比如要找的半径在10英里内的话:

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={
        "location": {
          "$nearSphere": {
            "__type": "GeoPoint",
            "latitude": 30.0,
            "longitude": -20.0
          },
          "$maxDistanceInMiles": 10.0
        }
      }' \
  https://leancloud.cn/1.1/classes/PlaceObject
```

同样做查询寻找在一个特定的范围里面的对象也是可以的,为了找到在一个矩形的区域里的对象,按下面的格式加入一个约束 {"$within": {"$box": {[southwestGeoPoint, northeastGeoPoint]}}}.

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={
        "location": {
          "$within": {
            "$box": [
              {
                "__type": "GeoPoint",
                "latitude": 37.71,
                "longitude": -122.53
              },
              {
                "__type": "GeoPoint",
                "latitude": 30.82,
                "longitude": -122.37
              }
            ]
          }
        }
      }' \
  https://leancloud.cn/1.1/classes/PizzaPlaceObject
```

###警告

这是有一些问题是值得留心的:

1. 每一个AVObject类只能有一个键指向一个AVGeoPoint对象
2. Points不应该等于或者超出它的界. 纬度不应该是-90.0或者90.0,经度不应该是-180.0或者180.0. 试图在GeoPoint上使用超出范围内的经度和纬度会导致问题.

## 用户反馈组件 API

如果使用我们的用户反馈组件，可以通过下列 API 来提交一条新的用户反馈：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
         "app_sign": "app id和 app key 连接的字符串做md5签名",
         "status"  : "open",
         "content" : "反馈的文字内容",
         "contact" : "联系方式，QQ或者邮箱手机等"
       }' \
  https://leancloud.cn/1.1/feedback
```

提交后的用户反馈在可以在组件菜单的用户反馈里看到。

## 短信验证 API

在一些场景下，你可能希望用户验证手机号码后才能进行一些操作，例如充值等。这些操作跟账户系统没有关系，可以通过我们提供的的短信验证 API 来实现。

短信 API 每个 AVOS Cloud帐户有 100 个免费额度，超过就即时收费。使用这些 API 需要开启`启用手机号码短信认证 （针对 /1.1/verifySmsCode/:code 接口）` 选项。

给某个手机号码发送验证短信通过：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://leancloud.cn/1.1/requestSmsCode
```

验证收到的 6 位数字验证码是否正确通过：

```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  https://leancloud.cn/1.1/verifySmsCode/6位数字验证码
```

其中 `code` 是手机收到的 6 位数字验证码。

如果您创建了短信模板，可以指定 `template` 参数指定模板名称来使用您的模板，并且可以传入变量渲染模板：


```
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx", "template":"activity","date":"2014 年 10 月 31 号"}' \
  https://leancloud.cn/1.1/requestSmsCode
```

##实时通信 API

###获取聊天记录

获取某个应用的聊天记录

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/rtm/messages/logs
```

<table>
  <tr>
    <td>参数名</td>
    <td>含义</td>
  </tr>
  <tr>
    <td>convid</td>
    <td>对话 id ，可选，默认表示全部对话</td>
  </tr>
  <tr>
    <td>from</td>
    <td>发送者 id ，可选，只查询某个用户发送的消息</td>
  </tr>
  <tr>
    <td>timestamp</td>
    <td>查询起始的时间戳，返回比这个时间早的记录，可选，默认是当前时间</td>
  </tr>
  <tr>
    <td>limit</td>
    <td>返回条数限制，可选，默认100条，最大1000条</td>
  </tr>
</table>

注意：不能同时指定 convid 和 from 作为查询参数。

返回数据格式，json 数组，按消息记录从新到旧排序。

```json
[
  {
    timestamp: 1408008498571,
    conv-id: "219946ef32e40c515d33ae6975a5c593",
    data: "今天天气不错！",
    app-id: "ndxxv7lefvpj7z9jk4hh6o686790i8mxgxmf27da7vr6767s"
    from: "u111872755_9d0461adf9c267ae263b3742c60fa"
  },
  ...
]
```

#### 构建对话 ID

实时通信中的 `convid` 构建规则：

* 对点对点通信，convid 为所有对话参与者的 peer id **排序**后以`:`分隔，做 md5 所得。如对话参与者 peer id 为 `u1234` 和 `u0988`，那么对话 ID 为 `bcd26a54e98687390b0abb4d83683d4b`。
* 对群组功能，convid 即群组 ID。

### 取未读消息数

您可以从服务器端通过 REST API 调用获取实时通信中，某个 peer 的未读消息
数。注意这个消息数仅包含单聊的未读数，群组不算在内。

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  https://leancloud.cn/1.1/rtm/messages/unread/PEER_ID
```

返回：

```json
{"count": 4}
```

##统计数据API

统计API可以获取一个应用的统计数据。因为统计数据的隐私敏感性，统计API必须使用master key的签名方式鉴权，请参考 更安全的鉴权方式 一节。

获取某个应用的基本信息，包括各平台的应用版本，应用发布渠道。（注意：下面示例直接使用`X-AVOSCloud-Master-Key`，不过我们推荐您在实际使用中采用[新鉴权方式](https://leancloud.cn/docs/rest_api.html#%E6%9B%B4%E5%AE%89%E5%85%A8%E7%9A%84%E9%89%B4%E6%9D%83%E6%96%B9%E5%BC%8F)加密，不要明文传递Key。）

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Master-Key: {{masterkey}}" \
  https://leancloud.cn/1.1/stats/appinfo
```

返回的json数据

```
{
  "iOS": {
      "versions": ["2.3.10","2.3","2.4","2.5","2.6","2.7","2.8","2.6.1"],
      "channels": ["App Store"]
  },
  "Android": {
      "versions": ["1.7.2.1","1.4.0","1.5.0","1.6.0","1.5.1","1.7.0","1.6.1","1.8.0","1.7.1","1.8.1","1.7.2","1.8.2"],
      "channels": []
  }
}
```

获取某个应用的具体统计数据

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Master-Key: {{masterkey}}" \
  "https://leancloud.cn/1.1/stats/appmetrics?platform=iOS&start=20140301&end=20140315&metrics=active_user"
```

具体支持的参数：

<table>
  <tr><th>参数名</th><th>含义</th></tr>
  <tr><td>start</td><td>开始日期(yyyyMMdd)</td></tr>
  <tr><td>end</td><td>结束日期(yyyyMMdd)</td></tr>
  <tr><td>metrics</td><td>统计数据项</td></tr>
  <tr><td>platform</td><td>应用平台：iOS, Android，可选，默认是全部</td></tr>
  <tr><td>appversion</td><td>选择应用版本，可选，默认是全部</td></tr>
  <tr><td>channel</td><td>选择发布渠道，可选，默认是全部</td></tr>
</table>

metrics参数可选项解释：

<table>
  <tr><th>参数值</th><th>含义</th></tr>
  <tr><td>accumulate_user</td><td>累计用户数</td></tr>
  <tr><td>new_user</td><td>新增用户数</td></tr>
  <tr><td>active_user</td><td>活跃用户数</td></tr>
  <tr><td>session</td><td>启动次数</td></tr>
  <tr><td>wau</td><td>周活跃用户数</td></tr>
  <tr><td>mau</td><td>月活跃用户数</td></tr>
  <tr><td>avg_user_time</td><td>日平均用户使用时长</td></tr>
  <tr><td>avg_session_time</td><td>日次均使用时长</td></tr>
  <tr><td>avg_page_count</td><td>日均访问页面数</td></tr>
  <tr><td>retention_n</td><td>n天后的存留用户数 (n可取值：1-7,14,30 如 retention_1)</td></tr>
  <tr><td>push_login</td><td>推送用户数</td></tr>
  <tr><td>push_ack</td><td>推送到达数</td></tr>
  <tr><td>push_session</td><td>聊天用户数</td></tr>
  <tr><td>push_direct</td><td>聊天消息数</td></tr>
</table>

返回的json数据

```
{
  "data": {
    "2014-03-01": 270,
    "2014-03-02": 275,
    "2014-03-03": 238,
    "2014-03-04": 259,
    "2014-03-05": 246,
    "2014-03-06": 306,
    "2014-03-07": 362,
    "2014-03-08": 347,
    "2014-03-09": 381,
    "2014-03-10": 255,
    "2014-03-11": 233,
    "2014-03-12": 232,
    "2014-03-13": 227,
    "2014-03-14": 215,
    "2014-03-15": 222
  },
  "metrics": "new_user"
}
```
获取某个应用的实时统计数据

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Master-Key: {{masterkey}}" \
  "https://leancloud.cn/1.1/stats/rtmetrics?platform=iOS&metrics=current_active"
```

具体支持的参数：

<table>
  <tr><th>参数名</th><th>含义</th></tr>
  <tr><td>metrics</td><td>统计数据项</td></tr>
  <tr><td>platform</td><td>应用平台：iOS, Android，可选，默认是全部</td></tr>
</table>

metrics参数可选项解释：

<table>
  <tr><th>参数值</th><th>含义</th></tr>
  <tr><td>current_active</td><td>活跃用户数</td></tr>
  <tr><td>30min_active</td><td>近30分钟的活跃用户数</td></tr>
  <tr><td>pages</td><td>用户停留页面</td></tr>
  <tr><td>events</td><td>用户触发事件</td></tr>
  <tr><td>locations</td><td>用户所在地</td></tr>
</table>

返回数据

```
{data:97, metrics:"current_active"}

{data:[1,3,5,..], metrics:"30min_active"}

{data:[{name:"pageA", count:3}, {name:"pageB",count:2}, ...], metrics:"pages"}

{data:[{name:"eventA", count:3}, {name:"eventB",count:2}, ...], metrics:"events"}

{data:[{location:"上海", count:3}, {location:"江苏", count:2}, ...], metrics:"locations"}

```


批量获取：

当需要批量获取统计数据时，可以将多个 metrics 值用 , 拼接传入，返回结果将是一个数组，结果值和参数值次序对应，例如：

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Master-Key: {{masterkey}}" \
  "https://leancloud.cn/1.1/stats/appmetrics?platform=iOS&start=20140301&end=20140315&metrics=new_user,retention_1"
```
将返回

```
[
  {"data":
    {"2014-03-01":371,
     "2014-03-02":493,
     "2014-03-03":400,
     "2014-03-04":407,
     "2014-03-05":383,
     "2014-03-06":377,
     "2014-03-07":416,
     "2014-03-08":425,
     "2014-03-09":434,
     "2014-03-10":364,
     "2014-03-11":434,
     "2014-03-12":416,
     "2014-03-13":400,
     "2014-03-14":394},
    "metrics":"active_user",
  {"data":
    {"2014-03-01":10,
     "2014-03-02":5,
     "2014-03-03":10,
     "2014-03-04":4,
     "2014-03-05":6,
     "2014-03-06":7,
     "2014-03-07":6,
     "2014-03-08":6,
     "2014-03-09":4,
     "2014-03-10":6,
     "2014-03-11":6,
     "2014-03-12":7,
     "2014-03-13":3,
     "2014-03-14":7},
     "metrics":"retention_1"}]
```

获取统计在线参数，可以获取发送策略，是否开启的设置情况，和自定义的在线配置参数

```
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Master-Key: {{appkey}}" \
  https://leancloud.cn/1.1/statistics/apps/{{appid}}/sendPolicy
```

返回结果：

```
{
  "policy":6, // 发送策略
  "enable":true, // 是否开启
  "parameters":{ // 自定义在线参数
    "showBeauty":"1"
  }
}
```

## 事件流 API

参考 [事件流 REST API](./status_system.html#rest-api)。

## 应用内搜索 API

参考 [搜索 API](./app_search_guide.html#搜索-api)。