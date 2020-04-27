来自：[Swift - RxSwift的使用详解63 (通知NotificationCenter的使用)](https://www.jianshu.com/p/6eec28e17be7)



​    这里所说的通知不是指发给用户看的通知消息，而是系统内部进行消息传递的通知。我在之前也写过一篇文章介绍如何使用 `NotificationCenter` 来发送、接收通知（[点击查看](http://www.hangge.com/blog/cache/detail_828.html)）。

​    其实 `RxSwift` 也对 `NotificationCenter` 进行了扩展，使用起来变得十分简洁，下面通过样例进行演示。



## 一、系统通知的注册与响应

### 1，监听应用进入后台的通知

（1）效果图

程序编译运行后，当按下设备的 `home` 键，程序进入后台的同时会在控制台中输出相关信息。

![img](https:////upload-images.jianshu.io/upload_images/3788243-bded1ef99139306b.png?imageMogr2/auto-orient/strip|imageView2/2/w/298)

（2）样例代码

程序进入后台时除了会执行 `AppDelegate.swift` 里的 `applicationDidEnterBackground` 方法外，还会发送 `UIApplicationDidEnterBackground` 通知，这里我们使用 `NotificationCenter` 的 `Rx` 扩展方法来监听这个通知。

##### 关于 `.takeUntil(self.rx.deallocated)：`
<u> 它的作用是**保证页面销毁的时候自动移除通知注册，避免内存浪费或奔溃。**</u>

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_63ViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        
        //监听应用进入后台通知
        _ = NotificationCenter.default.rx.notification(UIApplication.didEnterBackgroundNotification)
            .takeUntil(self.rx.deallocated) //页面销毁自动移除通知监听
            .subscribe(onNext: { (_) in
                print("程序进入到后台了")
            })
    }
}
```



### 2，监听键盘的通知

（1）效果图

我们分别监听虚拟键盘的打开和关闭通知，并在控制台中输出相关信息。

![img](https://upload-images.jianshu.io/upload_images/3788243-28cdf11cd5bd5102.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https://upload-images.jianshu.io/upload_images/3788243-16acd6b3990d98ae.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https://upload-images.jianshu.io/upload_images/3788243-d27059bf0186409d.png?imageMogr2/auto-orient/strip|imageView2/2/w/251)

（2）样例代码

```swift
import UIKit
import RxSwift
import RxCocoa


class SHRxswift_63ViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //添加文本输入框
        let textField = UITextField(frame: CGRect(x: 20, y: 100, width: 200, height: 30))
        textField.borderStyle = .roundedRect
        textField.returnKeyType = .done
        view.addSubview(textField)
        
        //点击键盘上的完成按钮后，收起键盘
        textField.rx.controlEvent(.editingDidEndOnExit)
            .subscribe(onNext: { (_) in
                //收起键盘
                textField.resignFirstResponder()
            })
            .disposed(by: disposeBag)
        
        //监听键盘弹出通知
        _ = NotificationCenter.default.rx
            .notification(UIApplication.keyboardWillShowNotification)
            .takeUntil(self.rx.deallocated)
            .subscribe(onNext: { (_) in
                print("键盘出现了")
            })
        
        //监听键盘隐藏通知
        _ = NotificationCenter.default.rx
            .notification(UIApplication.keyboardWillHideNotification)
            .takeUntil(self.rx.deallocated)
            .subscribe(onNext: { (_) in
                print("键盘消失了")
            })
        
    }
}
```





## 二、自定义通知的发送与接收

### 1，基本介绍

通知类型其实就是一个字符串，所以我们也可以使用自己定义的通知（同时还可以传递用户自定义数据）。

### 2，样例演示

（1）`ViewController.swift`（我们发出一个携带有自定义数据的通知，同时创建两个观察者来接收这个通知。）

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_63ViewController: UIViewController {

    let disposeBag = DisposeBag()
    
    let observers = [SHRxswift_63MyObserver(name: "观察器1"),SHRxswift_63MyObserver(name: "观察器2")]

    override func viewDidLoad() {
        super.viewDidLoad()
        
        print("发送通知")
        let notificationName = Notification.Name(rawValue: "DownloadImageNotification")
        NotificationCenter.default.post(name: notificationName, object: self, userInfo: ["value1": "univer2012.com", "value2": 123456])
        print("通知完毕")
    }
}
```



（2）`MyObserver.swift`（观察者在收到通知后的执行的处理函数中，添加了个 **3** 秒的等待。）

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_63MyObserver: NSObject {
    
    var name: String = ""
    
    init(name: String) {
        super.init()
        
        self.name = name
        
        //接收通知：
        let notificationName = Notification.Name(rawValue: "DownloadImageNotification")
        _ = NotificationCenter.default.rx
            .notification(notificationName)
            .takeUntil(self.rx.deallocated)
            .subscribe(onNext: { (notification) in
                
                //获取通知数据
                let userInfo = notification.userInfo as! [String: AnyObject]
                let value1 = userInfo["value1"] as! String
                let value2 = userInfo["value2"] as! Int
                
                print("\(name) 获取到通知，用户数据是 [\(value1),\(value2)]")
                sleep(3)
                print("\(name) 执行完毕")
                
            })
    }
}
```



（3）运行结果如下。可以看出，通知发送后的执行是同步的，也就是说观察者全部处理完毕后，主线程才继续往下进行。

![img](https://upload-images.jianshu.io/upload_images/3788243-678cc7afa69f92cc.png?imageMogr2/auto-orient/strip|imageView2/2/w/295)

[RxSwift使用详解系列](https://www.jianshu.com/p/f61a5a988590)
 [原文出自:www.hangge.com转载请保留原文链接](http://www.hangge.com/blog/cache/detail_2042.html)



---

【完】