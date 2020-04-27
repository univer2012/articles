来自：[Swift 5 新特性之三 Dictionary支持compactMapValues](https://www.jianshu.com/p/7b4934770a51)



在理解新方法compactMapValues之前，建议先了解已有的方法map,filter, reduce。如有疑惑可以参看[这里](https://links.jianshu.com/go?to=https%3A%2F%2Fuseyourloaf.com%2Fblog%2Fswift-guide-to-map-filter-reduce%2F)。

### Dictionary支持compactMapValues

###### 提案出处:

[https://github.com/apple/swift-evolution/blob/master/proposals/0218-introduce-compact-map-values.md](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapple%2Fswift-evolution%2Fblob%2Fmaster%2Fproposals%2F0218-introduce-compact-map-values.md)

##### 细节

首先我们来看看支持compactMapValues的动机。

Swift语言标准库已经为Array和Dictionary提供了两个很有用的函数：

- The `map(_:)` 函数对Array内的元素执行一个函数，并返回一个Array。  `compactMap(_:)` 函数提供类似功能，但其可过滤掉值为 `nil` 的元素。
- The `mapValues(_:)` 函数对Dictionary做类似 `map(_:)`的操作，但不过滤为值为`nil` 的部分。

由于Dictionary已有函数不支持过滤的功能，`compactMapValues`为此而生，结合了 `compactMap(_:)` 函数 和 `mapValues(_:)` 函数的功能。

我们来看下如何使用`compactMapValues`的案例：

```swift
let d:[String: String?] = ["a": "1", "b": nil, "c": "3"]
let r4 = d.compactMapValues({$0})
print(r4)
///打印：
///["c": "3", "a": "1"]

let ages = [
    "Mary": "42",
    "Bob": "twenty-five har har har!!",
    "Alice": "39",
    "John": "22"]

let filteredAges = ages.compactMapValues({ Int($0) })
print(filteredAges)
///打印：
///["John": 22, "Mary": 42, "Alice": 39]
```

结合上面的案例，`compactMapValues`应用的场景是对已有的未处理数据集合，通过自定义过滤条件，如过滤`nil` 或不符合格式的数据，从而得到期待的数据结果。

有兴趣的童鞋可以阅读下实现`compactMapValues`的源码

```swift
extension Dictionary {
    public func compactMapValues<T>(_ transform: (Value) throws -> T?) rethrows -> [Key: T] {
        return try self.reduce(into: [Key: T](), { (result, x) in
            if let value = try transform(x.value) {
                result[x.key] = value
            }
        })
    }
}
```

