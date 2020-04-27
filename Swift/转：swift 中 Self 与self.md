来自：[swift 中 Self 与self](https://www.jianshu.com/p/a6bcdebd83f5)

##### 前提

在swift 开发过程中，尤其是第三方库中，我们多次看到首字母大写的Self，很多时候不明白其中意思，Self 与self 又有何区别呢？

今天在这里简单解释一下：

##### self

self 大家都熟悉，在UIViewController 或者UIView 里面，我们通常用到`self.view`,`self.navigationController` 以及`self.属性`,这是跟OC里面的用法都差不多，我这里解释的是`.self`

> `.self`可以用在类型后面取得类型本身，也可以用在实例后面取得这个实例本身

- **在swift 中有这么一个定义**

  > public typealias AnyClass = AnyObject.Type

  

  * AnyClass 在 Swift 中被一个typealias 所定义，通过 AnyObject.Type 这种方式得到的是一个`元类型（Meta）`*

  具体我们来看实例：

```swift
A.Type //代表的是A这个类型的类型【也就是类 类型】，也就是说声明一个元类型来存储A这个类型本身 

let typeA: A.Type = A.self //用在类型后面取得的是 类型本身,用来获得一个表示该类型的值 

let obj = A().self //用在实例后面取得的是 这个实例本身，用来拿到实例本身

```



```swift
 class A {
    class func method() {
        print("hello")
    }
  
    func objMehtod() {
        print("world")
    }
 }


A.method()
typeA.method()
      
//或者
let anyClass: AnyClass = A.self
(anyClass as! A.Type).method()
      
obj.objMehtod()
```



##### `.self` 使用的场景

```swift
class MusicViewController: UIViewController {
  
}

class AbumViewController: UIViewController {
  
}

//调用
//元类使用的意义
let usingVCTypes: [AnyClass] = [MusicViewController.self ,AbumViewController.self]
      
func setupViewController(_ vcTypes: [AnyClass]) {
    for vctype in vcTypes {
        if vctype is UIViewController.Type {
              let vc = (vctype as! UIViewController.Type).init()
                  print(vc)
              }
          }
      }
      
setupViewController(usingVCTypes)

```



##### Self

在定义协议的时候Self 用的频率很高，比如rx

> Self 不仅指代的是 实现该协议的类型本身，也包括了这个类型的子类

- *我们来看一个例子*

```swift
protocol MProtocolTest01 {

    // 协议定一个方法，接受实现该协议的自身类型并返回一个同样的类型
    func testMethod(c: Self) -> Self

    //不能在协议中定义 范型 进行限制
    //Self 不仅指代的是 实现该协议的类型本身，也包括了这个类型的子类
}
```



* *实例：模拟copy 方法*

```swift
protocol Copyable {
    func copy() -> Self
}

class MMyClass: Copyable {
    var num = 1
    func copy() -> Self {
        let result = type(of: self).init()
        result.num  = num
        return result
    }

    //必须实现
    //如果不实现：Constructing an object of class type 'Self' with a metatype value must use a 'required' initializer。错误
    required init() {
    }
}
```



##### 拓展视野

```swift
/**
 *  如需自定义命名空间，只需需修改MajqNamespaceProtocal协议中的关联属性 majq 即可
 */
import Foundation
import UIKit
//MARK: - 仿命名空间
// 定义一个protocol -> NamespaceWrappable,含有关联属性： hand【关联类型(HandWrapperType)】
public protocol MajqNamespaceProtocal {
    
    //关联类型
    associatedtype HandWrapperType
    
    //关联属性: 实例属性
    var majq: HandWrapperType { get }
    
    //关联属性： 类 属性
    static var majq: HandWrapperType.Type { get }
    
}

//扩展协议 NamespaceWrappable
public extension MajqNamespaceProtocal {
    //实现协议： 属性majq类型为：MajqNamespaceStruct<Self>，get 方法 返回NamespaceWrapper<Self> 实例对象
    //btn.majq 返回值类型为：MajqNamespaceStruct<UIButton> 具体看：https://www.jianshu.com/p/5059d2993509
    var majq: MajqNamespaceStruct<Self> {
        return MajqNamespaceStruct(value: self)
    }

    //类方法
    static var majq: MajqNamespaceStruct<Self>.Type {
        return MajqNamespaceStruct.self
    }

    
}


//结构体：NamespaceWrapper【结构体能用于 范型 类型】
public struct MajqNamespaceStruct<T> {
    
    //属性：wrappedValue 类型为范型T
    public let wrappedValue: T
    
    //初始化方法，参数类型为范型T
    public init(value: T) {
        self.wrappedValue = value
    }
}


//MARK: - 用法

//NSObject类实现这个协议（任何一个NSObject 类都可以含有属性 majq）
///已默认 NSObject 类及其子类实现，需要添加其他的方法使用 where 匹配模式扩展
///如下面： mcApplyAppearance 方法
extension NSObject: MajqNamespaceProtocal {
    
}

//String 实现这个MajqNamespaceProtocal协议
extension String: MajqNamespaceProtocal {
    
}

//为 String拓展一个方法
extension MajqNamespaceStruct where T == String {
    var majqTest: String {
        return wrappedValue
    }
    
}


//由于 namespace 相当于将原来的值做了封装,所以如果在写扩展方法时需要用到原来的值,就不能再使用self,而应该使用wrappedValue
public extension MajqNamespaceStruct where T: UIView {
    
    /// 默认的 UI 控件外形设置通用方法
    ///
    /// - Parameter settings: 一个包含外形设置代码的闭包: 参数为范型T，返回值Void 
    /// - Returns: UIView自身实例
    @discardableResult
    public func mcApplyAppearance(_ settings: (_ v: T) -> Void) -> T {
        settings(wrappedValue)  //wrappedValue 是协议 MajqNamespaceStruct 本身的属性，可以直接使用
        return wrappedValue
    }
    
    
}
```



---

【完】