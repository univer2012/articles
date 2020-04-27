## 用Smart App Banners提升Apps

来自：[Promoting Apps with Smart App Banners](https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariWebContent/PromotingAppswithAppBanners/PromotingAppswithAppBanners.html#//apple_ref/doc/uid/TP40002051-CH6-SW1)

---

Safari在iOS6以后有一个新的Smart App Banner特性，这提供了一种标准化的方法，可以从网站上推广App Store上的应用程序，如图像8-1所示。



图像8-1 一个Apple Store应用程序的Smart App Banner

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/smartbanner_2x.png)

> 注意：Smart App Banners 只在iOS上展示，不能在OS X展示。



Smart App Banners 极大地提升用户浏览体验，比较于其他宣传方法。因为banners是在iOS6上被实现的，它们将在web上提供一致的外观和感觉，用户将逐渐认识到这一点。用户会相信，点击banner将带领他们到App Store，而不是第三方广告。他们将欣赏在网页顶部不显眼地被展示的那些banners，而不是作为一个全屏广告妨碍web内容。而且，有一个大而突出的关闭按钮，banner很容易被用户dismiss。当用户返回到网页，banner将不再出现。

如果该app已经安装在用户设备上，banner能够智能地改变它的行为，点击banner仅仅打开应用程序。如果在用户的设备上没有你的app，点击banner将引导用户到在App Store上的app入口。当他返回到你的网站，一个表明多久下载完成的进度条出现在banner上。当app下载完成，“查看”按钮变成一个“打开”按钮，点击banner将打开app，同时保留用户在网站的上下文。

Smart App Banners自动决定app在用户设备上是否支持。如果加载banner的设备不支持你的app，或者你的app在用户的位置不可用，banner将不展示。

### 在你的网站上实现一个 Smart App Banner
为了加一个Smart App Banner到你的网站，包含下面的`meta`标签在你想banner出现的每一个页面的头部：
```
<meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL">
```
你可以包含3个逗号分隔的参数在`content`属性上：
1. `app-id`：(Required)你的app的唯一标志符。要从[iTunes Link Maker](https://linkmaker.itunes.apple.com/zh-cn)找到你app ID，在搜索框键入你app的名字，并旋转适当的国家和媒体类型。在结果中，找到你的app并选择在列表右侧的“iPhone App Link“。你的app ID是介于`id`和`?mt`之间的9个数字。
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/smartbanner_3.png)
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/smartbanner_4.png)
2. `affiliate-data`：(Optional.)你的iTunes附属字符串，如果你是iTunes会员。如果你不是，请在http://www.apple.com/itunes/affiliates/ 了解更多关于成为iTunes会员的信息。
3. `app-argument`：(Optional.)一个提供上下文到你原生app的URL。如果你包含这个，并且用户安装了你的app，她能够从你的网站跳到你的iOS app中相应的位置。通常，保留导航上下文是有益的，因为：
		1. 如果用户位于你的网站的导航层次结构中，你可以传递文件的整个URL，并且在你的app中解析它，从而将它重新路由到你的app的正确位置。
		2. 如果用户在你的网站执行了一个搜索，你可以传递搜索字符串以至于她可以无缝地在你的app继续搜索，无需重新键入用户的疑问。
		3. 如果用户正在创建内容，你可以传递session ID在你的app中下载web session state，这样用户就可以无损得继续工作。

你可以用服务端脚本动态地生成每一页的`app-argument`。然而你想要，你可以格式化它，只要她是一个有效的URL。

> 注意：你无法在a frame内显示Smart App Banners。banners不能出现在iOS Simulator中。

### 提供导航上下文到你的app

在你的app中，实现在AppDelegate中的`application:openURL:sourceApplication:annotation:`方法，当你的app从一个URL启动时，它将触发。然后提供能够解释你传递的URL的逻辑。你设置的`app-argument`参数值可以作为`NSURL` url对象使用。

在[清单8-1]的例子演示了一个将数据传递给一个原生iOS app的网站。为了完成这个，检测URL是否包含`/profile`字符串。如果是，则打开`profile`视图控制器并传递查询字符串中的profile ID number.

清单8-1 路由用户到正确的view controller
```swift
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    // 在本例中，用户来自的URL是 http://example.com/profile/?12345
    // 确定用户是否正在查看一个profile
    if ([[url path] isEqualToString:@"/profile"]) {
        // 切换到 profile view controller
        [self.tabBarController setSelectedViewController:profileViewController];
        // pull the profile id number found in the query string
        NSString *profileID = [url query];
        // pass profileID to profile view controller
        [profileViewController loadProfile:profileID];
    }
    return YES;
}
```

