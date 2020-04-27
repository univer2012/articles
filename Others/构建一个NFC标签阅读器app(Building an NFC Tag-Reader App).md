# 构建一个NFC标签阅读器app(Building an NFC Tag-Reader App)

来自：[Building an NFC Tag-Reader App](https://developer.apple.com/documentation/corenfc/building_an_nfc_tag-reader_app#overview)



在你的app中读取带有NDEF消息的NFC标记。

[下载](https://docs-assets.developer.apple.com/published/3111a8e27d/BuildingAnNFCTagReaderApp.zip)



### 概述

这个示例代码项目(sample code project)展示了如何在应用程序中使用Core NFC来读取类型1到5的近场通信(NFC)标记，这些标记包含NFC数据交换格式(NFC Data Exchange Format ,NDEF)数据。要使用此示例，请下载项目并使用Xcode构建它。在你的iPhone上运行示例app。点击扫描按钮开始扫描标签，然后拿着手机靠近NFC标签(NFC tag)。

要读取标签，示例app创建一个NFC NDEF阅读器会话(reader session)并提供一个委托。正在运行的阅读器会话轮询NFC标记，并在发现包含NDEF消息的标签时调用委托，将消息传递给委托。然后委托存储消息，以便用户稍后查看。



### 配置app来检测NFC标签

通过配置你的app来检测NFC标记，开始构建你的标记阅读器。在project's target的Capabilities选项卡下，打开Near Field Communication Tag Reading(参见[Add a capability to a target](https://help.apple.com/xcode/mac/current/#/dev88ff319e7))。这一步:

* 将NFC标签读取功能添加到App ID中。

* 将[`Near Field Communication Tag Reader Session Formats Entitlement`](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_nfc_readersession_formats)添加到权限文件(entitlements file)中。

  

  接下来，将`NFCReaderUsageDescription`键作为字符串项添加到`Info.plist`文件。对于该值，输入一个字符串，该字符串描述了应用程序需要访问设备的NFC阅读器的原因。如果应用程序试图在不提供此键和字符串的情况下读取标记，则应用程序退出。

### 开始一个阅读器会话

通过调用[`init(delegate:queue:invalidateAfterFirstRead:)`](https://developer.apple.com/documentation/corenfc/nfcndefreadersession/2882064-init)初始化方法来创建一个[`NFCNDEFReaderSession`](https://developer.apple.com/documentation/corenfc/nfcndefreadersession)对象，并传入：

* 阅读器会话(reader session)委托对象。
* 调用委托上的方法时使用的调度队列(dispatch queue)。
* `invalidateAfterFirstRead`标志，用于确定阅读器会话只读取单个标记还是多个标记。



在创建阅读器会话之后，通过设置`alertMessage`属性向用户提供说明。例如，你可以告诉用户，“`Hold your iPhone near the item to learn more about it.`”当手机扫描NFC标签时，系统会向用户显示这条信息。最后，调用`begin()`来启动读取器会话。这使无线频率轮询电话，电话开始扫描标签。

当用户点击Scan按钮时，样例app启动一个阅读器会话。应用程序在读取第一个标签后，配置阅读器会话使会话无效。要读取其他标签，用户需要再次点击扫描按钮。

```swift
@IBAction func beginScanning(_ sender: Any) {
    guard NFCNDEFReaderSession.readingAvailable else {
        let alertController = UIAlertController(
            title: "Scanning Not Supported",
            message: "This device doesn't support tag scanning.",
            preferredStyle: .alert
        )
        alertController.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
        self.present(alertController, animated: true, completion: nil)
        return
    }

    session = NFCNDEFReaderSession(delegate: self, queue: nil, invalidateAfterFirstRead: false)
    session?.alertMessage = "Hold your iPhone near the item to learn more about it."
    session?.begin()
}
```



### 采用阅读器会话委托协议(Reader Session Delegate Protocol)

阅读器会话需要一个符合`NFCNDEFReaderSessionDelegate`协议的委托对象。采用此协议允许委托接收来自阅读器会话的通知，当它:

* 读取NDEF消息
* 由于会话结束或遇到错误而变得无效

```swift
class MessagesTableViewController: UITableViewController, NFCNDEFReaderSessionDelegate {
```



### 读取一个NDEF消息

每次阅读器会话检索一个新的NDEF消息时，会话通过调用`readerSession(_:didDetectNDEFs:)`方法将消息发送给委托。这是app利用数据做一些有用的事情的机会。例如，示例app存储消息，以便用户稍后查看。

```swift
func readerSession(_ session: NFCNDEFReaderSession, didDetectNDEFs messages: [NFCNDEFMessage]) {
    DispatchQueue.main.async {
        // Process detected NFCNDEFMessage objects.
        self.detectedMessages.append(contentsOf: messages)
        self.tableView.reloadData()
    }
}
```



### 处理无效的读取器会话

当阅读器会话结束时，它调用委托方法`readerSession(_:didInvalidateWithError:)`并传递一个`error`对象，该对象给出了结束会话的原因。可能的原因包括:

* 手机成功读取NFC标签，读取器会话被配置为在读取第一个标签后使会话无效。错误代码是`NFCReaderError.Code.readerSessionInvalidationErrorFirstNDEFTagRead`。
* 用户取消了会话，或者调用`invalidate()`来结束会话。错误代码是 `NFCReaderError.Code.readerSessionInvalidationErrorFirstNDEFTagRead`.
* 读取器会话期间发生错误。有关错误代码的完整列表，请参见[`NFCReaderError.Code`](https://developer.apple.com/documentation/corenfc/nfcreadererror/code)。

在样例app中，当阅读器会话出于除在单标记阅读器会话期间读取第一个标记或用户取消会话之外的任何原因结束时，委托将显示一个警报。此外，您不能重用无效的读取器会话，因此示例应用程序将设置`self.session`为`nil`。

```swift
func readerSession(_ session: NFCNDEFReaderSession, didInvalidateWithError error: Error) {
    // Check the invalidation reason from the returned error.
    if let readerError = error as? NFCReaderError {
        // Show an alert when the invalidation reason is not because of a
        // successful read during a single-tag read session, or because the
        // user canceled a multiple-tag read session from the UI or
        // programmatically using the invalidate method call.
        if (readerError.code != .readerSessionInvalidationErrorFirstNDEFTagRead)
            && (readerError.code != .readerSessionInvalidationErrorUserCanceled) {
            let alertController = UIAlertController(
                title: "Session Invalidated",
                message: error.localizedDescription,
                preferredStyle: .alert
            )
            alertController.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
            DispatchQueue.main.async {
                self.present(alertController, animated: true, completion: nil)
            }
        }
    }

    // To read new tags, a new session instance is required.
    self.session = nil
}
```



### 写一个NDEF消息

要向标记写入内容，样例app将启动一个新的读取会话。此会话必须处于活动状态，才能向标记写入NDEF消息，因此这一次`invalidateAfterFirstRead`设置为`false`，以防止在读取标记后会话无效。

```swift
@IBAction func beginWrite(_ sender: Any) {
    session = NFCNDEFReaderSession(delegate: self, queue: nil, invalidateAfterFirstRead: false)
    session?.alertMessage = "Hold your iPhone near an NDEF tag to write the message."
    session?.begin()
}
```

当读取会话检测到一个标记时，它调用`readerSession(_:didDetectNDEFs:)`委托方法。但是，由于在读取第一个标记之后会话不会变得无效，因此会话可以检测到多个标记。**示例应用程序只写入一个标记，因此它检查会话是否只检测到一个标记**。如果会话检测到多个标记，应用程序会要求用户删除标记，然后重新开始轮询，以扫描新的标记。

After the app confirms that it has only one tag, it connects to the tag  and verifies that it’s writable. The app then writes _the NDEF message it read earlier_ to the tag.

在应用程序确认它只有一个标签之后，它连接到标签并验证它是可写的。然后，应用程序将之前读取的NDEF消息写入标记。

```swift
func readerSession(_ session: NFCNDEFReaderSession, didDetect tags: [NFCNDEFTag]) {
    if tags.count > 1 {
        // Restart polling in 500 milliseconds.
        let retryInterval = DispatchTimeInterval.milliseconds(500)
        session.alertMessage = "More than 1 tag is detected. Please remove all tags and try again."
        DispatchQueue.global().asyncAfter(deadline: .now() + retryInterval, execute: {
            session.restartPolling()
        })
        return
    }
    
    // Connect to the found tag and write an NDEF message to it.
    let tag = tags.first!
    session.connect(to: tag, completionHandler: { (error: Error?) in
        if nil != error {
            session.alertMessage = "Unable to connect to tag."
            session.invalidate()
            return
        }
        
        tag.queryNDEFStatus(completionHandler: { (ndefStatus: NFCNDEFStatus, capacity: Int, error: Error?) in
            guard error == nil else {
                session.alertMessage = "Unable to query the NDEF status of tag."
                session.invalidate()
                return
            }

            switch ndefStatus {
            case .notSupported:
                session.alertMessage = "Tag is not NDEF compliant."
                session.invalidate()
            case .readOnly:
                session.alertMessage = "Tag is read only."
                session.invalidate()
            case .readWrite:
                tag.writeNDEF(self.message, completionHandler: { (error: Error?) in
                    if nil != error {
                        session.alertMessage = "Write NDEF message fail: \(error!)"
                    } else {
                        session.alertMessage = "Write NDEF message successful."
                    }
                    session.invalidate()
                })
            @unknown default:
                session.alertMessage = "Unknown NDEF tag status."
                session.invalidate()
            }
        })
    })
}
```

### 支持背景标签读取

要了解如何设置你的app来处理iOS在后台读取的标记，请参见 [Adding Support for Background Tag Reading](https://developer.apple.com/documentation/corenfc/adding_support_for_background_tag_reading)。