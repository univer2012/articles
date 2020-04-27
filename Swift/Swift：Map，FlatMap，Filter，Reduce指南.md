来自：[Swift：Map，FlatMap，Filter，Reduce指南](https://www.jianshu.com/p/87b97dfbf17b)



Swift是支持一门函数式编程的语言，拥有`Map`，`FlatMap`,`Filter`,`Reduce`针对集合类型的操作。在使用Objective-C开发时，如果你没接触过函数式编程，那你可能没听说过这些名词，希望此篇文章可以帮助你了解Swift中的`Map`，`FlatMap`,`Filter`,`Reduce`。

# Map

首先我们来看一下`map`在`Swift`中的的定义，我们看到它可以用在 [Optionals](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/type/Optional/) 和 [SequenceType](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/protocol/SequenceType/) 上（如：数组、词典等）。

```swift
public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
    /// If `self == nil`, returns `nil`.  Otherwise, returns `f(self!)`.
    @warn_unused_result
    public func map<U>(@noescape f: (Wrapped) throws -> U) rethrows -> U?
}

extension CollectionType {
    /// Returns an `Array` containing the results of mapping `transform`
    /// over `self`.
    ///
    /// - Complexity: O(N).
    @warn_unused_result
    public func map<T>(@noescape transform: (Self.Generator.Element) throws -> T) rethrows -> [T]
}
```



**@warn_unused_result**：表示如果没有检查或者使用该方法的返回值，编译器就会报警告。
 [**@noescape**](https://link.jianshu.com?t=http://wiki.jikexueyuan.com/project/swift/chapter2/07_Closures.html#nonescaping_closures)：表示`transform`这个闭包是**非逃逸闭包**，它只能在当前函数`map`中执行，不能脱离当前函数执行。这使得编译器可以明确的知道运行时的上下文环境（因此，在非逃逸闭包中可以不用写`self`），进而进行一些优化。



##### 对 [Optionals](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/type/Optional/)进行`map`操作

简要的说就是，如果这个可选值有值，那就解包，调用这个函数，之后返回一个可选值，需要注意的是，返回的可选值类型可以与原可选值类型不一致：

```swift
///原来类型： Int?,返回值类型：String?
var value:Int? = 1
var result = value.map { String("result = \($0)") }
print(result)
///打印：
///Optional("result = 1")
```

```swift
///原来类型： Int?,返回值类型：String?
var value:Int? = nil
var result = value.map { String("result = \($0)") }
print(result)
///打印：
///nil
```



##### 对[SequenceType](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/protocol/SequenceType/)进行`map`操作

**我们可以使用`map`方法遍历数组中的所有元素，并对这些元素一一进行一样的操作（函数方法）。**`map`方法返回完成操作后的数组。


![img](https:////upload-images.jianshu.io/upload_images/1869329-e17438ddf8171bbf.png?imageMogr2/auto-orient/strip|imageView2/2/w/1188)


 我们可以用完成类似的操作：

```swift
var values = [1,3,5,7]
var results = [Int]()
for var value in values {
    value *= 2
    results.append(value)
}
print(results)
///打印：
///[2, 6, 10, 14]
```

这看起来有点麻烦，我们得先定义一个变量`var results`然后将`values`里面的元素遍历，进行我们的操作以后，将其添加进`results`，我们比较下使用`map`又会怎么样：

```swift
var values = [1,3,5,7]
let results = values.map { (element) -> Int in
    return element * 2
}
print(results)
///打印：
///[2, 6, 10, 14]
```

我们向`map`传入了一个闭包，对数组中的所有元素都 `乘以2`，将返回的新的数组赋值为`results`，是不是精简了许多？还能更精简！

#### 精简写法

```swift
var values = [1,3,5,7]
let results = values.map { $0 * 2 }
print(results)
///打印：
///[2, 6, 10, 14]
```

what the fuck...沉住气，让我们一步步来解析怎么就精简成这样了，保证让你神清气爽。翻开[The Swift Programming Language](https://link.jianshu.com?t=http://wiki.jikexueyuan.com/project/swift/chapter2/07_Closures.html#closure_expressions)中对于闭包的定义你就能找到线索。

##### 第一步：

由于闭包的函数体很短，所以我们将其改写成一行：

```swift
var values = [1,3,5,7]
let results = values.map ({ (element) -> Int in return element * 2 })
print(results)
///打印：
///[2, 6, 10, 14]
```

##### 第二步：

由于我们的闭包是作为`map`的参数传入的，系统可以推断出其参数与返回值，因为其参数必须是(Element) -> Int类型的函数。因此，返回值类型，`->`及围绕在参数周围的括号都可以被忽略：

```swift
var values = [1,3,5,7]
let results = values.map ({ element in return element * 2 })
print(results)
///打印：
///[2, 6, 10, 14]
```



##### 第三步：

**单行表达式闭包**可以通过省略`return`来隐式返回闭包的结果：

```swift
var values = [1,3,5,7]
let results = values.map ({ element in element * 2 })
print(results)
///打印：
///[2, 6, 10, 14]
```



由于闭包函数体只含有`element * 2`这单一的表达式，该表达式返回`Int`类型，与我们例子中`map`所需的闭包的返回值类型一致（其实是泛型），所以，可以省略`return`。

##### 第四步：

**参数名称缩写（Shorthand Argument Names）**，由于`Swift`自动为内联闭包提供了参数缩写功能，你可以直接使用`$0`,`$1`,`$2`...依次获取闭包的第1，2，3...个参数。
 如果您在闭包表达式中使用参数名称缩写，您可以在闭包参数列表中省略对其的定义，并且对应参数名称缩写的类型会通过函数类型进行推断。`in`关键字也同样可以被省略：

```swift
var values = [1,3,5,7]
let results = values.map ({ $0 * 2 })
print(results)
///打印：
///[2, 6, 10, 14]
```

例子中的$0即代表闭包中的第一个参数。

##### 最后一步：

**尾随闭包**，由于我们的闭包是作为**最后一个**参数传递给`map`函数的，所以我们可以将闭包表达式尾随：

```swift
var values = [1,3,5,7]
let results = values.map(){ $0 * 2 }
print(results)
///打印：
///[2, 6, 10, 14]
```

如果函数只需要闭包表达式**一个**参数，当您使用尾随闭包时，您甚至可以把()省略掉：

```swift
var values = [1,3,5,7]
let results = values.map{ $0 * 2 }
print(results)
///打印：
///[2, 6, 10, 14]
```

如果还有不明白的，可以多翻阅翻阅[The Swift Programming Language](https://link.jianshu.com?t=http://wiki.jikexueyuan.com/project/swift/chapter2/07_Closures.html#closure_expressions)。

# FlatMap

与map一样，它可以用在 [Optionals](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/type/Optional/)和 [SequenceType](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/protocol/SequenceType/) 上（如：数组、词典等）。我们先来看看针对Optional的定义：

##### 对 [Optionals](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/type/Optional/)进行`flatMap`操作

```swift
public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
    /// Returns `nil` if `self` is `nil`, `f(self!)` otherwise.
    @warn_unused_result
    public func flatMap<U>(@noescape f: (Wrapped) throws -> U?) rethrows -> U?
}
```

就闭包而言，这里有一个明显的不同，这次`flatMap`期望一个 `(Wrapped) -> U?)`闭包。对于可选值， flatMap 对于输入一个可选值时应用闭包返回一个可选值，之后这个结果会被压平，也就是返回一个解包后的结果。本质上，相比 `map`,`flatMap`也就是在可选值层做了一个解包。

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



##### 对[SequenceType](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/protocol/SequenceType/)进行`flatMap`操作

我们先来看看`Swift中的定义`

```swift
extension SequenceType {
    /// 返回一个将变换结果连接起来的数组
    /// `transform` over `self`.
    ///     s.flatMap(transform)
    /// is equivalent to
    ///     Array(s.map(transform).flatten())
    @warn_unused_result
    public func flatMap<S : SequenceType>(transform: (Self.Generator.Element) throws -> S) rethrows -> [S.Generator.Element]
}

extension SequenceType {
    /// 返回一个包含非空值的映射变换结果
    @warn_unused_result
    public func flatMap<T>(@noescape transform: (Self.Generator.Element) throws -> T?) rethrows -> [T]
}
```

通过这两个描述，就体现了`flatMap`对[SequenceType](https://link.jianshu.com?t=http://swiftdoc.org/v2.0/protocol/SequenceType/)的两个作用：

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



> 注意：
>
> 用`faltMap`做空值过滤时，会警告：`'flatMap' is deprecated: Please use compactMap(_:) for the case where closure returns an optional value`
>
> 我们用`compactMap(_:)`的实现如下:
>
> ```swift
> var values: [Int?] = [1,3,5,7,9,nil,10]
> let flattenResults = values.compactMap{ $0 }
> print(flattenResults)
> ///打印：
> ///[1, 3, 5, 7, 9, 10]
> ```



# Filter

同样，我先来看看`Swift`中的定义：

```swift
extension SequenceType {
    /// 返回包含原数组中符合条件的元素的数组
    /// Returns an `Array` containing the elements of `self`,
    /// in order, that satisfy the predicate `includeElement`.
    @warn_unused_result
    public func filter(@noescape includeElement: (Self.Generator.Element) throws -> Bool) rethrows -> [Self.Generator.Element]
}
```

`filter`函数接受一个`(Element) -> Bool)`的闭包，来判断原数组中的元素是否符合条件，这个方法用来过滤数组中的一些元素再好不过了：

```swift
var values = [1,3,5,7,9]
let flattenResults = values.filter{$0 % 3 == 0}
print(flattenResults)
///打印：
///[3, 9]
```



我们向flatMap传入了一个闭包，筛选出了能被3整除的数据。

# Reduce

我们先来看下`Swift`中的定义：

```swift
extension SequenceType {
    /// Returns the result of repeatedly calling `combine` with an
    /// accumulated value initialized to `initial` and each element of
    /// `self`, in turn, i.e. return
    /// `combine(combine(...combine(combine(initial, self[0]),
    /// self[1]),...self[count-2]), self[count-1])`.
    @warn_unused_result
    public func reduce<T>(initial: T, @noescape combine: (T, Self.Generator.Element) throws -> T) rethrows -> T
}
```

给定一个初始化的combine结果，假设为result,从数组的第一个元素开始，不断地调用`combine`闭包，参数为：（result，数组中的元素)，返回的结果值继续调用`combine函数`，直至元素最后一个元素，返回最终的result值。来看下面的代码（为了更方便你理解这个过程，代码就不简写了）：

```swift
var values = [1,3,5]
let initialResult = 0
let reduceResults = values.reduce(initialResult) { (tempResult, element) -> Int in
    return tempResult + element
}
print(reduceResults)
///打印：
///9
```

我们存在一个数组`[1,3,5]`，给定了一个初始化的结果 `initialResult = 0`，向`reduce`函数传了 `(tempResult, element) -> Int`的闭包，`tempResut`便是每次闭包返回的结果值，并且其初始值为我们之前设置的`initialResult`为`0`，`element`即为我们数组中的元素（可能为`1`,`3`,`5`）。reduce会一直调用`combine`闭包，直至数组最后一个元素。下面的代码更形象地描述了整个过程，这其实跟`reduce`所做的操作是等价的：

```swift
let initialResult = 0
func combine(_ tempResult: Int, element: Int) -> Int {
    return tempResult + element
}
let reduceResult = combine(combine(combine(initialResult, element: 1), element: 3), element: 5)
print(reduceResult)
///打印：
///9
```



以上所用的一些示例代码可以在我的[Github](https://link.jianshu.com?t=https://github.com/wuleijun/Swift-Map-Filter-Reduce)中找到，如果您有什么建议可以在评论区留言。

