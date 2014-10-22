# Unity 统计 SDK 开发指南
LeanCloud Unity SDK 提供一套接口方便开发者可以对了解游戏在客户端运行时的一些参数或者是指标。

## 下载 SDK

您可以从 [SDK 下载页面](https://leancloud.cn/docs/sdk_down.html) 下载LeanCloud Unity SDK。

## 使用统计功能
在使用统计SDK之前需要初始化应用的参数，参数包括App ID以及App Key ，您可以在[设置](app.html?appid={{appid}}#/key)查询到这2个参数，如果在应用的其他地方已经进行初始化了，无需重复。
```
AVClient.Initialize("{{appid}}", "{{appkey}}");
```

**统计功能为默认打开, 并且可以在线配置.** 您可以进入应用的 [统计分析 -> 统计设置 -> 数据发送策略](/stat.html?appid={{appid}}#/statconfig/trans_strategoy) 在线更改SDK端的数据报告发送策略.

* 建议使用AV_BATCH形式，减少App与网络的交互，为用户节约流量。

默认是`启动时发送`策略: 应用程序每次会在启动时会向服务器发送一次消息，在应用程序过程中产生的所有统计信息(包括自定义事件和本次使用时长)都会在下次启动时候发送。 如果应用程序启动时处在不联网状态，那么消息将会缓存在本地，下次再尝试发送。

##  游戏场景访问统计

使用Unity开发游戏的时候必然会遇到需要统计玩家在各个游戏场景进入的时间，退出的时间等信息，以供开发者提高游戏体验以及优化游戏资源分配的参数，LeanCloud Unity SDK已经提供了这样的方法。

比如在主界面点击“进入副本”按钮，开始加载副本场景：

```
void OnGUI()//Home场景中的一个Behaviour中的OnGUI事件
{
	if (GUI.Button (new Rect (50, 50, 200, 50), "进入副本"))
	{
		Application.LoadLevel("BT");//加载“黑暗神殿”的场景
		AVAnalytics.OnSceneStart ("BT");//记录开始加载时间
		AVAnalytics.OnSceneEnd ("Home");//游戏引擎的特殊性，没有事件驱动供SDK抓取，所以需要显式的调用一次OnSceneEnd
	}
}
```

同样的，在副本结束之后，在副本场景中的页面上，点击“离开副本”的按钮返回主界面：

```
void OnGUI()//BT场景中的一个Behaviour中的OnGUI事件
{
	if (GUI.Button (new Rect (50, 50, 200, 50), "离开副本"))
	{
		Application.LoadLevel("Home");//返回主界面
		AVAnalytics.OnSceneStart ("Home");
		AVAnalytics.OnSceneEnd ("BT");
	}
}
```

## 使用自定义事件


### 基本简单事件

```
public static void TrackEvent(string name);
```

除了基本统计分析功能外，LeanCloud Unity SDK 还支持您自定义的事件分析，例如您可以统计玩家在游戏中点击商店的动作，购买东西的动作。


```
if (GUI.Button (new Rect (100, 600, 200, 50), "商店"))
{
	AVAnalytics.TrackEvent("进入商店");
	words = "为了兽人的荣耀！";
}
GUI.Label (new Rect (100, 400, 200, 50), words);

```

### 多标签事件
```
public static void TrackEvent(string name, IDictionary<string, object> dimensions);
```
实际上在很多游戏中简单的进入商店，或者与某NPC对话这些简单的事件未必能满足数据分析的需求，假如需要统计玩家进入商店购买了那些东西，花了多少时间，这些如果能跟一个事件关联一起来，在做数据分析的时候就有足够采样价值。

```
bool stepedInshop=false;
private string words=string.Empty;
private DateTime stepedInShopTime=DateTime.MinValue;
void OnGUI()
{
	if (!stepedInshop)
	{
		if (GUI.Button (new Rect (100, 200, 200, 50), "商店"))
		{
			stepedInshop=true;
			AVAnalytics.TrackEvent ("进入商店");
			stepedInShopTime = DateTime.Now;
			words = "为了Dota的荣耀";
		}
	}
	GUI.Label (new Rect (100, 50, 200, 50), words);
	if (stepedInshop)
	{
		if (GUI.Button (new Rect (100, 80, 200, 50), "购买雷神之锤"))
		{
			var baughtTime = DateTime.Now;
			var dimensions = new Dictionary<string, object>
			{
				{ "baughtTime", DateTime.Now},
				{ "duration",  baughtTime-stepedInShopTime},
				{ "location", "HomeScene" }
			};
			AVAnalytics.TrackEvent ("购买雷神之锤", dimensions);
		}
	}
}
```
## 设置数据发送策略

您可以进入应用的 [统计分析 -> Android统计 -> 统计设置 -> 数据发送策略](/stat.html?appid={{appid}}&os=android#/statconfig/trans_strategoy) 在线更改SDK端的数据报告发送策略。在没有取到在线配置的发送策略的情况下，会使用默认的发送策略.

以下均为在线配置中的可选策略


### 启动时发送（推荐使用）
应用程序每次会在启动时会向服务器发送一次消息，在应用程序过程中产生的所有消息(包括自定义事件和本次使用时长)都会在下次启动时候发送。 如果应用程序启动时处在不联网状态，那么消息将会缓存在本地，下次再尝试发送。

发送策略默认为启动时发送。

### 批量发送
批量发送，默认当消息数量达到30条时发送一次。

### 按最小间隔发送
间隔一段时间发送，每隔一段时间一次性发送到服务器。
