来自：[Swift - 类扩展（extension）](https://www.hangge.com/blog/cache/detail_525.html)



**Swift语言的类扩展是一个强大的工具，我们可以通过类扩展完成如下事情：** 

1. 给已有的类添加计算属性和计算静态属性
2. 定义新的实例方法和类方法
3. 提供新的构造器
4. 定义下标脚本
5. 是一个已有的类型符合某个协议

> **注意：**
>
> 1. 扩展只能添加新的计算型属性，不能添加存储型属性，也不能添加新的`didSet`、`willSet`属性监视器。
>
> 2. 在 **Swift 4** 中，**extension** 里可以访问 **private** 的属性。（过去 Swift3 中是不行的）
>
> 

例如：

```swift
class Calcuator {
    var a: Int = 1
    private var b: Int = 1

    var sum: Int {
        get {
            return a + b
        }
        set {
            b = newValue - a
        }
    }
}
extension Calcuator {
    //计算型属性
    ///1、extension可以添加计算型属性。
    ///2、Swift 4及以上版本中，可以访问private属性。
    var multiplication: Int {
        get {
            return a * b
        }
        set {
            b = newValue / a
        }
    }
    
    //计算型静态属性
    static var div: Int {
        return 8 / 2
    }
    //计算型属性
    var subtraction: Int {
        return 4
    }
    
    //存储型属性
    ///报错：Extensions must not contain stored properties
    //var c:Int = 1
    
    //新的构造器
    convenience init(a:Int, b: Int) {
        self.init()
        self.a = a
        self.b = b
    }
    
}
```



**示例1：给字符串String类添加下标脚本，支持索引访问**

```swift
extension String {
    public subscript(start: Int, length: Int) -> String {
        get {
            let index1 = self.index(self.startIndex, offsetBy: start)
            let index2 = self.index(index1, offsetBy: length)
            return String(self[index1 ..< index2])
        }
        set {
            let tmp = self
            var s = ""
            var e = ""
            for (idx, item) in tmp.enumerated() {
                if idx < start {
                    s += "\(item)"
                }
                if idx >= start + length {
                    e += "\(item)"
                }
            }
            self = s + newValue + e
        }
    }
    
    public subscript(index: Int) -> String {
        get {
            return String(self[self.index(self.startIndex,offsetBy: index)])
        }
        
        set {
            let tmp = self
            self = ""
            for (idx, item) in tmp.enumerated() {
                if idx == index {
                    self += "\(newValue)"
                } else {
                    self += "\(item)"
                }
            }
        }
    }
}

var str = "univer2012.com"
print(str[11,3])
print(str[1])

str[11,3] = "COM"
str[0] = "U"

print(str[0,14])
```

运行结果如下：

```
com
c
Univer2012.COM
```





**示例2：给Double增加mm，cm等方法，进行进制转换** 

```swift
extension Double {
    func mm() -> String {
        return "\(self/1)mm"
    }
    func cm() -> String {
        return "\(self/10)cm"
    }
    func dm() -> String {
        return "\(self/100)dm"
    }
    func m() -> String {
        return "\(self/1000)m"
    }
    func km() -> String {
        return "\(self/(1000 * 1000))km"
    }
}

let value = 2000000000.0
print(value.mm())
print(value.cm())
print(value.dm())
print(value.m())
print(value.km())
```

运行结果如下：

```
2000000000.0mm
200000000.0cm
20000000.0dm
2000000.0m
2000.0km
```

---

【完】