# `mutating`修饰符


参考：
1. [关于Swift中的mutating关键字](https://www.jianshu.com/p/829c0ca3e84b)

mutating： 可变化，可改变

`mutating`可使用到的地方:

1. 结构体，枚举类型中的方法中声明为`mutating`
2. `extension`中的方法声明为`mutating`
3. `protocol`方法声明为`mutating`


1. 修改`struct`，`enum`的属性变量：周知swift中结构体(structure)和枚举(enumeration)中是可以包含类方法和实例方法，可是官方不建议在实例方法中修改其属性变量(immutable)。在func前加入mutating关键字后，使其属性变量可修改(mutable)
2. 在extension中同理
3. 可能会被`struct`或`enum`实现的`protocol`方法，需要声明为`mutating`，使实现时可修改其自己的属性变量。同时因为`mutating`对于`class`来说是完全透明的，不必担心使用`class`实现该`protocol`方法会带来影响

>单词：
>
>immutable    [ɪˈmjuːtəbl] adj. 不变的；不可变的；不能变的
>
>

#### 例1 
为`String`添加一个`extention` 实现`appendString(string: String)`
```swift
extension String {
    mutating func append(string: String) {
        self = self.appending(string)
    }
}
//测试
var a: String = "abc"
a.append(string: "abc")
print(a)    //打印：abcabc
```

#### 例2
为`Array`添加一个``extention 实现`removeObject(object :Generator.Element)`
```swift
extension Array where Element: Equatable {
    mutating func remove(object: Element) {
        if let index = self.firstIndex(of: object) {
            print("index: \(index)")
            self.remove(at: index)
        }
    }
}
//测试
var arr: Array = ["a", "b", "c", "d", "c"]
arr.remove(object: "c")
print(arr)
/*output:
 index: 2
 ["a", "b", "d", "c"]
 */
```
#### 例3
`struct`实现`protocol`方法
```swift
protocol MyProtocol {
    mutating func MyProtocolDidChange(index: Int)
}

struct MyStruct: MyProtocol {
    var index: Int
    mutating func MyProtocolDidChange(index: Int) {
        self.index = index
    }
}
var ab: MyStruct = MyStruct(index: 4)
ab.MyProtocolDidChange(index: 5)
print(ab.index) //5
```


