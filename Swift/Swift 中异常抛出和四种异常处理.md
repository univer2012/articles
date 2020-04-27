来自：[Swift 中异常抛出和四种异常处理](https://blog.csdn.net/qq_30970529/article/details/52766601)



在Swift中你可以像其他语言一样抛出异常处理异常，今天我们就详细地说说Swift中的异常抛出和处理。 
 在一开始我们要定义错误或者说是异常，Swift中的一些简单异常可以使用枚举定义，注意这个枚举要继承一个空协议Error,如下代码：

```swift
enum OperationErro: Error {
    case ErrorOne
    case ErrorTwo
    case ErrorThree(String)
    case ErrorOther
}
```

这里定义了一个异常值的枚举，接下来我们再写个函数来使用这些异常值，**能够抛出异常的函数一定要在函数的表达式后面添加关键字 throws  （这种函数一般称作`throwing函数`），如果这个函数有返回值 ，`throws` 关键字要写在 `->ReturnType`前面**，看代码：

```swift
func numberTest(num: Int) throws {
    if num == 1 {
        print("成功")
    } else if num == 2 {
        throw OperationError.ErrorTwo
    } else if num == 3 {
        throw OperationError.ErrorThree("失败")
    } else {
        throw OperationError.ErrorOther
    }
}
```

这是个很简单的函数，可以根据传入参数的值来确定是否抛出异常，抛出何种异常值。 



 下面看第一种：**异常处理错误传递法**，<u>顾名思义就是函数自己不处理异常将异常抛出给上一级，让上一级处理</u>，如下代码所示：

```swift
//错误传递
func throwDeliver(num: Int) throws -> String {
    print("错误传递")
    try numberTest(num: num)
    print("未传递错误")
    return "无错误"
}
```

> 单词：
>
> Deliver  [dɪˈlɪvə(r)] 
>
>  vt. 交付；发表；递送；释放；给予（打击）；给…接生
>
> vi. 实现；传送；履行；投递
>
> n. 投球



throwDeliver这个throwing函数它本身并没有处理numberTest函数可能抛出的异常，而是把异常抛给了调用throwDeliver这个函数的地方处理了。**能够传递异常的它本身一定是throwing**



第二种：**使用do-catch捕获处理异常**，<u>在do闭包里面执行会抛出异常的代码，在catch 分支里面匹配异常处理异常</u>，看代码：

```swift
//do-catch错误捕获
do {
    print("do-catch错误捕获")
    try throwDeliver(num: 5)
    print("未捕获错误")
} catch OperationError.ErrorOne {
    print("ErrorOne")
} catch OperationError.ErrorTwo {
    print("ErrorTwo")
} catch OperationError.ErrorThree(let discription) {
    print("ErrorThree:" + discription)
} catch let discription {
    print(discription)
}
/*打印：
 do-catch错误捕获
 错误传递
 ErrorOther   */
```



第三种，**将异常转换成可选值**，如果一个语句会抛出异常那么它将返回`nil`，无论这个语句本来的返回值是什么：

```swift
//错误转成可选值
if let retureMessage = try? throwDeliver(num: 1) {
    print("可选值非空：" + retureMessage)
}
/*打印：
 错误传递
 成功
 未传递错误
 可选值非空：无错误 */
```

第四种，**禁止异常传递**，<u>只有当你确定这个语句不会抛出异常你才可以这么做，否则会引发运行时错误</u>：

```swift
//禁止错误传递
print(try! throwDeliver(num: 1) + ":禁止错误传递")
/*打印：
 错误传递
 成功
 未传递错误
 无错误:禁止错误传递 */
```

