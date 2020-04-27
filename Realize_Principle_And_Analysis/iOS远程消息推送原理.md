参考：
1. [iOS远程消息推送原理](https://www.jianshu.com/p/2595dfc5e7cd)


### 1. 什么是远程消息推送？
APNs：Apple Push Notification server 苹果推送通知服务
苹果的APNs允许设备和苹果的推送通知服务器保持连接，支持开发者推送消息给用户设备对应的应用程序。

### 2. 常见用途

常常用于消息的订阅

1、 电商：我有新品发布啦！

我的某某产品在搞活动，五折优惠！

2、 新闻媒体：今天又有新鲜事发生了！

3、 社交：某某给你留言了！

某某对你的文章发表评论了！

### 3. 实现消息推送的步骤
1. 注册：为应用程序申请消息推送服务。此时你的设备会向APNs服务器发送注册请求。
2. APNs服务器接受请求，并将deviceToken返给你设备上的应用程序
3. 客户端应用程序将deviceToken发送给后台服务器程序，后台接收并储存。
4. 后台服务器向APNs服务器发送推送消息
5. APNs服务器将消息发给deviceToken对应设备上的应用程序

### 4. 消息推送原理

![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E8%BF%9C%E7%A8%8B%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E5%8E%9F%E7%90%86_1.png)

### 5. UIApplication 与 UIApplicationDelegate
`UIApplication`的核心作用是提供iOS程序运行期间的控制和协作工作。

`UIApplication`的实例会被赋予一个代理对象（`UIApplicationDelegate`），以处理应用程序的生命周期事件，系统事件。

### 6. 远程消息注册
![2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E8%BF%9C%E7%A8%8B%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E5%8E%9F%E7%90%86_2.png)

#### 1. 注册成功
![3](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E8%BF%9C%E7%A8%8B%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E5%8E%9F%E7%90%86_3.png)

1. 注册成功之后会弹出提示框征求用户的同意
2. 当用户选择允许之后会在这个方法里取得设备的deviceToken，然后发送给服务器
3. ==测试环境与发布环境所连接的服务器地址是不同的，所获取到的deviceToken值也是不同的。deviceToken与应用无关。==

#### 2. 注册失败
![4](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E8%BF%9C%E7%A8%8B%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E5%8E%9F%E7%90%86_4.png)

#### 3. 收到远程消息
![5](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E8%BF%9C%E7%A8%8B%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E5%8E%9F%E7%90%86_5.png)

想要收到推送消息，就必须要有后台服务器向APNs的服务器发请求。
1. 公司自己开发后台服务器程序
2. 采用第三方的后台服务程序，比如：百度云推送、极光推送、友盟推送
![6](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E8%BF%9C%E7%A8%8B%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E5%8E%9F%E7%90%86_6.png)