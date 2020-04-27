来自：[RAC(ReactiveCocoa)介绍（十）——RACMulticastConnection](https://www.jianshu.com/p/ceb84f847212)



在实际项目开发过程中，经常会在多处不同地方对同一信号进行订阅。比如：在网络请求时，收到返回数据要针对页面多处进行更新操作。

 通常会出现以下RAC写法：

```objc
- (void)multicastConnection {
    _exampleSignal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"message received"];
        NSLog(@"这里进行了一次耗时操作");
        return [RACDisposable disposableWithBlock:^{
            NSLog(@"disposable!");
        }];
    }];
    
    [_exampleSignal subscribeNext:^(id  _Nullable x) {
        NSLog(@"第一处，收到订阅信号：%@",x);
    }];
    
    [_exampleSignal subscribeNext:^(id  _Nullable x) {
        NSLog(@"第二处，收到订阅信号：%@",x);
    }];
}
```





这样的错误写法打印结果如下：

```objc
2019-11-23 21:07:57.499885+0800 ObjectiveCDemo160728[17563:844808] 第一处，收到订阅信号：message received
2019-11-23 21:07:57.500676+0800 ObjectiveCDemo160728[17563:844808] 这里进行了一次耗时操作
2019-11-23 21:07:57.501360+0800 ObjectiveCDemo160728[17563:844808] disposable!
2019-11-23 21:07:57.502047+0800 ObjectiveCDemo160728[17563:844808] 第二处，收到订阅信号：message received
2019-11-23 21:07:57.502236+0800 ObjectiveCDemo160728[17563:844808] 这里进行了一次耗时操作
2019-11-23 21:07:57.502389+0800 ObjectiveCDemo160728[17563:844808] disposable!
```



从打印结果来看，很明显地`RACSignal`拥有两个订阅者时，发送信号与销毁信号都执行了两次。当拥有更多订阅者或者需要在发送信号前耗时处理时，这种写法大大降低了运行效率，浪费了不必要的资源。



 针对这种情况，就需要使用`RACMulticastConnection`来处理。



 将上述问题代码加入进`RACMulticastConnection`处理之后

```objc
- (void)multicastConnection1 {
    _exampleSignal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"message received"];
        NSLog(@"这里进行了一次耗时操作");
        return [RACDisposable disposableWithBlock:^{
            NSLog(@"disposable!");
        }];
    }];
    
    RACMulticastConnection *connection = [_exampleSignal publish];
    
    
    [connection.signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"第一处，收到订阅信号：%@",x);
    }];
    
    [connection.signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"第二处，收到订阅信号：%@",x);
    }];
    
    [connection connect];
}
```



这时再查看打印结果：

```objc
2019-11-23 21:16:41.935753+0800 ObjectiveCDemo160728[17638:855053] 第一处，收到订阅信号：message received
2019-11-23 21:16:41.936129+0800 ObjectiveCDemo160728[17638:855053] 第二处，收到订阅信号：message received
2019-11-23 21:16:41.937097+0800 ObjectiveCDemo160728[17638:855053] 这里进行了一次耗时操作
2019-11-23 21:16:41.953657+0800 ObjectiveCDemo160728[17638:855053] disposable!
```

**发送信号时只执行了一次，两个订阅者都收到了各自的订阅信号。**





为什么`RACMulticastConnection`可以实现一次发送多处订阅者就可以收到信号？

 带着问题去查看其内部实现过程：

```objc
///RACSignal+Operations.m
- (RACMulticastConnection *)publish {
	RACSubject *subject = [[RACSubject subject] setNameWithFormat:@"[%@] -publish", self.name];
	RACMulticastConnection *connection = [self multicast:subject];
	return connection;
}

- (RACMulticastConnection *)multicast:(RACSubject *)subject {
	[subject setNameWithFormat:@"[%@] -multicast: %@", self.name, subject.name];
	RACMulticastConnection *connection = [[RACMulticastConnection alloc] initWithSourceSignal:self subject:subject];
	return connection;
}
```



从上图代码实现中，可以发现`RACMulticastConnection`是基于`RACSubject`来实现的，并将`RACSubject`封装成了`RACMulticastConnection`对象。打印`connection.signal`，可以发现是为`RACSubject`类的对象。

 再继续进入至`- (instancetype)initWithSourceSignal:(RACSignal *)source subject:(RACSubject *)subject`方法中。

RACMulticastConnection内部实现方法：

```objc
///RACMulticastConnection.m
@implementation RACMulticastConnection

#pragma mark Lifecycle

- (instancetype)initWithSourceSignal:(RACSignal *)source subject:(RACSubject *)subject {
	NSCParameterAssert(source != nil);
	NSCParameterAssert(subject != nil);

	self = [super init];

	_sourceSignal = source;
	_serialDisposable = [[RACSerialDisposable alloc] init];
	_signal = subject;
	
	return self;
}

#pragma mark Connecting

- (RACDisposable *)connect {
	BOOL shouldConnect = OSAtomicCompareAndSwap32Barrier(0, 1, &_hasConnected);

	if (shouldConnect) {
		self.serialDisposable.disposable = [self.sourceSignal subscribe:_signal];
	}

	return self.serialDisposable;
}
```



此处使用了方法

```c
bool OSAtomicCompareAndSwap32Barrier( int32_t __oldValue, int32_t __newValue, volatile int32_t *__theValue );
```
用于将`_hasConnected`与`0(oldValue)`进行比较，如果两者是匹配的，则方法结果返回YES并将`_hasConnected`赋值为`1(newValue)`。**即由未连接状态，修改成已连接状态。**

 当由未连接状态变为已连接状态时，执行信号的订阅方法。



总结一下`RACMulticastConnection`实现过程：

1. 创建`connect`，`connect.sourceSignal`-> `RACSignal(原始信号)`、` connect.signal` -> `RACSubject`。
2. 订阅`connect.signal`，会调用`RACSubject`的`subscribeNext`，创建订阅者，而且把订阅者保存起来，不会执行`block`。
3. `[connect connect]`内部会订阅`RACSignal(原始信号)`，并且订阅者是`RACSubject`
4. `RACSubject`的`sendNext`,会遍历`RACSubject`所有订阅者发送信号，并执行他们的`nextBlock( )`

---

【完】

