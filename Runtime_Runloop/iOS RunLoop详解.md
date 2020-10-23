来自：[iOS RunLoop详解](https://juejin.im/post/5d5537d56fb9a06aea618ddc)



---



![](https://user-gold-cdn.xitu.io/2019/8/15/16c94e230f8fd6df?imageView2/0/w/1280/h/960/ignore-error/1)



Runloop 是和线程紧密相关的一个基础组件，是很多线程有关功能的幕后功臣。尽管在平常使用中几乎不太会直接用到，理解 Runloop 有利于我们更加深入地理解 iOS 的多线程模型。

本文从如下几个方面理解RunLoop的相关知识点。

- RunLoop概念
- RunLoop实现
- RunLoop运行
- RunLoop应用

### RunLoop概念

#### RunLoop介绍

> RunLoop 是什么？RunLoop 还是比较顾名思义的一个东西，说白了就是一种循环，只不过它这种循环比较高级。一般的 while 循环会导致 CPU 进入忙等待状态，而 RunLoop 则是一种“闲”等待，这部分可以类比 Linux 下的 epoll。当没有事件时，RunLoop 会进入休眠状态，有事件发生时， RunLoop 会去找对应的 Handler 处理事件。RunLoop 可以让线程在需要做事的时候忙起来，不需要的话就让线程休眠。

从代码上看，RunLoop其实就是一个对象，它的结构如下，源码看这里：

```c
struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;  /* locked for accessing mode list */
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

可见，一个RunLoop对象，主要包含了一个线程，若干个Mode，若干个commonMode，还有一个当前运行的Mode。

#### RunLoop与线程

当我们需要一个常驻线程，可以让线程在需要做事的时候忙起来，不需要的话就让线程休眠。我们就在线程里面执行下面这个代码，一直等待消息，线程就不会退出了。

```c
do {
   //获取消息
   //处理消息
} while (消息 ！= 退出)
复
```

上面的这种循环模型被称作 Event Loop，事件循环模型在众多系统里都有实现，RunLoop 实际上就是一个对象，这个对象管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部 "接受消息->等待->处理" 的循环中，直到这个循环结束（比如传入 quit 的消息），函数返回。

下图描述了Runloop运行流程（基本描述了上面Runloop的核心流程，当然可以查看官方`The Run Loop Sequence of Events`描述）：



![img](https://user-gold-cdn.xitu.io/2019/8/15/16c94e526bf8baaa?imageView2/0/w/1280/h/960/ignore-error/1)



整个流程并不复杂（需要注意的就是_黄色_区域的消息处理中并不包含source0，因为它在循环开始之初就会处理），整个流程其实就是一种Event Loop的实现，其他平台均有类似的实现，只是这里叫做RunLoop。

RunLoop与线程的关系如下图



![img](https://user-gold-cdn.xitu.io/2019/8/15/16c94e5ff5fb254d?imageView2/0/w/1280/h/960/ignore-error/1)



> 图中展现了 Runloop 在线程中的作用：从 input source 和 timer source 接受事件，然后在线程中处理事件。

Runloop 和线程是绑定在一起的。每个线程（包括主线程）都有一个对应的 Runloop 对象。我们并不能自己创建 Runloop 对象，但是可以获取到系统提供的 Runloop 对象。

主线程的 Runloop 会在应用启动的时候完成启动，其他线程的 Runloop 默认并不会启动，需要我们手动启动。

#### RunLoop Mode

Mode可以视为事件的管家，一个Mode管理着各种事件，它的结构如下：


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

一个CFRunLoopMode对象有一个name，若干source0、source1、timer、observer和若干port，可见事件都是由Mode在管理，而RunLoop管理Mode。

从源码很容易看出，Runloop总是运行在某种特定的 `CFRunLoopModeRef` 下（每次运行**`__CFRunLoopRun()` 函数时必须指定Mode）。而通过`CFRunloopRef`对应结构体的定义可以很容易知道，每种Runloop都可以包含若干个Mode，每个Mode又包含Source/Timer/Observer。每次调用Runloop的主函数 `__CFRunLoopRun()`时必须指定一种Mode，这个Mode称为 `_currentMode`**，当切换Mode时必须退出当前Mode，然后重新进入Runloop，以保证不同Mode的Source/Timer/Observer互不影响。



![img](https://user-gold-cdn.xitu.io/2019/8/15/16c94e7597fc94ed?imageView2/0/w/1280/h/960/ignore-error/1)



如图所示，Runloop Mode 实际上是 Source，Timer 和 Observer 的集合，不同的 Mode 把不同组的 Source，Timer 和 Observer 隔绝开来。Runloop 在某个时刻只能跑在一个 Mode 下，处理这一个 Mode 当中的 Source，Timer 和 Observer。

苹果文档中提到的 Mode 有五个，分别是：

- NSDefaultRunLoopMode
- NSConnectionReplyMode     注：Connection 连接  Reply  回复
- NSModalPanelRunLoopMode     注：Modal  adj. 模式的；情态的；  Panel  n. 仪表板；嵌板；
- NSEventTrackingRunLoopMode
- NSRunLoopCommonModes

iOS 中公开暴露出来的只有 `NSDefaultRunLoopMode` 和 `NSRunLoopCommonModes`。 NSRunLoopCommonModes 实际上是一个 Mode 的集合，默认包括 NSDefaultRunLoopMode 和 NSEventTrackingRunLoopMode（注意：并不是说Runloop会运行在kCFRunLoopCommonModes这种模式下，而是相当于分别注册了 NSDefaultRunLoopMode和 UITrackingRunLoopMode。当然你也可以通过调用`CFRunLoopAddCommonMode()` 方法将自定义Mode放到 `kCFRunLoopCommonModes` 组合）。 五种Mode的介绍如下图：



![img](https://user-gold-cdn.xitu.io/2019/8/15/16c94e839312f7d6?imageView2/0/w/1280/h/960/ignore-error/1)



#### RunLoop Source

Run Loop Source分为Source、Observer、Timer三种，他们统称为ModeItem。

#### CFRunLoopSource

根据官方的描述，CFRunLoopSource是对input sources的抽象。CFRunLoopSource分source0和source1两个版本，它的结构如下：

```c
struct __CFRunLoopSource {
    CFRuntimeBase _base;
    uint32_t _bits; //用于标记Signaled状态，source0只有在被标记为Signaled状态，才会被处理
    pthread_mutex_t _lock;
    CFIndex _order;         /* immutable */
    CFMutableBagRef _runLoops;
    union {
        CFRunLoopSourceContext version0;     /* immutable, except invalidation */
        CFRunLoopSourceContext1 version1;    /* immutable, except invalidation */
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
    void    (*perform)(void *info); //执行，表演
} CFRunLoopSourceContext;
```

source0是非基于Port的。只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用 `CFRunLoopSourceSignal(source)` ，将这个 Source 标记为待处理，然后手动调用 `CFRunLoopWakeUp(runloop)`  来唤醒 RunLoop，让其处理这个事件。

source1由RunLoop和内核管理，source1带有 `mach_port_t`，可以接收内核消息并触发回调。以下是source1的结构体

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
    mach_port_t (*getPort)(void *info); //比source0 多的字段
    void *  (*perform)(void *msg, CFIndex size, CFAllocatorRef allocator, void *info);
#else
    void *  (*getPort)(void *info); //比source0 多的字段
    void    (*perform)(void *info);
#endif
} CFRunLoopSourceContext1;
```

Source1除了包含回调指针外，包含一个mach port，Source1可以监听系统端口和通过内核和其他线程通信，接收、分发系统事件，它能够主动唤醒RunLoop(由操作系统内核进行管理，例如CFMessagePort消息)。**官方也指出可以自定义Source，因此对于CFRunLoopSourceRef来说它更像一种协议**，框架已经默认定义了两种实现，如果有必要开发人员也可以自定义，详细情况可以查看官方文档。

#### CFRunLoopObserver

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



#### CFRunLoopTimer

CFRunLoopTimer是定时器，可以在设定的时间点抛出回调，它的结构如下：

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

另外根据官方文档的描述，`CFRunLoopTimer` 和 `NSTimer` 是toll-free bridged的，可以相互转换。

> CFRunLoopTimer is “toll-free bridged” with its Cocoa Foundation counterpart, NSTimer. This means that the Core Foundation type is interchangeable in function or method calls with the bridged Foundation object.

所以CFRunLoopTimer具有以下特性：

- CFRunLoopTimer 是定时器，可以在设定的时间点抛出回调
- CFRunLoopTimer和NSTimer是toll-free bridged的，可以相互转换


## RunLoop实现

下面从以下3个方面介绍RunLoop的实现。

- 获取RunLoop
- 添加Mode
- 添加Run Loop Source

#### 获取RunLoop

从苹果开放的API来看，不允许我们直接创建RunLoop对象，只能通过以下几个函数来获取RunLoop:

- CFRunLoopRef CFRunLoopGetCurrent(void)
- CFRunLoopRef CFRunLoopGetMain(void)
- +(NSRunLoop *)currentRunLoop
- +(NSRunLoop *)mainRunLoop

前两个是Core Foundation中的API，后两个是Foundation中的API。

那么RunLoop是什么时候被创建的呢？

我们从下面几个函数内部看看。

#### CFRunLoopGetCurrent

```c
//取当前所在线程的RunLoop
CFRunLoopRef CFRunLoopGetCurrent(void) {
    CHECK_FOR_FORK();
    CFRunLoopRef rl = (CFRunLoopRef)_CFGetTSD(__CFTSDKeyRunLoop);
    if (rl) return rl;
    //传入当前线程
    return _CFRunLoopGet0(pthread_self());
}
```

在 `CFRunLoopGetCurrent` 函数内部调用了 `_CFRunLoopGet0()`，传入的参数是当前线程 `pthread_self()` 。这里可以看出，<font color=#038103>** `CFRunLoopGetCurrent` 函数必须要在线程内部调用，才能获取当前线程的RunLoop。也就是说子线程的RunLoop必须要在子线程内部获取。**</font>



#### CFRunLoopGetMain

```c
//取主线程的RunLoop
CFRunLoopRef CFRunLoopGetMain(void) {
    CHECK_FOR_FORK();
    static CFRunLoopRef __main = NULL; // no retain needed
    //传入主线程
    if (!__main) __main = _CFRunLoopGet0(pthread_main_thread_np()); // no CAS needed
    return __main;
}
```

在CFRunLoopGetMain函数内部也调用了 `_CFRunLoopGet0()`，传入的参数是主线程`pthread_main_thread_np()`。可以看出，<font color=#038103>`_CFRunLoopGetMain()`不管在主线程还是子线程中调用，都可以获取到主线程的RunLoop。</font>

####  _CFRunLoopGet0

前面两个函数都是使用了CFRunLoopGet0实现传入线程的函数，下面看下CFRunLoopGet0的结构是咋样的。

```c
static CFMutableDictionaryRef __CFRunLoops = NULL;
static CFSpinLock_t loopsLock = CFSpinLockInit;
 
// t==0 is a synonym for "main thread" that always works
//根据线程取RunLoop
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    if (pthread_equal(t, kNilPthreadT)) {
        t = pthread_main_thread_np();
    }
    __CFSpinLock(&loopsLock);
    //如果存储RunLoop的字典不存在
    if (!__CFRunLoops) {
        __CFSpinUnlock(&loopsLock);
        //创建一个临时字典dict
        CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        //创建主线程的RunLoop
        CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
        //把主线程的RunLoop保存到dict中，key是线程，value是RunLoop
        CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
        //此处NULL和__CFRunLoops指针都指向NULL，匹配，所以将dict写到__CFRunLoops
        if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
            //释放dict
            CFRelease(dict);
        }
        //释放mainrunloop
        CFRelease(mainLoop);
        __CFSpinLock(&loopsLock);
    }
    //以上说明，第一次进来的时候，不管是getMainRunloop还是get子线程的runloop，主线程的runloop总是会被创建
    //从字典__CFRunLoops中获取传入线程t的runloop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFSpinUnlock(&loopsLock);
    //如果没有获取到
    if (!loop) {
        //根据线程t创建一个runloop
        CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFSpinLock(&loopsLock);
        loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
        if (!loop) {
            //把newLoop存入字典__CFRunLoops，key是线程t
            CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
            loop = newLoop;
        }
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFSpinUnlock(&loopsLock);
        CFRelease(newLoop);
    }
    //如果传入线程就是当前线程
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            //注册一个回调，当线程销毁时，销毁对应的RunLoop
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```

这段代码可以得出以下结论：

- RunLoop和线程的一一对应的，对应的方式是以key-value的方式保存在一个全局字典中
- **主线程的RunLoop会在初始化全局字典时创**
- 子线程的RunLoop会在第一次获取的时候创建，如果不获取的话就一直不会被创建
- RunLoop会在线程销毁时销毁

#### 添加Mode

在Core Foundation中，针对Mode的操作，苹果只开放了以下3个API(Cocoa中也有功能一样的函数，不再列出):

- CFRunLoopAddCommonMode(CFRunLoopRef rl, CFStringRef mode)
- CFStringRef CFRunLoopCopyCurrentMode(CFRunLoopRef rl)
- CFArrayRef CFRunLoopCopyAllModes(CFRunLoopRef rl)

> CFRunLoopAddCommonMode Adds a mode to the set of run loop common modes. 向当前RunLoop的common modes中添加一个mode。 CFRunLoopCopyCurrentMode Returns the name of the mode in which a given run loop is currently running. 返回当前运行的mode的name CFRunLoopCopyAllModes Returns an array that contains all the defined modes for a CFRunLoop object. 返回当前RunLoop的所有mode

我们没有办法直接创建一个CFRunLoopMode对象，但是我们可以调用CFRunLoopAddCommonMode传入一个字符串向RunLoop中添加Mode，传入的字符串即为Mode的名字，Mode对象应该是此时在RunLoop内部创建的。下面来看一下源码。

### CFRunLoopAddCommonMode

```c
void CFRunLoopAddCommonMode(CFRunLoopRef rl, CFStringRef modeName) {
    CHECK_FOR_FORK();
    if (__CFRunLoopIsDeallocating(rl)) return;
    __CFRunLoopLock(rl);
    //看rl中是否已经有这个mode，如果有就什么都不做
    if (!CFSetContainsValue(rl->_commonModes, modeName)) {
        CFSetRef set = rl->_commonModeItems ? CFSetCreateCopy(kCFAllocatorSystemDefault, rl->_commonModeItems) : NULL;
        //把modeName添加到RunLoop的_commonModes中
        CFSetAddValue(rl->_commonModes, modeName);
        if (NULL != set) {
            CFTypeRef context[2] = {rl, modeName};
            /* add all common-modes items to new mode */
            //这里调用CFRunLoopAddSource/CFRunLoopAddObserver/CFRunLoopAddTimer的时候会调用
            //__CFRunLoopFindMode(rl, modeName, true)，CFRunLoopMode对象在这个时候被创建
            CFSetApplyFunction(set, (__CFRunLoopAddItemsToCommonMode), (void *)context);
            CFRelease(set);
        }
    } else {
    }
    __CFRunLoopUnlock(rl);
}
```

可以看得出：

- modeName不能重复，modeName是mode的唯一标识符
- RunLoop的_commonModes数组存放所有被标记为common的mode的名称
- 添加commonMode会把commonModeItems数组中的所有source同步到新添加的mode
- CFRunLoopMode对象在CFRunLoopAddItemsToCommonMode函数中调用CFRunLoopFindMode时被创建

#### CFRunLoopCopyCurrentMode/CFRunLoopCopyAllModes

CFRunLoopCopyCurrentMode和CFRunLoopCopyAllModes的内部逻辑比较简单，直接取RunLoop的_currentMode和_modes返回，就不贴源码了。

#### 添加Run Loop Source（ModeItem）

我们可以通过以下接口添加/移除各种事件:

- void CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef mode)
- void CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef mode)
- void CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef mode)
- void CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef * mode)
- void CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode)
- void CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode)

#### CFRunLoopAddSource

CFRunLoopAddSource的代码结构如下：

```c
//添加source事件
void CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef rls, CFStringRef modeName) {    /* DOES CALLOUT */
    CHECK_FOR_FORK();
    if (__CFRunLoopIsDeallocating(rl)) return;
    if (!__CFIsValid(rls)) return;
    Boolean doVer0Callout = false;
    __CFRunLoopLock(rl);
    //如果是kCFRunLoopCommonModes
    if (modeName == kCFRunLoopCommonModes) {
        //如果runloop的_commonModes存在，则copy一个新的复制给set
        CFSetRef set = rl->_commonModes ? CFSetCreateCopy(kCFAllocatorSystemDefault, rl->_commonModes) : NULL;
       //如果runl _commonModeItems为空
        if (NULL == rl->_commonModeItems) {
            //先初始化
            rl->_commonModeItems = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
        }
        //把传入的CFRunLoopSourceRef加入_commonModeItems
        CFSetAddValue(rl->_commonModeItems, rls);
        //如果刚才set copy到的数组里有数据
        if (NULL != set) {
            CFTypeRef context[2] = {rl, rls};
            /* add new item to all common-modes */
            //则把set里的所有mode都执行一遍__CFRunLoopAddItemToCommonModes函数
            CFSetApplyFunction(set, (__CFRunLoopAddItemToCommonModes), (void *)context);
            CFRelease(set);
        }
        //以上分支的逻辑就是，如果你往kCFRunLoopCommonModes里面添加一个source，那么所有_commonModes里的mode都会添加这个source
    } else {
        //根据modeName查找mode
        CFRunLoopModeRef rlm = __CFRunLoopFindMode(rl, modeName, true);
        //如果_sources0不存在，则初始化_sources0，_sources0和_portToV1SourceMap
        if (NULL != rlm && NULL == rlm->_sources0) {
            rlm->_sources0 = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
            rlm->_sources1 = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
            rlm->_portToV1SourceMap = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, NULL);
        }
        //如果_sources0和_sources1中都不包含传入的source
        if (NULL != rlm && !CFSetContainsValue(rlm->_sources0, rls) && !CFSetContainsValue(rlm->_sources1, rls)) {
            //如果version是0，则加到_sources0
            if (0 == rls->_context.version0.version) {
                CFSetAddValue(rlm->_sources0, rls);
                //如果version是1，则加到_sources1
            } else if (1 == rls->_context.version0.version) {
                CFSetAddValue(rlm->_sources1, rls);
                __CFPort src_port = rls->_context.version1.getPort(rls->_context.version1.info);
                if (CFPORT_NULL != src_port) {
                    //此处只有在加到source1的时候才会把souce和一个mach_port_t对应起来
                    //可以理解为，source1可以通过内核向其端口发送消息来主动唤醒runloop
                    CFDictionarySetValue(rlm->_portToV1SourceMap, (const void *)(uintptr_t)src_port, rls);
                    __CFPortSetInsert(src_port, rlm->_portSet);
                }
            }
            __CFRunLoopSourceLock(rls);
            //把runloop加入到source的_runLoops中
            if (NULL == rls->_runLoops) {
                rls->_runLoops = CFBagCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeBagCallBacks); // sources retain run loops!
            }
            CFBagAddValue(rls->_runLoops, rl);
            __CFRunLoopSourceUnlock(rls);
            if (0 == rls->_context.version0.version) {
                if (NULL != rls->_context.version0.schedule) {
                    doVer0Callout = true;
                }
            }
        }
        if (NULL != rlm) {
            __CFRunLoopModeUnlock(rlm);
        }
    }
    __CFRunLoopUnlock(rl);
    if (doVer0Callout) {
        // although it looses some protection for the source, we have no choice but
        // to do this after unlocking the run loop and mode locks, to avoid deadlocks
        // where the source wants to take a lock which is already held in another
        // thread which is itself waiting for a run loop/mode lock
        rls->_context.version0.schedule(rls->_context.version0.info, rl, modeName); /* CALLOUT */
    }
}
```

通过添加source的这段代码可以得出如下结论：

- 如果modeName传入kCFRunLoopCommonModes，则该source会被保存到RunLoop的_commonModeItems中
- 如果modeName传入kCFRunLoopCommonModes，则该source会被添加到所有commonMode中
- 如果modeName传入的不是kCFRunLoopCommonModes，则会先查找该Mode，如果没有，会创建一个
- 同一个source在一个mode中只能被添加一次

#### CFRunLoopRemoveSource

remove操作和add操作的逻辑基本一致，很容易理解。

```c
//移除source
void CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef rls, CFStringRef modeName) { /* DOES CALLOUT */
    CHECK_FOR_FORK();
    Boolean doVer0Callout = false, doRLSRelease = false;
    __CFRunLoopLock(rl);
    //如果是kCFRunLoopCommonModes，则从_commonModes的所有mode中移除该source
    if (modeName == kCFRunLoopCommonModes) {
        if (NULL != rl->_commonModeItems && CFSetContainsValue(rl->_commonModeItems, rls)) {
            CFSetRef set = rl->_commonModes ? CFSetCreateCopy(kCFAllocatorSystemDefault, rl->_commonModes) : NULL;
            CFSetRemoveValue(rl->_commonModeItems, rls);
            if (NULL != set) {
                CFTypeRef context[2] = {rl, rls};
                /* remove new item from all common-modes */
                CFSetApplyFunction(set, (__CFRunLoopRemoveItemFromCommonModes), (void *)context);
                CFRelease(set);
            }
        } else {
        }
    } else {
        //根据modeName查找mode，如果不存在，返回NULL
        CFRunLoopModeRef rlm = __CFRunLoopFindMode(rl, modeName, false);
        if (NULL != rlm && ((NULL != rlm->_sources0 && CFSetContainsValue(rlm->_sources0, rls)) || (NULL != rlm->_sources1 && CFSetContainsValue(rlm->_sources1, rls)))) {
            CFRetain(rls);
            //根据source版本做对应的remove操作
            if (1 == rls->_context.version0.version) {
                __CFPort src_port = rls->_context.version1.getPort(rls->_context.version1.info);
                if (CFPORT_NULL != src_port) {
                    CFDictionaryRemoveValue(rlm->_portToV1SourceMap, (const void *)(uintptr_t)src_port);
                    __CFPortSetRemove(src_port, rlm->_portSet);
                }
            }
            CFSetRemoveValue(rlm->_sources0, rls);
            CFSetRemoveValue(rlm->_sources1, rls);
            __CFRunLoopSourceLock(rls);
            if (NULL != rls->_runLoops) {
                CFBagRemoveValue(rls->_runLoops, rl);
            }
            __CFRunLoopSourceUnlock(rls);
            if (0 == rls->_context.version0.version) {
                if (NULL != rls->_context.version0.cancel) {
                    doVer0Callout = true;
                }
            }
            doRLSRelease = true;
        }
        if (NULL != rlm) {
            __CFRunLoopModeUnlock(rlm);
        }
    }
    __CFRunLoopUnlock(rl);
    if (doVer0Callout) {
        // although it looses some protection for the source, we have no choice but
        // to do this after unlocking the run loop and mode locks, to avoid deadlocks
        // where the source wants to take a lock which is already held in another
        // thread which is itself waiting for a run loop/mode lock
        rls->_context.version0.cancel(rls->_context.version0.info, rl, modeName);   /* CALLOUT */
    }
    if (doRLSRelease) CFRelease(rls);
}
```

觉得小编写的不错的可以点赞和评论



---

【完】

