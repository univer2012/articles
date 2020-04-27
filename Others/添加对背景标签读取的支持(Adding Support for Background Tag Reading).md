# 添加对背景标签读取的支持(Adding Support for Background Tag Reading)

来自：[Adding Support for Background Tag Reading](https://developer.apple.com/documentation/corenfc/adding_support_for_background_tag_reading)

允许用户扫描NFC标签，而无需app使用背景标签读取(background tag reading)。



### 概述

在支持后台标签读取的iphone上，系统扫描和读取NFC数据，而不需要用户使用app扫描标签。每次读取新标签时，系统都会显示一个弹出式通知。用户点击通知后，系统将标签数据发送到相应的应用程序。如果iPhone处于锁定状态，系统会提示用户在向应用程序提供标签数据之前先解锁手机。

> 注意
>
> iPhone XS及以后的版本支持背景标签读取。



为了避免无意的标签读取，系统只在用户使用iPhone时才读取后台的标签。另外，要注意有时显示是打开的，背景标签无法读取，比如:

* 该设备从未解锁过。
* 一个核心的NFC阅读器会话正在进行中。
* Apple Pay钱包正在使用中。
* 照相机正在使用中。
* 启用飞行模式。



### 扫描标签的过程

当设备在后台标签读取模式下扫描NFC标签后，系统通过查找具有以下属性值的`NFCNDEFPayload`对象，来检查标签的NDEF消息，以获取URI记录:

* [`typeNameFormat`](https://developer.apple.com/documentation/corenfc/nfcndefpayload/2875566-typenameformat)等于[`NFCTypeNameFormat.nfcWellKnown`](https://developer.apple.com/documentation/corenfc/nfctypenameformat/nfcwellknown)

* `type`类型等于“U”

**如果NDEF消息包含多个URI记录，则系统将使用第一个记录。_URI记录必须包含通用链接(universal link)或受支持的URL模式(URL scheme)_。**



### 使用通用链接

对于通用链接，在用户点击通知后，系统会启动与通用链接相关联的app，或将与通用链接相关联的app带到前台。系统将NDEF消息作为`NSUserActivity`对象发送给app。如果没有安装与通用链接相关的app，系统将在Safari中打开该链接。

### 使用URL模式

The system processes *NDEF payloads containing a URI for a URL scheme* in  the same way as universal links. The system displays a notification  after reading the tag. When the user taps the notification, the system  launches the app that supports the URL scheme. 

系统以与通用链接相同的方式，来处理NDEF有效负载(NDEF payloads)，该有效负载(NDEF payloads)包含作为URL模式的URI。系统在读取标签后显示通知。当用户点击通知时，系统会启动支持URL模式的app。

后台标签读取支持以下URL模式(URL schemes):

| URL Scheme               | Example                                                      |
| ------------------------ | ------------------------------------------------------------ |
| Website URL (HTTP/HTTPS) | https://www.example.com                                      |
| Email                    | mailto:user@example.com                                      |
| SMS                      | sms:+14085551212                                             |
| Telephone                | tel:+14085551212                                             |
| FaceTime                 | facetime://user@example.com                                  |
| FaceTime Audio           | facetime-audio://user@example.com                            |
| Maps                     | http://maps.apple.com/?address=Apple%20Park,Cupertino,California |
| HomeKit Accessory Setup  | X-HM://12345                                                 |

有关URL模式的更多信息，请参见[Apple URL Scheme Reference](https://developer.apple.com/library/archive/featuredarticles/iPhoneURLScheme_Reference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007899)。



> 注意
>
> 背景标签读取不支持自定义URL模式。使用通用链接代替。



### 配置你的app

通过打开在项目的Capabilities选项卡下的Associated Domains，来给app添加背景标签读取支持。此步骤将[`Associated Domains Entitlement`](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains)添加到你的项目权限文件(your project's entitlement file)和app ID中。接下来，为你的app支持的每个通用链接(universal link)输入域(domain)。



![图1 为每个受支持的通用链接添加域](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Adding_Support_for_Background_Tag_Reading1.png)



### 处理标签/标记交付(Tag Delivery)

要处理从标签中读取的NDEF消息，请在你的app委托(app delegate)中实现`application(_:continue:restorationHandler:)`方法。系统调用此方法将标签数据传递(deliver)到`NSUserActivity`对象中的应用程序。该用户活动(user activity)有一个`NSUserActivityTypeBrowsingWeb`的`activityType`枚举，标签数据在`ndefMessagePayload`属性中可用。

对于不是由背景标记读取(background tag reading)生成的用户活动(user activity)，`ndefMessagePayload`将返回一条消息，该消息仅包含一个`NFCNDEFPayload`记录。该记录有一个`NFCTypeNameFormat.empty`的`typeNameFormat`枚举。



清单1 处理从后台读取的标签中的数据

```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([Any]?) -> Void) -> Bool {

    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb else {
        return false
    }

    // Confirm that the NSUserActivity object contains a valid NDEF message.
    let ndefMessage = userActivity.ndefMessagePayload
    guard ndefMessage.records.count > 0,
        ndefMessage.records[0].typeNameFormat != .empty else {
            return false
    }

    // Send the message to `MessagesTableViewController` for processing.
    guard let navigationController = window?.rootViewController as? UINavigationController else {
        return false
    }

    navigationController.popToRootViewController(animated: true)
    let messageTableViewController = navigationController.topViewController as? MessagesTableViewController
    messageTableViewController?.addMessage(fromUserActivity: ndefMessage)

    return true
}
```

并不是所有的设备都支持后台标签读取，所以一定要为用户提供直接从app读取标签的选项。有关更多信息，请参见 [Building an NFC Tag-Reader App](https://developer.apple.com/documentation/corenfc/building_an_nfc_tag-reader_app)。

