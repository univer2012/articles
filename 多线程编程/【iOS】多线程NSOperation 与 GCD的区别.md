---
title: 【iOS】多线程NSOperation 与 GCD的区别
date: 2017-12-02 17:29:58
tags:
---

### 1.1.1 NSOperation相对于GCD：
1. `NSOperation`拥有更多的函数可用，具体查看api。`NSOperationQueue`是在GCD基础上实现的，只不过是GCD更高一层的抽象。
2. 在`NSOperationQueue`中，可以建立各个`NSOperation`之间的依赖关系。
3. `NSOperationQueue`支持KVO。可以监测operation是否正在执行（isExecuted）、是否结束（isFinished），是否取消（isCanceld）
4. GCD只支持FIFO的队列(即先进先出的队列)，而`NSOperationQueue`可以调整队列的执行顺序（通过调整权重）。`NSOperationQueue`可以方便的管理并发、`NSOperation`之间的优先级。


### 1.1.2 使用NSOperation的情况：
各个操作之间有依赖关系、操作需要取消暂停、并发管理、控制操作之间优先级，限制同时能执行的线程数量.让线程在某时刻停止/继续等。

### 1.1.3 使用GCD的情况：
一般需求很简单的多线程操作，用GCD都可以了，简单高效。

# 二、NSOperation的简单操作
* `NSOperation`(操作)配`NSoperationQueue`(队列)来实现多线程。

* 没有像GCD一样串行并行什么的, 直接拿来就能用。

* 操作依赖:`NSOperation`可以通过设置依赖来保证执行顺序。某一个操作的执行, 必须等待另一个操作完成才会继续执行。
  使用：`[op1 addDependency:op2]`  **依赖关系可以跨队列指定(不能弄成循环依赖)**。

* 可以指定队列的优先级。


## 2.1 使用NSOperation的方法(3者是等价的)：
（注意`NSOperation`是抽象类, 必须使用他的子类才能实现）

1. 用NSOperation的子类`NSInvocationOperation`来创建操作。
2. 用NSOperation的子类`NSBlockOperation`来创建操作. (与前者效果一样)。
3. 直接弄个队列(可以是主队列,先跑起来,再`addOperationWithBlock`更加简便。
4. 直接自定义`NSOperation`，实现相应的方法。



实现NSOperation的方法，以及挂起、暂停，设置最大并发数，设置依赖关系等。

```Objective-C
#import "ViewController.h"


@interface ViewController ()
//NSOperation操作队列
@property(nonatomic,strong)NSOperationQueue * queue;
@property (weak, nonatomic) IBOutlet UIButton * demo6Button;
@property (weak, nonatomic) IBOutlet UIButton * demo5Button;
@property (weak, nonatomic) IBOutlet UIButton * demo4Button;
@property (weak, nonatomic) IBOutlet UIButton * demo3Button;
@property (weak, nonatomic) IBOutlet UIButton * demo2Button;
@property (weak, nonatomic) IBOutlet UIButton * demo1Button;


@end

@implementation ViewController


- (void)viewDidLoad {
    [super viewDidLoad];
    [self.demo6Button addTarget:self action:@selector(opDemo6) forControlEvents:UIControlEventTouchUpInside];
    [self.demo5Button addTarget:self action:@selector(opDemo5) forControlEvents:UIControlEventTouchUpInside];
    [self.demo4Button addTarget:self action:@selector(opDemo4) forControlEvents:UIControlEventTouchUpInside];
    [self.demo3Button addTarget:self action:@selector(opDemo3) forControlEvents:UIControlEventTouchUpInside];
    [self.demo2Button addTarget:self action:@selector(opDemo2) forControlEvents:UIControlEventTouchUpInside];
    [self.demo1Button addTarget:self action:@selector(opDemo1) forControlEvents:UIControlEventTouchUpInside];

}
-(NSOperationQueue * )queue {
    if (!_queue) {
        _queue = [[NSOperationQueue alloc] init];
    }
    return _queue;
}
//MARK: 暂停挂起
//暂停操作
- (IBAction)pause:(id)sender {
    //1.判断队列汇总是否有操作
    if (self.queue.operationCount == 0) {
        NSLog(@"没有操作");
        return;
    }
    //2.如果没有被挂起(正在执行)，才需要暂停
    //只会挂起当前队列中还没有被调度（没有被安排到线程上工作的操作）才会被挂起
    if (!self.queue.isSuspended) {
        NSLog(@"暂停");
        [self.queue setSuspended:YES];
    }
    else {
        NSLog(@"已经暂停");
    }
}
//继续操作
- (IBAction)resume:(id)sender {
    //1.判断队列中是否有操作
    if (self.queue.operationCount ==  0) {
        NSLog(@"没有操作");
        return;
    }
    //2.如果有被挂起的操作，才需要继续(恢复)
    if (self.queue.isSuspended) {
        NSLog(@"继续");
        [self.queue setSuspended:NO];
    }
    else {
        NSLog(@"正在执行");
    }
}
//MARK: NSOperation指定操作之间的依赖关系
-(void)opDemo6 {
    NSBlockOperation * op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"正在下载苍老师全集... %@",[NSThread currentThread]);
    }];
    NSBlockOperation * op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"正在解压缩苍老师全集... %@",[NSThread currentThread]);
    }];
    NSBlockOperation * op3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"正在保存到磁盘... %@",[NSThread currentThread]);
    }];
    NSBlockOperation *op4 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"下载完成... %@",[NSThread currentThread]);
    }];

    /*指定操作之间的”依赖“关系，某一个操作的执行，必须等待另一个操作完成才会开始
     依赖关系是可以跨队列指定的   */
    [op2 addDependency:op1];
    [op3 addDependency:op2];
    [op4 addDependency:op3];

    // *** 添加依赖的时候，注意不要出现循环依赖
    //    [op3 addDependency:op4];

    [self.queue addOperation:op1];
    [self.queue addOperation:op2];
    [self.queue addOperation:op3];
    //主队列更新UI
    [[NSOperationQueue mainQueue] addOperation:op4];
}
//MARK: 设置最大并发数
-(void)opDemo5 {
    // 设置队列的最大并发数，队列是负责调度操作的
    /** 最大并发数的应用场景：
     1> 用户在使用3G的时候          限制线程的数量，省电，省流量(省钱)
     2> 用户使用WIFI的时候（局域网） 增加线程数量，提高用户的体验
     maxConcurrentOperationCount 如果＝＝ 1，类似于串行队列异步方法  */
    self.queue.maxConcurrentOperationCount = 1;
    for (int i = 0; i < 10; i++) {
        [self.queue addOperationWithBlock:^{
            NSLog(@"正在下载: %@ %d", [NSThread currentThread], i);
        }];
    }
}
//MARK: Block操作，添加执行块
-(void)opDemo4 {
    //实例化block操作
    NSBlockOperation *op = [[NSBlockOperation alloc] init];
    //设置最大并发（操作）数，不会限制执行块！
    self.queue.maxConcurrentOperationCount = 2;

    //添加执行块
    [op addExecutionBlock:^{
        NSLog(@"下载苍老师全集1 %@", [NSThread currentThread]);
    }];

    [op addExecutionBlock:^{
        NSLog(@"下载苍老师全集2 %@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"下载苍老师全集3 %@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"下载苍老师全集4 %@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"下载苍老师全集5 %@", [NSThread currentThread]);
    }];

    /* 启动操作，在主线程执行
     1. 如果执行块的数量超过1，就会自动进入其他线程执行(异步)
     2. 具体开启线程的数量，由系统决定
     3. 执行块的调度与操作的调度非常像  */
//    [op start];
    [self.queue addOperation:op];
}

//MARK:直接添加块操作
-(void)opDemo3 {
//只要将操作添加到队列，就会立即被调度（执行）
    for (int i = 0; i < 10; i++) {
        [self.queue addOperationWithBlock:^{
            NSLog(@"下载开始 %@ - %@", [NSThread currentThread], @(i));
        }];
    }

    //向主队列中添加操作
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        NSLog(@"下载开始 %@ -111 %@", [NSThread currentThread], nil);
    }];
}
//MARK: NSBlockOperation
-(void)opDemo2 {
    for (int i = 0; i < 10; i++) {
        //指定一个块操作
        NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
            NSLog(@"下载开始%@ - %@", [NSThread currentThread], @(i));
        }];
        //将块操作添加到队列，新开线程
        [self.queue addOperation:op1];
        /*打印结果还是  0~9  */
    }
}
//MARK: NSInvocationOperation
-(void)opDemo1 {
    for (int i = 0; i < 10; i++) {
        NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(download:) object:@(i)];
        // 如果直接启动，会在主线程执行
        //    [op1 start];
        // 添加到队列，就会新建线程，异步执行
        [self.queue addOperation:op1];
        /*打印结果还是  0~9  ***/
    }

}
-(void)download:(id)obj {
    NSLog(@"开始下载 %@ - %@", [NSThread currentThread], obj);
}
@end
```


# 3 自定义NSOperation

1. 继承NSOperation
2. 重新main方法

```Objective-C
@interface XNMyOperation : NSOperation  
@end  
//=上为.h头文件，下为.m文件====

@implementation XNMyOperation  

// 只要重写main就可以了  
- (void)main {  
    // 自定义操作，一定要自己添加自动释放池  
    @autoreleasepool {  
        //。。。。。。。。。。。  
    }  
}  
@end  
```

