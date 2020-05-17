来自：[iOS底层原理探索—内存管理(一)](https://www.jianshu.com/p/70cc9623caa9)



---



### 往期回顾

[iOS底层原理探索 — OC对象的本质](https://www.jianshu.com/p/ffd742041946)
 [iOS底层原理探索 — class的本质](https://www.jianshu.com/p/265412c910be)
 [iOS底层原理探索 — KVO的本质](https://www.jianshu.com/p/1ae6a02ffb42)
 [iOS底层原理探索 — KVC的本质](https://www.jianshu.com/p/91eb7eea2cdb)
 [iOS底层原理探索 — Category的本质(一)](https://www.jianshu.com/p/6735b7ec0fe5)
 [iOS底层原理探索 — Category的本质(二)](https://www.jianshu.com/p/bc3e9fa647cc)
 [iOS底层原理探索 — 关联对象的本质](https://www.jianshu.com/p/1227f7202665)
 [iOS底层原理探索 — block的本质(一)](https://www.jianshu.com/p/cebc6e8e7a2d)
 [iOS底层原理探索 — block的本质(二)](https://www.jianshu.com/p/26ff309f00a0)
 [iOS底层原理探索 — Runtime之isa的本质](https://www.jianshu.com/p/a3fb0919877c)
 [iOS底层原理探索 — Runtime之class的本质](https://www.jianshu.com/p/941cb11538aa)
 [iOS底层原理探索 — Runtime之消息机制](https://www.jianshu.com/p/c8d98d39f7ee)
 [iOS底层原理探索 — RunLoop的本质](https://www.jianshu.com/p/31237ae8c539)
 [iOS底层原理探索 — RunLoop的应用](https://www.jianshu.com/p/9475317619f7)
 [iOS底层原理探索 — 多线程的本质](https://www.jianshu.com/p/1faf2d78136c)
 [iOS底层原理探索 — 多线程的经典面试题](https://www.jianshu.com/p/3192a73fa7c2)
 [iOS底层原理探索 — 多线程的“锁”](https://www.jianshu.com/p/b33090b13d24)

# 前言

内存管理在APP开发过程中占据着一个很重要的地位，在iOS中，系统为我们提供了`ARC`的开发环境，帮助我们做了很多内存管理的内容，其实在`MRC`时代，内存管理对于开发者是个很头疼的问题。我们会通过几篇文章的分析，来帮助我们了解iOS中内存管理的原理，以及在`ARC`的开发环境下系统帮助我们做了哪些内存管理的操作。

# iOS程序的内存布局

我们通过一张图展示iOS程序的内存布局：

![img](https:////upload-images.jianshu.io/upload_images/1760191-9747305ab3b475d4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

内存布局.png


 在iOS程序的内存中，从底地址开始，到高地址一次分为：`程序区域`、`数据区域`、`堆区`、`栈区`。其中`程序区域`主要是`代码段`，`数据区域`包括`数据段`和`BSS段`。我们具体分析一下各个区域所代表的含义：



> **代码段**: 存放编译后的代码，内存区域较小。程序结束时系统会自动回收存储在代码段中的数据。

> **数据段**: 也叫常量区，保存已初始化的全局变量、静态变量等。直到程序结束的时候才会被回收。

> **BSS段**: 也叫静态区，保存未被初试化的全局变量、静态变量。一旦初始化就会被回收，并且将数据转存到数据段中。

> **堆区(heap)**: 保存由`alloc`创建出来的对象，动态分配内存。需要程序员来进行内存管理。从底地址到高地址分配内存空间

> **栈区(stack)**: 保存局部变量，自动分配内存，系统管理。当局部变量的作用域执行完毕后就会被系统立即回收。从高地址到底地址分配内存空间

# Tagged Pointer技术

在 2013 年 9 月，苹果推出了 iPhone5s 。iPhone5s 配备了首个采用 64 位架构的 A7 双核处理器。为了节省内存和提高执行效率，苹果提出了`Tagged Pointer`的概念，用于优化`NSNumber`、`NSDate`、`NSString`等小对象的存储。

在没有使用`Tagged Pointer`之前， `NSNumber`等对象需要动态分配内存、维护引用计数等，`NSNumber`指针存储的是堆中`NSNumber`对象的`地址值`。
 例如下面这句代码：

```objectivec
NSNumber *number = @10;
```

**在没有使用`Tagged Pointer`之前，内存中包括一个占8字节的指针变量 `number`，和一个占16字节的`NSNumber`对象，指针变量 `number` 指向 `NSNumber` 对象的地址。这样需要耗费24个字节内存空间。**

![img](https:////upload-images.jianshu.io/upload_images/1760191-c6e2ccbaa42f1b1d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

未使用TaggedPointer.png

**使用`Tagged Pointer`之后，`NSNumber`指针里面存储的数据变成了：`Tag + Data`，也就是将数据直接存储在了指针中。**

直接将数据10保存在指针变量`number`中，这样仅占用8个字节。

![img](https:////upload-images.jianshu.io/upload_images/1760191-a1ec7d858720748d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

使用了TaggedPointer.png


 当然，当指针不够存储数据时，就会使用动态分配内存的方式来存储数据。



我们用代码来验证一下：

![img](https:////upload-images.jianshu.io/upload_images/1760191-be76477a257026ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

测试.png


 在测试代码中创建7个`NSNumber`类型的对象，分别赋值后打印地址，可以看出使用`Tagged Pointer`之后，`NSNumber`指针里面存储着对象的值。其中`number7`由于赋了一个很大的值，指针不够存储，就使用了动态分配内存的方式来存储`number7`的值。



当然，以上测试代码要运行在64位环境下。

接下来我们通过**一道面试题**来帮助我们理解：

### 以下两段代码的执行结果是什么？

```objectivec
    //第1段代码
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    for (int i = 0; i < 1000; i++) {
        dispatch_async(queue, ^{
            self.name = [NSString stringWithFormat:@"asdasdefafdfa"];
        });
    }
    NSLog(@"end");
```

```objectivec
     //第2段代码
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    for (int i = 0; i < 1000; i++) {
        dispatch_async(queue, ^{
            self.name = [NSString stringWithFormat:@"abc"];
        });
    }
    NSLog(@"end");
```

> 答案是第1段代码会崩溃，报出坏内存访问的错误；第2段代码正常打印end

这是为什么呢？

这就涉及到我们上文讲到的`Tagged Pointer`技术。

我们先来看第1段代码中`self.name = [NSString stringWithFormat:@"asdasdefafdfa"];`这句代码，这句代码的意思将后面的值赋给`self.name`。注意，此时要赋的值是一长串字符串，`name`的指针的8个字节已经存储不下这个字符串了，那么就会动态分配内存的方式来存储，就是调用`name`的`set`方法。

我们知道，在`set`方法内部，会首先调用`[_name release]`释放旧值，再赋新值。但是我们赋值的代码是在子线程中异步执行的，那么就存在同时会有多条线程同时调用`[_name release]`，这就出现问题了。

问题的解决方法很简单，可以把 `name` 的 `nonatomic` 修饰符改成`atomic`，这一点我们在[iOS底层原理探索 —多线程的读写安全](https://www.jianshu.com/p/aeae6e6d2ed5)中讲到过`atomic`的作用，这里不再赘述。或者最直接有效的解决方案就是在异步复制时进行加锁和解锁即可。以保证线程安全。



那么第2段代码为什么能执行成功呢？原因很简单，由于`Tagged Pointer`技术，`name`的指针的8个字节足以存放字符串`abc`，就不涉及调用`name`的`set`方法。所以能够成功打印`end`。



# MRC中的内存管理

在iOS中，使用`引用计数`的技术来管理`OC对象`的内存：
 一个新创建的`OC对象`引用计数默认是1，当引用计数减为0，`OC对象`就会销毁，释放其占用的内存空间。调用`retain`会让OC对象的引用计数+1，调用`release`或者`autorelease`会让OC对象的引用计数-1

我们在上文中提到了在`set`方法内部，会首先调用`[_name release]`释放旧值，再赋新值。

在`MRC`时代，程序员需要手动的去管理内存，创建一个对象时，需要在`set`方法和`get`方法内部添加释放对象的代码。并且在对象的`dealloc`里面添加释放的代码。
 我们用几个简单的例子来看一下：

- 使用`assign`关键字修饰的数据常量，`set`方法和`get`方法内部直接赋值和取值

```objectivec
@property (nonatomic, assign) int age;

- (void)setAge:(int)age {
    _age = age;
}

- (int)age {
    return _age;
}
```

使用`strong`关键字修饰的对象，`set`方法内部需要先释放旧值，再`retain`新值

```objectivec
@property (nonatomic, strong) Person *person;

- (void)setPerson:(Person *)person {
    if (_person != person) {
        [_person release];
        _person = [person retain];
    }
}

- (Person *) person {
    return _person;
}
```

- 使用`copy`关键字修饰的对象，`set`方法内部需要先释放旧值，再`copy`新值

```objectivec
@property (nonatomic, copy) NSArray *data;

- (void)setData:(NSArray *)data {
    if (_data != data) {
        [_data release];
        _data = [data copy];
    }
}
```



# ARC的内存管理

在`ARC`环境中，我们不再像以前一样自己手动管理内存，系统帮助我们做了`release`或者`autorelease`等事情。
 `ARC`是`LLVM编译器`和`RunTime`协作的结果。其中`LLVM编译器`自动生成`release`、`reatin`、`autorelease`的代码，像`weak`弱引用这些则靠`RunTime`在运行时释放。



# 引用计数

上文我们讲到在iOS中，使用`引用计数`的技术来管理`OC对象`的内存，那么`引用计数`是如何存储的呢？我们之前在[iOS底层原理探索 — Runtime之isa的本质](https://www.jianshu.com/p/a3fb0919877c)一文中讲过**在`__arm64__`架构之后，`isa`指针不单单只存储了类对象和元类对象的内存地址，而是使用共用体的方式存储了更多信息**。其中就包括`引用计数`。
 我们再来回顾一下`isa`指针内部存储的内容：

```objectivec
struct {
    // 0代表普通的指针，存储着类对象、元类对象的内存地址。
    // 1代表优化后的使用位域存储更多的信息。
    uintptr_t nonpointer        : 1; 

   // 是否有设置过关联对象，如果没有，释放时会更快
    uintptr_t has_assoc         : 1;

    // 是否有C++析构函数，如果没有，释放时会更快
    uintptr_t has_cxx_dtor      : 1;

    // 存储着类对象、元类对象对象的内存地址信息
    uintptr_t shiftcls          : 33; 

    // 用于在调试时分辨对象是否未完成初始化
    uintptr_t magic             : 6;

    // 是否有被弱引用指向过。
    uintptr_t weakly_referenced : 1;

    // 对象是否正在释放
    uintptr_t deallocating      : 1;

    // 引用计数器是否过大无法存储在isa中
    // 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中
    uintptr_t has_sidetable_rc  : 1;

    // 里面存储的值是引用计数器减1
    uintptr_t extra_rc          : 19;
};
```

我们可以看到，在`extra_rc`里面存储的值是引用计数器减1，但是当`extra_rc`的19位内存不够存储引用计数时，`has_sidetable_rc`的值就会变为`1`，那么此时引用计数会存储在一个叫`SideTable`的类的属性中。

![img](https:////upload-images.jianshu.io/upload_images/1760191-d9abb9ab113c5400.png?imageMogr2/auto-orient/strip|imageView2/2/w/1168)

SideTable.png


`SideTable`类中有一个`RefcountMap`类型的散列表，这个散列表中就存放着引用计数。



我们来到源码文件`NSObject.mm`文件看一下源码：
 在源码中，`retainCount`方法内部会调用`rootRetainCount`方法，在`rootRetainCount`方法，内部会做一系列的引用计数操作：

![img](https:////upload-images.jianshu.io/upload_images/1760191-ee37337e5ac97266.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

rootRetainCount源码.png


 经过一系列判断，如果`has_sidetable_rc`的值就会为`1`时，说明此时引用计数会存储在`SideTable`的类`RefcountMap`散列表中。然后通过`sidetable_getExtraRC_nolock()`函数去获取引用计数。

![img](https:////upload-images.jianshu.io/upload_images/1760191-9bc4ec3156a37165.png?imageMogr2/auto-orient/strip|imageView2/2/w/1012)

sidetable_getExtraRC_nolock.png


`sidetable_getExtraRC_nolock`函数内部，也是先通过`key`找到对应的`SideTable`，在`SideTable`中通过`key`找到`RefcountMap`散列表，在散列表中拿到`refcnts`，即引用计数，然后返回。



今天对于内存管理的分析就到这里，我会在后续的文章中继续为大家分析有关内存管理的知识。



---

【完】

