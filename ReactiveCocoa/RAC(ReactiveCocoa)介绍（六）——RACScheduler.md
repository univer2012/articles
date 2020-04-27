来自：[RAC(ReactiveCocoa)介绍（六）——RACScheduler](https://www.jianshu.com/p/320e231aca52)



上一篇介绍了一下RAC订阅信号中，方法实现`RACCompoundDisposable`真正的订阅者类的相关介绍。
 这一篇，将在订阅信号的方法中继续探究，当真正的订阅者初始化之后，后续代码涉及到了`RACScheduler`类的使用。将针对`RACScheduler`类进行深入的剖析。

```objc
///RACDynamicSignal.m
- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	NSCParameterAssert(subscriber != nil);

	RACCompoundDisposable *disposable = [RACCompoundDisposable compoundDisposable];
	subscriber = [[RACPassthroughSubscriber alloc] initWithSubscriber:subscriber signal:self disposable:disposable];

	if (self.didSubscribe != NULL) {
		RACDisposable *schedulingDisposable = [RACScheduler.subscriptionScheduler schedule:^{
			RACDisposable *innerDisposable = self.didSubscribe(subscriber);
			[disposable addDisposable:innerDisposable];
		}];

		[disposable addDisposable:schedulingDisposable];
	}
	
	return disposable;
}
```

先从上图的代码中可以发现，`RACScheduler`的`block`代码块中，有执行`self.didSubscribe( )`代码块的代码，即执行创建信号时的代码块，也就意味着此处的`schedule`代码块是必须执行。

 跳转进入查看`schedule`方法，可以发现有多个基于`RACScheduler`的子类，都有该实现方法。
 在`RACScheduler`类中，有三种子类：`RACImmediateScheduler`、`RACSubscriptionScheduler`和`RACQueueScheduler`

![img](https:////upload-images.jianshu.io/upload_images/1243805-c80d45a11ded6493.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)


 在`RACImmediateScheduler`子类看到`schedule`方法中，是立即执行block代码块。 

```objectivec
- (RACDisposable *)schedule:(void (^)(void))block {
    NSCParameterAssert(block != NULL);

    block();
    return nil;
}
```



接着是`RACSubscriptionScheduler`子类中的`schedule`方法实现。

```objectivec
///RACSubscriptionScheduler.m
- (RACDisposable *)schedule:(void (^)(void))block {
    NSCParameterAssert(block != NULL);

    if (RACScheduler.currentScheduler == nil) return [self.backgroundScheduler schedule:block];

    block();
    return nil;
}
```

在`RACSubscriptionScheduler`子类`schedule`方法中，在执行代码块之前，会先判断当前线程、当前队列是否为`nil`。若不为`nil`，则会在后台开启一个串行异步线程队列。而`self.backgroundScheduler`通过`[RACScheduler scheduler]`创建并实例化的：

```objc
///RACSubscriptionScheduler.m
- (instancetype)init {
	self = [super initWithName:@"org.reactivecocoa.ReactiveObjC.RACScheduler.subscriptionScheduler"];

	_backgroundScheduler = [RACScheduler scheduler];

	return self;
}
```



最终可以追溯到以下代码中：

```objc
///RACScheduler.m
+ (RACScheduler *)schedulerWithPriority:(RACSchedulerPriority)priority name:(NSString *)name {
	return [[RACTargetQueueScheduler alloc] initWithName:name targetQueue:dispatch_get_global_queue(priority, 0)];
}
```



上述代码意味着是由`RACTargetQueueScheduler`类初始化，而`RACTargetQueueScheduler`是`RACQueueScheduler`的子类。
 而`RACTargetQueueScheduler`类实例化方法实现中，使用GCD在目标线程里创建了一个串行队列。

```objc
@implementation RACTargetQueueScheduler

#pragma mark Lifecycle

- (instancetype)initWithName:(NSString *)name targetQueue:(dispatch_queue_t)targetQueue {
	NSCParameterAssert(targetQueue != NULL);

	if (name == nil) {
		name = [NSString stringWithFormat:@"org.reactivecocoa.ReactiveObjC.RACTargetQueueScheduler(%s)", dispatch_queue_get_label(targetQueue)];
	}

	dispatch_queue_t queue = dispatch_queue_create(name.UTF8String, DISPATCH_QUEUE_SERIAL);
	if (queue == NULL) return nil;

	dispatch_set_target_queue(queue, targetQueue);

	return [super initWithName:name queue:queue];
}

@end
```



那么，此时来看下`RACQueueScheduler`类的`schedule`方法实现：

```objc
///RACQueueScheduler.m
- (RACDisposable *)schedule:(void (^)(void))block {
	NSCParameterAssert(block != NULL);

	RACDisposable *disposable = [[RACDisposable alloc] init];

	dispatch_async(self.queue, ^{
		if (disposable.disposed) return;
		[self performAsCurrentScheduler:block];
	});

	return disposable;
}
```

此处使用GCD开启了一个异步线程，在后台执行下一步操作，作为当前的`Scheduler`去执行。所以说，**`RACQueueScheduler`类实现的`schedule`方法是在后台创建一个串行队列异步线程来实现最终的代码块执行。**

```objc
///RACScheduler.m
- (void)performAsCurrentScheduler:(void (^)(void))block {
	NSCParameterAssert(block != NULL);

	// If we're using a concurrent queue, we could end up in here concurrently,
	// in which case we *don't* want to clear the current scheduler immediately
	// after our block is done executing, but only *after* all our concurrent
	// invocations are done.

	RACScheduler *previousScheduler = RACScheduler.currentScheduler;
	NSThread.currentThread.threadDictionary[RACSchedulerCurrentSchedulerKey] = self;

	@autoreleasepool {
		block();
	}

	if (previousScheduler != nil) {
		NSThread.currentThread.threadDictionary[RACSchedulerCurrentSchedulerKey] = previousScheduler;
	} else {
		[NSThread.currentThread.threadDictionary removeObjectForKey:RACSchedulerCurrentSchedulerKey];
	}
}
```



在该方法中，首先找到当前的`Scheduler`队列(`RACScheduler.currentScheduler`)；然后从当前线程字典中找到关于`RACSchedulerCurrentSchedulerKey`键值并将`RACScheduler`自己赋值给它。

 如果当前`Scheduler`队列不为空，则会把当前的`Scheduler`队列存入到当前线程字典的`RACSchedulerCurrentSchedulerKey`键值中；若为空，则把当前线程字典的`RACSchedulerCurrentSchedulerKey`键值内容全部删除。一旦删除，意味着当前线程中的队列已不存在。当队列不存在时，会利用`Objective-C`的动态机制，会自动修复重启当前的队列。



 上面有一段代码：

```objc
@autoreleasepool {
  block();
}
```

此段代码中的`block`执行时，也就是创建`RACSignal`信号的`block`中，会有临时变量的产生，`autoreleasepool`意味着延迟释放。其中涉及到`runloop`知识范围，此处不做深入讨论。





以上内容主要讲解了`RACScheduler`类以及三种子类的作用与实现过程原理，后续会继续探究RAC内部实现的具体流程。



---

【完】