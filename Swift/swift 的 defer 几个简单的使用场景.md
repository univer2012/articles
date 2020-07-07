来自：[swift 的 defer 几个简单的使用场景](https://www.jianshu.com/p/a71d87d92459)



---



准备把 swift 文档再扫一遍，发现了`defer`这个关键字，恕本人愚钝，以前还从来没有用过这个呢~ 简单地列一下这个东西有哪些可以用得上的情景吧~~

## defer 是干什么用的

很简单，用一句话概括，就是 `defer` block 里的代码会在函数 return 之前执行，无论函数是从哪个分支 return 的，还是有 throw，还是自然而然走到最后一行。

这个关键字就跟 Java 里的 try-catch-finally 的`finally`一样，不管 try catch 走哪个分支，它都会在函数 return 之前执行。而且它比 Java 的`finally`还更强大的一点是，它可以独立于 try catch 存在，所以它也可以成为整理函数流程的一个小帮手。在函数 return 之前无论如何都要做的处理，可以放进这个 block 里，让代码看起来更干净一些~

下面是 swift 文档上的例子：

```swift
var fridgeIsOpen = false
let fridgeContent = ["milk", "eggs", "leftovers"]
 
func fridgeContains(_ food: String) -> Bool {
    fridgeIsOpen = true
    defer {
        fridgeIsOpen = false
    }
    
    let result = fridgeContent.contains(food)
    return result
}
fridgeContains("banana")
print(fridgeIsOpen)
```



这个例子里执行的顺序是，先`fridgeIsOpen = true`，然后是函数体正常的流程，最后在 return 之前执行 `fridgeIsOpen = false`。

## 几个简单的使用场景

#### try catch 结构

最典型的场景，我想也是 `defer` 这个关键字诞生的主要原因吧：

```swift
func foo() {
  defer {
    print("finally")
  }
  do {
    throw NSError()
    print("impossible")
  } catch {
    print("handle error")
  }
}
```

不管 do block 是否 throw error，有没有 catch 到，还是 throw 出去了，都会保证在整个函数 return 前执行 `defer`。在这个例子里，就是先 print 出 "handle error" 再 print 出 "finally"。

do block 里也可以写 `defer`：

```swift
do {
  defer {
    print("finally")
  }
  throw NSError()
  print("impossible")
} catch {
  print("handle error")
}
```



那么它执行的顺序就会是在 catch block 之前，也就是先 print 出 "finally" 再 print 出 "handle error"。

#### 清理工作、回收资源

跟 swift 文档举的例子类似，`defer`一个很适合的使用场景就是用来做清理工作。文件操作就是一个很好的例子：

关闭文件

```swift
func foo() {
  let fileDescriptor = open(url.path, O_EVTONLY)
  defer {
    close(fileDescriptor)
  }
  // use fileDescriptor...
}
```

这样就不怕哪个分支忘了写，或者中间 throw 个 error，导致 `fileDescriptor` 没法正常关闭。还有一些类似的场景：

dealloc 手动分配的空间

```swift
func foo() {
  let valuePointer = UnsafeMutablePointer<T>.allocate(capacity: 1)
  defer {
    valuePointer.deallocate(capacity: 1)
  }
  // use pointer...
}
```

加/解锁：下面是 swift 里类似 Objective-C 的 synchronized block 的一种写法，可以使用任何一个 NSObject 作 lock

```swift
func foo() {
  objc_sync_enter(lock)
  defer { 
    objc_sync_exit(lock)
  }
  // do something...
}
```

像这种成对调用的方法，可以用 defer 把它们放在一起，一目了然。

#### 调 completion block

这是一个让我感觉“如果当时知道 `defer` ”就好了的场景，就是有时候一个函数分支比较多，可能某个小分支 return 之前就忘了调 completion block，结果藏下一个不易发现的 bug。用 `defer` 就可以不用担心这个问题了：



---

 【完】