来自：[2019 iOS面试题-----RunLoop数据结构、RunLoop的实现机制、RunLoop的Mode、RunLoop与NSTimer和线程](https://juejin.im/post/5e134a025188253aa666acba)

###### RunLoop概念

###### RunLoop的数据结构

###### RunLoop的Mode

###### RunLoop的实现机制

###### RunLoop与NSTimer

###### RunLoop和线程

#### 一、RunLoop概念

###### RunLoop是通过内部维护的事件循环(Event Loop)来对事件/消息进行管理的一个对象。

1、没有消息处理时，休眠已避免资源占用，由用户态切换到内核态([CPU-内核态和用户态](https://www.jianshu.com/p/3bb1cdd44ef0))
 2、有消息需要处理时，立刻被唤醒，由内核态切换到用户态

###### 为什么main函数不会退出？

```objectivec
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

UIApplicationMain内部默认开启了主线程的RunLoop，并执行了一段无限循环的代码（不是简单的for循环或while循环）

```objectivec
//无限循环代码模式(伪代码)
int main(int argc, char * argv[]) {        
    BOOL running = YES;
    do {
        // 执行各种任务，处理各种事件
        // ......
    } while (running);

    return 0;
}
```

UIApplicationMain函数一直没有返回，而是不断地接收处理消息以及等待休眠，所以运行程序之后会保持持续运行状态。

#### 二、RunLoop的数据结构

`NSRunLoop(Foundation)`是`CFRunLoop(CoreFoundation)`的封装，提供了面向对象的API
 RunLoop 相关的主要涉及五个类：

- `CFRunLoop`：RunLoop对象
- `CFRunLoopMode`：运行模式
- `CFRunLoopSource`：输入源/事件源
- `CFRunLoopTimer`：定时源
- `CFRunLoopObserver`：观察者



###### 1、CFRunLoop

由`pthread`(线程对象，说明RunLoop和线程是一一对应的)、`currentMode`(当前所处的运行模式)、`modes`(多个运行模式的集合)、`commonModes`(模式名称字符串集合)、`commonModelItems`(Observer,Timer,Source集合)构成。

RunLoop其实就是一个对象，它的结构如下，源码看这里：

```c
struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;  /* locked for accessing mode list */ //互斥锁
    __CFPort _wakeUpPort;   // used for CFRunLoopWakeUp 内核向该端口发送消息可以唤醒runloop
    Boolean _unused;
    volatile _per_run_data *_perRunData; // reset for runs of the run loop
    pthread_t _pthread;             //RunLoop对应的线程
    uint32_t _winthread;
    CFMutableSetRef _commonModes;    //存储的是字符串，记录所有标记为common的mode
    CFMutableSetRef _commonModeItems;//存储所有commonMode的item(source、timer、observer)
    CFRunLoopModeRef _currentMode;   //当前运行的mode
    CFMutableSetRef _modes;          //存储的是CFRunLoopModeRef
    struct _block_item *_blocks_head;//doblocks的时候用到
    struct _block_item *_blocks_tail;
    CFTypeRef _counterpart;
};
```



###### 2、CFRunLoopMode

由name、source0、source1、observers、timers构成

它的结构如下：

```c
struct __CFRunLoopMode {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;  /* must have the run loop locked before locking this */
    CFStringRef _name;   //mode名称
    Boolean _stopped;    //mode是否被终止
    char _padding[3];
    //几种事件
    CFMutableSetRef _sources0;  //sources0
    CFMutableSetRef _sources1;  //sources1
    CFMutableArrayRef _observers; //通知
    CFMutableArrayRef _timers;    //定时器
    CFMutableDictionaryRef _portToV1SourceMap; //字典  key是mach_port_t，value是CFRunLoopSourceRef
    __CFPortSet _portSet; //保存所有需要监听的port，比如_wakeUpPort，_timerPort都保存在这个数组中
    CFIndex _observerMask;
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    dispatch_source_t _timerSource;
    dispatch_queue_t _queue;
    Boolean _timerFired; // set to true by the source when a timer has fired
    Boolean _dispatchTimerArmed;
#endif
#if USE_MK_TIMER_TOO
    mach_port_t _timerPort;
    Boolean _mkTimerArmed;
#endif
#if DEPLOYMENT_TARGET_WINDOWS
    DWORD _msgQMask;
    void (*_msgPump)(void);
#endif
    uint64_t _timerSoftDeadline; /* TSR */
    uint64_t _timerHardDeadline; /* TSR */
};
```



###### 3、CFRunLoopSource

分为source0和source1两种

- `source0`:
   即非基于port的，也就是用户触发的事件。需要手动唤醒线程，将当前线程从内核态切换到用户态
- `source1`:
   基于port的，包含一个 mach_port 和一个回调，可监听系统端口和通过内核和其他线程发送的消息，能主动唤醒RunLoop，接收分发系统事件。
   具备唤醒线程的能力



根据官方的描述，CFRunLoopSource是对input sources的抽象。CFRunLoopSource分source0和source1两个版本，它的结构如下：

```c
struct __CFRunLoopSource {
    CFRuntimeBase _base;
    uint32_t _bits; //用于标记Signaled状态，source0只有在被标记为Signaled状态，才会被处理
    pthread_mutex_t _lock;
    CFIndex _order;         /* immutable 不可变的 */ 
    CFMutableBagRef _runLoops;
    union {
        CFRunLoopSourceContext version0;  //source0   /* immutable, except invalidation 不可变的，除非失效 */
        CFRunLoopSourceContext1 version1; //source1   /* immutable, except invalidation 不可变的，除非失效 */
    } _context;
};
```

source0是App内部事件，**由App自己管理的UIEvent、CFSocket都是source0。当一个source0事件准备执行的时候，必须要先把它标记为signal状态**。以下是source0的结构体：

```c
typedef struct {
    CFIndex version;
    void *  info;
    const void *(*retain)(const void *info);
    void    (*release)(const void *info);
    CFStringRef (*copyDescription)(const void *info);
    Boolean (*equal)(const void *info1, const void *info2);
    CFHashCode  (*hash)(const void *info);
    void    (*schedule)(void *info, CFRunLoopRef rl, CFStringRef mode); //安排，预定
    void    (*cancel)(void *info, CFRunLoopRef rl, CFStringRef mode); //取消
    void    (*perform)(void *info);	//执行，表演
} CFRunLoopSourceContext;
```

source0是非基于Port的。只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用 `CFRunLoopSourceSignal(source)` ，将这个 Source 标记为待处理，然后手动调用 `CFRunLoopWakeUp(runloop)`  来唤醒 RunLoop，让其处理这个事件。



source1由RunLoop和内核管理，source1带有 `mach_port_t`，可以接收内核消息并触发回调。以下是source1的结构体：

```c
typedef struct {
    CFIndex version;
    void *  info;
    const void *(*retain)(const void *info);
    void    (*release)(const void *info);
    CFStringRef (*copyDescription)(const void *info);
    Boolean (*equal)(const void *info1, const void *info2);
    CFHashCode  (*hash)(const void *info);
#if (TARGET_OS_MAC && !(TARGET_OS_EMBEDDED || TARGET_OS_IPHONE)) || (TARGET_OS_EMBEDDED || TARGET_OS_IPHONE)
    mach_port_t (*getPort)(void *info);	//比source0 多的字段
    void *  (*perform)(void *msg, CFIndex size, CFAllocatorRef allocator, void *info);
#else
    void *  (*getPort)(void *info);	//比source0 多的字段
    void    (*perform)(void *info);
#endif
} CFRunLoopSourceContext1;
```

Source1除了包含回调指针外，包含一个mach port，Source1可以监听系统端口和通过内核和其他线程通信，接收、分发系统事件，它能够主动唤醒RunLoop(由操作系统内核进行管理，例如 CFMessagePort 消息)。**官方也指出可以自定义Source，因此对于 `CFRunLoopSourceRef` 来说它更像一种协议**，框架已经默认定义了两种实现，如果有必要开发人员也可以自定义，详细情况可以查看官方文档。



###### 4、CFRunLoopTimer

基于时间的触发器，基本上说的就是NSTimer。在预设的时间点唤醒RunLoop执行回调。**因为它是基于RunLoop的，因此它不是实时的**（就是NSTimer 是不准确的。 因为RunLoop只负责分发源的消息。如果线程当前正在处理繁重的任务，就有可能导致Timer本次延时，或者少执行一次）。



`CFRunLoopTimer` 是定时器，可以在设定的时间点抛出回调，它的结构如下：

```c
struct __CFRunLoopTimer {
    CFRuntimeBase _base;
    uint16_t _bits;  //标记fire状态
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;        //添加该timer的runloop
    CFMutableSetRef _rlModes;     //存放所有 包含该timer的 mode的 modeName，意味着一个timer可能会在多个mode中存在
    CFAbsoluteTime _nextFireDate;
    CFTimeInterval _interval;     //理想时间间隔  /* immutable */
    CFTimeInterval _tolerance;    //时间偏差      /* mutable */
    uint64_t _fireTSR;          /* TSR units */
    CFIndex _order;         /* immutable */
    CFRunLoopTimerCallBack _callout;    /* immutable */
    CFRunLoopTimerContext _context; /* immutable, except invalidation */
};
```

另外根据官方文档的描述，**`CFRunLoopTimer` 和 `NSTimer` 是toll-free bridged的，可以相互转换。**

> CFRunLoopTimer is “toll-free bridged” with its Cocoa Foundation counterpart, NSTimer. This means that the Core Foundation type is interchangeable in function or method calls with the bridged Foundation object.

所以CFRunLoopTimer具有以下特性：

- CFRunLoopTimer 是定时器，可以在设定的时间点抛出回调
- CFRunLoopTimer和NSTimer是toll-free bridged的，可以相互转换



###### 5、CFRunLoopObserver

CFRunLoopObserver是观察者，可以观察RunLoop的各种状态，并抛出回调。

```c
struct __CFRunLoopObserver {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;
    CFIndex _rlCount;
    CFOptionFlags _activities;      /* immutable 不可变的 */
    CFIndex _order;         /* immutable 不可变的 */
    CFRunLoopObserverCallBack _callout; /* immutable */
    CFRunLoopObserverContext _context;  /* immutable, except invalidation 不可变的，除非失效 */
};
```

`CFRunLoopObserver` 可以观察的状态有如下6种:

```c
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),           //即将进入run loop
    kCFRunLoopBeforeTimers = (1UL << 1),    //即将处理timer
    kCFRunLoopBeforeSources = (1UL << 2),   //即将处理source
    kCFRunLoopBeforeWaiting = (1UL << 5),   //即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),    //被唤醒但是还没开始处理事件
    kCFRunLoopExit = (1UL << 7),            //run loop已经退出
    kCFRunLoopAllActivities = 0x0FFFFFFFU   //监听所有状态
};
```

Runloop 通过监控 Source 来决定有没有任务要做，除此之外，我们还可以用 Runloop Observer 来监控 Runloop 本身的状态。 Runloop Observer 可以监控上面的 Runloop 事件，具体流程如下图。



![img](https://user-gold-cdn.xitu.io/2019/8/15/16c94eae501bee6b?imageView2/0/w/1280/h/960/ignore-error/1)



#### 

监听以下时间点:`CFRunLoopActivity`

- `kCFRunLoopEntry`             
   RunLoop准备启动
- `kCFRunLoopBeforeTimers`      
   RunLoop将要处理一些Timer相关事件
- `kCFRunLoopBeforeSources`     
   RunLoop将要处理一些Source事件
- `kCFRunLoopBeforeWaiting`        
   RunLoop将要进行休眠状态，即将由用户态切换到内核态
- `kCFRunLoopAfterWaiting`
   RunLoop被唤醒，即从内核态切换到用户态后
- `kCFRunLoopExit`
   RunLoop退出
- `kCFRunLoopAllActivities`
   监听所有状态



###### 6、各数据结构之间的联系

线程和RunLoop一一对应， RunLoop和Mode是一对多的，Mode和source、timer、observer也是一对多的



![img](https://user-gold-cdn.xitu.io/2020/1/6/16f7b5954d21a667?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image.png

#### 三、RunLoop的Mode

关于Mode首先要知道**一个RunLoop 对象中可能包含多个Mode，且每次调用 RunLoop 的主函数时，只能指定其中一个 Mode(CurrentMode)。切换 Mode，需要重新指定一个 Mode 。主要是为了分隔开不同的 Source、Timer、Observer，让它们之间互不影响。**

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f7b59547459a49?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image.png



当RunLoop运行在Mode1上时，是无法接受处理Mode2或Mode3上的Source、Timer、Observer事件的

总共是有五种`CFRunLoopMode`:

- `kCFRunLoopDefaultMode`：默认模式，主线程是在这个运行模式下运行
- `UITrackingRunLoopMode`：跟踪用户交互事件（用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他Mode影响）
- `UIInitializationRunLoopMode`：在刚启动App时第进入的第一个 Mode，启动完成后就不再使用
- `GSEventReceiveRunLoopMode`：接受系统内部事件，通常用不到
- `kCFRunLoopCommonModes`：伪模式，不是一种真正的运行模式，是同步Source/Timer/Observer到多个Mode中的一种解决方案

#### 四、RunLoop的实现机制

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f7b5953de1489a?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image.png

这张图在网上流传比较广。

对于RunLoop而言最核心的事情就是保证线程在没有消息的时候休眠，在有消息时唤醒，以提高程序性能。RunLoop这个机制是依靠系统内核来完成的（苹果操作系统核心组件Darwin中的Mach）。



![img](https://user-gold-cdn.xitu.io/2020/1/6/16f7b5955374dc43?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image.png

 RunLoop通过`mach_msg()`函数接收、发送消息。它的本质是调用函数`mach_msg_trap()`，相当于是一个系统调用，会触发内核状态切换。在用户态调用 `mach_msg_trap()`时会切换到内核态；内核态中内核实现的`mach_msg()`函数会完成实际的工作。

即基于port的source1，监听端口，端口有消息就会触发回调；而source0，要手动标记为待处理和手动唤醒RunLoop


[Mach消息发送机制](https://www.jianshu.com/p/a764aad31847)

大致逻辑为：

1. 通知观察者 RunLoop 即将启动。
2. 通知观察者即将要处理Timer事件。
3. 通知观察者即将要处理source0事件。
4. 处理source0事件。
5. 如果基于端口的源(Source1)准备好并处于等待状态，进入步骤9。
6. 通知观察者线程即将进入休眠状态。
7. 将线程置于休眠状态，由用户态切换到内核态，直到下面的任一事件发生才唤醒线程。

	- 一个基于 port 的Source1 的事件(图里应该是source0)。
	- 一个 Timer 到时间了。
	- RunLoop 自身的超时时间到了。
	- 被其他调用者手动唤醒。

8. 通知观察者线程将被唤醒。
9. 处理唤醒时收到的事件。

	- 如果用户定义的定时器启动，处理定时器事件并重启RunLoop。进入步骤2。
	- 如果输入源启动，传递相应的消息。
	- 如果RunLoop被显示唤醒而且时间还没超时，重启RunLoop。进入步骤2

10. 通知观察者RunLoop结束。



#### 五、RunLoop与NSTimer

一个比较常见的问题：滑动tableView时，定时器还会生效吗？

默认情况下RunLoop运行在`kCFRunLoopDefaultMode`下，而当滑动tableView时，RunLoop切换到`UITrackingRunLoopMode`，而Timer是在`kCFRunLoopDefaultMode`下的，就无法接受处理Timer的事件。

怎么去解决这个问题呢？把Timer添加到`UITrackingRunLoopMode`上并不能解决问题，因为这样在默认情况下就无法接受定时器事件了。

所以我们需要把Timer同时添加到`UITrackingRunLoopMode`和`kCFRunLoopDefaultMode`上。

那么如何把timer同时添加到多个mode上呢？就要用到`NSRunLoopCommonModes`了

```objectivec
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

Timer就被添加到多个mode上，这样即使RunLoop由`kCFRunLoopDefaultMode`切换到`UITrackingRunLoopMode`下，也不会影响接收Timer事件。



#### 六、RunLoop和线程

- 线程和RunLoop是一一对应的，其映射关系是保存在一个全局的 Dictionary 里；
- 自己创建的线程默认是没有开启RunLoop的。



###### 1、怎么创建一个常驻线程？

1. 为当前线程开启一个RunLoop（第一次调用 [NSRunLoop currentRunLoop]方法时实际是会先去创建一个RunLoop）
2. 向当前RunLoop中添加一个Port/Source等维持RunLoop的事件循环（如果RunLoop的mode中一个item都没有，RunLoop会退出）
3. 启动该RunLoop。

```objectivec
   @autoreleasepool {
        
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        
        [[NSRunLoop currentRunLoop] addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        
        [runLoop run];
        
    }
```



###### 2、输出下边代码的执行顺序

```objectivec
 NSLog(@"1");

dispatch_async(dispatch_get_global_queue(0, 0), ^{
    
    NSLog(@"2");

    [self performSelector:@selector(test) withObject:nil afterDelay:10];
    
    NSLog(@"3");
});

NSLog(@"4");

- (void)test
{
    
    NSLog(@"5");
}
```

答案是1423，test方法并不会执行。

原因是：**带 `afterDelay` 的延时函数，会在内部创建一个 NSTimer，然后添加到当前线程的RunLoop中。也就是如果当前线程没有开启RunLoop，该方法会失效。**

那么我们改成：

```objectivec
dispatch_async(dispatch_get_global_queue(0, 0), ^{
        
        NSLog(@"2");
        
        [[NSRunLoop currentRunLoop] run];
        
        [self performSelector:@selector(test) withObject:nil afterDelay:10];
  
        NSLog(@"3");
    });
```

然而test方法依然不执行。

原因是：**如果RunLoop的mode中一个item都没有，RunLoop会退出。即在调用RunLoop的run方法后，由于其mode中没有添加任何item去维持RunLoop的时间循环，RunLoop随即还是会退出。**

所以我们自己启动RunLoop，一定要在添加item后：

```objectivec
dispatch_async(dispatch_get_global_queue(0, 0), ^{
        
        NSLog(@"2");
        
        [self performSelector:@selector(test) withObject:nil afterDelay:10];
        
        [[NSRunLoop currentRunLoop] run];
  
        NSLog(@"3");
    });
```



###### 3、怎样保证子线程数据回来更新UI的时候不打断用户的滑动操作？

当我们在子请求数据的同时滑动浏览当前页面，如果数据请求成功要切回主线程更新UI，那么就会影响当前正在滑动的体验。

我们就可以**将更新UI事件放在主线程的`NSDefaultRunLoopMode`上执行即可，这样就会等用户不再滑动页面，主线程RunLoop由`UITrackingRunLoopMode`切换到`NSDefaultRunLoopMode`时再去更新UI** 。

```objectivec
[self performSelectorOnMainThread:@selector(reloadData) withObject:nil waitUntilDone:NO modes:@[NSDefaultRunLoopMode]];
```



---

【完】


















