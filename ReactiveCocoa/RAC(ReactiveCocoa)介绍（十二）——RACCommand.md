来自：[RAC(ReactiveCocoa)介绍（十二）——RACCommand](https://www.jianshu.com/p/75681ea2256e)





#### 额外的知识点：

FRP： [Functional Reactive Programming](https://link.jianshu.com?t=https://en.wikipedia.org/wiki/Functional_reactive_programming)，函数响应式编程，相关技术：Reactive Cocoa

OOP：Object Oriented Programming，面向对象编程

OOA：面向对象分析

OOD：面向对象设计

---

参考：

[[iOS - 对OOA、OOD、OOP的理解]](https://www.cnblogs.com/dingding3w/p/4843946.html)





`RACCommand`作为ReactiveCocoa基本组件之一，通常在项目开发过程中`RACSignal`与`RACSubject`组合使用就可以满足大部分的开发需求。

**<u> `RACCommand`个人理解为事件响应信号的管理者。其本身并不继承自`RACSignal`或者`RACStream`类，而是继承于`NSObject`，用于管理`RACSignal`类的创建与订阅的类</u>**。关于`RACCommand`类的作用，找到了一篇国外的博客[原文链接](https://links.jianshu.com/go?to=http%3A%2F%2Fcodeblog.shape.dk%2Fblog%2F2013%2F12%2F05%2Freactivecocoa-essentials-understanding-and-using-raccommand%2F)，其中写道：

![RACCommand类作用](https:////upload-images.jianshu.io/upload_images/1243805-1e5bd81ce836c592.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)




 大意为，`RACCommand`类用于响应动作事件的执行，执行命令通常由用户交互页面的手势操作来触发。该类可以实现多种不同情况下的响应事件处理，除了可以快速绑定交互页面，还可以确保其在未使用时不会执行信号操作。

因此**在实际项目开发中，`RACCommand`使用的场景多用于交互手势操作响应事件，以及网络请求时不同请求状态的处理封装处理**。当需要响应事件或网络请求时，直接执行对应`RACCommand`就可以发送信号，执行操作。当`RACCommand`内部收到请求时，把处理的结果返回给外部，这时要通过`signalBlock`返回的信号进行数据传递。



接下来，看看`RACCommand`类的基本使用

```objc
- (void)p_command {
    //创建command信号
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
        
        //返回RACSignal信号类型对象
        return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
            
            [subscriber sendNext:@"this is signal of command"];
            NSError *error = [NSError errorWithDomain:NSCocoaErrorDomain code:1234 userInfo:@{@"key": @"error"}];
            [subscriber sendError:error];
            
            return [RACDisposable disposableWithBlock:^{
                NSLog(@"销毁了");
            }];
        }];
    }];
    
    //command信号是否正在执行
    [command.executing subscribeNext:^(NSNumber * _Nullable x) {
        NSLog(@"executing == %@",x);
    }];
    
    //错误信号
    [command.errors subscribeNext:^(NSError * _Nullable x) {
        NSLog(@"errors == %@",x);
    }];
    
    //command信号中signal信号发送出来的订阅信号
    [command.executionSignals subscribeNext:^(id  _Nullable x) {
        NSLog(@"executionSignals == %@",x);
    }];\
    
    //必须执行命令，否则所有信号都不会订阅到
    [command execute:@"command执行"];
}
```



打印：

```objc
2019-11-23 23:49:20.895475+0800 ObjectiveCDemo160728[18706:926542] executing == 0
2019-11-23 23:49:20.914362+0800 ObjectiveCDemo160728[18706:926542] executing == 1
2019-11-23 23:49:20.914714+0800 ObjectiveCDemo160728[18706:926542] executionSignals == <RACDynamicSignal: 0x600001d63d00> name:
2019-11-23 23:49:20.915656+0800 ObjectiveCDemo160728[18706:926542] 销毁了
2019-11-23 23:49:20.960019+0800 ObjectiveCDemo160728[18706:926542] errors == Error Domain=NSCocoaErrorDomain Code=1234 "(null)" UserInfo={key=error}
2019-11-23 23:49:20.960399+0800 ObjectiveCDemo160728[18706:926542] executing == 0
```



从打印结果中可以发现，`executing`属性在信号开始时，一定会返回0，代表RACCommand未执行。在实际应用中，并不需要监听`command`第一次未执行状态。此处可将属性`executing`的代码修改成自动跳过第一个信号。

```objc
[[command.executing skip:1] subscribeNext:^(NSNumber * _Nullable x) {
    NSLog(@"executing_skip1 == %@",x);
}];
```



打印结果中，已经打印出command中的RACSignal信号，若需要监听发送信号中的具体内容，可将executionSignals的订阅信号代码修改成：

```objc
//监听最后一次发送的信号
[[command.executionSignals switchToLatest] subscribeNext:^(id  _Nullable x) {
    NSLog(@"executionSignals_switchToLatest == %@",x);
}];
```

这时的打印结果就变成了预期想要的结果：

```objc
2019-11-23 23:59:56.205540+0800 ObjectiveCDemo160728[18881:938746] executing == 0
2019-11-23 23:59:56.224563+0800 ObjectiveCDemo160728[18881:938746] executing == 1
2019-11-23 23:59:56.225280+0800 ObjectiveCDemo160728[18881:938746] 销毁了
2019-11-23 23:59:56.245675+0800 ObjectiveCDemo160728[18881:938746] errors == Error Domain=NSCocoaErrorDomain Code=1234 "(null)" UserInfo={key=error}
2019-11-23 23:59:56.246395+0800 ObjectiveCDemo160728[18881:938746] executing == 0

```



至于为什么error信号，没有被`executionSignals`订阅到，而是被errors给订阅到了？

**原因在于这个errors是一个被包装在`RACSignal`信号类对象，进行错误处理的时候，我们不应该使用`subscribeError:`对`RACCommand`的`executionSignals`
 进行错误的订阅，因为`executionSignals`这个信号是不会发送error事件的，而应该使用`subscribeNext:`去订阅错误信号。**



errors信号对象内部实现：

![img](https:////upload-images.jianshu.io/upload_images/1243805-f619c45bd44f6c5e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



在`RACCommand`使用中，创建的`RACSignal`信号时，发送完订阅信号时，若没有`error`信号发送，则需要执行`[subscriber sendCompleted]`命令来销毁信号。否则该`RACCommand`会一直处于执行状态而无法正常释放。

接下来以一个简单的登录页面小demo来介绍RACCommand的作用与使用方法。
 GitHub链接[demo链接](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FRoganZheng%2FRACCommandDemo)

---

【完】