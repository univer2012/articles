**1，Swift中类可以使用计算属性，即使用get和set来间接获取/改变其他属性的值**，代码如下 

```swift
class Calcuator {
    var a: Int = 1
    var b: Int = 1

    var sum: Int {
        get {
            return a + b
        }
        set(val) {
            b = val - a
        }
    }
}

let cal = Calcuator();
print(cal.sum)      //打印：2
cal.sum = 5
print(cal.b)        //打印：4
```



**2，对于set有简写方法，简写时，新赋的值默认为newValue** 

```swift
class Calcuator {
    var a: Int = 1
    var b: Int = 1

    var sum: Int {
        get {
            return a + b
        }
        set {
            b = newValue - a
        }
    }
}
```



**3，如果只要get，不要set方法时可以简写成如下代码** 

```
class Calcuator {
    var a: Int = 1
    var b: Int = 1
    
    var sum: Int {
        get {
            return a + b
        }
    }
}
```

---

【完】