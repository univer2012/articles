## Mark Up Web Content

来自：[Mark Up Web Content](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/WebContent.html#//apple_ref/doc/uid/TP40016308-CH8-SW1)

---

如果你的app的部分或全部内容在你的网站上也是可用的，你可以使用web标记(web markup)，让用户在搜索结果中访问你的内容。使用web标记可以让Applebot web爬虫在苹果的服务器端索引中索引你的内容，这使得所有iOS用户都可以在Spotlight和Safari搜索结果中使用它。

除了添加web标记之外，强烈建议你支持通用链接。添加对通用链接的支持，通过当用户点击到你网站的链接时打开你的原生app(的方式)，进一步增强了用户体验(如果您的应用程序没有安装，点击结果将打开Safari)。这种行为有助于包含大量数据的网站被所有用户索引和搜索。

要了解如何使用通用链接，请参阅[Support Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)。清单5-1展示了使用通用链接让用户点击一个网站链接并打开app的例子。

清单5-1 使用通用链接来支持在应用程序中打开网站链接
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Mark_Up_Web_Content_1.png)

要使用web标记搜索你的内容，请遵循这3个步骤：
1. 确保Apple能够发现和索引你的网站；
2. 给从你的网站到你的app的深度链接添加标记。
3. 通过为结构化数据添加标记，来丰富搜索结果。

> NOTE
> 因为被Applebot索引的项已经是公开的，所以它们村粗在Apple的服务端索引中。

### 让你的网站被Apple发现
确保Applebot web爬虫抓取你的网站的最简单的方法是，在提交你的app时，指定URL作为你的支持或营销网站。要了解有关指定此URL的更多信息，请参阅Version Infromation。

清单5-2 指定支持和营销的URLs
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Mark_Up_Web_Content_2.png)

此外，修改`robots.txt`文件以允许Applebot抓取你的网站也是非常重要的。Applebot检查你的`robots.txt`文件，以确定应该抓取你网站的哪部分。(你可以在[Wikipeida](https://en.wikipedia.org/wiki/Robots_exclusion_standard)了解更多有关`robots.txt`文件的信息) 要检测Applebot，可以使用正则表达式。

> NOTE
> Applebot 忽略URL的片段标志符组件。

使用App Search API验证工具来验证Applebot能够从您的网站中提取的数据。来自验证工具的信息可以帮助您识别应该添加的信息片段，并帮助您发现优化网站元数据的方法。你可以在这里访问这个严重工具：https://search.developer.apple.com/appsearch-validation-tool.

### 给你的网站添加深度链接

帮助你的网站的用户发现你的app的最好的方式是，采用Smart App Banners(你可以通过阅读[Promoting Apps with Smart App Banners](https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariWebContent/PromotingAppswithAppBanners/PromotingAppswithAppBanners.html#//apple_ref/doc/uid/TP40002051-CH6)了解更多有关Smart App Banners的信息)。在你网站上的Smart App Banner，会邀请没有安装你的app的用户从App Store下载它，为已经按照你的app的用户提供了一个打开页面的简单方法。清单5-3展示了一个Smart App Banner的例子。

清单5-3 一个将用户重定向到你已安装的app的 Smart App Banner
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Mark_Up_Web_Content_3.png)

在你的Smart App Banner标记中包含一个`app-argument`值，可以用Apple索引你的内容。为了包含一个`app-argument`值，你可以使用类似的标记:

```
<meta name="myApp" content="app-id=123, app-argument=http://example.com/about">
```
至关重要的是，`app-argument`值将URL包含到与用户当前正在查看的特定web内容对应的原生app中。不要将`app-argument`值设置为app打开屏幕的URL。

除了添加一个Smart App Banner，强烈建议你使用通用链接和你的深链接，而不是使用自定义URL方案。当你支持通用链接，iOS能用Handoff来启动你的app，并提供给你用户正在浏览的指定web URL。了解如何支持通用链接，请查阅[Support Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)。

作为使用Smart App Banners的另一种选择，你可以使用苹果支持的开放标准之一，在你的网站上提供深度链接，如Twitter Cards 和 App Links。你可以使用类似下面的Twitter Cards 标记：
```
<meta name="twitter:app:name:iphone" content="myAppName">
<meta name="twitter:app:id:iphone" content="myAppID">
<meta name="twitter:app:url:iphone" content="myURL">
```
或者你可以用类似的方式使用App Links标记：
```
<meta property="al:ios:app_name" content="myAppName">
<meta property="al:ios:app_store_id" content="myAppID">
<meta property="al:ios:url" content="myURL">
```
有关Twitter Cards的更多信息，请查阅 https://dev.twitter.com/cards/mobile  ；有关App Links,的更多信息，请查阅 http://applinks.org 。

当你用深度链接标记你的网站后，你要确保你的app能够处理它们。当你支持通用链接时，iOS调用你的`application:continueUserActivity:restorationHandler: `代理方法，当用户点击深度链接进入你的app时。如果你用自定义方案，iOS调用`openURL:`来打开你的app。清单5-1展示了一个用`openURL:`来处理来自Smart App Banner的深度链接的例子。

清单5-1 处理深度链接
```
    // In this example, the URL is "http://example.com/profile/?123".
    func application(application: UIApplication, openURL url: NSURL, sourceApplication: String?, annotation: AnyObject) -> Bool {
        if let components = NSURLComponents(URL: url, resolvingAgainstBaseURL: true), let path = components.path, let query = components.query {
            if path == "/profile" {
                // Pass the profile ID from the URL to the view controller.
                return profileViewController.loadProfile(query)
            }
        }
        return false
    }
```

> NOTE
> 要了解更多在你的app中处理通用链接，请查阅 [Preparing Your App to Handle Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW2)。

### 丰富搜索结果

在你的网站上标记结构化数据有助于Apple更好地解析和理解你的内容，并提供更丰富的搜索结果。例如，除了提供一个项的标题和描述，你还可以包含元数据，如图像、价格、评级(ratings)和其他。提供接过话数据的最大优势在于，它可以帮助你提高搜索结果的排名：用户倾向于更多的参与包含更丰富信息的结果；而且，得到更多互动的结果显示得更频繁。

要对web内容进行注释，以便用户可以看到丰富的搜索结果，可以对结构化数据使用基于标准的标记，如[Schema.org](http://schema.org/)中定义的标记。例如，清单5-2中所示的代码结合了标记的不同类型，以给用户丰富的信息，如图5-4所示。

清单5-2 用标记提供丰富的信息
```
<title>Beats by Dr. Dre Solo2 Wireless Headphones - Apple</title>
<meta property="og:description" content="Beats by Dr. Dre Solo2 Wireless Headphones let you take your music anywhere you go. Get fast, free shipping when you buy online.">
<span itemprop="reviewCount">924</span>
<meta itemprop="ratingValue" content="4.5">
<meta itemprop="priceCurrency" content="USD">
```

图像5-4 标记结构化数据，以给用户丰富信息
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Mark_Up_Web_Content_4.png)

虽然必须提供深度链接、标题和描述元素来索引一个项，但仍强烈你建议也包括一张特定内容的图片。

目前支持以下模式：

1. 总体评价
2. 出价
3. 价格区间
4. 交互数
5. 组织
6. 食谱
7. 搜索动作
8. 图片对象

除了使用如[Schema.org](http://schema.org/)中定义的结构化数据标记，你可以使用Open Graph协议标记（在[opg.me](http://ogp.me/)中定义）来指定一个与结果相伴的图片、标题、音频、视频和描述。你可以使用[Schema.org](http://schema.org/)标记来启用搜索结果中的操作。例如，您可以让用户拨打电话号码或获得地址的方向。要启用用户可以在搜索结果中执行的操作，可以使用如清单5-3所示的代码。(你可以看到用户如何访问这些在[ Search Consists of Several Technologies](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/index.html#//apple_ref/doc/uid/TP40016308-CH4-SW2)中的操作的例子。)

清单5-3 执行各种各样的用web标记的操作
```
    <!— Enable dialing a phone number. ->
      <div itemscope itemtype=
          "http://schema.org/Organization">
          <span itemprop="telephone">
          (408) 123-4567</span>
      </div>
     
      <!— Enable getting directions to an address. ->
      <div itemprop="address" itemscope
      itemtype="http://schema.org/PostalAddress">
          <span itemprop="streetAddress">
        1 Infinite Loop
    </span>
    <span itemprop="addressLocality">
        Cupertino</span>,
    <span itemprop="addressRegion">
        CA</span>
    <span itemprop="postalCode">
        95014</span>
      </div>
```

正如[Making Your Website Discoverable by Apple](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/WebContent.html#//apple_ref/doc/uid/TP40016308-CH8-SW5)中提到的，使用App Search API验证工具检查你添加到网站的元数据的正确性是一个好主意。

---
[完]