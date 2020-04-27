## Search Drives User Engagement

来自：[Search Drives User Engagement](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/index.html#//apple_ref/doc/uid/TP40016308-CH4-SW1)

---
在iOS9及以上版本，搜索给人们提供了访问你app内部信息的好方法，即使(你的app)没有被安装。当你让你的内容可搜索时，用户可以通过Spolight和Safari搜索结果，Handoff、Siri Suggestions和Reminders，来访问你的app内部活动和内容。让你的内容可搜索将帮助你提高你app的用户体验，并提高它的被发现率。

对你来说，App搜索很容易使用和定制。你不需要任何实现搜索的经验，并且你可以控制哪些内容被索引、哪些信息要显示在搜索结果中、以及用户点击与你内容相关的结果后去那里。

### 2种索引有助于保护用户隐私
隐私是iOS中搜索的一个基本特性。为了给用户提供最佳的搜索体验，同时保护他们的私人数据，有两个索引可以存储与内容相关的可搜索项。您可以使用各种与搜索相关的api为每个条目指定适当的索引，因此了解每个索引的工作方式非常重要。

iOS提供了以下索引：
1. 一个私有的设备上的索引(**A private on-device index**)：每个设备都包含一个私有索引，该索引的信息从未与苹果共享或在设备之间同步。当在用户设备上的索引中提供项时，只有该用户才能在搜索结果中查看该项。
2. 苹果服务端索引(**Apple’s server-side index**)：服务端索引存储的只有公开可用的、你的网站上做了适当标记的数据。

### 搜索由几种技术组成
为了给用户最佳的搜索体验，并增加与你的app的互动，你应使用不同APIs和技术的组合。在你开始之前，评估你的内容，以确定内容存储在何处，以及谁能够访问有关内容信息。