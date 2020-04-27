委托（`delegate`） iOS 开发中十分常见。不管是使用系统自带的库，还是一些第三方组件时，我们总能看到 `delegate` 的身影。使用 `delegate` 可以实现代码的松耦合，减少代码复杂度。但如果我们项目中使用 `RxSwift`，那么原先的 `delegate` 方式与我们链式编程方式就不相称了。

  解决办法就是将代理方法进行一层 `Rx` 封装，这样做不仅会减少许多不必要的工作（比如原先需要遵守不同的代理，并且要实现相应的代理方法），还会使得代码的聚合度更高，更加符合响应式编程的规范。

  其实在 RxCocoa 源码中我们也可以发现，它已经对标准的 `Cocoa` 做了大量的封装（比如 `tableView` 的 `itemSelected`）。下面我将通过样例演示如何将代理方法进行 `Rx` 化。

## 一、对 Delegate进行Rx封装原理

### 1，DelegateProxy

（1）`DelegateProxy` 是代理委托，我们可以将它看作是代理的代理。

（2）`DelegateProxy` 的作用是做为一个中间代理，他会先把系统的 `delegate` 对象保存一份，然后拦截 `delegate` 的方法。也就是说在每次触发 `delegate` 方法之前，会先调用 `DelegateProxy` 这边对应的方法，我们可以在这里发射序列给多个订阅者。

### 2，流程图

这里以 `UIScrollView` 为例，`Delegate proxy` 便是其代理委托，它遵守 `DelegateProxyType` 与 `UIScrollViewDelegate`，并能响应 `UIScrollViewDelegate` 的代理方法，这里我们可以为代理委托设计它所要响应的方法（即为订阅者发送观察序列）。

```xml
/***
  
 +-------------------------------------------+
 |                                           |
 | UIView subclass (UIScrollView)            |
 |                                           |
 +-----------+-------------------------------+
             |
             | Delegate
             |
             |
 +-----------v-------------------------------+
 |                                           |
 | Delegate proxy : DelegateProxyType        +-----+---->  Observable<T1>
 |                , UIScrollViewDelegate     |     |
 +-----------+-------------------------------+     +---->  Observable<T2>
             |                                     |
             |                                     +---->  Observable<T3>
             |                                     |
             | forwards events                     |
             | to custom delegate                  |
             |                                     v
 +-----------v-------------------------------+
 |                                           |
 | Custom delegate (UIScrollViewDelegate)    |
 |                                           |
 +-------------------------------------------+
 
 **/
```

## 二、获取地理定位信息样例

这个是 `RxSwift` 的一个官方样例，演示的是如何对 `CLLocationManagerDelegate` 进行 `Rx` 封装。

### 1，效果图

（1）第一次运行时会申请定位权限，如果当前`App`可以使用定位信息时，界面上会实时更新显示当前的经纬度。

![img](https://upload-images.jianshu.io/upload_images/3788243-45d29c514bb1f019.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)



![img](https://upload-images.jianshu.io/upload_images/3788243-e6abf7ad590b2f3a.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)

（2）如果当前 `App` 被禁止使用定位信息，界面上会出现一个提示按钮，点击后会自动跳转到系统权限设置页面。

![img](https://upload-images.jianshu.io/upload_images/3788243-e16cbfc420d25f6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)



![img](https://upload-images.jianshu.io/upload_images/3788243-4967a6deabfdb0a4.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)

### 2，准备工作

（1）`RxCLLocationManagerDelegateProxy.swift`

首先我们继承 `DelegateProxy` 创建一个关于定位服务的代理委托，同时它还要遵守 `DelegateProxyType` 和 `CLLocationManagerDelegate` 协议。

```swift
import CoreLocation
import RxSwift
import RxCocoa

extension CLLocationManager: HasDelegate {
    public typealias Delegate = CLLocationManagerDelegate
}

public class RxCLLocationManagerDelegateProxy:
    DelegateProxy<CLLocationManager, CLLocationManagerDelegate>,
    DelegateProxyType,
CLLocationManagerDelegate {
    
    public init(locationManager: CLLocationManager) {
        
        super.init(parentObject: locationManager, delegateProxy: RxCLLocationManagerDelegateProxy.self)
        
    }
    
    public static func registerKnownImplementations() {
        self.register { (parent) -> RxCLLocationManagerDelegateProxy in
            RxCLLocationManagerDelegateProxy(locationManager: parent)
        }
    }
    
    internal lazy var didUpdateLocationsSubject = PublishSubject<[CLLocation]>()
    internal lazy var didFailWithErrorSubject = PublishSubject<Error>()
    
    public func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        
        _forwardToDelegate?.locationManager(manager, didUpdateLocations: locations)
        didUpdateLocationsSubject.onNext(locations)
    }
    
    public func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        
        _forwardToDelegate?.locationManager(manager, didFailWithError: error)
        didFailWithErrorSubject.onNext(error)
    }
    
    deinit {
        self.didUpdateLocationsSubject.on(.completed)
        self.didFailWithErrorSubject.on(.completed)
    }
    
}
```



（2）`CLLocationManager+Rx.swift`

接着我们对 `CLLocationManager` 进行`Rx` 扩展，作用是将`CLLocationManager`与前面创建的代理委托关联起来，将定位相关的 `delegate` 方法转为可观察序列。

注意：下面代码中将 `methodInvoked`方法替换成 `sentMessage` 其实也可以，它们的区别可以看另一篇文章：

- [Swift - RxSwift的使用详解61（sendMessage和methodInvoked的区别）](https://www.jianshu.com/p/0d2875c30083)

```swift
import CoreLocation
import RxSwift
import RxCocoa

extension Reactive where Base: CLLocationManager {
    
    /**
     Reactive wrapper for `delegate`.
     
     For more information take a look at `DelegateProxyType` protocol documentation.
     */
    public var delegate: DelegateProxy<CLLocationManager, CLLocationManagerDelegate> {
        
        return RxCLLocationManagerDelegateProxy.proxy(for: base)
    }
    
    // MARK: Responding to Location Events
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didUpdateLocations: Observable<[CLLocation]> {
        
        return RxCLLocationManagerDelegateProxy.proxy(for: base)
            .didUpdateLocationsSubject.asObserver()
    }
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didFailWithError: Observable<Error> {
        return RxCLLocationManagerDelegateProxy.proxy(for: base)
            .didFailWithErrorSubject.asObserver()
    }
//    #if ox(iOS) || os(macOS)
    
    public var didFinishDeferredUpdatesWithError: Observable<Error?> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didFinishDeferredUpdatesWithError:)))
            .map{ a in
                return try castOptionalOrThrow(Error.self, a[1])
        }
    }
    
//    #endif
    
    #if os(iOS)
    // MARK: Pausing Location Updates
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didPauseLocationUpdates: Observable<Void> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManagerDidPauseLocationUpdates(_:)))
            .map{ _ in
                return ()
        }
    }
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didResumeLocationUpdates: Observable<Void> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManagerDidResumeLocationUpdates(_:)))
            .map{_ in
                return ()
        }
    }
    
    // MARK: Responding to Heading Events
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didUpdateHeading: Observable<CLHeading> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didUpdateHeading:)))
            .map{ a in
                return try castOrThrow(CLHeading.self, a[1])
        }
    }
    
    // MARK: Responding to Region Events
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didEnterRegion: Observable<CLRegion> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didEnterRegion:)))
            .map{ a in
                return try castOrThrow(CLRegion.self, a[1])
        }
    }
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didExitRegion: Observable<CLRegion> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didExitRegion:)))
            .map{ a in
                return try castOrThrow(CLRegion.self, a[1])
        }
    }
    #endif
    
    #if os(iOS) || os(macOS)
    
    /**
     Reactive wrapper for `delegate` message.
     */
    @available(OSX 10.10, *)
    public var didDetermineStateForRegion: Observable<(state: CLRegionState, region: CLRegion)> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didDetermineState:for:)))
            .map{ a in
                let stateNumber = try castOrThrow(NSNumber.self, a[1])
                let state = CLRegionState(rawValue: stateNumber.intValue) ?? CLRegionState.unknown
                let region = try castOrThrow(CLRegion.self, a[2])
                return (state: state, region: region)
        }
    }
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var monitoringDidFailForRegionWithError: Observable<(region: CLRegion?, error: Error)> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:monitoringDidFailFor:withError:)))
            .map{ a in
                let region = try castOptionalOrThrow(CLRegion.self, a[1])
                let error = try castOrThrow(Error.self, a[2])
                return (region: region, error: error)
        }
    }
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didStartMonitoringForRegion: Observable<CLRegion> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didStartMonitoringFor:)))
            .map{ a in
                return try castOrThrow(CLRegion.self, a[1])
        }
    }
    
    #endif
    
    #if os(iOS)
    
    // MARK: Responding to Ranging Events
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didRangeBeaconsInRegion: Observable<(beacons: [CLBeacon], region: CLBeaconRegion)> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didRangeBeacons:in:)))
            .map{ a in
                let beacons = try castOrThrow([CLBeacon].self, a[1])
                let region = try castOrThrow(CLBeaconRegion.self, a[2])
                return (beacons: beacons, region: region)
        }
    }
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var rangingBeaconsDidFailForRegionWithError: Observable<(region: CLBeaconRegion, error: Error)> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:rangingBeaconsDidFailFor:withError:)))
            .map{ a in
                let region = try castOrThrow(CLBeaconRegion.self, a[1])
                let error = try castOrThrow(Error.self, a[2])
                return (region: region, error: error)
        }
    }
    
    // MARK: Responding to Visit Events
    
    /**
     Reactive wrapper for `delegate` message.
     */
    @available(iOS 8.0, *)
    public var didVisit: Observable<CLVisit> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didVisit:)))
            .map{ a in
                return try castOrThrow(CLVisit.self, a[1])
        }
    }
    
    #endif
    
    // MARK: Responding to Authorization Changes
    
    /**
     Reactive wrapper for `delegate` message.
     */
    public var didChangeAuthorizationStatus: Observable<CLAuthorizationStatus> {
        return delegate.methodInvoked(#selector(CLLocationManagerDelegate
            .locationManager(_:didChangeAuthorization:)))
            .map{ a in
                let number = try castOrThrow(NSNumber.self, a[1])
                return CLAuthorizationStatus(rawValue: Int32(number.intValue)) ?? .notDetermined
        }
    }
    
}

fileprivate func castOrThrow<T>(_ resultType: T.Type, _ object: Any) throws -> T {
    guard let returnValue = object as? T else {
        throw RxCocoaError.castingError(object: object, targetType: resultType)
    }
    
    return returnValue
}

fileprivate func castOptionalOrThrow<T>(_ resultType: T.Type, _ object: Any) throws -> T? {
    if NSNull().isEqual(object) {
        return nil
    }
    
    guard let returnValue = object as? T else {
        throw RxCocoaError.castingError(object: object, targetType: resultType)
    }
    
    return returnValue
}
```



（3）`GeolocationService.swift`

虽然现在我们已经可以直接 `CLLocationManager` 的 `rx` 扩展方法获取位置信息了。但为了更加方便使用，我们这里对 `CLLocationManager` 再次进行封装，定义一个地理定位的 `service` 层，作用如下：

- 自动申请定位权限，以及授权判断。
- 自动开启定位服务更新。
- 自动实现经纬度数据的转换。

```swift
import CoreLocation
import RxSwift
import RxCocoa

//地理定位服务层
class GeolocationService {
    //单例模式
    static let instance = GeolocationService()
    
    //定位权限序列
    private (set) var authorized: Driver<Bool>
    
    //经纬度信息序列
    private (set) var location: Driver<CLLocationCoordinate2D>
    
    //定位管理器
    private let locationManager = CLLocationManager()
    
    private init() {
        
        //更新距离
        locationManager.distanceFilter = kCLDistanceFilterNone
        
        //设置定位精度
        locationManager.desiredAccuracy = kCLLocationAccuracyBestForNavigation
        
        //获取定位权限序列
        authorized = Observable.deferred{ [weak locationManager] in
            let status = CLLocationManager.authorizationStatus()
            guard let locationManager = locationManager else {
                return Observable.just(status)
            }
            return locationManager
                .rx.didChangeAuthorizationStatus
                .startWith(status)
        }
        .asDriver(onErrorJustReturn: CLAuthorizationStatus.notDetermined)
            .map{
                switch $0 {
                case .authorizedAlways:
                    return true
                default:
                    return false
                }
        }
        
        //获取经纬度信息序列
        location = locationManager.rx.didUpdateLocations
        .asDriver(onErrorJustReturn: [])
            .flatMap{
                return $0.last.map(Driver.just) ?? Driver.empty()
        }
            .map{ $0.coordinate }
        
        //发送授权申请
        locationManager.requestAlwaysAuthorization()
        //允许使用定位服务的话，开启定位服务更新
        locationManager.startUpdatingLocation()
    }
}
```



### 3，使用样例

（1）要获取定位信息，首先我们需要在 `info.plist` 里加入相关的定位描述：

-  `Privacy - Location Always and When In Use Usage Description`：我们需要通过您的地理位置信息获取您周边的相关数据
-  `Privacy - Location When In Use Usage Description`：我们需要通过您的地理位置信息获取您周边的相关数据

![img](https:////upload-images.jianshu.io/upload_images/3788243-fac8f36fa5a1661b.png?imageMogr2/auto-orient/strip|imageView2/2/w/753)

（2）Main.storyboard

在 `StoryBoard` 中添加一个`Label` 和 `Button`，分别用来显示经纬度信息，以及没有权限时的提示。并将它们与代码做 `@IBOutlet` 绑定。

![img](https:////upload-images.jianshu.io/upload_images/3788243-fe995128b8a9bed2.png?imageMogr2/auto-orient/strip|imageView2/2/w/325)

（3）UILabel+Rx.swift

为了能让 `Label` 直接绑定显示经纬度信息，这里对其做个扩展。

```swift
//UILabel的Rx扩展
import RxSwift
import RxCocoa

import CoreLocation

extension Reactive where Base: UILabel {
    var coordinates: Binder<CLLocationCoordinate2D> {
        return Binder(base) {label, location in
            label.text = "经度: \(location.longitude)\n纬度: \(location.latitude)"
        }
    }
}
```

（4）ViewController.swift
 主视图控制器代码如下，可以看到我们获取定位信息变得十分简单。

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_58ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    @IBOutlet weak private var button: UIButton!
    
    @IBOutlet weak var label: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //获取地理定位服务
        let geolocationService = GeolocationService.instance
        
        //定位权限绑定到按钮上（是否可见）
        geolocationService.authorized
        .drive(button.rx.isHidden)
        .disposed(by: disposeBag)
        
        //经纬度信息绑定到label上显示
        geolocationService.location
        .drive(label.rx.coordinates)
        .disposed(by: disposeBag)
        
        //按钮点击
        button.rx.tap
            .bind { [weak self](_) -> Void in
                self?.openAppPreferences()
            }
            .disposed(by: disposeBag)
        
        
    }
    
    //跳转到应有偏好的设置页面
    private func openAppPreferences() {
        UIApplication.shared.open(URL(string: UIApplication.openSettingsURLString)!)
    }
    
}
```

> 注意：
>
> 如果用模拟器运行，定位的经纬度没有显示出来，需要选中模拟器 -->Debug --> Location --> 选择一个选项进行设置。默认是`none`。

---

【完】