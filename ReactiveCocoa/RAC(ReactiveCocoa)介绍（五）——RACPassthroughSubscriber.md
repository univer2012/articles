来自：[RAC(ReactiveCocoa)介绍（五）——RACPassthroughSubscriber](https://www.jianshu.com/p/4e4ebee7a782)



上一篇整体分析了RAC的信号流程，这样对RAC的工作原理有了整体的认识。
 接下来将逐步深入了解RAC实现的底层。


#### `RACPassthroughSubscriber`类
在上一篇文章的流程分析中，真正的订阅者是`RACPassthroughSubscriber`类，它将创建信号、订阅者与销毁者进行了关联。
在`RACPassthroughSubscriber`类的头文件中，声明了一个`RACSignal`类的成员变量。

```objc
///RACPassthroughSubscriber.m
@property (nonatomic, unsafe_unretained, readonly) RACSignal *signal;
```

在声明的属性中，定义了`signal`为`unsafe_unretained`，意味着`signal`为弱引用，但又与`weak`属性不同。<u>`weak`属性当为`nil`时，对象会自动执行释放功能，导致该对象无法继续被使用。而`unsafe_unretained`同样作为弱引用，当为`nil`时却指针不会自动释放，保留一个野指针。如果使用此指针，程序会抛出` BAD_ACCESS `的异常。</u>



在此处多问一个为什么，此处信号为什么要使用unsafe_unretained而不是weak属性修饰？
 找了网上的几处分析，都在说下面这句：

> 这里之所以不是weak，是因为引用`RACSignal`仅仅只是一个`DTrace probes`动态跟踪技术的探针。如果设置成`weak`，会造成没必要的性能损失。



个人理解：`unsafe_unretained`属性不会自动将`signal`变为`nil`，而`weak`属性会自动置为`nil`。当`signal`置为`nil`时，当前signal不再是一个信号类型的对象，就无法再继续执行订阅信号、发送信号动作。所以要使用`unsafe_unretained`属性保留当前`signal`的类型，即使变成了野指针。



```objc
///RACPassthroughSubscriber.m
- (instancetype)initWithSubscriber:(id<RACSubscriber>)subscriber signal:(RACSignal *)signal disposable:(RACCompoundDisposable *)disposable {
	NSCParameterAssert(subscriber != nil);

	self = [super init];

	_innerSubscriber = subscriber;
	_signal = signal;					///_signal成员变量传值
	_disposable = disposable;

	[self.innerSubscriber didSubscribeWithDisposable:self.disposable];
	return self;
}
```



首先，此处的成员变量`signal`声明为`RACSignal`类，是`RACPassthroughSubscriber`类实例化方法执行时传入的。而该信号又是从`RACDynamicSignal`传来的，`RACDynamicSignal`持有了当前的`subscribe`；`subscribe`继续向上溯源，找到了在`RACSignal`类中的`[self subscribe:o]`方法。那么答案已经显而易见了，**`unsafe_unretained`属性的声明，就是为了防止在此过程中的循环引用。**



![RACSignal弱引用导图](https:////upload-images.jianshu.io/upload_images/1243805-149ae5a5772363c5.png?imageMogr2/auto-orient/strip|imageView2/2/w/704)



在`RACSignal`类的订阅方法`subscribeNext`方法中，当执行订阅信号时，`self`通过`LLDB`打印出的却是`RACDynamicSignal`类。

![self打印信息](https:////upload-images.jianshu.io/upload_images/1243805-991b6500bca81b15.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)





首先`RACDynamicSignal`类是`RACSignal`类的子类，此处进行了`RACSignal`类的分类扩展在分类中实现了`subscribeNext`方法。在创建信号时，信号类为`RACDynamicSignal`类。

![RACSignal分类](https:////upload-images.jianshu.io/upload_images/1243805-be9bdf33bad7f512.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



在发送信号时，打印`subscriber`会发现其所属类为`RACPassthroughSubscriber`。

打印结果：

![img](https:////upload-images.jianshu.io/upload_images/1243805-907eed11ac10b061.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



`self.innerSubscriber`对象为`RACSubscriber`类创建的，最终是执行`RACSubscriber`类下的`sendNext`方法，执行`nextBlock`。在执行时，`@synchronized (self)`用于保证线程安全

![RACPassthroughSubscriber发送信号实现](https:////upload-images.jianshu.io/upload_images/1243805-cc2cd813e2b181b3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



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



以上就是针对`RACPassthroughSubscriber`类的实现进行了一个简单的分析，后续会不断补充、修改。





