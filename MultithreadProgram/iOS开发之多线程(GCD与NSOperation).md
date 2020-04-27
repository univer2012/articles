---
title: 27多线程
date: 2017-12-02 12:39:32
tags:
---

来自：[iOS开发之多线程(GCD与NSOperation)](http://www.jianshu.com/p/5593af00c597)


# 1 概述
&nbsp; &nbsp; &nbsp; &nbsp;iOS开发中，多线程是必然碰到的，自己这两天有空稍微总结了一下。简单的概念如线程/进程等就不说了。

## 1.1 何为多线程？
&nbsp; &nbsp; &nbsp; &nbsp;多线程其实针对单核的CPU来设计的，CPPU同一时间只能执行一条线程，耳朵线程就是让CPU快速的在多个线程之间进行调度。

</br>
多线程优点：
* 能够适当提高资源利用率
* 能够适当提高资源利用率

</br>
缺点：
* 开线程需要一定的内存空间，默认一条线程占用栈区间512KB
* 线程过多会导致COU在线程上调度的开销比较大
* 程序设计比较复杂，比如线程间通信，多线程的数据共享

&nbsp; &nbsp; &nbsp; &nbsp;在iOS中其实有4套多线程方案，它们分别是：
* pthread
* NSThread
* GCD
* NSOperation

四种方案对比如下：
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E5%BC%80%E5%8F%91%E4%B9%8B%E5%A4%9A%E7%BA%BF%E7%A8%8B(GCD%E4%B8%8ENSOperation)_1.png)

&nbsp; &nbsp; &nbsp; &nbsp;由于平时大多数只用到GCD和NSOperation，下面就主要讨论这两种多线程方案实现。


# 2 GCD简介
&nbsp; &nbsp; &nbsp; &nbsp;GCD以block为基本单位，一个block中的代码可以为一个任务。下文中提到 任务 ，可以理解为执行某个block

&nbsp; &nbsp; &nbsp; &nbsp;GCD有两大重要概念，分别是`队列`和`执行方式`；使用block的过程，概括来说就是把block放进合适的队列，并选择合适的执行方式去执行block的过程。

GCD有三种队列：
1. 串行队列（先进入队列的任务先出队列，每次只执行一个任务）
2. 并发队列 (依然是先进先出，不过可以形成多个任务并发)
3. 主队列 (这是一个特殊的串行队列，而且队列中的任务 一定会在主线程中执行)

两种执行方式：
1. 同步执行
2. 异步执行

关于同步异步、串行并行和线程的关系，如下表格所示

|          | 同步             | 异步           |
| -------- | ---------------- | -------------- |
| 主队列   | 在主线程中执行   | 在主线程中执行 |
| 串行队列 | 在当前线程中执行 | 新建线程执行   |
| 并发队列 | 在当前线程中执行 | 新建线程执行   |

&nbsp; &nbsp; &nbsp; &nbsp;可以看到，同步方法不一定在本线程，异步方法亦不一定新开线程(主队列)

&nbsp; &nbsp; &nbsp; &nbsp;所以，我们在编程时考虑的是`同步`Or`异步` 以及 `串行`Or `并行`，而不是仅仅考虑是否新开线程。

## 2.1 GCD死锁问题
&nbsp; &nbsp; &nbsp; &nbsp;在**使用GCD的过程中，如果向当前串行队列中同步派发一个任务，就会导致死锁**，例如：
```Objective-C
- (void)viewDidLoad {
    [super viewDidLoad];
    NSLog(@"1"); //任务1
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2"); //任务2
    });
    NSLog(@"3"); //任务3    
}
```
&nbsp; &nbsp; &nbsp; &nbsp;以上代码就发生了死锁，控制台只能打印1。因为我们目前在主队列中，又将要同步地添加一个block到主队列(串行)中。

### 2.1.1 理论分析
&nbsp; &nbsp; &nbsp; &nbsp;`dispatch_sync`表示同步的执行任务，也就是说执行`dispatch_sync`后，当前队列会阻塞。而`dispatch_sync`中的block如果要在当前队列中执行，就得等待当前队列执行完成。

&nbsp; &nbsp; &nbsp; &nbsp;上面例子中，首先主队列执行任务1，然后执行`dispatch_sync`，随后在队列中新增一个任务2。**<span style="border-bottom:2px dashed red;">因为主队列是同步队列，所以任务2要等`dispatch_sync`执行完才能执行，但是`dispatch_sync`是同步派发 ，要等任务2执行完才算是结束。在主队列中的两个任务互相等待，导致了死锁<\span>**。当然，由于死锁，后面添加的任务3也不会执行了。

### 2.1.2 解决方案
&nbsp; &nbsp; &nbsp; &nbsp;通常情况下我们不必要用`dispatch_sync`，**因为`dispatch_async`能够更好地利用CPU，提升程序运行速度**。

&nbsp; &nbsp; &nbsp; &nbsp;只有当我们需要去把队列中的任务必须顺序执行时，才考虑使用`dispatch_sync`。在使用`dispatch_sync`的时候应该分析当前处于哪个队列，以及任务会提交到哪个队列。


## 2.2 GCD任务组
&nbsp; &nbsp; &nbsp; &nbsp;**在开发中有这个需求，在A,B,C,D这四个任务全部结束后进行一些处理，那么我们怎么知道四个任务都已经执行完了呢？** 这时候我们就需要用到dispatch_group了。
```Objective-C
dispatch_queue_t dispatchQueue = dispatch_queue_create("sgh.queue.next", DISPATCH_QUEUE_CONCURRENT);
    dispatch_group_t dispatchGroup = dispatch_group_create();

    dispatch_group_async(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务A");
    });
    dispatch_group_async(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务B");
    });
    dispatch_group_async(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务C");
    });
    dispatch_group_async(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务D");
    });

    dispatch_group_notify(dispatchGroup, dispatchQueue, ^{
        NSLog(@"end");
    });
```
执行3次的打印结果如下：
```Objective-C
//执行第1次
 任务A
 任务B
 任务C
 任务D
 end
//执行第2次
 任务B
 任务A
 任务C
 任务D
 end
//执行第3次
 任务A
 任务B
 任务D
 任务C
 end
```
&nbsp; &nbsp; &nbsp; &nbsp;首先我们要通过`dispatch_group_create`方法生成一个组然后我们把`dispatch_async`方法换成`dispatch_group_async`。这个方法多了一个参数，第一个参数填刚创建的分组。

&nbsp; &nbsp; &nbsp; &nbsp;最后调用`dispatch_group_notify`方法。**这个方法表示把第三个参数block传入第二个参数队列中去。而且可以保证第三个参数block执行时，group中所有任务已经全部完成**。

### 2.2.1 dispatch_group_wait
`dispatch_group_wait` 的完整定义如下：
```Objective-C
long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout);
```
&nbsp; &nbsp; &nbsp; &nbsp;第一个参数表示要等待的group,第二个则表示等待时间。**返回值表示，经过指定的等待时间，属于这个group的任务是否已经全部执行完。如果是则返回0，否则返回非0**。

&nbsp; &nbsp; &nbsp; &nbsp;第二个参数`dispatch_time_t`类型的参数还有两个特殊值：`DISPATCH_TIME_NOW`和`DISPATCH_TIME_FOREVER`，**前者表示立刻检查这个group的任务是否已经完成，后者则表示一直到属于这个group的任务全部完成**。


### 2.2.2 dispatch_after
通过GCD还**可以进行简单的定时的操作，比如在1秒后执行某个block**。代码如下：
```Objective-C
NSLog(@"前");
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"1秒后执行");
    });
    NSLog(@"后");
```
输出为：
```Objective-C
2017-12-02 08:26:46.741331+0800 iOSSingleViewApp[4435:3113889] 前
2017-12-02 08:26:46.741533+0800 iOSSingleViewApp[4435:3113889] 后
2017-12-02 08:26:47.741596+0800 iOSSingleViewApp[4435:3113889] 1秒后执行
```
&nbsp; &nbsp; &nbsp; &nbsp;**`dispatch_after`有三个参数。第一个表示时间，也就是从现在起往后多少秒钟。第二个参数表示提交到哪个队列**，第三个参数表示要提交的任务。

&nbsp; &nbsp; &nbsp; &nbsp;**需要注意的是，`dispatch_after`仅表示在指定时间后提交任务 ，而非执行任务**。如果任务提交到主队列，它将在main runloop中执行，对于每隔1/60秒执行一个的RunLoop1，任务最多可能在1+1/60秒后执行。


# 3 GCD进阶
GCD也有一些强大的特性。接下来我们主要讨论以下几个部分：
* `dispatch_suspend` 和 `dispatch_resume`
* `dispatch_once`
* `dispatch_barrier_async`
* `dispatch_semaphore`

&nbsp; &nbsp; &nbsp; &nbsp;我们知道`NSOperationQueue`有暂停`suspend`和恢复`resume`。其实GCD中的队列也有类似的功能。
```Objective-C
//延缓,推迟，暂停
void dispatch_suspend(dispatch_object_t object);
//恢复
void dispatch_resume(dispatch_object_t object);
```
&nbsp; &nbsp; &nbsp; &nbsp;这些函数不会影响到队列中已经执行的任务。**队列暂停后，已经添加到队列中但是还没有执行的任务，不会执行，直到队列被恢复**。

## 3.1 dispatch_once
&nbsp; &nbsp; &nbsp; &nbsp;`dispatch_once`在单例模式被广泛使用。**`dispatch_once`函数可以确保某个block在应用程序执行过程中只被处理一次，而且它是线程安全的**。所以单例模式可以很简单的实现，代码实现如下：
```Objective-C
+(instancetype)shareInstance {
    static dispatch_once_t onceToken;
    static TKBeginnerGuideHelper * shareInstance;
    dispatch_once(&onceToken, ^{
        shareInstance = [self new];
    });
    return shareInstance;
}
```
&nbsp; &nbsp; &nbsp; &nbsp;这段代码中我们创建一个值为nil的 shareInstance 静态对象，然后把它的初始化代码放到dispatch_once中完成。这样，只有第一次调用sharedInstance方法时才会进行对象的初始化，以后每次只是返回 shareInstance 而已。

## 3.2 dispatch_barrier_async
&nbsp; &nbsp; &nbsp; &nbsp;我们知道在写入时，不能再其他线程读取或写入。但是多个线程同时读取数据是没有问题的。所以我们可以把读取任务放入并行队列，把写入任务放入串行队列，并且保证写入任务执行过程中没有读取任务可以执行。这样的需求就比较常见，GCD提供了一个非常简单的解决方法  `dispatch_barrier_async` 。

&nbsp; &nbsp; &nbsp; &nbsp;假设我们有四个读取任务，在第二、三个任务之间有一个写入任务，代码大概如下：
```Objective-C
dispatch_queue_t dispatchQueue = dispatch_queue_create("sgh.queue.next", DISPATCH_QUEUE_CONCURRENT);

    dispatch_block_t block1_for_reading, block2_for_reading, block_for_writing;
    dispatch_block_t block3_for_reading, block4_for_reading;
    
    block1_for_reading = ^{
        NSLog(@"block1前");
        sleep(3);
        NSLog(@"block1--- 3秒后执行");
    };
    
    block2_for_reading = ^{
        NSLog(@"block2前");
        sleep(2);
        NSLog(@"block2--- 2秒后执行");
    };
    
    block_for_writing = ^{
        NSLog(@"writing前");
        sleep(5);
        NSLog(@"writing--- 5秒后执行");
    };
    
    block3_for_reading = ^{
        NSLog(@"block3前");
        sleep(3);
        NSLog(@"block3--- 3秒后执行");
        
    };
    block4_for_reading = ^{
        NSLog(@"block4前");
        sleep(2);
        NSLog(@"block4--- 2秒后执行");
        
    };
    
    dispatch_async(dispatchQueue, block1_for_reading);
    dispatch_async(dispatchQueue, block2_for_reading);
    
    //这里插入写入任务，比如：
    dispatch_async(dispatchQueue, block_for_writing);
     
    dispatch_async(dispatchQueue, block3_for_reading);
    dispatch_async(dispatchQueue, block4_for_reading);
```
打印如下：
```objc
2020-04-26 14:03:38.820240+0800 OCDemo20200321[58299:845488] block1前
2020-04-26 14:03:38.820323+0800 OCDemo20200321[58299:845481] block2前
2020-04-26 14:03:38.820366+0800 OCDemo20200321[58299:845483] writing前
2020-04-26 14:03:38.820393+0800 OCDemo20200321[58299:845487] block3前
2020-04-26 14:03:38.820652+0800 OCDemo20200321[58299:845485] block4前
2020-04-26 14:03:40.823233+0800 OCDemo20200321[58299:845485] block4--- 2秒后执行
2020-04-26 14:03:40.823233+0800 OCDemo20200321[58299:845481] block2--- 2秒后执行
2020-04-26 14:03:41.823274+0800 OCDemo20200321[58299:845488] block1--- 3秒后执行
2020-04-26 14:03:41.823323+0800 OCDemo20200321[58299:845487] block3--- 3秒后执行
2020-04-26 14:03:43.823128+0800 OCDemo20200321[58299:845483] writing--- 5秒后执行
```
&nbsp; &nbsp; &nbsp; &nbsp;如果代码这样写，**由于几个block是并发执行，就有可能在前两个block中读取到已经修改了的数据。如果是有多写入任务，那问题更严重，可能会有数据竞争**。


如果使用 `dispatch_barrier_async` 函数，代码就可以这么写：
```Objective-C
    dispatch_queue_t dispatchQueue = dispatch_queue_create("sgh.queue.next", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_block_t block1_for_reading, block2_for_reading, block_for_writing;
    dispatch_block_t block3_for_reading, block4_for_reading;
    
    block1_for_reading = ^{
        NSLog(@"block1前");
        sleep(3);
        NSLog(@"block1--- 3秒后执行");
    };
    
    block2_for_reading = ^{
        NSLog(@"block2前");
        sleep(2);
        NSLog(@"block2--- 2秒后执行");
    };
    
    block_for_writing = ^{
        NSLog(@"writing前");
        sleep(5);
        NSLog(@"writing--- 5秒后执行");
    };
    
    block3_for_reading = ^{
        NSLog(@"block3前");
        sleep(3);
        NSLog(@"block3--- 3秒后执行");
        
    };
    block4_for_reading = ^{
        NSLog(@"block4前");
        sleep(2);
        NSLog(@"block4--- 2秒后执行");
        
    };
    
    dispatch_async(dispatchQueue, block1_for_reading);
    dispatch_async(dispatchQueue, block2_for_reading);
    
    //这里插入写入任务，比如：
    dispatch_barrier_async(dispatchQueue, block_for_writing);
    
    dispatch_async(dispatchQueue, block3_for_reading);
    dispatch_async(dispatchQueue, block4_for_reading);
```
打印如下：
```objc
2020-04-26 14:03:47.322186+0800 OCDemo20200321[58299:845700] block1前
2020-04-26 14:03:47.322220+0800 OCDemo20200321[58299:845488] block2前
2020-04-26 14:03:49.326612+0800 OCDemo20200321[58299:845488] block2--- 2秒后执行
2020-04-26 14:03:50.322697+0800 OCDemo20200321[58299:845700] block1--- 3秒后执行
2020-04-26 14:03:50.322916+0800 OCDemo20200321[58299:845700] writing前
2020-04-26 14:03:55.327519+0800 OCDemo20200321[58299:845700] writing--- 5秒后执行
2020-04-26 14:03:55.327819+0800 OCDemo20200321[58299:845700] block3前
2020-04-26 14:03:55.327819+0800 OCDemo20200321[58299:845488] block4前
2020-04-26 14:03:57.331132+0800 OCDemo20200321[58299:845488] block4--- 2秒后执行
2020-04-26 14:03:58.331165+0800 OCDemo20200321[58299:845700] block3--- 3秒后执行
```
dispatch_barrier_async会把队列的运行周期分为这三个过程：
1. 首先等目前追加到并行队列中所有任务都执行完成。
2. **开始执行 `dispatch_barrier_async` 中的任务时，即便向并行队列提交任务，也不会执行**。
3. `dispatch_barrier_async` 中任务执行完成后，并行队列恢复正常。

&nbsp; &nbsp; &nbsp; &nbsp;总的来说，**==`dispatch_barrier_async`起到了承上启下的作用。它保证此前的任务都先于自己执行，此后的任务也迟于自己执行==**。正如barrier的含义一样，它起到一个栅栏或者分水岭的作用。

使用并行队列和 `diapatch_barrier_async` 方法，就可以高效的进行数据和文件读写了。


## 3.3 dispatch_semaphore
&nbsp; &nbsp; &nbsp; &nbsp;首先介绍一下信号量(semaphore)的概念。**信号量是持有计数的信号**，举个生活中的例子来看：

&nbsp; &nbsp; &nbsp; &nbsp;假设有一个房子，它对应进程的概念，房子里的人就对应着线程。一个进程可以包括多个线程。这个房子(进程)有很多资源，比如花园、客厅灯，是所有人(线程)共享的。

&nbsp; &nbsp; &nbsp; &nbsp;但是有些地方，比如卧室，最多只有两个人进去睡觉。怎么办呢？在卧室门口挂上两把钥匙。进去的人(线程)就拿着钥匙进去，没有钥匙就不能进去，出来的时候把钥匙放回门口。

&nbsp; &nbsp; &nbsp; &nbsp;这时候，**门口的钥匙数量就称为信号量(Semaphore)**。很明显，信号量为0时需要等待，信号量不为零时，减去1而且不等待。

在GCD中，创建信号量的代码如下：
```Objective-C
dispatch_semaphore_t semaphore = dispatch_semaphore_create(3);
```
&nbsp; &nbsp; &nbsp; &nbsp;这句代码通过`diapatch_semaphore_create`方法创建一个信号量初始值为3。然后就可以调用`dispatch_semaphore_wait`方法了。
```Objective-C
long hasSemaphore = dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
```

**==`dispatch_semaphore_wait` 方法表示一直等待，直到信号量的值大于等于1，当这个方法执行后，会把第一个参数信号量的值减1。第二个参数是一个`dispatch_time_t`类型的时间，它表示这个方法最大的等待时间 。==**

返回值也和dispatch_group_wait方法一样，**==返回0表示在规定的时间内，第一个参数信号量的值已经大于等于1，否则表示已超过规定等待时间，但信号量的值还是0。==**

==dispatch_semaphore_wait方法返回0，因为此时的信号量的值大于等于1，任务获得了可以执行的权限。==**==这时候我们就可以安全的执行需要进行排他控制的任务了==**。

**==任务结束时还需要调用dispatch_semaphore_signal()方法，将信号量的值加1==**。这类似于之前所说的，从卧室出来要把锁放回门上，否则后来的人就无法进入了。示例代码如下：

代码示例是：**利用dispatch_semaphore_t将数据，在异步线程追加到数组**

```Objective-C
- (void)demo3_3 {
    //创建一个信号量初始值为1
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    
    dispatch_queue_t dispatchQueue = dispatch_queue_create("sgh.gcd.next", DISPATCH_QUEUE_CONCURRENT);
    NSLog(@"开始");
    NSMutableArray *array = [NSMutableArray array];
    for (int i = 0; i < 10000; i++) {
        dispatch_async(dispatchQueue, ^{
            /* 某个线程执行到这里，如果信号量为1，那么wait方法返回1，开始执行接下来的操作。
             与此同时，因为信号量变为0，其他执行到这里的线程必须等待  *****/
            dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

            /* 执行了wait方法后，信号量变成了0，可以进行接下来的操作。
             这时候其他线程都得等待wait方法返回。
             对array修改的线程在任意时刻都只有一个，可以安全的修改array *****/
            [array addObject:@(i)];
            NSLog(@"被锁在房间,%d",i);
            
            /** 排他操作执行结束，记得要调用signal方法，把信号量的值加1。
             这样，如果有别的线程在等待wait函数返回，就由最先等待的线程执行  ****/
            dispatch_semaphore_signal(semaphore);
            
            if (i + 1 == 10000) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    NSLog(@"结束");
                });
            }
            
        });
        
    }
    NSLog(@"结束2");
}
```

另外还有ANNetworking中的`AFURLSessionManager.m`代码，则是**用GCD的信号量来实现异步线程同步操作**。代码如下：

```objc
- (NSArray *)tasksForKeyPath:(NSString *)keyPath {
    __block NSArray *tasks = nil;
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    [self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        if ([keyPath isEqualToString:NSStringFromSelector(@selector(dataTasks))]) {
            tasks = dataTasks;
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(uploadTasks))]) {
            tasks = uploadTasks;
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(downloadTasks))]) {
            tasks = downloadTasks;
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(tasks))]) {
            tasks = [@[dataTasks, uploadTasks, downloadTasks] valueForKeyPath:@"@unionOfArrays.self"];
        }

        dispatch_semaphore_signal(semaphore);
    }];

    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

    return tasks;
}
```



# 4 NSOperation

`NSOperation` 和 `NSOperationQueue` 主要介绍以下几个方面：

1. `NSOperation`和 `NSOperationQueue`的用法介绍
2. `NSOperation`的暂停、恢复和取消
3. 通过KVO对`NSOperation`的状态进行检测
4. 多个`NSOperation`之间的依赖关系
5. 进程间通信

&nbsp; &nbsp; &nbsp; &nbsp;`NSOperation`是对GCD中的block进行的封装，它也表示一个要被执行的任务。和GCD的block类似，`NSOperation`对象有一个`start()`方法表示开始执行这个任务。

&nbsp; &nbsp; &nbsp; &nbsp;不仅如此，NSOperation表示的任务还可以被取消。它还有三种状态`isExecuted`、`isFinished`和`isCancelled`以方便我们通过KVC对它的状态进行监听。

想要**开始执行一个任务**可以这么写：
```Objective-C
NSBlockOperation * op = [NSBlockOperation blockOperationWithBlock:^{
				//要执行的任务，这个任务主线程中执行
        NSLog(@"task----%@",[NSThread currentThread]);
    }];
    [op start];
```
打印结果如下：
```Objective-C
 task----<NSThread: 0x600000262b80>{number = 1, name = main}
```
我们创建了一个NSBlockOperation,**并且设置好它的block，也就是要执行的任务。这个任务就会在主线程中执行**。

&nbsp; &nbsp; &nbsp; &nbsp;为什么不直接使用NSOperation呢？**因为NSOperation本身是一个抽象类**，要使用可以通过以下几个方法：
* 使用`NSInvocationOperation`
* 使用`NSBlockOperation`
* 自定义`NSOperation`的子类

&nbsp; &nbsp; &nbsp; &nbsp;`NSBlockOperation`可以用来封装一个或多个block。同时，还可以**调用addExecutionBlock方法追加几个任务，这些任务会并行执行**(也就是说很有可能运行在别的线程里)。最后，**调用start方法让NSOperation方法运行起来。start是一个同步方法**。

# 5 NSOperationQueue
&nbsp; &nbsp; &nbsp; &nbsp;从上面我们知道，**默认的NSOperation是同步执行的。简单的看一下NSOperation类的定义会发现它只有一个只读属性`asynchronous`。这意味着如果想要异步执行，就需要自定义`NSOperation`的子类。或者使用`NSOperationQueue`**。

&nbsp; &nbsp; &nbsp; &nbsp;`NSOperationQueue`类似于GCD中的队列。我们知道 <u>**GCD中的队列有三种：主队列、串行队列和并行队列。`NSOperationQueue`更简单，只有两种：主队列和非主队列**</u> 。

&nbsp; &nbsp; &nbsp; &nbsp;**我们自己生成的`NSOperationQueue`对象都是非主队列，主队列可以用`[NSOperationQueue mainQueue]`取得。`NSOperationQueue`的主队列是串行队列，而且其中所有`NSOperation`都会在主线程中执行**。

对于非主队列来说，一旦一个`NSOperation`被放入其中，那这个`NSOperation`一定是并发执行的。因为`NSOperationQueue`会为每一个NSOperation创建线程并调用它的start方法。

**`NSOperationQueue`有一个属性叫`maxConcurrentOperationCount`，它表示最多支持多少个`NSOperation`并发执行**。如果`maxConCurrentOperationCount`被设置为1，就以为这个队列是串行队列。因此，NSOperationQueue和GCD中的队列有这样的对应关系：



|          | NSOperation                               | GCD                                                 |
| -------- | ----------------------------------------- | --------------------------------------------------- |
| 主队列   | [NSOperationQueue mainQueue]              | dispatch_get_mian_queue()                           |
| 串行队列 | 自建队列 maxConcurrentOperationCount为1   | dispatch_queue_create("",DISPATCH_QUEUE_SERIAL)     |
| 并发队列 | 自建队列 maxConcurrentOperationCount大于1 | dispatch_queue_create("",DISPATCH_QUEUE_CONCURRENT) |



  如何利用NSOperationQueue实现异步操作？代码如下：
```Objective-C
  //自建队列
      NSOperationQueue *queue = [[NSOperationQueue alloc] init];
      NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
          NSLog(@"task0 :%@", [NSThread currentThread]);
      }];
      [op addExecutionBlock:^{
          NSLog(@"task1: %@", [NSThread currentThread]);
      }];
      [op addExecutionBlock:^{
          NSLog(@"task2: %@", [NSThread currentThread]);
      }];
      [op addExecutionBlock:^{
          NSLog(@"task3: %@", [NSThread currentThread]);
      }];
      [queue addOperation:op];
      NSLog(@"操作结束");
```
执行3次结果如下：
```Objective-C
//第1次
 操作结束
 task0 :<NSThread: 0x60400046db00>{number = 3, name = (null)}
 task3: <NSThread: 0x60400046dc00>{number = 6, name = (null)}
 task2: <NSThread: 0x6000004668c0>{number = 5, name = (null)}
 task1: <NSThread: 0x60400046db40>{number = 4, name = (null)}
 //第2次
  操作结束
 task0 :<NSThread: 0x60400006d240>{number = 3, name = (null)}
 task2: <NSThread: 0x604000274740>{number = 5, name = (null)}
 task3: <NSThread: 0x600000276180>{number = 6, name = (null)}
 task1: <NSThread: 0x600000276080>{number = 4, name = (null)}
 //第3次
  操作结束
 task1: <NSThread: 0x604000461800>{number = 4, name = (null)}
 task0 :<NSThread: 0x6040004617c0>{number = 3, name = (null)}
 task3: <NSThread: 0x600000460600>{number = 6, name = (null)}
 task2: <NSThread: 0x6040004618c0>{number = 5, name = (null)}
```
&nbsp; &nbsp; &nbsp; &nbsp;使用`NSOperationQueue`来执行任务与之前的区别在于，**首先创建一个非主队列。然后用`addOperation`方法替换之前的`start`方法。刚刚已经说过，`NSOperationQueue`会为每一个`NSOperation`创建线程并调用它们的`start`方法**。

观察一下运行结果 ，**<u>所有的NSOperation都没有在主线程执行，从而成功的实现了异步、并行处理</u>**。

## 5.1 取消任务
&nbsp; &nbsp; &nbsp; &nbsp;如果我们有两次网络请求，第二次请求会用到第一次的数据。假设此时网络情况不好，第一次请求超时了，那么第二次请求也没有必要发送了。当然，用户也有可能人为地取消某个`NSOperation`。

&nbsp; &nbsp; &nbsp; &nbsp;当某个`NSOperation`被取消时，我们应该尽可能的清除`NSOperation`内部的数据并且把`cancel`和`finished`设为true，把`executing`设为false。
```Objective-C
//取消某个NSOperation
    [operation cancel];
    //取消某个NSOperationQueue剩余的NSOperation
    [queue cancelAllOperations];
```

## 5.2 设置依赖
&nbsp; &nbsp; &nbsp; &nbsp;有时候一个网络请求是用到另一个网络请求获得的数据，这时候我们要确保第二次请求时，第一个请求已经执行完。但是我们同时还希望用到`NSOperationQueue`的并发特性(因为可能不止这两个任务)

这时候我们**可以设置`NSOperation`之间的依赖关系**，很简单，代码如下：
```Objective-C
[operation1 addDependency: operation2];
```
需要注意的是，**<u>`NSOperation`之间的相互依赖会导致死锁</u>。**

## 5.3 NSOperationQueue暂停与恢复
这个也很简单，只要修改`suspended`属性即可：
```Objective-C
queue.suspended = true; //暂停queue中所有operation
queue.suspended = false; //恢复queue中所有operation
```

## 5.4 NSOperation优先级
&nbsp; &nbsp; &nbsp; &nbsp;***GCD中，任务(block)是没有优先级的，而队列具有优先级。和GCD相反，我们一般考虑NSOperation的优先级***。

`NSOperation`有一个`NSOperationQueuePriority`枚举类型的属性`queuePriority`。
```Objective-C
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
    NSOperationQueuePriorityVeryLow = -8L,
    NSOperationQueuePriorityLow = -4L,
    NSOperationQueuePriorityNormal = 0,
    NSOperationQueuePriorityHigh = 4,
    NSOperationQueuePriorityVeryHigh = 8
};
```
需要注意的是，**`NSOperationQueue`也不能完全保证优先级高的任务 一定先执行**。

## 5.5 进程间通信

有时候我们在子线程中执行完一些操作的时候，需要回到主线程做一些事情(如进行UI操作)，因此需要从当前线程回到主线程，以下载并显示图片为例，代码如下：
```Objective-C
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 子线程下载图片
[queue addOperationWithBlock:^{
    NSURL * url = [NSURL URLWithString:@"http://img.pconline.com.cn/images/photoblog/9/9/8/1/9981681/200910/11/1255259355826.jpg"];
    NSData * data = [NSData dataWithContentsOfURL:url];
    UIImage * image = [[UIImage alloc] initWithData:data];
    //回到主线程进行显示
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        self.imageView.image = image;
    }];
}];
```

# 6 小结
## 6.1 NSOperation和GCD如何选择
1. GCD以block为单位，代码简洁。同时GCD中的队列、组、信号量、source、barriers都是组成并行编程的基本原语。对于一次性的计算，或者仅仅为了加快现有方法的运行速度，选择轻量化的GCD就更加方便。

2. `NSOperation`可以用来规划一组任务之间的依赖关系，设置它们的优先级，任务能被取消。队列可以暂停、恢复。`NSOperation`还可以自定义子类。这些都是GCD没有具备的。

3. 可以根据情况有效结合`NSOperation`和GCD一起使用。


最后，有个很经典的面试题，**GCD和NSOperation有什么区别？**

答案基本就是对上面所说的的总结：

1. GCD是纯C语言的API,`NSOperation`是基于GCD的OC版本封装。
2. **GCD只支持FIFO的队列，`NSOperation`可以很方便地调整执行顺序，设置最大并发数量**。
3. `NSOperationQueue`可以轻松在operation间设置依赖关系，而GCD需要写很多代码才能实现。
4. **`NSOperationQueue`支持KVO，可以检测operation是否正在执行(isExecuted)，是否结束(isFinisn),是否取消(isCancel)**。
5. GCD的执行速度比`NSOperation`快。