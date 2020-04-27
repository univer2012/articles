## 1.public和open的区别
答：
来自：[private、fileprivate、internal、public和open的区别](https://www.jianshu.com/p/2e320f27afda)

在Swift语言中，访问修饰符有五种，分别为fileprivate，private，internal，public和open，其中 fileprivate和open是Swift 3新添加的。由于过去 Swift对于访问权限的控制是基于文件的，不是基于类的。这样会有问题，所以Swift 3新增了两个修饰符对原来的private、public进行细分。

### private

 private所修饰的属性或者方法只能在当前类里访问

 private所修饰类只能在当前.swift文件里访问



### fileprivate 

fileprivate访问级别所修饰的属性或者方法在当前的Swift源文件里可以访问。

###  internal（默认访问级别，internal修饰符可写可不写）

internal访问级别所修饰的属性或方法，在源代码所在的整个模块都可以访问、被继承、被重写

如果是框架或者库代码，则在整个框架内部都可以访问，框架由外部代码所引用时，则不可以访问。即使使用import，也会提示错误：

> No such module '...'

如果是App代码，也是在整个App代码，也是在整个App内部可以访问。

### public

可以被任何类访问。<font color=#FF0000>但其他module中不可以被override和继承，而在module内可以被override和继承。</font>

> Cannot inherit from non-open class '...' outside of its defining module

# open

可以被任何类使用，包括override和继承。

访问权限排序从高到低排序：open>public>interal > fileprivate >private

### 总结

属性和方法的访问控制通俗的分级顺序应该是：

> 当前类（private）、当前swift文件（fileprivate）、 当前模块（internal）、其它模块（open、public）
>
> 属性和方法的修饰：在当前模块internal、open、public是同一级别，在外模块open、public是同一级别

类的访问级别:

> 当前类（private）、当前swift文件（fileprivate）、 当前模块（internal）、其它模块(open需要import)、是否可被继承被重写(public需要import)
>
> 可以发现cocopod导入的第三方库，作为外模块一般都是使用的open、public修饰类，来提供给项目使用





## 2.throws和rethrows的用法与作用，try?和try!是什么意思

来自：

1.[Swift中一些常见的关键字一(inout,defer,throw等)](https://www.jianshu.com/p/0b43a0b5bfd6) 

2.[Swift学习记录 -- Swift中throws处理方式](https://www.jianshu.com/p/d407ae190569)



throws的使用很简单，只需要<u>在可能出现异常的函数或者方法后面添加throws。经过这个关键字修饰的函数，在调用的时候，需要程序员加上do-catch来调用。</u>

对于错误类型开发者来说，只需要使用Throws进行修饰，就保证了以后的调用者必然需要对相应的错误进行处理（当然也可以不处理，但无论如何，错误被throw携带出来了，以后的维护和优化不需要重新做错误处理的设计，直接加上错误处理的逻辑即可）。



用法：在可能出现异常的函数或者方法后面添加throws。经过这个关键字修饰的函数，在调用的时候，需要程序员加上do-catch来调用。

作用：错误被throw携带出来了，以后的维护和优化不需要重新做错误处理的设计，直接加上错误处理的逻辑即可。保证以后的调用者必然需要对相应的错误进行处理，

代码如下：

```swift
//错误类型枚举
enum MyError : Error {
    case ErrorOne
    case ErrorTwo
    case ErrorThree
    case ErrorOther
}

func willThrow(_ type:NSInteger)throws -> NSString{
    print("开始处理错误")
    if type == 1 {
        throw MyError.ErrorOne
    }else if type == 2 {
        throw MyError.ErrorTwo
    }else if type == 3{
        throw MyError.ErrorThree
    }else if type == 4{
        throw MyError.ErrorOther
    }
    return "一切都很好"
}
    
//调用
do {
    let str = try willThrow(2)
    //以下是非错误时的代码
    print(str) //如果有错误出现，这里将不会执行
}catch let err as MyError{
    print(err)
}catch{
    //这里必须要携带一个空的catch 不然会报错。 原因是可能遗漏
}
/*打印：
 开始处理错误
 ErrorTwo   */
```





rethrows是异常往上传递的关键字。<u>rethrows针对的不是函数或者方法的本身，而是它携带的闭包类型的参数，当它的闭包类型的参数throws的时候，我们要使用rethrows继续将这个异常往上传递， 直到被调用者使用到</u>。这相比throws多一个传递的环节。

用法：rethrows针对的不是函数或者方法的本身，而是它携带的闭包类型的参数，当它的闭包类型的参数throws的时候，我们要使用rethrows继续将这个异常往上传递， 直到被调用者使用到。

```swift
func willRethrow(_ throwCall: (Int) throws -> String) rethrows
```



作用：使用rethrows继续将这个异常往上传递， 直到被调用者使用到。

代码如下：

```swift
//错误类型枚举
enum MyError : Error {
    case ErrorOne
    case ErrorTwo
    case ErrorThree
    case ErrorOther
}

func willThrow(_ type:Int)throws -> String{
    print("开始处理错误")
    if type == 1 {
        throw MyError.ErrorOne
    }else if type == 2 {
        throw MyError.ErrorTwo
    }else if type == 3{
        throw MyError.ErrorThree
    }else if type == 4{
        throw MyError.ErrorOther
    }
    return "一切都很好"
}
func willRethrow(_ throwCall: (Int) throws -> String) rethrows {
    do {
        let result = try throwCall(2)
        print(result)
    } catch let err as MyError {
        throw err       //这里进行了 再次throw
    } catch {
        
    }
}

//MARK:调用
let afunc = willThrow
do {
    try willRethrow(afunc)
} catch let err as MyError {
    print("rethrows ",err)
} catch {
    
}
/*打印：
 开始处理错误
 rethrows  ErrorTwo  */
```

代码中看到，`willRethrow`本身并不对错误进行处理，原因是它本身并不会差生错误。另外的，它的参数`throwCall`进行了错误的处理，`willRethrow`对`throwCall`的错误进行再次`throw`。

 简单来说，`rethorws`就是`throws`的传递，也即是对于throw的一个层次级别的应用。我们甚至可以进行多级传递，但是会导致代码过于复杂，不建议这么做。



`try`:  程序员手动捕捉异常

```swift
// try方式
do {
    try String.init(contentsOfFile: htmlPate!, encoding: String.Encoding.utf8)
} catch {
    //抛出异常
    print(error)
}
```



`try?`方式:  系统帮助我们处理异常 ,如果该方法出现了异常, 则方法返回nil ,如果没有异常,则返回对应的对象

```swift
//安全校验
guard let htmlCont = try? String.init(contentsOfFile: htmlPate!, encoding: String.Encoding.utf8) else {
    return
}
```

`try!`方式：  直接告诉系统,该方法没有异常,如果该方法出现了错误,直接崩溃

```swift
let htmlCont = try! String.init(contentsOfFile: htmlPate!, encoding: String.Encoding.utf8)
```





## 3.Self的使用场景，associatetype的作用

来自：[swift 中 Self 与self](https://www.jianshu.com/p/a6bcdebd83f5)

在定义协议的时候Self 用的频率很高，比如rx。`Self` 不仅指代的是 实现该协议的类型本身，也包括了这个类型的子类。

示例1：

```swift
protocol MProtocolTest01 {

    // 协议定一个方法，接受实现该协议的自身类型并返回一个同样的类型
    func testMethod(c: Self) -> Self

    //不能在协议中定义 范型 进行限制
    //Self 不仅指代的是 实现该协议的类型本身，也包括了这个类型的子类
}
```

示例2：模拟copy 方法

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



关联类型为协议中的某个类型提供了一个占位名（或者说别名），其代表的实际类型在协议被采纳时才会被指定。你可以通过 `associatedtype` 关键字来指定关联类型。比如使用协议声明更新cell的方法：

```swift
//模型
struct Model {
    let age: Int
}

//协议，使用关联类型
protocol TableViewCell {
    associatedtype T
    func updateCell(_ data: T)
}

//遵守TableViewCell,需要实现 `typealias T = ...` 和 `updateCell`方法
class MyTableViewCell: UITableViewCell, TableViewCell {
    typealias T = Model
    func updateCell(_ data: Model) {
        // do something ...
    }
    
}
```



## 4.map、filter、reduce的作用，map与flatmap的区别

来自：1.[Swift中 Map,Flatmap,Filter,Reduce的用法](https://www.jianshu.com/p/b5de33ec83f5)

2.[Swift：Map，FlatMap，Filter，Reduce指南](https://www.jianshu.com/p/87b97dfbf17b)



1. `map`:map方法作用是把数组[T]通过闭包函数把每一个数组中的元素变成U类型的值，最后组成数组[U]。定义如下：

   `func map(transform: (T) -> U) -> [U]`

   ```swift
   @inlinable public func map<T>(_ transform: (Element) throws -> T) rethrows -> [T]
   ```

   

2. `filter`就是筛选的功能，参数是一个用来判断是否筛除的筛选闭包，根据闭包函数返回的Bool值来过滤值。为True则加入到结果数组中。定义如下：
    `func filter(includeElement: (T) -> Bool) -> [T]`

   ```swift
   @inlinable public func filter<T>(_ isIncluded: (Element) throws -> Bool) rethrows -> [T]
   ```

   

3. `reduce`的作用给定一个类型为U的初始值，把数组[T]中每一个元素传入到combine的闭包函数里面，通过计算得到最终类型为U的结果值。定义如下：
    `func reduce(initial: U, combine: (U, T) -> U) -> U`

   ```swift
   @inlinable public func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Self.Element) throws -> Result) rethrows -> Result
   ```



**map与flatmap的区别**:

本质上，相比 `map`，`flatMap`也就是在可选值层做了一个解包。

```swift
var value: String? = "1"
var result = value.map{ Int($0)}
print(result)
///打印：
///Optional(Optional(1))
```

```swift
var value: String? = "1"
var result = value.flatMap{ Int($0)}
print(result)
///打印：
///Optional(1)
```

使用flatMap就可以在链式调用时，不用做额外的解包工作：

```swift
var value: String? = "1"
var result = value.flatMap{ Int($0)}.map { $0 * 2 }
print(result)
///打印：
///Optional(2)
```



`flatMap`对[SequenceType](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/protocol/SequenceType/)的两个作用：

##### 一：压平

```swift
var values = [[1,3,5,7],[9]]
let flattenResults = values.flatMap{ $0 }
print(flattenResults)
///打印：
///[1, 3, 5, 7, 9]
```

##### 二：空值过滤

```swift
var values: [Int?] = [1,3,5,7,9,nil,10]
let flattenResults = values.flatMap{ $0 }
print(flattenResults)
///打印：
///[1, 3, 5, 7, 9, 10]
```



## 5.GCD与NSOperationQueue有哪些异同

来自：[iOS面试题01-多线程网络（★★★）](https://www.jianshu.com/p/3f6108b28598)

1>GCD是纯C语言的API，NSOperationQueue是基于GCD的OC版本封装
 2>GCD只支持FIFO的队列，NSOperationQueue可以很方便地调整执行顺
 序、设置最大并发数量
 3>NSOperationQueue可以在轻松在Operation间设置依赖关系，而GCD
 需要写很多的代码才能实现
 4>NSOperationQueue支持KVO，可以监测operation是否正在执行
 （isExecuted）、是否结束（isFinished），是否取消（isCanceld）
 5>GCD的执行速度比NSOperationQueue快
 任务之间不太互相依赖：GCD
 任务之间有依赖\或者要监听任务的执行情况：NSOperationQueue



## 6.图解MVC，MVVM架构

来自：[iOS 架构模式--解密 MVC，MVP，MVVM以及VIPER架构](https://www.cnblogs.com/oc-bowen/p/6255475.html)



[传统的MVC模式](https://en.wikipedia.org/wiki/Model–view–controller):

![](http://cc.cocimg.com/api/uploads/20160107/1452152223816977.png)

**“传统的MVC架构不适用于当下的iOS开发”**

**苹果推荐的MVC--****愿景**

![](http://cc.cocimg.com/api/uploads/20160107/1452152304301101.png)

**苹果推荐的MVC--****事实**



![](http://cc.cocimg.com/api/uploads/20160107/1452152425723031.png)

**“就开发速度而言，Cocoa MVC是最好的架构选择方案。”**

**MVP 实现了Cocoa的MVC的愿景**

![](http://cc.cocimg.com/api/uploads/20160107/1452152673569726.png)



**MVVM--最新且是最伟大的MV(X)系列的一员**

![](http://cc.cocimg.com/api/uploads/20160107/1452153249521047.png)

**“MVVM很诱人，因为它集合了上述方法的优点，并且由于在View层的绑定，它并不需要其他附加的代码来更新View，尽管这样，可测试性依然很强。”**

**VIPER--把LEGO建筑经验迁移到iOS app的设计**

![](http://cc.cocimg.com/api/uploads/20160107/1452153564205784.png)




## 7.如何提升tableview的流畅度

## 8.图解一下TCP发起连接和断开连接的过程。

来自：

1.[iOS面试题01-多线程网络（★★★）](https://www.jianshu.com/p/3f6108b28598)

2.[IOS-基于CocoaAsyncSocket的服务端的监听（二）](https://www.jianshu.com/p/efc169a1c8f9)

Socket通信流程:

![Socket通信流程](https://upload-images.jianshu.io/upload_images/4418111-551f70ee84474cb6.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/903)





###### 十二、什么是 TCP连接的三次握手

第一次握手：客户端发送 syn 包(syn=j)到服务器，并进入 SYN_SEND 状态，等待服务器确认；
 第二次握手：服务器收到 syn 包，必须确认客户的 SYN（ack=j+1），同时自己也发送一个 SYN 包（syn=k），即 SYN+ACK 包，此时服务器进入 SYN_RECV 状态；
 第三次握手：客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入 ESTABLISHED 状态，完成三次握手。
 握手过程中传送的包里不包含数据，三次握手完毕后，客户端与服务器才正式开始传送数据。理想状态下，TCP 连接一旦建立，在通信双方中的任何一方主动关闭连接之前，TCP   连接都将被一直保持下去。断开连接时服务器和客户端均可以主动发起断开 TCP 连接的请求，断开过程需要经过“四次握手”（过程就不细写了，就是服务器和客户端交互，最终确定断开）



![](https://upload-images.jianshu.io/upload_images/3788243-4830cacee1f1e7d3.png?imageMogr2/auto-orient/strip|imageView2/2/w/875)