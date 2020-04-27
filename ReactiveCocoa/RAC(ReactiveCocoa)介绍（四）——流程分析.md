之前的文章一直在写关于RAC框架中某些方法的实现原理、具体使用作用以及代码实现，基本上也对RAC有了一个初步的认识。这次不打算继续记录关于RAC中具体的方法，准备放到后面慢慢分析，这篇准备继续深入探究一下RAC中信号的流程分析。这样对RAC认识会更加深刻，有助于后续的理解。

 首先，贴上之前的代码为例

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    //1.创建signal信号
    RACSignal *signal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        //subscriber并不是一个对象
        //3. 发送信号
        [subscriber sendNext:@"send one Message"];
        
        //发送error信号
        NSError *error = [NSError errorWithDomain:NSURLErrorDomain code:1001 userInfo:@{@"errorMsg": @"this is a error message"}];
        [subscriber sendError:error];
        
        //4.销毁信号
        return [RACDisposable disposableWithBlock:^{
            NSLog(@"signal已销毁");
        }];
    }];
    
    //2.1 订阅信号
    [signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x);
    }];
    
    //2.2 针对实际中可能出现的逻辑错误，RAC提供了订阅error信号
    [signal subscribeError:^(NSError * _Nullable error) {
        NSLog(@"%@",error);
    }];
}
```



```objc
2019-11-20 09:07:51.141354+0800 ObjectiveCDemo160728[3281:107000] send one Message
2019-11-20 09:07:51.142004+0800 ObjectiveCDemo160728[3281:107000] signal已销毁
2019-11-20 09:07:51.142621+0800 ObjectiveCDemo160728[3281:107000] Error Domain=NSURLErrorDomain Code=1001 "(null)" UserInfo={errorMsg=this is a error message}
2019-11-20 09:07:51.143190+0800 ObjectiveCDemo160728[3281:107000] signal已销毁
```



按照代码中的执行步骤：
 在创建信号`signal`之后，订阅了两个信号`signal`；**当分别发送两个信号`signal`时，订阅信号`signal`方法block中首先打印出发送信号`signal`的内容，然后执行信号`signal`销毁。**注意：RAC方法中的block代码块，



#### 1. 创建信号`signal`
    在创建信号`signal`的方法`createSignal`中，可以看到是由`RACDynamicSignal`类创建的信号`signal`：

```objc
+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe {
	RACDynamicSignal *signal = [[self alloc] init];
	signal->_didSubscribe = [didSubscribe copy];
	return [signal setNameWithFormat:@"+createSignal:"];
}
```

方法中可以看出，将block代码块didSubscribe复制copy给了当前的实例变量_didSubscribe



#### 2. 订阅信号signal

**订阅信号方法`subscribeNext`中，使用`RACSubscriber`类进行了初始化并将`next`的`block`保存**。在该实例化方法中，也可将`error`的block与`completed`的block传入，此处因外部使用`subscribeNext`方法，所以只传入`next`。**与之相对应的，`subscribeError`方法则是初始化并保存了error的block，`subscribeCompleted`方法是初始化并保存了`completed`的block。**

```objc
///RACSignal.m
- (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock error:(void (^)(NSError *error))errorBlock completed:(void (^)(void))completedBlock {
	NSCParameterAssert(nextBlock != NULL);
	NSCParameterAssert(errorBlock != NULL);
	NSCParameterAssert(completedBlock != NULL);
	
	RACSubscriber *o = [RACSubscriber subscriberWithNext:nextBlock error:errorBlock completed:completedBlock];
	return [self subscribe:o];
}
```

```objc
///RACSubscriber.m
+ (instancetype)subscriberWithNext:(void (^)(id x))next error:(void (^)(NSError *error))error completed:(void (^)(void))completed {
	RACSubscriber *subscriber = [[self alloc] init];

	subscriber->_next = [next copy];
	subscriber->_error = [error copy];
	subscriber->_completed = [completed copy];

	return subscriber;
}
```



```objc
///RACSignal.m
- (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock {
	NSCParameterAssert(nextBlock != NULL);
	
	RACSubscriber *o = [RACSubscriber subscriberWithNext:nextBlock error:NULL completed:NULL];
	return [self subscribe:o];
}
```



订阅信号实现代码中，`[self subscribe:o]`方法中的`self`的类是`RACDynamicSignal`（刚才第1步中创建信号类为`RACDynamicSignal`），进入`RACDynamicSignal`类中的`subscribe`方法。

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

`RACCompoundDisposable`为组合式销毁栈，用于处理信号的销毁。至于其中如何运作的，之后专门用一篇幅来详细介绍。

<u>**`subscriber`被重新指向由`RACPassthroughSubscriber`类初始化的对象，通过该类创建的对象才是信号真正的订阅者**</u>。通过该方法将信号、订阅者、销毁者全部传入进去，换句话说，就是该方法将信号、订阅者、销毁者进行了关联，从而实现不断订阅、不断销毁。此处亦是RAC的核心之一。



```objc
RACDisposable *innerDisposable = self.didSubscribe(subscriber);
[disposable addDisposable:innerDisposable];
```

此处代码要注意到，是执行了`self.didSubscribe(id<RACSubscriber> subscriber)`代码块。而`self.didSubscribe(id<RACSubscriber> subscriber)`是在创建信号的时候，进行了`copy`赋值操作，在订阅者这里进行执行操作。所以，在第2步订阅信号创建的时候，会进入第1步创建信号的block代码块中寻找并执行自定义添加的代码，也就是准备开始执行第一张代码图中的第3步、第4步操作。



#### 第3步，发送信号signal
 通过进入`RACSubscriber`类中的`sendNext`方法查看实现代码

```objc
///RACSubscriber.m
- (void)sendNext:(id)value {
	@synchronized (self) {
		void (^nextBlock)(id) = [self.next copy];
		if (nextBlock == nil) return;

		nextBlock(value);
	}
}
```

保证`RACSubscriber`类的线程安全，将`self.next`代码块`copy`至一个本地代码块，并将其执行。此处的`self.next`正是在第2步订阅信号过程中，保存的`nextBlock`代码块。 最终在执行发送信号`signal`的时候，会查找并执行在第2步订阅信号方法中的block代码块。

因此，整个信号流程的执行，也正是函数式编程思想的完美体现。

最后再配上一张流程图，来总结一下上述实现的步骤。



![RAC流程分析简图](https://upload-images.jianshu.io/upload_images/1243805-b92daa17b8a044a6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

---

【完】