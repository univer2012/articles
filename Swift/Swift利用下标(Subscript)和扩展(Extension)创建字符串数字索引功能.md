来自：[Swift利用下标(Subscript)和扩展(Extension)创建字符串数字索引功能](https://blog.csdn.net/NecoSann/article/details/52296190)



大家都知道在编程语言Swift当中，字符串是不可以用数字作为索引值来进行操作的。
Swift是可以用索引值来操作字符串的，但索引值的类型为String.Index，意味着我们只能使用startIndex、endIndex、advancedBy(n)(offsetBy(n))、predecessor()、successor() 这样的属性或方法来操作字符串。然而我们可以利用Swift的下标和扩展功能来自己写一个字符串像C++等语言中用数字来索引的功能。
具体代码如下：

```swift
// Swift 2.x
extension String {
    subscript(index: Int) -> Character {
        get {
            return self[self.startIndex.advancedBy(index)]
        }
        set {
            let rangeIndex = self.startIndex.advancedBy(index)
            self.replaceRange(rangeIndex...rangeIndex, with: String(newValue))
        }
    }
}
```



```swift
// Swift 3.0
extension String {
    subscript(index: Int) -> Character {
        get {
            return self[self.index(startIndex, offsetBy: index)]
        }
        set {
            let rangeIndex = self.index(startIndex, offsetBy: index)
            self.replaceSubrange(rangeIndex...rangeIndex, with: String(newValue))
        }
    }
}
```

测试结果如下：

```swift
var str = "Hello"
print(str[4])     // 打印 o
str[0] = "C"
print(str)        // 打印 Cello
```

---

【完】