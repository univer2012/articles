# Swift_where关键字

来自：[Swift - where关键字使用详解（额外条件语句）](https://www.hangge.com/blog/cache/detail_1826.html)

[Swift  where 关键字](https://www.jianshu.com/p/1546594b856b)

**where** 语句可以用来设置约束条件、限制类型，让代码更加简洁、易读。 

### 1，Swift3 中 where 语句的作用

（1）可以使用 **where** 关键词在 **switch**、**for in** 语句上做些条件限制。 

```swift
let scores = [20, 8, 59, 60, 70, 80]

//switch语句中使用
scores.forEach {
    switch $0 {
    case let x where x >= 60:
        print("及格")
    default:
        print("不及格")
    }
}

//for语句中使用
for score in scores where score >= 60 {
    print("这个是及格的：\(score)")
}
```

打印结果：

```
打印结果：
不及格
不及格
不及格
及格
及格
及格
这个是及格的：60
这个是及格的：70
这个是及格的：80
```

（2）在 **do catch** 里面使用

```swift
enum ExceptionError: Error {
    case httpCode(Int)
}
func throwError() throws {
    //print("hello!")  //正常的流程
    throw ExceptionError.httpCode(500) //抛出错误500
    //throw ExceptionError.httpCode(405) //抛出错误405
}

do {
    try throwError()
} catch ExceptionError.httpCode(let httpCode) where httpCode >= 500 {
    print("server error")
} catch {
    print("other error")
}
```

打印结果：

```
server error
```

（3）与协议结合 

```swift
protocol aProtocol { }

//只给遵守aProtocol协议的UIView添加了拓展
extension aProtocol where Self: UIView {
    func getString() -> String {
        return "string"
    }
}

class myView: UIView, aProtocol {
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
//    //myView属于UIView的子类，所以要实现aProtocol协议
//    func getString() -> String {
//        return "It is a UIView."
//    }
}
//myViewController不属于UIView的子类，即使遵从aProtocol协议，但是不用实现该协议里面的内容
class myViewController: UIViewController, aProtocol {
    
}
```

（4）与泛型结合
```swift
///第一种写法
func genericFunction<S>(str: S) where S: ExpressibleByStringLiteral {
    print(str)
}
///第二种写法
func genericFunction<S: ExpressibleByStringLiteral>(str: S) {
    print(str)
}
```


#### 注意：过去 `if let`、`guard` 语句中的`where`现在都用 `,` 替代了
```swift
var optionName: String? = "Hangge"
if let name = optionName, name.hasPrefix("H") {
    print("\(name)")
}
```

打印：

```
Hangge
```

####  以下是Swift3.0取消的 where 写法

本来在while循环中也能使用的where，现在变成了这个样子
 1、while

```swift
//while   swift2.0中的写法
var arrayTwo:[Int]? = []
while let arr = arrayTwo where arr.count < 5 {
    arrayTwo?.append(1)
}
//Swift3.0的写法
var arrayTwo:[Int]? = []
while let arr = arrayTwo , arr.count < 5 {
    arrayTwo?.append(1)
}
```

2、if 和 guard    现在我也不知道怎么使用Swift化的写法来写这个表达式，希望能知道的朋友告诉一下，谢谢

```swift
// swift2.0中的写法``
let sString = ""
if let str = sString where str == ""{
}

let string:String? = "小刚"
guard let str = string where str != "小明" else {
    fatalError("g看错人了") //
}
print("确实是小明")
```


###  2，Swift4 中的改进 

（1）可以在 **associatedtype** 后面声明的类型后追加 **where** 约束语句

```swift 3
//in Swift 3
protocol MyProtocol {
    associatedtype Element
    associatedtype SubSequence: Sequence where SubSequence.Iterator.Element == Iterator.Element
}
```

（2）由于 **associatedtype** 支持追加 `where` 语句，现在 **Swift 4** 标准库中 `Sequence` 中 `Element` 的声明如下： 

```swift
public protocol Sequence {

    /// A type representing the sequence's elements.
    associatedtype Element where Self.Element == Self.Iterator.Element
    // ...
}
```

它限定了 **Sequence** 中 **Element** 这个类型必须和 **Iterator.Element** 的类型一致。原先 **Swift 3** 中到处露脸的 **Iterator.Element**，现在瘦身成 **Element**。 



> 单词：
>
> Sequence  [ˈsiːkwəns] n. [数][计] 序列；顺序；续发事件     vt. 按顺序排好
> Numeric  [njuː'merɪk] adj. 数值的（等于 numerical）；数字的   		n. 数；数字

比如我们对 **Sequence** 做个扩展，增加个 **sum** 方法（计算总和）：

```swift
extension Sequence where Element: Numeric {
    var sum: Element {
        var result: Element = 0
        for item in self {
            result += item
        }
        return result
    }
}

print([1, 2, 3, 4].sum)
```

打印结果：

```
10
```

上面的扩展，如果数组的元素(Element)不是数字，那就不能调用这个`sum`方法：

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Swift_where_ keyword1.png)

同样地，**SubSequence** 也通过 **where** 语句的限定，保证了类型正确，避免在使用 **Sequence** 时做一些不必要的类型判断。

```swift
///in Swift 3
protocol Sequence {
    associatedtype SubSequence: Sequence
        where SubSequence.SubSequence == SubSequence, SubSequence.Element == Element
}
```

（3）除了 **Sequence**，**Collection** 同样新增了 **where** 语句约束。比如下面样例，我们扩展 **Collection** 时所需约束变得更少了。 

```swift
// 在Swift 3时代, 这种扩展需要很多的约束:
//extension Collection where Iterator.Element: Equatable,
//    SubSequence: Sequence,
//    SubSequence.Iterator.Element == Iterator.Element
//
// 而在Swift 4, 编译器已经提前知道了上述3个约束中的2个, 因为可以用相关类型的where语句来表达它们.
extension Collection where Element: Equatable {
    func prefieIsEqualSuffix(_ n: Int) -> Bool {
        let head = prefix(n)
        let suff = suffix(n).reversed()
        return head.elementsEqual(suff)
    }
}

print([1,2,3,4,2,1].prefieIsEqualSuffix(2))
```

打印结果：

```
true
```


【完】

