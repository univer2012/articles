来自：[Swift - RxSwift的使用详解60（DelegateProxy样例3：应用生命周期的状态变化）](https://www.jianshu.com/p/5c941464bdf8)



我们知道 `UIApplicationDelegate` 协议中定义了关于程序启动各个过程的回调，比如：

- `applicationWillResignActive` 方法：在应用从活动状态进入非活动状态的时候会被调用（比如电话来了）。

- `applicationWillTerminate`方法：在应用终止的时候会被调用。

  过去我们通常都是在 `AppDelegate.swift` 里的相关回调方法中编写相应的业务逻辑。但一旦功能复杂些，这里就会变得十分混乱难以维护。而且有时想在其它模块中使用这些回调也不容易。

  本文演示如何通过对 `UIApplication` 进行 `Rx` 扩展，利用 `RxSwift` 的 `DelegateProxy` 实现 `UIApplicationDelegate` 相关回调方法的封装。从而让 `UIApplicationDelegate` 回调可以在任何模块中都可随时调用。

## 四、监测应用生命周期的状态变化

### 1，准备工作

（1）`RxUIApplicationDelegateProxy.swift`

首先我们继承 `DelegateProxy` 创建一个关于应用生命周期变化的代理委托，同时它还要遵守 `DelegateProxyType`、`UIApplicationDelegate` 协议。

```swift
import UIKit
import RxSwift
import RxCocoa

public class RxUIApplicationDelegateProxy:
    DelegateProxy<UIApplication, UIApplicationDelegate>,
    UIApplicationDelegate, DelegateProxyType {
    
    public weak private(set) var application: UIApplication?
    
    init(application: ParentObject) {
        
        self.application = application
        super.init(parentObject: application, delegateProxy: RxUIApplicationDelegateProxy.self)
    }
    
    public static func registerKnownImplementations() {
        self.register{ RxUIApplicationDelegateProxy(application: $0) }
    }
    
    public static func currentDelegate(for object: UIApplication) -> UIApplicationDelegate? {
        return object.delegate
    }
    
    public static func setCurrentDelegate(_ delegate: UIApplicationDelegate?, to object: UIApplication) {
        
        object.delegate = delegate
    }
    
    public override func setForwardToDelegate(_ delegate: UIApplicationDelegate?, retainDelegate: Bool) {
        
        super.setForwardToDelegate(delegate, retainDelegate: true)
    }
    
}
```



（2）`UIApplication+Rx.swift`

接着我们对 `UIApplication` 进行 `Rx` 扩展，作用是将 `UIApplication` 与前面创建的代理委托关联起来，将状态变化相关的 `delegate` 方法转为可观察序列。

注意1：我们在开头自定义了一个表示应用状态枚举（`AppState`），不使用系统自带的的 `UIApplicationState` 是因为后者没有 `terminated`（终止）这个状态。

注意2：下面代码中将 `methodInvoked` 方法替换成 `sentMessage` 其实也可以，它们的区别可以看另一篇文章：

- [Swift - RxSwift的使用详解61（sendMessage和methodInvoked的区别）](https://www.jianshu.com/p/0d2875c30083)

```swift
import UIKit
import RxSwift
import RxCocoa

public enum AppState {
    case active
    case inactive
    case background
    case terminated
}

//扩展
extension UIApplication.State {
    func toAppState() -> AppState {
        switch self {
        case .active:
            return .active
        case .inactive:
            return .inactive
        case .background:
            return .background
        default:
            return .terminated
        }
    }
}

extension Reactive where Base: UIApplication {
    
    //代理委托
    var delegate: DelegateProxy<UIApplication, UIApplicationDelegate> {
        return RxUIApplicationDelegateProxy.proxy(for: base)
    }
    
    //应用重新回到活动状态
    var didBecomActive: Observable<AppState> {
        return delegate.methodInvoked(#selector(UIApplicationDelegate.applicationDidBecomeActive(_:)))
            .map{ _ in return .active}
    }
    
    //应用从活动状态进入非活动状态
    var willResignActive: Observable<AppState> {
        return delegate.methodInvoked(#selector(UIApplicationDelegate.applicationWillResignActive(_:)))
            .map{ _ in return .inactive}
    }
    
    //应用从后台恢复至前台（还不是活动状态）
    var willEnterForeground: Observable<AppState> {
        return delegate.methodInvoked(#selector(UIApplicationDelegate.applicationWillEnterForeground(_:)))
            .map{ _ in return .inactive }
    }
    
    //应用进入到后台
    var didEnterBackground: Observable<AppState> {
        return delegate.methodInvoked(#selector(UIApplicationDelegate.applicationDidEnterBackground(_:)))
            .map{ _ in return .background }
    }
    
    //应用终止
    var willTerminate: Observable<AppState> {
        return delegate.methodInvoked(#selector(UIApplicationDelegate.applicationWillTerminate(_:)))
            .map{ _ in return .terminated }
    }
    
    var state: Observable<AppState> {
        return Observable.of(
            didBecomActive,
            willResignActive,
            willEnterForeground,
            didEnterBackground,
            willTerminate
        )
        .merge()
            .startWith(base.applicationState.toAppState())  //为了让开始订阅是就能获取到当前状态
    }
}
```



### 2，使用样例1

（1）我们可以对各个状态变化行为分别进行订阅：

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_60ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //应用重新回到活动状态
        UIApplication.shared.rx
            .didBecomActive
            .subscribe(onNext: { (_) in
                print("应用进入活动状态")
            })
            .disposed(by: disposeBag)
        
        //应用从活动状态进入非活动状态
        UIApplication.shared.rx
            .willResignActive
            .subscribe(onNext: { (_) in
                print("应用从活动状态进入非活动状态")
            })
            .disposed(by: disposeBag)
        
        //应用从后台恢复至前台（还不是活动状态）
        UIApplication.shared.rx
            .willEnterForeground
            .subscribe(onNext: { (_) in
                print("应用从后台恢复至前台（还不是活动状态）")
            })
            .disposed(by: disposeBag)
        
        //应用进入到后台
        UIApplication.shared.rx
            .didEnterBackground
            .subscribe(onNext: { (_) in
                print("应用进入到后台")
            })
            .disposed(by: disposeBag)
        
        //应用终止
        UIApplication.shared.rx
            .willTerminate
            .subscribe(onNext: { (_) in
                print("应用终止")
            })
            .disposed(by: disposeBag)
        
        
    }

}
```

（2）我们进行在如下一系列操作后，控制台里内容如下：

- 编译运行程序
- 按下 `home` 键程序进入后台
- 双击 `home` 键，选择程序并回到前台

![img](https:////upload-images.jianshu.io/upload_images/3788243-1c207b6904dee55c.png?imageMogr2/auto-orient/strip|imageView2/2/w/231)



![img](https:////upload-images.jianshu.io/upload_images/3788243-46236ba4a9820b15.png?imageMogr2/auto-orient/strip|imageView2/2/w/232)



![img](https:////upload-images.jianshu.io/upload_images/3788243-52cdae58ba7cd742.png?imageMogr2/auto-orient/strip|imageView2/2/w/232)

### 3，使用样例2

（1）我们也可以对状态变化序列进行订阅：

```swift
import UIKit
import RxSwift
import RxCocoa


class SHRxswift_60ViewController: UIViewController {
    let disposeBag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        //应用重新回到活动状态
        UIApplication.shared.rx
            .state
            .subscribe(onNext: { (state) in
                switch state {
                case .active:
                    print("应用进入活动状态")
                case .inactive:
                    print("应用进入非活动状态")
                case .background:
                    print("应用进入到后台")
                case .terminated:
                    print("应用终止。")
                }
            })
        .disposed(by: disposeBag)
        
    }
}
```



（2）我们进行在如下一系列操作后，控制台里内容如下：

- 编译运行程序
- 按下 `home` 键程序进入后台
- 双击 `home` 键，选择程序并回到前台

![img](https:////upload-images.jianshu.io/upload_images/3788243-0585cfe2d19e8cac.png?imageMogr2/auto-orient/strip|imageView2/2/w/232)



![img](https:////upload-images.jianshu.io/upload_images/3788243-1db5f8ffab5f8832.png?imageMogr2/auto-orient/strip|imageView2/2/w/252)



![img](https:////upload-images.jianshu.io/upload_images/3788243-95e0792ae65b3f41.png?imageMogr2/auto-orient/strip|imageView2/2/w/230)



[RxSwift使用详解系列](https://www.jianshu.com/p/f61a5a988590)
 [原文出自:www.hangge.com转载请保留原文链接](http://www.hangge.com/blog/cache/detail_2055.html)



---

【完】