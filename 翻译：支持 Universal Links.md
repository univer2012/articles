## 支持 Universal Links

来自：[Support Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)

---
当你支持通用链接，iOS用户可以点击你网站的链接，无需通过Safari就可以无缝重定向到已安装的你的app。如果你的app没有被安装，点击你网站的链接将在Safari打开你的网站。

通用链接给了你好几个当你使用自定义URL模式时无法获得的关键好处。特别地，通用链接是：

1. **Unique**  不像自定义URL模式，通用链接不能被其他app声明，因为它们使用标准的HTTP或HTTPS链接到你的网站。
2. **Secure** 当用户安装了你的app，iOS会检查一个你上传至你的web服务器的文件，以确保你的网站允许你的app代表它打开URLs。只有你可以创建和上传这个文件，所以你的网站和你的app的关联是安全的。
3. **Flexible** 通用链接即使当你的app没有安装时也生效。当你的app没被安装时，点击你网站的链接，就会像用户期望的那样，在Safari打开内容。
4. **Simple** 一个URL对你的网站和app都生效。(一个URL同时适用于你的网站和app。)
5. **Private** 其他app可以与你的App通信，而不需要知道你的app是否被安装。

> NOTE: 
> 通用链接让用户打开你的app，当他们在`WKWebView`和`UIWebView`视图和Safari页面中指向你网站的链接时，此外还有一些链接会导致调用`openURL:`，比如在Mail、Messages和其他apps中出现的链接。
> 
> 当用户在Safari浏览你的网站时，他们点击一个指向当前网页有相同域名的URL的通用链接。iOS会尊重用户最有可能的意图，并在Safari中打开该链接。如果用户点击了一个指向不同域名的通用链接，iOS会在你的app打开该链接。
> 
> 对于运行iOS版本早于 9.0的用户，点击一个指向你网站的通用链接，就会在Safari打开该链接。

为通用链接添加支持是很容易的。你需要采取三个步骤：
1. 创建一个`apple-app-site-association`文件，其中包含有关你的app可以处理的URLs的JSON数据。
2. 上传`apple-app-site-association`文件到你的HTTPS web服务器。你可以把该文件放在你服务器的根目录或者在`.well-known` 子目录。
3. 准备你的app处理通用链接。

你可以在一台设备上测试通用链接。

## 创建和上传关联文件
为了在你的网站和app之间创建一个安全的连接，你要在它们之间建立一个信任关系。你通过2部分来建立这个关系：
* 一个你添加到你的网站的`apple-app-site-association`的文件
* 一个你添加到你的app的`com.apple.developer.associated-domains`权限(这部分被描述在[Preparing Your App to Handle Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW2))

你可以在[Shared Web Credentials Reference](https://developer.apple.com/documentation/security/shared_web_credentials)中了解更多关于你的app和网站如何共享凭据的信息。

> NOTE:
> 如果你的app运行在iOS9或更高版本，并且你用HTTPS来提供`apple-app-site-association`文件，你可以创建一个使用`application/json`MIME 类型的纯文本文件，并且不需要签名。如果在iOS8中支持Handoff和 Shared Web Credentials，仍需要像在[Shared Web Credentials Reference]()描述的方式对文件进行签名。

你需要为每个域名提供一个单独的`apple-app-site-association`文件，该域名包含你的app支持的unique内容。例如`apple.com`和`developer.apple.com`需要单独的`apple-app-site-association`文件，因为这些域名提供不同的内容。相比之下，`apple.com`和`www.apple.com`不需要单独的网站关联文件——因为两者的域名提供相同的内容——但这2个域名都必须使文件可用。对运行在iOS 9.3.1及以上的apps，`apple-app-site-association`文件的未压缩size必须不大于128KB，无论文件是否签名。

在`apple-app-site-association`文件中，你指定了应该作为通用链接处理的网站路径，以及不应该作为通用链接处理的路径。保持路径列表相当短，并依赖通配符来匹配更大的路由集合。清单6-1展示了一个`apple-app-site-association`文件的示例，该文件标识了3个应该作为通用链接处理的路径。

清单6-1 创建一个`apple-app-site-association`文件
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

> NOTE:
> 不要将`.json`附加到`apple-app-site-association`文件名中。

在`apple-app-site-association`文件中的`apps`键必须提供，且它的值必须是空数组，如清单6-1所示。`details`键的值是一个字典数组，每一个网站支持的app有一个字典。数组中字典的顺序决定了系统在查找匹配项时遵循的顺序，因此你可以指定一个app来处理网站的特点部分。

每一个app指定的字典包含一个`appID`键和`paths`键。`appID`键的值是team ID或者 app ID前缀，然后是bundle ID。（`appID`的值与构建后app权限中的“application-identifier”键关联的值相同。）`paths`键的值是一个字符串数组，它指定app支持的网站部分和不想要与app关联的网站部分。为了指定不应该被通用链接处理的区域，添加`NOT `(`T`后面包含一个空格)到路径字符串的开头。例如，清单6-1所示的`apple-app-site-association`文件可以通过如下所示更新`paths`数组，防止将网站的`/videos/wwdc/2010/*`区域作为通用链接被处理：
```
"paths": [ "/wwdc/news/", "NOT /videos/wwdc/2010/*", "/videos/wwdc/201?/*"]
```
因为系统按照指定的顺序计算`paths`数组中的每个路径——并且在发现整匹配或负匹配时停止计算——所以你应该在低优先级路径之前指定高优先级路径。注意，只有URL的路径组件用于比较。其他组件，如查询字符串或片段标志符将被忽略。

有多种方式指定在`apple-app-site-association`文件中的网站路径。例如，你可以：

1. 用`*`来指定你整个网站
2. 包含一个特定的URL，如`/wwdc/news`，来指定一个特特定的链接
3. 将`*`附加到一个特定的URL，如`/videos/wwdc/2015/*`，来指定你网站的某部分。

除了用`*`匹配任何子字符串，你也可以用`?`来匹配任何单个字符。你可以将2个通配符组合在一个路径中，如`/foo/*/bar/201?/mypage`。

> NOTE：
> 用于在`paths`数组中指定路径的字符串，区分大小写。

你创建`apple-app-site-association`文件后，上传它到HTTPS web服务器的根目录或者`.well-known`子目录。该文件需要通过HTTPS访问——不需要任何重定向——在`https://<domain>/apple-app-site-association`或 `https://<domain>/.well-known/apple-app-site-association`。接下来，你需要在你的app中处理通用链接。

### 准备你的app处理通用链接

通用链接使用2种技术：第一种与web浏览器和原生app之间的Handoff机制相同，第二种是共享网络凭据(Shared Web Credentials，有关这些技术的更多信息，请看[Web Browser–to–Native App Handoff](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/Handoff/AdoptingHandoff/AdoptingHandoff.html#//apple_ref/doc/uid/TP40014338-CH2-SW10)和[Shared Web Credentials Reference](https://developer.apple.com/documentation/security/shared_web_credentials))。当用户点击了通用链接，iOS启动你的app，并发送一个`NSUserActivity`对象给你的app，你可以通过这个对象查询你的app是如何启动的。

要支持在你app中的通用链接，请执行以下步骤：
1. 添加指定你的app支持的域名的权限
2. 更新你的app delegate，使其在收到`NSUserActivity`对象时做出适当的响应。

在你的`com.apple.developer.associated-domains`权限中，包含了一个你的app想要作为通用链接处理的域名的清单。要在Xcode中做这个，打开`Capabilities`选项卡中的`Associated Domains`部分，并为你的app支持的每个域名添加一个entry(条目)，该条目的前缀为`applinks:`，例如`applinks:www.mywebsite.com`。将这个列表限制在不超过20到30个域名。

要匹配关联域名的所有子域名，可以通过在特定域名的开始之前加上前缀`*.`指定通配符（需要使用句点）。域名匹配基于`applinks`条目中最长的子字符串。例如，如果你指定了条目`applinks:*.mywebsite.com`和`applinks:*.users.mywebsite.com`，则对较长的条目`*.users.mywebsite.com`执行域名`emily.users.mywebsite.com`进行匹配。注意条目`*.mywebsite.com`不能匹配`mywebsite.com`因为星号后面有句点（`.`）。要同时匹配`*.mywebsite.com`和`mywebsite.com`，你需要为它们分别提供一个`applinks`条目。

你匹配你关联的域名后，使用对Handoff的`UIApplicationDelegate`方法(特别是` application:continueUserActivity:restorationHandler:`)，以便你的app可以收到一个链接并适当地处理它。

当用户点击同意链接后，iOS启动你的app，你会收到一个`NSUserActivity`对象，它的`activityType`值为`NSUserActivityTypeBrowsingWeb`。该activity对象的`webpageURL`属性包含用户正在访问的URL。该网页URL属性总是包含一个HTTP或HTTPS URL，你可以用` NSURLComponents`APIs来操作该URL组件。

当用户点击你要处理的通用链接时，iOS也检查用户的最新选择来决定去打开你的app还是你的网站。例如，用户点击了通用链接来打开你的app，他稍后可以通过点击状态栏的 痕迹导航(breadcrumb)按钮来选择在Safari中打开你的网站。用户选择这个选项后，iOS继续在Safari打开你的网站，直到用户通过点击网页上的Smart App Banner上的 OPEN 按钮，来选择打开你的app。

> NOTE
> 如果你实例化一个SFSafariViewController、WKWebView或者UIWebView对象来处理一个通用链接，iOS会在Safari打开你的网站，而不是你的app。但是，如果用户从嵌入式SFSafariViewController、WKWebView或UIWebView对象中点击了一个通用链接，iOS就会打开你的app。

如果你的app使用`openURL:`打开一个指向你网站的通用链接，该链接不会打开你的app，理解这一点很重要。在这个场景中，iOS识别出调用来自你的app，因此该调用不应该作为通用链接被你的app处理。

如果你在activity对象中接收到一个无效的URL，那么优雅得失败是很重要的。要处理不支持的URL，你可以在共享应用对象上调用`openURL:`，在Safari打开该链接。如果不能执行此调用，则向用户显示一条错误消息说明出错的原因。

> IMPORTANT
> 为了保护用户隐私和安全，在需要传输数据时不应该使用HTTP；相反，应使用一个像HTTPS那样的安全传输协议。



