# Universal Links探究

参考：

1. [iOS 通用链接（UniversalLinks）+ 分享功能的一些看法](https://www.jianshu.com/p/8ae3576b12b0)
2. [iOS Universal Links(通用链接)的使用](https://www.jianshu.com/p/1970fd59de12)
3. [Support Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)
4. [Web Browser–to–Native App Handoff](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/Handoff/AdoptingHandoff/AdoptingHandoff.html#//apple_ref/doc/uid/TP40014338-CH2-SW10)
5. [iOS9 App Search编程(1)- 搜索基础](https://www.jianshu.com/p/01e6e998ecf5)

## 一、什么是Universal Links

> When you support universal links, iOS users can tap a link to your website and get seamlessly redirected to your installed app without going through Safari. If your app isn’t installed, tapping a link to your website opens your website in Safari. 
> 
> 当你支持通用链接，iOS用户可以点击你网站的链接，无需通过Safari就可以无缝重定向到已安装的你的app。如果你的app没有被安装，点击你网站的链接将在Safari打开你的网站。

Universal Links是iOS9推出的一项功能，使你的应用可以通过传统的HTTP或HTTPS链接来启动APP(如果iOS设备上已经安装了你的app，不管在微信里还是在哪里)， 或者打开网页(iOS设备上没有安装你的app)。

## 二、使用Universal Links的场景和优势
1. 可以使用通用链接，跳过微信对自定义URL Schemes的屏蔽
2. 其他只要能识别url，用户点击加了Universal Links的url就可以直接跳到已安装的APP里去，不需再经过Safari间接跳转，直接把用户引流到APP。
3. 对分享出去的网页，可以将对我们APP感兴趣的用户引入到APP。

## 三、怎么使用Universal Links
1. 先决条件：你必须有一个域名,且这个域名需要支持https。
2. 配置对应的App ID支持Associated Domains：要在开发者中心做配置：找到对应的App ID，在Application Services列表里有Associated Domains一条，把它变为Enabled。同时编辑对应的Provisioning Profiles，并且重新下载双击安装。
![](https://upload-images.jianshu.io/upload_images/1743443-dfea7134921c89c5.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
![](https://upload-images.jianshu.io/upload_images/1743443-5e7e06116373c01d.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
3. 配置项目中的Associated Domains：在Xcode中，选中对应的TARGETS，在Capabilities选项卡下，找到Associated Domains，打开开关，在其中的Domains中添加你要支持的域名，注意，**必须以`applinks:`为前缀**。
![](https://images2018.cnblogs.com/blog/837281/201809/837281-20180904104422264-1449519633.png)

4. 编写一个无后缀的`apple-app-site-association`：创建一个内容为json格式的文件，苹果将会在合适的时候，从我们在项目中填入的域名请求这个文件。**该文件名没有后缀。** 文件内容大概如下：
```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "9JA89QQLNQ.com.apple.wwdc",
                "paths": [ "/wwdc/news/", "/videos/wwdc/2015/*"]
            },
            {
                "appID": "ABCD1234.com.apple.wwdc",
                "paths": [ "*" ]
            }
        ]
    }
}
```
> 注意：
> 
> 1. `apps`必须使空数组。
> 2. `appID`组成是`<team identifier>` + `.` + `<bundle identifier>`，例如：`9JA89QQLNQ.com.apple.wwdc`。其中team ID可以在苹果Apple Developer -> Member Center ->Membership->Team ID中找到；bundle ID可以在工程的`Bundle Identifier`中找到。
> 3. `paths`设定的你的app支持的路径列表，只有这些指定的路径的链接，才能被app处理。
> 
> 	1. 可以用`*`匹配任何子字符串，`?`匹配任何单个字符。`NOT `(`T`后面包含一个空格)作前缀开头的字符串，代表不被通用链接处理的路径。例如
> 	`"paths": [ "/wwdc/news/", "NOT /videos/wwdc/2010/*", "/videos/wwdc/201?/*"]`
> 	2. `paths`数组中指定路径的字符串，区分大小写。

5. 将`apple-app-site-association`文件上传到服务器的`.well-known`文件夹下面。

	之前的官方文档是放在根目录下，后面改为了放在根目录和`.well-known`子目录都可以，不过 Handoff会先去`.well-known`子目录查找该文件，如果该子目录没有，才返回到根目录查找。

6. 验证上传的文件是否合格。
  
   第一个方法是把支持Universal Links的链接放在苹果官方的[App Search Validation Tool](https://search.developer.apple.com/appsearch-validation-tool)中测试，观察Applebot爬虫能否抓取到链接中的信息。
   第二个方法是直接请求`https://www.domains.com/apple-app-site-association`，观察能否把上传到服务器的`apple-app-site-association`文件中的JSON内容请求到。




## 四、验证

先决条件：要安装app。

验证方法：
1. 在iOS设备的备忘录中输入app能识别的链接，然后点击此链接，直接跳转到你的app代表成功。或是长按，在出现的弹出菜单中第二项是在’XXX’中打开，也代表着成功。
![](https://images2018.cnblogs.com/blog/837281/201809/837281-20180904104703237-609503870.png)

2. 将要测试的网址在safari中打开，在出现的网页上方下滑，可以看到有【在”XX”应用中打开】代表成功。
![](https://images2018.cnblogs.com/blog/837281/201809/837281-20180904104742793-240925888.png)

3. 在Message中发送该链接，然后点击，可以直接跳转到app代表成功。
4. 在微信中发送该链接，然后点击打开网页，再点击右上角的分享，在Safari中打开，可以直接跳转到app表示成功。

## 五、进入app后的处理
你匹配你关联的域名后，使用对Handoff的`UIApplicationDelegate`方法(特别是`application:continueUserActivity:restorationHandler:`)，以便你的app可以收到一个链接并适当地处理它。
```
//Objective-C
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler
{
    if ([userActivity.activityType isEqualToString:NSUserActivityTypeBrowsingWeb])
    {
        NSURL *url = userActivity.webpageURL;
        if (url是我们希望处理的)
        {
            //进行我们的处理
        }
        else
        {
            [[UIApplication sharedApplication] openURL:url];
        }
    }
    return YES;
}
```
```
//Swift
override func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([Any]?) -> Void) -> Bool {
        if userActivity.activityType == NSUserActivityTypeBrowsingWeb {
            let url = userActivity.webpageURL
            if url是我们希望处理的 {
                //进行我们的处理
            } else {
                if let url = url {
                    UIApplication.shared.openURL(url)
                }
            }
        }
        return true
    }
```

其他代理方法是：
* `override func application(_ application: UIApplication, willContinueUserActivityWithType userActivityType: String) -> Bool { }`
* `override func application(_ application: UIApplication, didFailToContinueUserActivityWithType userActivityType: String, error: Error) { }`
* `override func application(_ application: UIApplication, didUpdate userActivity: NSUserActivity) { }`

## facebook的样例

<video src="https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Universal_Links.mov" width="320" height="180"
controls="controls"></video>

在浏览器输入`https://www.facebook.com/apple-app-site-association`或者`https://www.facebook.com/.well-known/apple-app-site-association`，可以看到facebook的`apple-app-site-association`文件的内容：
```
{
  "activitycontinuation": {
    "apps": [
      "V9WTTPBFK9.com.facebook.drafts.internal",
      "4W5TH4RKQ2.com.facebook.drafts.internalDevelopment",
      "4W5TH4RKQ2.com.facebook.drafts.internalInHouse"
    ]
  },
  "webcredentials": {
    "apps": [
      "T84QZS65DQ.com.facebook.Facebook",
      "T84QZS65DQ.com.facebook.Wilde",
      "3NW3KR6Q88.com.facebook.FacebookDevelopment",
      "3NW3KR6Q88.com.facebook.WildeInHouse",
      "T84QZS65DQ.com.facebook.bishop",
      "T84QZS65DQ.com.facebook.bishop.localDevelopment",
      "3NW3KR6Q88.com.facebook.bishop.internalInHouse",
      "3NW3KR6Q88.com.facebook.bishop.internalDevelopment",
      "T84QZS65DQ.com.facebook.highlight.dev"
    ]
  },
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "T84QZS65DQ.com.facebook.MessengerDev",
        "paths": [
          "/messenger/diode/*"
        ]
      },
      {
        "appID": "T84QZS65DQ.com.facebook.Messenger",
        "paths": [
          "/messenger/diode/*"
        ]
      },
      {
        "appID": "3NW3KR6Q88.com.facebook.OrcaDevelopment",
        "paths": [
          "/messenger/diode/*"
        ]
      },
      //... ...
    ]
  }
}
```

## 增:项目需求

项目是原本是需要改善分享出去的h5活动页面，要做这样的事件点击：安装了app，点击直接跳到app里面相应的页面，如果没有安装app，点击按钮跳到App Store页面。之前项目采用自定义URL Scheme的方案，h5点击按钮后起一个定时器，指定时间没有跳走就是没有安装app，就跳去App Store。但是在跳去App Store的中间会出现 [`Safari浏览器打不开该网页，因为网址无效`] 的页面，影响用户体验。而且安装app了有时超过指定时间也没有跳转到app—— 也就是判断不准确的情况。

所以经过一番市场app调查和研究，发现[知乎]App的实现方式是比较合理且不影响用户体验的。经过研究，发现[知乎]App的实现逻辑如下：

同时准备好两套域名：A域名和B域名，这2个域名都支持了通用链接，用户要浏览的网页用A域名，[`App内打开`]按钮响应的链接是B域名。按钮响应链接页面可以是统一的下载页面，出现这个页面的时候就执行跳转App Store事件。

当用户点击 [`App内打开`]按钮 后，因为B域名支持通用链接，如果是安装了app，系统可以识别到，就会立即通过通用链接的方式，直接跳转到App。如果没有安装App，那么自然是无法用通用链接跳转的，这时打开了B域名的链接。当出现 B域名的统一下载页面时，就会执行跳转App Store，此时这个统一下载页面会在短暂的出现在用户面前，但是是我们自己的页面所以不会影响用户体验。

要准备A、B域名，其实是因为通用链接要在[这里](https://www.jianshu.com/p/475b398a117d)所说的 **跨域 **的情况下才会生效，同一个域名下的跳转是不会生效的，即使这个域名支持了通用链接。

> "注意看域名：oia.zhihu.com，这个就是知乎的universal link，之所以是oia开头，是因为universal link必须跨域。"

逻辑灵感来自：[h5跳转app指定页面及各种坑的总结](https://www.jianshu.com/p/475b398a117d)


## 关于`applinks:*.mywebsite.com`或者`applinks:*.users.mywebsite.com`
在Xcode --> `Capabilities`选项卡的`Associated Domains`中添加关联域名时，是支持使用通配符的。[官方文档](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)原文如下：

> To match all subdomains of an associated domain, you can specify a wildcard by prefixing `*.` before the beginning of a specific domain (the period is required). Domain matching is based on the longest substring in the `applinks` entries. For example, if you specify the entries `applinks:*.mywebsite.com` and `applinks:*.users.mywebsite.com`, matching for the domain `emily.users.mywebsite.com` is performed against the longer `*.users.mywebsite.com` entry. Note that an entry for `*.mywebsite.com` does not match `mywebsite.com` because of the period after the asterisk. To enable matching for both `*.mywebsite.com` and `mywebsite.com`, you need to provide a separate `applinks` entry for each. 
> 
[翻译](https://github.com/univer2012/personal-document/blob/master/Articles/Universal%20Links%E6%8E%A2%E7%A9%B6.md)如下：
> 要匹配关联域名的所有子域名，可以通过在特定域名的开始之前加上前缀`*.`指定通配符（需要使用句点）。域名匹配基于`applinks`条目中最长的子字符串。例如，如果你指定了条目`applinks:*.mywebsite.com`和`applinks:*.users.mywebsite.com`，则对较长的条目`*.users.mywebsite.com`执行域名`emily.users.mywebsite.com`进行匹配。注意条目`*.mywebsite.com`不能匹配`mywebsite.com`因为星号后面有句点（`.`）。要同时匹配`*.mywebsite.com`和`mywebsite.com`，你需要为它们分别提供一个`applinks`条目。
> 

但是需要注意以下问题：

1. 注意是“在特定域名的开始之前加上前缀`*.`指定通配符（需要使用句点）“，而不是使用`*`，例如：`applinks:*.mywebsite.com`可以识别，但是`applinks:*mywebsite.com`则无法识别。

2. 要使用`applinks:*.mywebsite.com`这样的通配符配置，不仅 `mywebsite.com` 的子域（例如`subdomain.mywebsite.com`）要支持通用链接， 主域 `mywebsite.com` 本身也要支持通用链接，才可以。

3. 通用链接不支持重定向。

关于这个issue的讨论，可以参考：[Apple Developer Forums - Universal link and wildcard ](https://forums.developer.apple.com/thread/47315) 和 [stackoverflow: iOS Universal Links with Wildcards not working](https://stackoverflow.com/questions/37301392/ios-universal-links-with-wildcards-not-working)

---
## 扩展
翻译的官方文档：
1. [web浏览器到原生app的Handoff](http://note.youdao.com/noteshare?id=06d06c3f5461cfda499d25048a10d713)，原文：[Web Browser–to–Native App Handoff](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/Handoff/AdoptingHandoff/AdoptingHandoff.html#//apple_ref/doc/uid/TP40014338-CH2-SW10)
2. [支持 Universal Links](http://note.youdao.com/noteshare?id=0e96204d15cbf5618a11d33aadcc8c96)，原文：[Support Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)
3. [Mark Up Web Content](http://note.youdao.com/noteshare?id=5604f6662b02b9826a34eef525ef1f54)，原文：[Mark Up Web Content](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/WebContent.html#//apple_ref/doc/uid/TP40016308-CH8-SW1)
4. [用Smart App Banners提升Apps](http://note.youdao.com/noteshare?id=da637b720ea5ecb86bacf3d85cef2706)，原文：[Promoting Apps with Smart App Banners](https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariWebContent/PromotingAppswithAppBanners/PromotingAppswithAppBanners.html#//apple_ref/doc/uid/TP40002051-CH6-SW1)


