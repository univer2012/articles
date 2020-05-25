来自：[iOS内存管理（MRC、ARC）深入浅出](http://www.cocoachina.com/articles/22518)



---



## 内存管理方式

首先明确一点，无论在MRC还是ARC情况下，Objective-C采用的是引用计数式的内存管理方式，这一方式的特点：

- 自己生成的对象，自己持有。例如：`NSObject * __strong obj = [[NSObject alloc]init];`。
- 非自己生成的对象，自己也能持有。例如：`NSMutableArray * __strong array = [NSMutableArray array];`。
- 不再需要自己持有对象时释放。
- 无法释放非自己持有的对象。

| 对象操作       | Objective-C方法                  |
| -------------- | -------------------------------- |
| 生成并持有对象 | alloc/new/copy/mutableCopy等方法 |
| 持有对象       | retain方法                       |
| 释放对象       | release方法                      |
| 废弃对象       | dealloc方法                      |

#### 自己生成的对象，自己持有

在iOS内存管理中有四个关键字，alloc、new、copy、mutableCopy，自身使用这些关键字产生对象,那么自身就持有了对象

```objectivec
    // 使用了alloc分配了内存，obj指向了对象，该对象本身引用计数为1,不需要retain 
    id obj = [[NSObject alloc] init]; 
    // 使用了new分配了内存,objc指向了对象，该对象本身引用计数为1，不需要retain 
    id obj = [NSObject new];
```

#### 非自己生成的对象，自己也能持有

```objectivec
    // NSMutableArray通过类方法array产生了对象(并没有使用alloc、new、copy、mutableCopt来产生对象),因此该对象不属于obj自身产生的
    // 因此，需要使用retain方法让对象计数器+1,从而obj可以持有该对象(尽管该对象不是他产生的)
    id obj = [NSMutableArray array];
    [obj retain];
```

#### 不再需要自己持有对象时释放

```objectivec
    id obj = [NSMutableArray array];  
    [obj retain];
    // 当obj不在需要持有的对象，那么，obj应该发送release消息
    [obj release];
    
    // 释放了对象还进行释放,会导致奔溃
    [obj release];
```

#### 无法释放非自己持有的对象

```objectivec
    // 释放一个不属于自己的对象
    id obj = [NSMutableArray array]; 
    
    // obj没有进行retain操作而进行release操作， 然后autoreleasePool也会对其进行一次release操作，导致奔溃。
    //后面会讲到autorelease。
    [obj release];
```

#### 针对[NSMutableArray array]方法取得的对象存在，自己却不持有对象，底层大致实现：

```objectivec
+ (id)object {
    //自己持有对象
    id obj = [[NSObject alloc]init];
    
    [obj autorelease];
    
    //取得的对象存在，但自己不持有对象
    return obj;
}
```

使用了autorelease方法，将obj注册到autoreleasePool中，不会立即释放，当pool结束时再自动调用release。这样达到取得的对象存在，自己不持有对象。

### autorelease

顾名思义：autorelease就是自动释放，它和C语言的局部变量类似，C语言局部变量在程序执行时，超出其作用域时，会被自动废弃。autorelease会像C语言局部变量那样对待对象实例，当超出作用域时，对象实例的release实例方法会被调用。我们可以设定autorelease的作用域。

**autorelease具体使用方法**

- 生成并持有NSAutoreleasePool对象
- 调用已分配对象的autorelease实例方法
- 废弃NSAutoreleasePool对象

![1.png](http://api.cocoachina.com/uploads//20180309/1520577566552663.png)

NSAutoreleasePool生命周期

NSAutoreleasePool对象的生命周期就是一个作用域，对于所有调用过autorelease实例方法的对象，在废弃NSAutoreleasePool对象时，都会调用其release实例方法。如上图所示。

用源代码表示如下：

```objectivec
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    id obj = [[NSObject alloc] init];
    [obj autorelease];
    [pool drain];
```

在[pool drain]调用时，NSAutoreleasePool被销毁，obj的release方法会被触发。obj被释放。

**autorelease苹果实现**

可以通过objc4库的`runtime/objc-arr.mm`来确认苹果中autorelease的实现。

```objectivec
    class AutoreleasePoolPage 
    {
        static inline void *push() 
        {
            //相当于生成或持有NSAutoreleasePool类对象
        }
        static inline void *pop(void *token)
        {
            //相当于废弃NSAutoreleasePool类对象
            releaseAll();
        }
        static inline id autorelease(id obj)
        {
            //相当于NSAutoreleasePool类的addObject类方法   
            AutoreleasePoolPage *autoreleasePoolPage = 取得正在使用的AutoreleasePoolPage实例; 
            autoreleasePoolPage->add(obj);
        }
        id *add(id obj) 
        {
            //将对象追加到内部数组中
        }
        void releaseAll() 
        {
            //调用内部数组中对象的release实例方法 
        }
    };
    void *objc_autoreleasePoolPush(void)
    {
        return AutoreleasePoolPage::push();
    }
    void objc_autoreleasePoolPop(void *ctxt)
    {
        AutoreleasePoolPage::pop(ctxt);
    }
    id *objc_autorelease(id obj) 
    {
        return AutoreleasePoolPage::autorelease(obj);
    }
```

我们使用调速器来观察NSAutoreleasePool类方法和autorelease方法的运行过程，如下所示，这些方法调用了关联于objc4库autorelease实现的函数。

```objectivec
    //等同于objc_autoreleasePoolPush()
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    id obj = [[NSObject alloc] init];
    
    //等同于objc_autorelease(obj)
    [obj autorelease];
    
    //等同于objc_autoreleasePoolPop(pool)
    [pool drain];
```

在iOS程序启动后，主线程会自动创建一个RunLoop，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 `_wrapRunLoopWithAutoreleasePoolHandler()`。

第一个 Observer **监视的事件是 Entry(即将进入Loop)**，其回调内会调用 `_objc_autoreleasePoolPush()` 创建自动释放池。其 order 是-2147483647，<font color=#FF0000>优先级最高，保证创建释放池发生在其他所有回调之前。</font>

第二个 Observer 监视了两个事件： **BeforeWaiting(准备进入休眠)** 时调用 `_objc_autoreleasePoolPop()` 和 `_objc_autoreleasePoolPush()` 释放旧的池并创建新池；**Exit(即将退出Loop)** 时调用  `_objc_autoreleasePoolPop()` 来释放自动释放池。这个 Observer 的 order 是  2147483647，<font color=#FF0000>优先级最低，保证其释放池子发生在其他所有回调之后。</font>

在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不一定非得显式创建 Pool 了。[runloop相关](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)



尽管如此，在大量产生autorelease的对象时，只要 `NSAutoreleasePool` 没有被废弃，那么产生的对象就不能被释放，因此会产生内存不足的现象。

例如：读入大量图像的同时改变其尺寸。图像文件读入到NSData对象，并从中生成UIImage对象，改变该对象尺寸后生成新的UIImage对象。这种情况会产生大量 `autorelease` 对象。

```objectivec
    for (int i = 0; i < 图片数; ++ i) {
        /*
         读入图片
         大量产生autorelease的对象
         由于没有废弃NSAutoreleasePool对象
         导致产生内存峰值，可能内存不足而奔溃
         */
    }
```

在这种情况下，应当在适当的位置生成、持有或废弃NSAutoreleasePool对象

```objectivec
    for (int i = 0; i < 图片数; ++ i) {
        NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
        /*
         读入图片
         大量产生autorelease的对象
         */
        [pool drain];
        /*
         通过释放pool
         autorelease的对象被release，就不会产生内存峰值
         */
    }
```



**使用容器的block版本的枚举器时，内部会自动添加一个AutoreleasePool**：

```objectivec
    [array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    // 这里被一个局部@autoreleasepool包围着
    }];
```



## 所有权修饰符及其原理

在 ARC 特性下有 4 种与内存管理息息相关的变量所有权修饰符值得我们关注：

- `__strong`
- `__weak`
- `__unsafe_unretaied`
- `__autoreleasing`

说到变量所有权修饰符，有人可能会跟属性修饰符搞混，这里做一个对照关系小结：

- assign 对应的所有权类型是 `__unsafe_unretained`。
- copy 对应的所有权类型是 `__strong`。
- retain 对应的所有权类型是 `__strong`。
- strong 对应的所有权类型是 `__strong`。
- unsafe_unretained对应的所有权类型是 `__unsafe_unretained`。
- weak 对应的所有权类型是 `__weak`。

### `__strong`

`__strong` 表示强引用，对应定义 property 时用到的  strong。当对象没有任何一个强引用指向它时，它才会被释放。如果在声明引用时不加修饰符，那么引用将默认是强引用。当需要释放强引用指向的对象时，需要保证所有指向对象强引用置为 nil。`__strong` 修饰符是 id 类型和对象类型默认的所有权修饰符。

**原理：**

```objectivec
    {
        id __strong obj = [[NSObject alloc] init];
    }
```

```objectivec
    //编译器的模拟代码
    id obj = objc_msgSend(NSObject,@selector(alloc));
    objc_msgSend(obj,@selector(init));
    // 出作用域的时候调用
    objc_release(obj);
```

虽然ARC有效时不能使用release方法，但由此可知编译器自动插入了release。



对象是通过除alloc、new、copy、multyCopy外方法产生的情况

```objectivec
   {
        id __strong obj = [NSMutableArray array];
    }
```

结果与之前稍有不同：

```objectivec
    //编译器的模拟代码
    id obj = objc_msgSend(NSMutableArray,@selector(array));
    objc_retainAutoreleasedReturnValue(obj);
    objc_release(obj);
```

`objc_retainAutoreleasedReturnValue` 函数主要用于优化程序的运行。它是用于持有(retain)对象的函数，它持有的对象应为返回注册在autoreleasePool中对象的方法，或是函数的返回值。像该源码这样，在调用 `array` 类方法之后，由编译器插入该函数。

而这种 `objc_retainAutoreleasedReturnValue` 函数是成对存在的，与之对应的函数是`objc_autoreleaseReturnValue` 。它用于array类方法返回对象的实现上。

下面看看 `NSMutableArray` 类的 `array` 方法通过编译器进行了怎样的转换：

```objectivec
    + (id)array 
    {
        return [[NSMutableArray alloc] init];
    }
```

```objectivec
    //编译器模拟代码
    + (id)array 
    {
        id obj = objc_msgSend(NSMutableArray,@selector(alloc));
        objc_msgSend(obj,@selector(init));
        
        // 代替我们调用了autorelease方法
        return objc_autoreleaseReturnValue(obj);
    }
```

我们可以看见调用了 `objc_autoreleaseReturnValue` 函数，且**这个函数会返回注册到自动释放池的对象，但是，这个函数有个特点，它会查看调用方的命令执行列表，如果发现接下来会调用 `objc_retainAutoreleasedReturnValue` 则不会将返回的对象注册到autoreleasePool中，而仅仅返回一个对象。达到了一种最优效果**。如下图：



![1.png](http://api.cocoachina.com/uploads//20180309/1520577862996624.png)

省略了autoreleasePool注册

### `__weak`

`__weak` 表示弱引用，对应定义 property 时用到的 weak。弱引用不会影响对象的释放，而当对象被释放时，所有指向它的弱引用都会自动被置为  nil，这样可以防止野指针。**使用 `__weak` 修饰的变量，即是使用注册到autoreleasePool中的对象**。`__weak`  最常见的一个作用就是用来避免循环引用。需要注意的是，`__weak` 修饰符只能用于 iOS5 以上的版本，在 iOS4 及更低的版本中使用  `__unsafe_unretained` 修饰符来代替。

`__weak` 的几个使用场景：

- 在 Delegate 关系中防止循环引用。
- 在 Block 中防止循环引用。
- 用来修饰指向由 Interface Builder 创建的控件。比如：`@property (weak, nonatomic) IBOutlet UIButton *testButton;` 。

**原理**

```objectivec
    {
        id __weak obj = [[NSObject alloc] init];
    }
```

编译器转换后的代码如下:

```objectivec
    id obj;
    id tmp = objc_msgSend(NSObject,@selector(alloc));
    objc_msgSend(tmp,@selector(init));
    objc_initweak(&obj,tmp);
    objc_release(tmp);
    objc_destroyWeak(&object);
```

对于 `__weak` 内存管理也借助了类似于引用计数表的散列表，它**通过对象的内存地址做为key，而对应的 `__weak` 修饰符变量的地址作为value注册到weak表中** 。在上述代码中， `objc_initweak` 就是完成这部分操作，而 `objc_destroyWeak` 则是销毁该对象对应的value。

当指向的对象被销毁时，会通过其内存地址，去weak表中查找对应的 `__weak` 修饰符变量，将其从weak表中删除。所以，weak在修饰只是让weak表增加了记录没有引起引用计数表的变化.



**对象通过 `objc_release` 释放对象内存的动作如下**:

1. objc_release
2. 因为引用计数为0所以执行dealloc
3. _objc_rootDealloc
4. objc_dispose
5. objc_destructInstance
6. objc_clear_deallocating

而在对象被废弃时最后调用了 `objc_clear_deallocating`，该函数的动作如下:

1. 从weak表中获取已废弃对象内存地址对应的所有记录；
2. 将已废弃对象内存地址对应的记录中所有以weak修饰的变量都置为nil；
3. 从weak表删除已废弃对象内存地址对应的记录；
4. 根据已废弃对象内存地址从引用计数表中找到对应记录删除。



据此可以解释为什么对象被销毁时对应的weak指针变量全部都置为nil，同时，也看出来，**销毁weak步骤较多，如果大量使用weak的话会增加CPU的负荷。**

还需要确认一点是：`__weak` 修饰符的变量，即是使用注册到autoreleasePool中的对象。

```objectivec
    {
        id __weak obj1 = obj; 
        NSLog(@"obj2-%@",obj1);
    }
```

编译器转换上述代码如下:

```objectivec
    id obj1;
    objc_initweak(&obj1,obj);
    id tmp = objc_loadWeakRetained(&obj1);
    objc_autorelease(tmp);
    NSLog(@"%@",tmp);
    objc_destroyWeak(&obj1);
```

`objc_loadWeakRetained` 函数获取附有 `__weak` 修饰符变量所引用的对象并retain,  `objc_autorelease` 函数将对象放入autoreleasePool中，据此当我们访问weak修饰指针指向的对象时，实际上是访问注册到自动释放池的对象。因此，**如果大量使用weak的话，在我们去访问weak修饰的对象时，会有大量对象注册到自动释放池，这会影响程序的性能。**

解决方案：要访问weak修饰的变量时，先将其赋给一个strong变量，然后进行访问。



问：为什么访问weak修饰的对象就会访问注册到自动释放池的对象呢?

因为weak不会引起对象的引用计数器变化，因此，该对象在运行过程中很有可能会被释放。所以，需要将对象注册到自动释放池中并在autoreleasePool销毁时释放对象占用的内存。



### `__unsafe_unretained`

ARC 是在 iOS5 引入的，而 `__unsafe_unretained`  这个修饰符主要是为了在ARC刚发布时兼容iOS4以及版本更低的系统，因为这些版本没有弱引用机制。这个修饰符在定义property时对应的是 `__unsafe_unretained`。`__unsafe_unretained` 修饰的指针纯粹只是指向对象，没有任何额外的操作，不会去持有对象使得对象的 retainCount  +1。而在指向的对象被释放时依然原原本本地指向原来的对象地址，不会被自动置为 nil，所以成为了野指针，非常不安全。

`__unsafe_unretained` 的应用场景：

在 ARC 环境下但是要兼容 iOS4.x 的版本，用`__unsafe_unretained` 替代 `__weak` 解决强循环循环的问题。



### `__autoreleasing`

将对象赋值给附有__autoreleasing修饰符的变量等同于MRC时调用对象的autorelease方法。

```objectivec
    @autoeleasepool {
        // 如果看了上面__strong的原理，就知道实际上对象已经注册到自动释放池里面了 
        id __autoreleasing obj = [[NSObject alloc] init];
    }
```

编译器转换上述代码如下:

```objectivec
    id pool = objc_autoreleasePoolPush(); 
    id obj = objc_msgSend(NSObject,@selector(alloc));
    objc_msgSend(obj,@selector(init));
    objc_autorelease(obj);
    objc_autoreleasePoolPop(pool);
```





```objectivec
	  @autoreleasepool {
        id __autoreleasing obj = [NSMutableArray array];
    }
```

编译器转换上述代码如下:

```objectivec
    id pool = objc_autoreleasePoolPush();
    id obj = objc_msgSend(NSMutableArray,@selector(array));
    objc_retainAutoreleasedReturnValue(obj);
    objc_autorelease(obk);
    objc_autoreleasePoolPop(pool);
```

上面两种方式，虽然第二种持有对象的方法从alloc方法变为了`objc_retainAutoreleasedReturnValue` 函数，都是通过objc_autorelease，注册到autoreleasePool中。

**ARC 模式规则**

ARC 模式下，还有一些需要注意的规则：

- 不能显式使用 retain/release/retainCount/autorelease。
- 不能使用 `NSAllocateObject` / `NSDeallocateObject`。
- 需要遵守内存管理的方法命名规则。在 ARC 模式和 MRC 模式下，以 alloc/new/copy/mutableCopy  开头的方法在返回对象时都必须返回给调用方所应当持有的对象。在 ARC 模式下，追加一条：以 init  开头的方法必须是实例方法并且必须要返回对象。返回的对象应为 id  类型或声明该方法的类的对象类型，或是该类的超类型或子类型。该返回的对象并不注册到 Autorelease Pool 中，基本上只是对 alloc 方法返回值的对象进行初始化处理并返回该对象。需要注意的是：- (void)initialize; 方法虽然是以 init  开头但是并不包含在上述规则中。
- 不要显式调用 dealloc。
- 使用 `@autoreleasepool` 块替代 `NSAutoreleasePool`。
- 不能使用区域（NSZone）。
- 对象型变量不能作为 C 语言结构体（struct/union）的成员。
- 显式转换 id 和 void *。



### Toll-Free Bridging

MRC 下的 Toll－Free Bridging 因为不涉及内存管理的转移，相互之间可以直接交换使用，当使用 ARC 时，由于Core  Foundation 框架并不支持 ARC，此时编译器不知道该如何处理这个同时有 ObjC 指针和 CFTypeRef  指向的对象，所以除了转换类型，还需指定内存管理所有权的改变，可通过 `__bridge`、`__bridge_retained` 和  `CFBridgingRetain`、`__bridge_transfer` 和 `CFBridgingRelease`。

#### 一、`__bridge`

只是声明类型转变，但是不做内存管理规则的转变。比如：

```objectivec
CFStringRef s1 = (__bridge CFStringRef) [[NSString alloc] initWithFormat:@"Hello, %@!", name];
```

只是做了 NSString 到 CFStringRef 的转化，但管理规则未变，依然要用 Objective-C 类型的 ARC 来管理 s1，你不能用 CFRelease() 去释放 s1。



#### 二、`__bridge_retained` or `CFBridgingRetain`

表示将指针类型转变的同时，将内存管理的责任由原来的 Objective-C 交给Core Foundation 来处理，也就是，将 ARC 转变为 MRC。比如，还是上面那个例子

```objectivec
NSString *s1 = [[NSString alloc] initWithFormat:@"Hello, %@!", name];
CFStringRef s2 = (__bridge_retained CFStringRef)s1;
// or CFStringRef s2 = (CFStringRef)CFBridgingRetain(s1);
// do something with s2
//...
CFRelease(s2); // 注意要在使用结束后加这个
```

我们在第二行做了转化，这时内存管理规则由 ARC 变为了 MRC，我们需要手动的来管理 s2 的内存，而对于 s1，我们即使将其置为 nil，也不能释放内存。



#### 三、`__bridge_transfer` or `CFBridgingRelease`

这个修饰符和函数的功能和上面那个 `__bridge_retained` 相反，它表示将管理的责任由 Core Foundation 转交给 Objective-C，即将管理方式由 MRC 转变为 ARC。比如：

```objectivec
CFStringRef result = CFURLCreateStringByAddingPercentEscapes(. . .);
NSString *s = (__bridge_transfer NSString *)result;
//or NSString *s = (NSString *)CFBridgingRelease(result);
return s;
```

这里我们将 result 的管理责任交给了 ARC 来处理，我们就不需要再显式地将 CFRelease() 了。



## 其他 

之前写的和内存管理有些许相关：[为什么声明NString，NSArray等需要使用copy，使用strong有什么问题，深拷贝和浅拷贝，block为什么使用copy。](https://www.jianshu.com/p/1e1a6f9c26f8)





参考文献和链接：

- [《Objective-C高级编程》](https://link.jianshu.com/?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FObjective-C%E9%AB%98%E7%BA%A7%E7%BC%96%E7%A8%8B%2F12345038%3Ffr%3Daladdin)
- [终于明白那些年知其然而不知其所以然的iOS内存管理方式](https://www.jianshu.com/p/4f49c5c81021)
- [深入理解RunLoop](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)



---

【完】