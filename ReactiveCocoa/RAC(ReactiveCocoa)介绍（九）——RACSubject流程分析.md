来自：[RAC(ReactiveCocoa)介绍（九）——RACSubject流程分析](https://www.jianshu.com/p/5f51ac4885b4)



之前介绍了RACSignal类以及使用方法，这次要介绍的是RACSignal的子类：RACSubject。
 先看下RACSubject基本使用方法：

```objc
//1.创建信号
RACSubject *subject = [RACSubject subject];
//2.订阅信号
[subject subscribeNext:^(id  _Nullable x) {
    NSLog(@"%@",x);
}];
//3.发送信号
[subject sendNext:@"this is a RACSubject"];
```



与`RACSignal`父类的使用方法相比，缺省了信号销毁的方法声明。

 查看RACSubject的内部实现：

```objc
///RACSubject.m
#pragma mark Lifecycle

+ (instancetype)subject {
	return [[self alloc] init];
}

- (instancetype)init {
	self = [super init];
	if (self == nil) return nil;

	_disposable = [RACCompoundDisposable compoundDisposable];
	_subscribers = [[NSMutableArray alloc] initWithCapacity:1];
	
	return self;
}

- (void)dealloc {
	[self.disposable dispose];
}

```



从这就可以看出，在初始化创建`RACSubject`类的对象同时，创建了一个组合式销毁栈，而且在`dealloc`方法中执行了组合式销毁栈的信号销毁操作，并没有和父类`RACSignal`一样，创建并保存了一个销毁信号的成员变量。也就是说`RACSubject`类是在内部通过组合式销毁栈`RACCompoundDisposable`自动完成信号销毁。

 那么与之对应的`RACSubject`与父类`RACSignal`的订阅、发送消息也有不同。



 在`RACSignal`类的`subscribe`方法中，执行了`_didSubscribe( )`代码块，也就是在创建信号时保存的成员变量`didSubscribe`销毁信号，而`RACSubject`类的subscribe方法则是通过`RACCompoundDisposable`来完成信号的销毁。下面两张图为`subscribe`方法的实现对比。



`RACSignal`类的`subscribe`方法实现：

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



`RACSubject`类的`subscribe`方法实现：

```objc
///RACSubject.m
- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	NSCParameterAssert(subscriber != nil);

	RACCompoundDisposable *disposable = [RACCompoundDisposable compoundDisposable];
	subscriber = [[RACPassthroughSubscriber alloc] initWithSubscriber:subscriber signal:self disposable:disposable];

	NSMutableArray *subscribers = self.subscribers;
	@synchronized (subscribers) {
		[subscribers addObject:subscriber];
	}
	
  ///备注：向组合式销毁栈添加信号
	[disposable addDisposable:[RACDisposable disposableWithBlock:^{
		@synchronized (subscribers) {
			// Since newer subscribers are generally shorter-lived, search
			// starting from the end of the list.
			NSUInteger index = [subscribers indexOfObjectWithOptions:NSEnumerationReverse passingTest:^ BOOL (id<RACSubscriber> obj, NSUInteger index, BOOL *stop) {
				return obj == subscriber;
			}];

			if (index != NSNotFound) [subscribers removeObjectAtIndex:index];
		}
	}]];

	return disposable;
}
```



接下来，该到了发送订阅消息的方法中：

```objc
///RACSubject.m
//备注：遍历所有的订阅者信号并发送一次
- (void)enumerateSubscribersUsingBlock:(void (^)(id<RACSubscriber> subscriber))block {
	NSArray *subscribers;
	@synchronized (self.subscribers) {
		subscribers = [self.subscribers copy];
	}

	for (id<RACSubscriber> subscriber in subscribers) {
		block(subscriber);
	}
}

#pragma mark RACSubscriber

- (void)sendNext:(id)value {
	[self enumerateSubscribersUsingBlock:^(id<RACSubscriber> subscriber) {
		[subscriber sendNext:value];
	}];
}

- (void)sendError:(NSError *)error {
	[self.disposable dispose];
	
	[self enumerateSubscribersUsingBlock:^(id<RACSubscriber> subscriber) {
		[subscriber sendError:error];
	}];
}

- (void)sendCompleted {
	[self.disposable dispose];
	
	[self enumerateSubscribersUsingBlock:^(id<RACSubscriber> subscriber) {
		[subscriber sendCompleted];
	}];
}
```



`RACSubject`类中在发送信号时，`RACSubject`类会将所有的订阅者信号全部遍历并发送一次。除此以外，`RACSubject`类与父类`RACSignal`的发送信号流程相同：执行相应的`nextBlock( )`

`RACSubject`类是将父类`RACSignal`的封装，将销毁信号的管理放在内部进行自动管理实现。为了更加形象的总结描述`RACSubject`类的流程，附加一张简单的流程分析图：



RACSubject类流程图：

![img](https:////upload-images.jianshu.io/upload_images/1243805-e02eb5e821688aeb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

---

【完】