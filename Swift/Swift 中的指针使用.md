来自：[Swift 中的指针使用](https://www.cnblogs.com/wj033/p/4510868.html)



---

SWIFT 中  指针被映射为泛型 

UnsafePointer<T> 

UnsafeMutablePointer<T>

表示一组连续数据指针的 UnsafeBufferPointer<T>

表示非完整结构的不透明指针 COpaquePointer 等等

 

UnsafePointer<T> 通过 memory 属性对其进行取值，如果这个指针是可变的 UnsafeMutablePointer<T> 类型，我们还可以通过 memory 对它进行赋值。

```swift
func incrementor(ptr: UnsafeMutablePointer<Int>) {
    ptr.memory += 1
}
  
var a = 10
incrementor(&a)
a  // 11
```

swift中&同样可以取地址, 但无法直接获取一个指针实例

```swift
var a = 10
//let ptr:UnsafeMutablePointer<Int> = &a // 'inout Int' is not convertible to 'UnsafeMutablePointer<Int>'
//let ptr2 = &a                          // 报错
func incrementor1(inout num: Int) {
    num += 1
}
 
var b = 10
incrementor1(&b)
b   // 11
```

 

```swift
[1,2,3] + 1  // 不报错，Playground显示一个地址值
([1,2,3] + 1)[-100]  // 不报错
([1,2,3] + 1)[30]
 
var array = [1,2,3]
//array + 1     //报错
 //let ptr:UnsafeMutableBufferPointer<Int> = array  //报错
```



当使用inout运算符时，使用var声明的变量和使用let声明的常量被分别转换到UnsafePointer和UnsafeMutablePoinger

 

 在 Swift 中不能直接取到现有对象的地址，我们还是可以创建新的 UnsafeMutablePointer 对象。与 Swift 中其他对象的自动内存管理不同，对于指针的管理，是需要我们手动进行内存的申请和释放的。

```swift
// 将向系统申请 1 个 Int 大小的内存，并返回指向这块内存的指针
var intPtr = UnsafeMutablePointer<Int>.alloc(1)
// 初始化
intPtr.initialize(10)
var intPtr2 = intPtr.successor()
intPtr2.initialize(50)
// 读取值
intPtr.memory   // 10
intPtr2.memory   // 20
 
//intPtr.dealloc(1)
//intPtr.destroy(1)//
intPtr.destroy()
intPtr2 = nil
//intPtr2.memory  // 奔溃
 
 
 
var array = [1,2,3]
let arrayPtr = UnsafeMutableBufferPointer<Int>(start: &array, count: array.count)
// baseAddress 是第一个元素的指针
var basePtr = arrayPtr.baseAddress as UnsafeMutablePointer<Int>
 
basePtr.memory // 1
basePtr.memory = 10
basePtr.memory // 10
 
//下一个元素
var nextPtr = basePtr.successor()
nextPtr.memory // 2
```

 直接操作变量地址 withUnsafePointer，withUnsafePointers

```swift
var test = 10
test = withUnsafeMutablePointer(&test, { (ptr: UnsafeMutablePointer<Int>) -> Int in
    ptr.memory += 1
    return ptr.memory
})
 
test // 11
```

 

**unsafeBitCast**

unsafeBitCast 是非常危险的操作，它会将一个指针指向的内存强制按位转换为目标的类型。因为这种转换是在 Swift 的类型管理之外进行的，因此编译器无法确保得到的类型是否确实正确，你必须明确地知道你在做什么。比如：

```swift
let arr = NSArray(object: "meow")
let str = unsafeBitCast(CFArrayGetValueAtIndex(arr, 0), CFString.self)
str // “meow”
 
let arr2 = ["meow2"]
let str2 = unsafeBitCast(CFArrayGetValueAtIndex(arr2, 0), CFString.self)
```

 

因为 NSArray 是可以存放任意 NSObject 对象的，当我们在使用 CFArrayGetValueAtIndex  从中取值的时候，得到的结果将是一个 UnsafePointer<Void>。由于我们很明白其中存放的是 String  对象，因此可以直接将其强制转换为 CFString。

关于 unsafeBitCast  一种更常见的使用场景是不同类型的指针之间进行转换。因为指针本身所占用的的大小是一定的，所以指针的类型进行转换是不会出什么致命问题的。这在与一些 C API 协作时会很常见。比如有很多 C API 要求的输入是 void *，对应到 Swift 中为  UnsafePointer<Void>。我们可以通过下面这样的方式将任意指针转换为 UnsafePointer。

```swift
var count = 100
var voidPtr = withUnsafePointer(&count, { (a: UnsafePointer<Int>) -> UnsafePointer<Void> in
    return unsafeBitCast(a, UnsafePointer<Void>.self)
})
// voidPtr 是 UnsafePointer<Void>。相当于 C 中的 void *
voidPtr.memory //Void
 
// 转换回 UnsafePointer<Int>
var intPtr = unsafeBitCast(voidPtr, UnsafePointer<Int>.self)
intPtr.memory //100
```



---

【完】