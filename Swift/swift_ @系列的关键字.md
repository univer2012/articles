## 一、`@discardableResult`

如果一个函数有返回值，调用的时候也必须接收返回值，否则报错；

加上 `@discardableResult` 可以避免这个报错；

也可以使用 `_` 通配符来接收；

```swift
///表示取消不使用返回值的警告
@discardableResult
    func isLogin(jump: Bool, success: Closure = nil, failed: Closure = nil) -> Bool
```



## 二、`@available`

Swift 2.0 中，引入了可用性的概念。对于函数，类，协议等，可以使用`@available`声明这些类型的生命周期依赖于特定的平台和操作系统版本。而`#available`用在判断语句中（if, guard, while等），在不同的平台上做不同的逻辑

### @available

用法
 @available放在函数（方法），类或者协议前面。表明这些类型适用的平台和操作系统。看下面一个例子：

```swift
@available(iOS 9, *)
func myMethod() {
    // do something
}
```

@available(iOS 9, *)必须包含至少2个特性参数，其中iOS 9表示必须在 iOS 9 版本以上才可用。如果你部署的平台包括 iOS 8 , 在调用此方法后，编译器会报错。
 另外一个特性参数：星号（*），表示包含了所有平台，目前有以下几个平台：
 iOS
 iOSApplicationExtension
 OSX
 OSXApplicationExtension
 watchOS
 watchOSApplicationExtension
 tvOS
 tvOSApplicationExtension
 一般来讲，如果没有特殊的情况，都使用 `*` 表示全平台。

 @available(iOS 9, *)是一种简写形式。全写形式是@available(iOS,  introduced=9.0)。

introduced=9.0参数表示指定平台(iOS)从 9.0  开始引入该声明。为什么可以采用简写形式呢？当只有introduced这样一种参数时，就可以简写成以上简写形式。同理：`@available(iOS 8.0, OSX 10.10, *)` 这样也是可以的。表示同时在多个平台上（iOS 8.0 及其以上；OSX 10.10及其以上）的可用性。

 另外，`@available` 还有其他一些参数可以使用，分别是：

- deprecated=版本号：从指定平台某个版本开始过期该声明
- obsoleted=版本号：从指定平台某个版本开始废弃（注意弃用的区别，deprecated是还可以继续使用，只不过是不推荐了，obsoleted是调用就会编译错误）该声明
- message=信息内容：给出一些附加信息
- unavailable：指定平台上是无效的
- renamed=新名字：重命名声明

```swift
@available(iOS, introduced: 6.0, deprecated: 8.0, message: "Manually forward viewWillTransitionToSize:withTransitionCoordinator: if necessary")
open func shouldAutomaticallyForwardRotationMethods() -> Bool
```



来自：[swift @available与#available](https://www.jianshu.com/p/fb9b0d91482f)



## 三、`@convention(block)`



我们经常将一个函数作为参数传入另一个函数。
 那么在iOS上能作为一个函数参数的东西有哪些呢

1. c的函数指针
2. oc的block
3. swift的闭包(closures)

ok回归正题，先说下 `@convention` 是干什么的。

他是用来修饰闭包的。他后面需要跟一个参数：

1. `@convention(swift)`  : 表明这个是一个swift的闭包
2. `@convention(block)` ：表明这个是一个兼容oc的block的闭包
3. `@convention(c)`   : 表明这个是兼容c的函数指针的闭包。



```swift
class Person:NSObject {

    func doAction(action: @convention(swift) (String)->Void, arg:String){
        action(arg)
    }
}

let saySomething_c : @convention(c) (String)->Void = {
    print("i said: \($0)")
}

let saySomething_oc : @convention(block) (String)->Void = {
    print("i said: \($0)")
}

let saySomething_swift : @convention(swift) (String)->Void = {
    print("i said: \($0)")
}

let person = Person()
person.doAction(action: saySomething_c, arg: "helloworld")
person.doAction(action: saySomething_oc, arg: "helloworld")
person.doAction(action: saySomething_swift, arg: "helloworld")
```



为啥今天要写这个呢？因为我在用runtime的 `imp_implementationWithBlock` 这个函数时不知道咋传参数。我用swift的闭包怎么都不对，看完 `@convention` 之后就知道该怎么办了。

```swift
class Person:NSObject {
    //数  数字 
    dynamic func countNumber(toValue:Int){
        for value in 0...toValue{
            print(value)
        }
    }
}
//现在我们要替换数数函数的实现，给他之前和之后加上点广告语。

//拿到method
let methond = class_getInstanceMethod(Person.self, #selector(Person.countNumber(toValue:)))
//通过method拿到imp， imp实际上就是一个函数指针
let oldImp = method_getImplementation(methond!)
//由于IMP是函数指针，所以接收时需要指定@convention(c)
typealias Imp  = @convention(c) (Person,Selector,NSNumber)->Void
//将函数指针强转为兼容函数指针的闭包
let oldImpBlock = unsafeBitCast(oldImp!, to: Imp.self)

//imp_implementationWithBlock的参数需要的是一个oc的block，所以需要指定convention(block)
let newFunc:@convention(block) (Person, NSNumber)->Void = {
    (sself,  toValue) in
    print("数之前， 祝大家新年快乐")
    oldImpBlock(sself, #selector(Person.countNumber(toValue:)), toValue)
    print("数之后， 祝大家新年快乐")
}


let imp = imp_implementationWithBlock(unsafeBitCast(newFunc, to: AnyObject.self))


method_setImplementation(methond!, imp)

let person = Person()
person.countNumber(toValue: 50)
/** output:
 数之前， 祝大家新年快乐
 0
 1
 2
 ... ...
 ... ...
 49
 50
 数之后， 祝大家新年快乐
 
*/
```



来自：[swift的@convention](https://www.jianshu.com/p/f4dd6397ae86)