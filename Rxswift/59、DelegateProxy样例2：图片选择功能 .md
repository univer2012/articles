接下来介绍的同样是 `RxSwift` 的官方样例，演示的是如何对 `UIImagePickerControllerDelegate` 进行 `Rx` 封装，方便我们在`RxSwift`项目中选择图片（可以通过拍照、或者从相簿中选取）

## 三、从本地相册、或摄像头获取图片

### 1，效果图

（1）点击“拍照”按钮，会打开摄像头进行拍照，拍照后自动将照片显示在下方的 `imageView` 中。

![img](https:////upload-images.jianshu.io/upload_images/3788243-c0b518bfec9b46ea.png?imageMogr2/auto-orient/strip|imageView2/2/w/248)



![img](https:////upload-images.jianshu.io/upload_images/3788243-6960d1e00d36dc0f.png?imageMogr2/auto-orient/strip|imageView2/2/w/248)



![img](https:////upload-images.jianshu.io/upload_images/3788243-db64b238efd80af2.png?imageMogr2/auto-orient/strip|imageView2/2/w/248)

（2）而点击“选择照片”或“选择照片并裁剪”按钮后，会打开本地相册选择照片，选择后自动将照片显示在下方的 `imageView` 中。不过后者在选择完毕后还多了个编辑步骤，可以把照片裁剪成正方形再显示。

![img](https:////upload-images.jianshu.io/upload_images/3788243-9b2def4e647a66f3.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-d2812ab0215a0864.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-e63afa70122acbcb.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)

### 2，准备工作

（1）`RxImagePickerDelegateProxy.swift`

首先我们继承 `DelegateProxy` 创建一个关于图片选择的代理委托，同时它还要遵守 `DelegateProxyType`、`UIImagePickerControllerDelegate`、`UINavigationControllerDelegate` 协议。

```swift
import RxSwift
import RxCocoa
import UIKit

public class RxImagePickerDelegateProxy:
    DelegateProxy<UIImagePickerController,UIImagePickerControllerDelegate & UINavigationControllerDelegate>,
    DelegateProxyType,
    UIImagePickerControllerDelegate,
    UINavigationControllerDelegate {
    
    public init(imagePicker: UIImagePickerController) {
        super.init(parentObject: imagePicker, delegateProxy: RxImagePickerDelegateProxy.self)
    }
    
    public static func registerKnownImplementations() {
        self.register{ RxImagePickerDelegateProxy(imagePicker: $0)}
    }
    
    public static func currentDelegate(for object: UIImagePickerController) -> (UIImagePickerControllerDelegate & UINavigationControllerDelegate)? {
        return object.delegate
    }
    
    public static func setCurrentDelegate(_ delegate: (UIImagePickerControllerDelegate & UINavigationControllerDelegate)?, to object: UIImagePickerController) {
        object.delegate = delegate
    }
}
```



（2）`UIImagePickerController+Rx.swift`

接着我们对 `UIImagePickerController` 进行 `Rx` 扩展，作用是将 `UIImagePickerController` 与前面创建的代理委托关联起来，将图片选择相关的 `delegate` 方法转为可观察序列。

注意：下面代码中将 `methodInvoked` 方法替换成 `sentMessage` 其实也可以，它们的区别可以看我的另一篇文章：

- [Swift - RxSwift的使用详解61（sendMessage和methodInvoked的区别）](https://www.jianshu.com/p/0d2875c30083)

```swift
import UIKit
import RxSwift
import RxCocoa

//图片选择控制器（UIImagePcikerController）的Rx扩展
extension Reactive where Base: UIImagePickerController {
    
    //代理委托
    public var pickerDelegate: DelegateProxy<UIImagePickerController, UIImagePickerControllerDelegate & UINavigationControllerDelegate> {
        return RxImagePickerDelegateProxy.proxy(for: base)
    }
    
    //图片选择完毕代理方法的封装
    public var didFinishPickingMediaWithInfo: Observable<[String: AnyObject]> {
        return pickerDelegate.methodInvoked(#selector(UIImagePickerControllerDelegate
            .imagePickerController(_:didFinishPickingMediaWithInfo:)))
            .map{ a in
                return try castOrThrow(Dictionary<String, AnyObject>.self, a[1])
        }
    }
    
    //图片取消选择代理方法的封装
    public var didCancel: Observable<()> {
        return pickerDelegate
            .methodInvoked(#selector(UIImagePickerControllerDelegate
                .imagePickerControllerDidCancel(_:)))
            .map{ _ in () }
    }
}

//转类型的函数（转换失败后，会发出Error）
fileprivate func castOrThrow<T>(_ resultType: T.Type, _  object: Any) throws -> T {
    guard let returnValue = object as? T else {
        throw RxCocoaError.castingError(object: object, targetType: resultType)
    }
    return returnValue
}
```



### 3，使用样例

（1）要获取照片或者进行拍照，首先我们需要在 `info.plist`里加入相关的描述：

-  `Privacy - Camera Usage Description`：App 需要访问您的相机
-  `Privacy - Photo Library Usage Description`：App 需要访问您的照片

![img](https:////upload-images.jianshu.io/upload_images/3788243-7c26e90097b47522.png?imageMogr2/auto-orient/strip|imageView2/2/w/533)

（2）`Main.storyboard`

在 `StoryBoard`中添加 **3** 个 `Button` 以及 **1** 个 `ImageView`，并将它们与代码做 `@IBOutlet` 绑定。

![img](https:////upload-images.jianshu.io/upload_images/3788243-82d839ebc5284b31.png?imageMogr2/auto-orient/strip|imageView2/2/w/333)

（3）`ViewController.swift`

主视图控制器代码如下，可以看到原来图片选择完毕这个代理方法现在已经变成响应式的了。

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_59ViewController: UIViewController {
    let disposeBag = DisposeBag()
    //拍照按钮
    @IBOutlet weak var cameraButton: UIButton!
    
    //选择照片按钮
    @IBOutlet weak var galleryButton: UIButton!
    
    //选择照片并裁减按钮
    @IBOutlet weak var cropButton: UIButton!
    
    //显示照片的imageView
    @IBOutlet weak var imageView: UIImageView!
    
    
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //初始化图片控制器
        let imagePicker = UIImagePickerController()
        
        //判断并决定“z拍照”按钮是否可用
        cameraButton.isEnabled = UIImagePickerController.isSourceTypeAvailable(.camera)
        
        //“拍照”按钮点击
        cameraButton.rx.tap.bind { [weak self] (_) -> Void in
            imagePicker.sourceType = .camera    //来源为相机
            imagePicker.allowsEditing = false   //不可编辑
            //弹出控制器，显示界面
            self?.present(imagePicker, animated: true)
        }.disposed(by: disposeBag)
        
        //“选择照片”按钮点击
        galleryButton.rx.tap.bind { [weak self] (_) -> Void in
            imagePicker.sourceType = .photoLibrary  //来源为相册
            imagePicker.allowsEditing = false       //不可编辑
            //弹出控制器，显示界面
            self?.present(imagePicker, animated: true)
        }.disposed(by: disposeBag)
        
        //“选择照片并裁剪”按钮点击
        cropButton.rx.tap.bind { [weak self] (_) -> Void in
            imagePicker.sourceType = .photoLibrary  //来源为相册
            imagePicker.allowsEditing = true        //不可编辑
            //弹出控制器，显示界面
            self?.present(imagePicker, animated: true)
        }.disposed(by: disposeBag)
        
        //图片选择完毕后，将其绑定到imageView上显示
        imagePicker.rx.didFinishPickingMediaWithInfo
            .map { info in
                if imagePicker.allowsEditing {
                    return info[UIImagePickerController.InfoKey.editedImage] as! UIImage
                    
                } else {
                    return info[UIImagePickerController.InfoKey.originalImage] as! UIImage
                }
        }.bind(to: imageView.rx.image)
        .disposed(by: disposeBag)
        
        
        //图片选择完毕后，退出图片控制器
        imagePicker.rx.didFinishPickingMediaWithInfo
            .subscribe(onNext: { (_) in
                imagePicker.dismiss(animated: true)
            })
            .disposed(by: disposeBag)
        
    }
}
```



## 附：功能改进

虽然前面我们对 `UIImagePickerController` 进行了 `Rx` 扩展，但使用起来还是有些不便，比如图片选择完毕后还需要在代码中手动退出选择器。下面对它做个功能改进，让其可以自动关闭退出。

### 1，UIImagePickerController+RxCreate.swift

这里再一次对 `UIImagePickerController` 进行 `Rx` 扩展，增加一个创建图片选择控制器的静态方法，后面当我们使用该方法初始化 `ImagePickerController` 时会自动将其弹出显示，并且在选择完毕后会自动关闭。

```swift
import UIKit
import RxSwift
import RxCocoa

//取消置顶视图控制器函数
func dismissViewController(_ viewController: UIViewController,animated: Bool) {
    if viewController.isBeingDismissed || viewController.isBeingPresented {
        DispatchQueue.main.async {
            dismissViewController(viewController, animated: animated)
        }
        return
    }
    
    if viewController.presentingViewController != nil {
        viewController.dismiss(animated: animated, completion: nil)
    }
}


//对UIImagePickerController进行Rx扩展
extension Reactive where Base: UIImagePickerController {
    static func createWithParent(_ parent: UIViewController?, animated: Bool = true, configureImagePicker: @escaping (UIImagePickerController) throws -> () = {x in}) -> Observable<UIImagePickerController> {
        
        //返回可观察序列
        return Observable.create { [weak parent] (observer) -> Disposable in
            
            //初始化一个图片选择控制器
            let imagePicker = UIImagePickerController()
            
            //不管图片选择完毕还是取消选择，都会发出.complete事件
            let dismissDisposable = Observable.merge(
                imagePicker.rx.didFinishPickingMediaWithInfo.map{_ in ()},
                imagePicker.rx.didCancel
            ).subscribe(onNext: { (_) in
                observer.on(.completed)
            })
            
            //设置图片选择控制器初始参数参数不准确则发出.error事件
            do {
                try configureImagePicker(imagePicker)
            } catch let error {
                observer.on(.error(error))
                return Disposables.create()
            }
            
            //判断parent是否存在，不存在则发出.completed事件
            guard let parent = parent else {
                observer.on(.completed)
                return Disposables.create()
            }
            
            //弹出控制器，显示界面
            parent.present(imagePicker, animated: animated, completion: nil)
            //发出.next事件（携带的是控制器对象）
            observer.on(.next(imagePicker))
            
            //销毁时自动退出图片控制器
            return Disposables.create(dismissDisposable, Disposables.create {
                dismissViewController(imagePicker, animated: animated)
            })
        }
    }
}
```

### 2，ViewController.swift

主视图控制器代码如下，可以看到我们现在不需要去关心图片选择界面如何关闭了。

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_59ViewController: UIViewController {
    let disposeBag = DisposeBag()
    //拍照按钮
    @IBOutlet weak var cameraButton: UIButton!
    
    //选择照片按钮
    @IBOutlet weak var galleryButton: UIButton!
    
    //选择照片并裁减按钮
    @IBOutlet weak var cropButton: UIButton!
    
    //显示照片的imageView
    @IBOutlet weak var imageView: UIImageView!
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //判断并决定“拍照”按钮是否可用
        cameraButton.isEnabled = UIImagePickerController.isSourceTypeAvailable(.camera)
        
        //"拍照"按钮点击
        cameraButton.rx.tap.flatMapLatest { [weak self] (_) in
            return UIImagePickerController.rx.createWithParent(self) { (picker: UIImagePickerController) in
                picker.sourceType = .camera
                picker.allowsEditing = false
            }.flatMap{ $0.rx.didFinishPickingMediaWithInfo }
        }.map { (info) in
            return info[UIImagePickerController.InfoKey.originalImage] as? UIImage
        }.bind(to: imageView.rx.image)
        .disposed(by: disposeBag)
        
        
        //"选择"按钮点击
        galleryButton.rx.tap.flatMapLatest { [weak self] (_) in
            return UIImagePickerController.rx.createWithParent(self) { (picker: UIImagePickerController) in
                picker.sourceType = .photoLibrary
                picker.allowsEditing = false
            }.flatMap{ $0.rx.didFinishPickingMediaWithInfo }
        }.map { (info) in
            return info[UIImagePickerController.InfoKey.originalImage] as? UIImage
        }.bind(to: imageView.rx.image)
        .disposed(by: disposeBag)
        
        
        //“选择照片并裁剪”按钮点击
        cropButton.rx.tap.flatMapLatest { [weak self] (_) in
            return UIImagePickerController.rx.createWithParent(self) { (picker: UIImagePickerController) in
                picker.sourceType = .photoLibrary
                picker.allowsEditing = true
            }.flatMap{ $0.rx.didFinishPickingMediaWithInfo }
        }.map { (info) in
            return info[UIImagePickerController.InfoKey.editedImage] as? UIImage
        }.bind(to: imageView.rx.image)
        .disposed(by: disposeBag)
    }
}
```

---

【完】

