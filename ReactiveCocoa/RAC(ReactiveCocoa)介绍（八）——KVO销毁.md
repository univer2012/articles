来自：[RAC(ReactiveCocoa)介绍（八）——KVO销毁](https://www.jianshu.com/p/a23c09ebe9e6)



上一篇探究了RAC的销毁机制，既然说到销毁，就不得不说下RAC中的KVO销毁。

在RAC中使用KVO时，仅需一行代码，即可完成对指定对象的属性变化值监听，而且不再需要时刻关注KVO销毁。在这一行代码中，RAC内部是如何自动完成KVO的销毁管理？

```objc
[RACObserve(self.testLabel, text) subscribeNext:^(id  _Nullable x) {
    NSLog(@"%@",x);
}];
```



宏定义RACObserve方法实现中，可以发现`- (RACSignal *)rac_valuesForKeyPath:(NSString *)keyPath observer:(__weak NSObject *)observer;`方法。

```objc
#define _RACObserve(TARGET, KEYPATH) \
({ \
	__weak id target_ = (TARGET); \
	[target_ rac_valuesForKeyPath:@keypath(TARGET, KEYPATH) observer:self]; \
})
```



进入`- (RACSignal *)rac_valuesForKeyPath:(NSString *)keyPath observer:(__weak NSObject *)observer`方法实现

```objc
- (RACSignal *)rac_valuesForKeyPath:(NSString *)keyPath observer:(__weak NSObject *)observer {
	return [[[self
		rac_valuesAndChangesForKeyPath:keyPath options:NSKeyValueObservingOptionInitial observer:observer]
		map:^(RACTuple *value) {
			// -map: because it doesn't require the block trampoline that -reduceEach: uses
			return value[0];
		}]
		setNameWithFormat:@"RACObserve(%@, %@)", RACDescription(self), keyPath];
}
```



在实现方法中，可以发现使用了`map`映射方法，当KVO监听属性时，会有多个数据通过元组的形式返回，此时只取元组中的第一个值，即`newValue`。为什么要取元组value的第一个值，这时打印一下value就会发现：



![img](https://upload-images.jianshu.io/upload_images/1243805-0e4b1c090fa9e675.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



`value`中以元组类型存在的数据，与KVO本身监听属性值变化时的打印值非常相像。那么value中的第一个元素，即为RAC KVO方法监听到的新属性值。

深入探究主要带着目的去查看：**如何实现内部管理销毁KVO**



在该RAC实现方法中，既然是要探究如何在RAC内部方法中如何销毁KVO，那么直接寻找返回`RACDisposable`类型的代码。在代码实现中，发现下面该方法返回了一个`RACDisposable`类型数据，即为销毁信号。

```objc
///NSObject+RACPropertySubscribing.m
return [self rac_observeKeyPath:keyPath options:options observer:observer block:^(id value, NSDictionary *change, BOOL causedByDealloc, BOOL affectedOnlyLastComponent) {
  
				[subscriber sendNext:RACTuplePack(value, change)];
  
			}];
```



继续深入该方法的实现过程，会在该方法中找到组合式销毁信号`RACCompoundDisposable`，组合式销毁与RACDisposable区别在于，组合式销毁能够将多个销毁信号保存并在执行销毁操作时，将保存的所有销毁信号进行销毁操作。

 `RACCompoundDisposable`类中，声明了一个CF可变数组`_disposables`，并提供了相应的初始化、添加销毁信号、移除销毁信号操作。

```objc
///RACCompoundDisposable.h
@interface RACCompoundDisposable : RACDisposable

/// Creates and returns a new compound disposable.
+ (instancetype)compoundDisposable;

/// Creates and returns a new compound disposable containing the given
/// disposables.
+ (instancetype)compoundDisposableWithDisposables:(nullable NSArray *)disposables;

/// Adds the given disposable. If the receiving disposable has already been
/// disposed of, the given disposable is disposed immediately.
///
/// This method is thread-safe.
///
/// disposable - The disposable to add. This may be nil, in which case nothing
///              happens.
- (void)addDisposable:(nullable RACDisposable *)disposable;

/// Removes the specified disposable from the compound disposable (regardless of
/// its disposed status), or does nothing if it's not in the compound disposable.
///
/// This is mainly useful for limiting the memory usage of the compound
/// disposable for long-running operations.
///
/// This method is thread-safe.
///
/// disposable - The disposable to remove. This argument may be nil (to make the
///              use of weak references easier).
- (void)removeDisposable:(nullable RACDisposable *)disposable;

@end
```



RACKVOTrampoline类初始化：

![img](https:////upload-images.jianshu.io/upload_images/1243805-82b09b9bc35e3a6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)




 进入到该方法中，会发现该方法将keyPath、block、target、strongTarget、observer属性保存至成员变量中。为什么要保存至成员变量中，是因为主要作用于RAC的信号销毁机制。RAC销毁机制可查看上一篇文章



```objc
///RACKVOTrampoline.m
#pragma mark Lifecycle

- (instancetype)initWithTarget:(__weak NSObject *)target observer:(__weak NSObject *)observer keyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options block:(RACKVOBlock)block {
	NSCParameterAssert(keyPath != nil);
	NSCParameterAssert(block != nil);

	NSObject *strongTarget = target;
	if (strongTarget == nil) return nil;

	self = [super init];

	_keyPath = [keyPath copy];

	_block = [block copy];
	_weakTarget = target;
	_unsafeTarget = strongTarget;
	_observer = observer;

	[RACKVOProxy.sharedProxy addObserver:self forContext:(__bridge void *)self];
	[strongTarget addObserver:RACKVOProxy.sharedProxy forKeyPath:self.keyPath options:options context:(__bridge void *)self];

	[strongTarget.rac_deallocDisposable addDisposable:self];
	[self.observer.rac_deallocDisposable addDisposable:self];

	return self;
}
```



其中一行代码实现的方法与系统中KVO方法对比后，会发现此处就是对系统KVO方法的封装。

```objc
//RAC内部方法
[strongTarget addObserver:RACKVOProxy.sharedProxy forKeyPath:self.keyPath options:options context:(__bridge void *)self];

//系统KVO实现方法
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
```



在`RACKVOTrampoline`类继承自`RACDisposable`父类，其中有实现`dispose`销毁方法。



在RAC探究过程中，已经知道在`RACSignal`信号销毁时，都会执行手动销毁+自动销毁信号的流程。当执行`dispose`方法销毁时，`RACKVOTrampoline`类实现了对系统KVO监听者的`remove`移除操作。

```objc
///RACKVOTrampoline.m
- (void)dispose {
	NSObject *target;
	NSObject *observer;

	@synchronized (self) {
		_block = nil;

		// The target should still exist at this point, because we still need to
		// tear down its KVO observation. Therefore, we can use the unsafe
		// reference (and need to, because the weak one will have been zeroed by
		// now).
		target = self.unsafeTarget;
		observer = self.observer;

		_unsafeTarget = nil;
		_observer = nil;
	}

	[target.rac_deallocDisposable removeDisposable:self];
	[observer.rac_deallocDisposable removeDisposable:self];
	
  ///备注：调用了KVO的remove移除方法
	[target removeObserver:RACKVOProxy.sharedProxy forKeyPath:self.keyPath context:(__bridge void *)self];
	[RACKVOProxy.sharedProxy removeObserver:self forContext:(__bridge void *)self];
}
```



**通过层层查看代码实现，最终找到了RAC内部是大体如何实现KVO监听者的remove移除操作。**

**RAC本身实际上是对系统KVO监听者的封装，将KVO监听者的创建与销毁操作放入销毁信号`RACDisposable`类的子类：`RACKVOTrampoline`。通过RAC的手动+自动销毁信号机制，在`dispose`方法中实现对KVO监听者的remove操作。最终达成RAC KVO的内部自动管理remove监听者KVO的目的。**

---

【完】