# 从你的iPhone创建NFC标签(Creating NFC Tags from Your iPhone)

来自：[Creating NFC Tags from Your iPhone](https://developer.apple.com/documentation/corenfc/creating_nfc_tags_from_your_iphone)

将数据保存到标签中，并使用本机标签协议(native tag protocols)与它们交互。



[下载](https://docs-assets.developer.apple.com/published/f6dc555515/CreatingNFCTagsFromYourIPhone.zip)



### 概述

> 注意
>
> 此示例代码项目与WWDC 2019会话(WWDC 2019 session)  [715: Core NFC Enhancements](https://developer.apple.com/videos/play/wwdc19/715)  相关。
>
> 单词：Enhancements  [in'ha:nsmənts]  n. 增强，增强功能（enhancement的复数形式）



#  2、[NFCTagReaderSession](https://developer.apple.com/documentation/corenfc/nfctagreadersession)

用于检测ISO7816、ISO15693、FeliCa和MIFARE标签的阅读器会话(reader session)。



### 宣言

```swift
class NFCTagReaderSession : NFCReaderSession
```

### 概述

使用`NFCTagReaderSession`与`NFCTagType`中列出的标记类型之一进行交互。要使用此阅读器会话，你必须:

* 在你的app中包含[`Near Field Communication Tag Reader Session Formats Entitlement`](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_nfc_readersession_formats)。
* 在你的app的`info.plist`文件中，为`NFCReaderUsageDescription`键提供非空字符串。

要与ISO 7816标签进行交互，请将app中支持的应用程序标识符(application identifiers)列表添加到`com.apple.developer.nfc.readersession.iso7816.select-identifier information`属性列表键。如果您包含应用程序标识符`D2760000850101`—MIFARE DESFire tags (NFC Forum T4T tag platform)上NDEF应用程序的标识符—并且阅读器会话发现一个与此标识符匹配的标签，它将向委托发送一个`NFCISO7816Tag`标记对象。要得到作为`NFCMiFareTag`对象的MIFARE DESFire标签，请不要在应用程序标识符列表中包含`D2760000850101`。

系统中每次只能有一个任何类型的阅读器会话是活动的。系统将添加的会话放入队列中，并按先进先出(FIFO)顺序处理它们。



> 重要
>
> Core NFC不支持与支付相关的应用程序id。

