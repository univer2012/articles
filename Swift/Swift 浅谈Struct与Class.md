来自：[Swift 浅谈Struct与Class](https://www.cnblogs.com/beckwang0912/p/8508299.html)



---



讨论Struct与Class之前，我们先来看一个概念：Value Type（值类型），Reference Type（引用类型）：

1. 值类型的变量直接包含他们的数据，对于值类型都有他们自己的数据副本，因此对一个变量操作不可能影响另一个变量；

2. 引用类型的变量存储对他们的数据引用，因此后者称为对象，因此对一个变量操作可能影响另一个变量所引用的对象。

这就是我们之前博客中提到的深拷贝与浅拷贝，博客传送《[iOS 图文并茂的带你了解深拷贝与浅拷贝](http://www.cnblogs.com/beckwang0912/p/7212075.html)》，两者的本质区别在于：深拷贝就是内容拷贝，浅拷贝就是指针拷贝

   **A. 是否开启新的内存地址**

   **B. 是否影响内存地址的引用计数**

 

讨论Struct与Class之前，我们先做好准备工作，首先分别创建一个Struct：【SNode】 与 Class：【CNode】

```swift
fileprivate struct SNode {
    var Data: Int?
}
```



```swift
fileprivate class CNode {
    var Data: Int?
    init(data: Int) {
        self.Data = data
    }
}
```

下面我们通过代码来理解两者都有哪些异同： 

**1、property初始化的不同**

```swift
let snode = SNode(Data: 4) // struct可直接在构造函数中初始化property
print("snode.data:\(String(describing: snode.Data))")


let cnode = CNode()  // class不可直接在构造函数中初始化property
cnode.Data = 5
print("cnode.data:\(String(describing: cnode.Data))")
```

打印结果：

```
snode.data:Optional(4)
cnode.data:Optional(5)
```

主要的差別就是 <font color=#993366>class 在初始化时不能直接把 property 放在 默认的constructor 的参数里，而是需要自己创建一个带参数的constructor</font> ，如：

```swift
fileprivate class CNode {
    var Data: Int?
    init(data: Int) {
        self.Data = data
    }
}

let cnode = CNode.init(data: 5)
```



**2、变量赋值方式不同（深浅copy）**

```swift
// struct
let snode = SNode(Data: 4)
var snode1 = snode
snode1.Data = 5
print("snode.data:\(String(describing: snode.Data))\n snode1.data:\(String(describing: snode1.Data))")
```

打印结果：

```
snode.data:Optional(4)
 snode1.data:Optional(5)
```

说明：struct 赋值“=”的时候，会copy一份完整相同的內容给另一個变量 --> <font color=#FF0000>【开辟了新的内存地址】</font>

```swift
// class
let cnode = CNode()
cnode.Data = 5
let cnode1 = cnode
cnode1.Data = 6
print("cnode.data:\(String(describing: cnode.Data))\n cnode1.data:\(String(describing: cnode.Data))")
```

打印结果：

```
cnode.data:Optional(6)
 cnode1.data:Optional(6)
```

说明：class 赋值“=”的时候，不会copy一份完整的内容给另一個变量，只是增加了原变量内存地址的引用而已 --> <font color=#FF0000>【没有开辟了新的内存地址】</font>



**3、immutable 变量**

   Swift 语言的特色之一就是可变动内容和不可变内容用 var 和 let 來甄别，如果初始为let的变量再去修改会发生编译错误。

- struct也遵循这一特性
- class不存在这样的问题：从上面的赋值代码能很清楚的看出来。

![WX20200514-235554.png](https://upload-images.jianshu.io/upload_images/843214-f25d873ab89a658b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**4、mutating function**

```swift
fileprivate struct SNode {
    var Data: Int?
}
extension SNode {
    
    mutating func changeData(value: Int) {
        self.Data = value
    }
}
```



```swift
fileprivate class CNode {
    var Data: Int?
}
extension CNode {
    func changeData(value: Int) {
        self.Data = value
    }
}
```

说明：<font color=#9400D3>struct 和 class 的差別是 struct 的 function 要去改变 property 的值的时候要加上 mutating，而 class 不用。</font>



**5、继承**

   struct不能继承，class可以继承。



**6、struct比class更“轻量级”**

   struct分配在栈中，class分配在堆中。



## 题外话：Swift 把 Struct 作为数据模型的注意事项

  优点：

  1、安全性：

```
因为 Struct 是用值类型传递的，它们没有引用计数。
```

  2、内存：

```
由于他们没有引用数，他们不会因为循环引用导致内存泄漏。
```

  3、速度:

```
 值类型通常来说是以栈的形式分配的，而不是用堆。因此他们比 Class 要快很多!  (http://stackoverflow.com/a/24243626/596821)
```

  4、拷贝：

```
 Objective-C 里拷贝一个对象,你必须选用正确的拷贝类型（深拷贝、浅拷贝）,而值类型的拷贝则非常轻松！
```

  5、线程安全

```
值类型是自动线程安全的。无论你从哪个线程去访问你的 Struct ，都非常简单。
```

 

  缺点：

  1、Objective-C

```
当你的项目的代码是 Swift 和 Objective-C 混合开发时，你会发现在 Objective-C 的代码里无法调用 Swift 的 Struct。因为要在 Objective-C 里调用 Swift 代码的话，对象需要继承于 NSObject。
Struct 不是 Objective-C 的好朋友。
```

  2、继承

```
Struct 不能相互继承。
```

  3、NSUserDefaults

```
Struct 不能被序列化成 NSData 对象。
```

 

所以：如果模型较小，并且无需继承、无需储存到 NSUserDefault 或者无需 Objective-C 使用时，建议使用 Struct。

- #### [Why Choose Struct Over Class?](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class)

 

**知识延伸：为什么访问struct比class快？**

“堆”和“栈”并不是数据结构上的Heap跟Stack，而是程序运行中的不同内存空间。栈是程序启动的时候，系统事先分配的，使用过程中，系统不干预；**堆是用的时候才向系统申请的，用完了需要交还，这个申请和交还的过程开销相对就比较大了。**



栈是编译时分配空间，而堆是动态分配（运行时分配空间），所以栈的速度快。

从两方面来考虑：

   1.分配和释放：堆在分配和释放时都要调用函数（MALLOC,FREE)，比如分配时会到堆空间去寻找足够大小的空间（因为多次分配释放后会造成空洞），这些都会花费一定的时间，而栈却不需要这些。

   2.访问时间：访问堆的一个具体单元，需要两次访问内存，第一次得取得指针，第二次才是真正得数据，而栈只需访问一次。

 

---

【完】