# 错误码详解

本文档尝试为所有服务端和SDK返回的错误码给出相对详细的解释，具体到各个SDK的错误码，请参考下列文档链接：

* iOS的[AVConstants](https://leancloud.cn/docs/api/iOS/docs/AVConstants.html)。
* Android的[AVException](https://leancloud.cn/docs/api/android/doc/index.html)。

## 1
* 信息 - `Internal server error. No information available.`
* 含义 - 服务器内部错误或者参数错误，一般是因为传入了错误的参数，或者没有在本文档里明确定义的运行时错误，都会以代码1指代。

## 100

* 信息 - `The connection to the AVOS servers failed.`
* 含义 - 无法建立TCP连接到LeanCloud服务器，通常是因为网络故障，或者我们服务器故障引起的，我们的服务器状态可以查看[健康状态检查](http://status.leancloud.cn/)。

## 101

* 信息 - `Object doesn't exist, or has an incorrect password.`
* 含义 - 查询的Class不存在，或者要关联的Pointer对象不存在。

## 103

* 信息 - `Missing or invalid classname. Classnames are case-sensitive. They must start with a letter, and a-zA-Z0-9_ are the only valid characters.`
* 含义 - 非法的class名称，class名称是大小写敏感的，并且必须以英文字母开头，有效的字符仅限在英文字母、数字以及下划线。

## 104

* 信息 - `Missing object id.`
* 含义 - 缺少ObjectId，通常是在查询的时候没有传入objectId，或者objectId非法。objectId只能为字母、数字组成的字符串。

## 105

* 信息 - `Invalid key name. Keys are case-sensitive. They must start with a letter, and a-zA-Z0-9_ are the only valid characters.`
* 含义 - 无效的key名称，也就是class的列名无效，列名必须以英文字母开头，有效的字符仅限在英文字母、数字以及下划线。

## 106

* 信息 - `Malformed pointer. Pointers must be arrays of a classname and an object id.`
* 含义 - 无效的Pointer格式，Pointer必须为形如`{className: 'GameScore', objectId:'xxxxxx'}`的JSON对象。

## 107

* 信息 - `Malformed json object. A json dictionary is expected.`
* 含义 - 无效的JSON对象，解析JSON数据失败。

## 108

* 信息 - `Tried to access a feature only available internally.`
* 含义 - 此API仅供内部使用。

## 111


* 信息 - `Field set to incorrect type.`
* 含义 - 想要存储的值不匹配列的类型，请检查你的数据管理平台中列定义的类型，查看存储的数据是否匹配这些类型。

## 112

* 信息 - `Invalid channel name. A channel name is either an empty string (the broadcast channel) or contains only a-zA-Z0-9_ characters and starts with a letter.`
* 含义 - 推送订阅的频道无效，频道名称必须不是空字符串，只能包含英文字母、数字以及下划线，并且只能以英文字母开头。

## 114

* 信息 - `Invalid device token.`
* 含义 - iOS推送存储的deviceToken无效，如何存储installation请阅读[消息推送开发指南](https://leancloud.cn/docs/push_guide.html#ios%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81)。

## 116

* 信息 - `The object is too large.`
* 含义 - 要存储的对象超过了大小限制，我们限制单个对象的最大大小在16m。

## 119

* 信息 - `That operation isn't allowed for clients.`
* 含义 - 该操作无法从客户端发起。

## 120

* 信息 - `The results were not found in the cache.`
* 含义 - 查询结果无法从缓存中找到，SDK在使用从查询缓存的时候，如果发生缓存没有命中，返回此错误。

## 121

* 信息 - `Keys in NSDictionary values may not include '$' or '.'.`
* 含义 - JSON对象中key的名称不能包含`$`和`.`符号。

## 122

* 信息 - `Invalid file name. A file name contains only a-zA-Z0-9_. characters and is between 1 and 36 characters.`
* 含义 - 无效的文件名称，文件名称只能是英文字母、数字和下划线组成，并且名字长度限制在1到36之间。

## 123

* 信息 - `Invalid ACL. An ACL with an invalid format was saved. This should not happen if you use AVACL.`
* 含义 - ACL格式错误，如果您是使用SDK提供的AVACL类，理论上这不应该发生，正确的ACL格式请参考[REST API](https://leancloud.cn/docs/rest_api.html#%E5%AE%89%E5%85%A8%E6%80%A7)。

## 124

* 信息 - `The request timed out on the server. Typically this indicates the request is too expensive.`
* 含义 - 请求超时，超过一定时间（默认10秒）没有返回，通常是因为网络故障或者该操作太耗时引起的。

## 125

* 信息 - `The email address was invalid.`
* 含义 - 电子邮箱地址无效。

## 126

* 信息 - `Invalid user id.`
* 含义 - 无效的用户 Id，可能用户不存在

## 127

* 信息 - `The mobile phone number was invalid.`
* 含义 - 手机号码无效

## 137

* 信息 - `A unique field was given a value that is already taken.`
* 含义 - 违反class中的唯一性索引约束(unique)，尝试存储重复的值。

## 139

* 信息 - `Role's name is invalid.`
* 含义 - 角色名称非法，角色名称只能以英文字母、数字或下划线组成。

## 140

* 信息 - `Exceeded an application quota. Upgrade to resolve.`
* 含义 - 超过应用的容量限额，请升级帐户等级。

## 141

* 信息 - `Cloud Code script had an error.`
* 含义 - 云代码脚本编译或者运行报错。

## 142

* 信息 - `Cloud Code validation failed.`
* 含义 - 云代码校验错误，通常是因为beforeSave、beforeDelete等函数返回error。

## 145

* 信息 - `Payment is disabled on this device`
* 含义 - 本设备没有启用支付功能。

## 150

* 信息 - `Fail to convert data to image.`
* 含义 - 转换数据到图片失败。

## 200

* 信息 - ` Username is missing or empty`
* 含义 - 没有提供用户名，或者用户名为空。

## 201

* 信息 - `Password is missing or empty`
* 含义 - 没有提供密码，或者密码为空。

## 202

* 信息 - `Username has already been taken`
* 含义 - 用户名已经被占用。

## 203

* 信息 - `Email has already been taken`
* 含义 - 电子邮箱地址已经被占用。

## 204

* 信息 - `The email is missing, and must be specified`
* 含义 - 没有提供电子邮箱地址。

## 205

* 信息 - `A user with the specified email was not found`
* 含义 - 找不到电子邮箱地址对应的用户。

## 206

* 信息 - `The user cannot be altered by a client without the session.`
* 含义 - 没有提供session，无法修改用户信息，这通常是因为没有登录的用户想修改信息。修改用户信息必须登录，除非在云代码里，或者使用master key调用REST API。

## 207

* 信息 - `Users can only be created through sign up`
* 含义 - 只能通过注册创建用户，不允许第三方登录。

## 208

* 信息 - `An existing account already linked to another user.`
* 含义 - 第三方帐号已经绑定到一个用户，不可绑定到其他用户。

## 210

* 信息 - `The username and password mismatch.`
* 含义 - 用户名和密码不匹配。

## 211

* 信息 - `Cloud not find user`
* 含义 - 找不到用户

## 212

* 信息 - `The mobile phone number is missing, and must be specified`
* 含义 - 请提供手机号码。

## 213

* 信息 - `An user with the specified mobile phone number was not found`
* 含义 - 手机号码对应的用户不存在

## 214

* 信息 - `Mobile phone number has already been taken`
* 含义 - 手机号码已经被注册

## 215

* 信息 - `Mobile phone number isn't verified.`
* 含义 - 未验证的手机号码

## 250

* 信息 - `Linked id missing from request`
* 含义 - 连接的第三方账户没有返回用户唯一标示id

## 251

* 信息 - `Invalid linked session`或者`Invalid Weibo session`
* 含义 - 无效的账户连接，一般是因为access token非法引起的。

## 300

* 信息 - `CQL syntax error.`
* 含义 - CQL 语法错误。详情参考 [CQL 语法详细指南](./cql_guide.html)

## 401

* 信息 - `Unauthorized.`
* 含义 - 未经授权的访问，没有提供App id，或者App id和App key校验失败，请检查配置。

## 403

* 信息 - `Forbidden to xxx by class permissions`
* 含义 - 操作被禁止，因为[class权限限制](https://leancloud.cn/docs/data_security.html#class-%E7%BA%A7%E5%88%AB%E7%9A%84%E6%9D%83%E9%99%90)。

## 503

* 信息 - `Rate limit exceeded.`
* 含义 - 超过流量访问限制，默认API并发1000访问每秒，通过数据管理平台每秒限制上传一个文件，并且每分钟最多上传30个文件，如需提升，请联系我们。
