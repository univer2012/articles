# 1 苹果的推送机制(APNS)

先来看一张苹果官方对其推送做出解释的概要图。

[![img](https://univer2012.github.io/2017/12/05/35实现App消息推送/20141114171953345.jpeg)](https://univer2012.github.io/2017/12/05/35实现App消息推送/20141114171953345.jpeg)

Provider是给你手机应用发出推送消息的服务器，而APNS（Apple Push Notification Service）则是苹果消息推送服务器。

你本地的服务器当需要给应用推送一条消息的时候，先要将消息发出到苹果推送服务器，然后再由苹果推送服务器将消息发到安装了该应用的手机。

[![img](https://univer2012.github.io/2017/12/05/35实现App消息推送/20141114173252426.jpeg)](https://univer2012.github.io/2017/12/05/35实现App消息推送/20141114173252426.jpeg)

解释下上图的逻辑：

1. 你的IOS应用需要去注册APNS消息推送功能。
2. 当苹果APNS推送服收到来自你应用的注册消息就会返回一串device token给你（很重要）
3. 将应用收到的device Token传给你本地的Push服务器。
4. 当你需要为应用推送消息的时候，你本地的推送服务器会将消息，以及Device Token打包发送到苹果的APNS服务器。
5. APNS再将消息推送给目的iphone。

# 2 制作CSR证书

## 1 从证书颁发机构颁发证书

打开你mac的钥匙串访问： [![钥匙串](https://univer2012.github.io/2017/12/05/35实现App消息推送/钥匙串.png)](https://univer2012.github.io/2017/12/05/35实现App消息推送/钥匙串.png)钥匙串 然后点击钥匙串访问。
[![钥匙串](https://univer2012.github.io/2017/12/05/35实现App消息推送/pic2.png)](https://univer2012.github.io/2017/12/05/35实现App消息推送/pic2.png)钥匙串
[![钥匙串](https://univer2012.github.io/2017/12/05/35实现App消息推送/20141117095900875.png)](https://univer2012.github.io/2017/12/05/35实现App消息推送/20141117095900875.png)钥匙串

随后它会弹出一个窗口用户电子邮件信息就填写你苹果开发者账号的名称即可（应该是一个邮件名称），点击保存到磁盘的选
项，点击继续，显示如下：