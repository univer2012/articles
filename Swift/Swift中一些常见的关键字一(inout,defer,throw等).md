来自：[Swift中一些常见的关键字一(inout,defer,throw等)](https://www.jianshu.com/p/0b43a0b5bfd6)



本文将 `inout`,`defer`,`throws`,`rethrows`这几个关键字进行介绍。



### inout

`inout`关键字用一句话概括：**将值类型的对象用引用的方式传递。**
 我们经常说`值类型`和`引用类型`实际上就是指对象的传递方式分别是 **按值传递** 和 **按址传递**。

 值类型 按值传递：

```swift
let a: Int = 5
var b: Int = a
b = b + 1
print(a, b)
///打印：
///5 6
```

引用类型（class对象） 按址传递

```swift
let p1 = Person()
p1.name = "name1"
let p2 = p1
p2.name = "name2"
print(p1.name, p2.name)
///打印：
///name2 name2
```



而<u>`inout`关键字则可以使得值类型的对象和引用类型的对象一样，以按址传递的方式进行操作，这里的操作不仅包含上述例子中的赋值操作，也包含了函数的参数传递行为</u>。



 通俗的举个例子：我们在使用函数传递一个`Int`类型对象的时候，通常会将这个对象的值传递进去了。但是如果使用inout修饰对象的类型，则可以将变量的地址传入函数。就像下面这个handle函数一样。

```swift
func test() {
    var a: Int = 5
    handle(a: &a)   //注意这里使用了取地址操作
    print(a)        //output: 6
}

func handle(a: inout Int) {
    print(a)        //output: 5
    a = a + 1       // 如果没有inout修饰的话，这句代码将会报错，主要意思是不能改变一个let修饰的常量
}

test()
```

最终，我们在test 函数中打印的变量`a`的值被改变了。
 除了 `Int`类型，诸如：CGFloat,Bool,Character,Array,struct等，这些值类型的对象都可以使用inout修饰，达到使用引用的方式传递的目的。



### defer

`defer`关键字用一句话概括：**修饰一段函数内任一段代码，使其必须在函数中的其余代码都执行完毕，函数即将结束前调用**。也可以理解成将延迟执行。
 为了更好的了解`defer`的作用，我写了一段代码：

```swift
func test() {
    print("函数开始")
    defer {
        print("执行defer1")
    }
    print("函数将结束")
    defer {
        print("执行defer2")
    }
}
test()
/*打印：
 函数开始
 函数将结束
 执行defer2
 执行defer1   */
```

我在test函数中，添加了两个 `defer`，分别打印了两条语句。从下面的打印中，可以看到，两个`defer`都执行了。并且他们都在`函数结束`之后执行的:**`defer`总在函数要结束之前调用，尽管我将它们放在了程序较靠前的位置**。

<u> defer2先于defer1执行，说明如果一个函数中包含数个`defer` 的话：**程序会按照自下而上的顺序执行`defer`**。</u>



 以上是在一个同步的代码中运行的`defer`，那如果是异步执行的代码`defer`的作用又是怎样的呢？

```swift
func test() {
    print("函数开始")
    defer {
        print("执行defer1")
    }
    
    defer {
        print("执行defer2")
    }
    
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        print("异步执行完毕")
    }
    print("函数将结束")
}
test()
/*打印：
 函数开始
 函数将结束
 执行defer2
 执行defer1
 异步执行完毕   */
```



**异步代码的执行，不会影响`defer`的执行时间。**事实上，`defer`的执行只和其所在的作用域有关，如果作用域即将被回收，那么会在回收之前执行`defer`。



### throws

Swift要求的严格的类型安全，我想它对于错误的处理也不太可能马虎吧。回想在OC中，使用`NSError`处理错误，而实际上`NSError`处理的错误有限，我们不太可能在日常所有的开发中使用`NSError`。



通常，对于一个需要进行错误处理的时候，我们习惯性使用一个 `nil`作为参数，就像这样：

```objc
[[NSFileManager defaultManager] contentsOfDirectoryAtURL:@"aURL" 
includingPropertiesForKeys:@"aKey" options:nil error:nil];
```

因为类似的错误出现的情况是很极端的场景，大多数时候使用nil是没有问题的。但是，这是一个隐患，不仅不能回馈错误，还增加了维护的成本：至少我得知道这个函数究竟在哪些极端情况下才会返回错误！ 而这一切是因为程序员的懒或者马虎造成的。



Swift决定从技术上解决这个问题：使用`throw`可以让程序员必须处理相关的异常代码！
 Swift中提供了`Error`协议，我们在开发中，如果要自定义自己的错误类型，一般会使用一个`Enum`来继承`Error`协议，目的是享用`Error`已经包含的一些特性。
 下面是一个例子：

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



throws的使用很简单，只需要在可能出现异常的函数或者方法后面添加throws。经过这个关键字修饰的函数，在调用的时候，需要程序员加上do-catch来调用。

对于错误类型开发者来说，只需要使用Throws进行修饰，就保证了以后的调用者必然需要对相应的错误进行处理（当然也可以不处理，但无论如何，错误被throw携带出来了，以后的维护和优化不需要重新做错误处理的设计，直接加上错误处理的逻辑即可）。





### rethrows

rethrows是异常往上传递的关键字。上面说了throws用在可能出现异常的函数或者方法中，而<u>rethrows针对的不是函数或者方法的本身，而是它携带的闭包类型的参数，当它的闭包类型的参数throws的时候，我们要使用rethrows继续将这个异常往上传递， 直到被调用者使用到</u>。这相比throws多一个传递的环节。

还是同样，使用一个简单的例子来看一看：

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

