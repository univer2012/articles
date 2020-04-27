来自：[RAC(ReactiveCocoa)介绍（七）——信号销毁](https://www.jianshu.com/p/940544653ede)



这一篇讲主要针对`RACSignal`信号销毁进行探究

在`RACSignal`信号发送命令执行之后，本着谁创建谁销毁的原则，最后一步必须要进行销毁操作。而销毁操作的执行则由`RACDisposable`类来完成。

 `RACDisposable`类在RAC中作为一个父类，由三种子类继承自它。`RACCompoundDisposable`、`RACSerialDisposable`以及`RACKVOTrampoline`。



首先来看下RACDisposable类在执行销毁`disposableWithBlock`方法时的操作。

```objc
///RACDisposable.m
- (instancetype)initWithBlock:(void (^)(void))block {
	NSCParameterAssert(block != nil);

	self = [super init];

	_disposeBlock = (void *)CFBridgingRetain([block copy]); 
	OSMemoryBarrier();

	return self;
}

+ (instancetype)disposableWithBlock:(void (^)(void))block {
	return [(RACDisposable *)[self alloc] initWithBlock:block];
}
```



可以看到，将销毁信号中的代码块进行了保存操作，赋值给了`_disposeBlock`。
 而`_dispostBlock`的声明方式为：

```objc
void * volatile _disposeBlock;
```



 `volatile`的作用是，每次取得数值的方式都是直接从内存中读取

`(void *)CFBridgingRetain( )`代码是Objective-C与C语言进行桥接的方法，使用`__bridge_retained`方法自行管理内存。

 桥接方法有三种：`__bridge`、`__bridge_retained`、`__bridge_transfer`。

* `__bridge`：是将Objective-C转换成C语言，OC对象交给CF对象同时其所有权不变化
*  `__bridge_retained`：将Objective-C转换成C语言，OC对象将所有权交给CF对象，但会解除自动管理内存机制ARC的所有权，意味着要自行进行内存管理。当管理对象需要释放时，必须要执行`CFBridgingRelease`方法来手动释放。



`__bridge_retained`内部方法：

```objc
NS_INLINE CF_RETURNS_RETAINED CFTypeRef _Nullable CFBridgingRetain(id _Nullable X) {
    return (__bridge_retained CFTypeRef)X;
}
```



`CFBridgingRelease`方法的内部实现，是为第三种方法`__bridge_transfer`的执行，将CF对象的所有权交给OC对象，给予管理对象自动管理内存机制ARC的所有权。

```objc
NS_INLINE id _Nullable CFBridgingRelease(CFTypeRef CF_CONSUMED _Nullable X) {
    return (__bridge_transfer id)X;
}
```

此处为什么将该block转换成C函数？将Objective-C对象转换成C函数的，而C函数可以直接拿到相应的函数指针，拿到函数指针之后就可以指向任意类型，即**重定向指针**。此处重定向指针之后，会在`dispose`方法进行指针处理。



`OSMemoryBarrier();`：被称为内存屏障，为了保证相应的对象按顺序依次执行。

 类似的，在`dispose`方法中使用到了`OSAtomicCompareAndSwapPtrBarrier( )`方法



```objc
bool	OSAtomicCompareAndSwapPtrBarrier( void *__oldValue, void *__newValue, void * volatile *__theValue );
```



对比第一个`oldValue`与 `& value`是否相等，若相等则返回BOOL值YES，并把第二个`newValue`赋值给 `& value`。



```objc
///RACDisposable.m
#pragma mark Disposal

- (void)dispose {
	void (^disposeBlock)(void) = NULL;

	while (YES) {
		void *blockPtr = _disposeBlock;
		if (OSAtomicCompareAndSwapPtrBarrier(blockPtr, NULL, &_disposeBlock)) {
			if (blockPtr != (__bridge void *)self) {
				disposeBlock = CFBridgingRelease(blockPtr);
			}

			break;
		}
	}

	if (disposeBlock != nil) disposeBlock();
}
```



`OSAtomicCompareAndSwapPtrBarrier( )`方法将`_disposeBlock`赋值给的`blockPtr`与`_disposeBlock`进行对比，如果相等就将`_disposeBlock`赋值为NULL，同时将`blockPtr`释放销毁，此处写法作用是将`_disposeBlock`置为NULL的操作，同时进入下一步判断`blockPtr`是否与`self`相同，若不同则将`blockPtr`的OC对象赋值给`disposeBlock`。

 那么，判断局部变量`disposeBlock`不为`nil`，意味着还存在销毁者，还不需要执行销毁操作，则继续执行`disposeBlock( )`，即销毁信号block中的代码块。



在`dispose`方法中，当`OSAtomicCompareAndSwapPtrBarrier( )`方法判断`_disposeBlock`与`blockPtr`不相同时，`_disposeBlock`无法赋值为NULL，就无法执行下一步操作。那么就在`dealloc`方法中执行置为NULL以及释放操作。



```objc
///RACDisposable.m
- (void)dealloc {
	if (_disposeBlock == NULL || _disposeBlock == (__bridge void *)self) return;

	CFRelease(_disposeBlock);
	_disposeBlock = NULL;
}
```



释放CF对象`_disposeBlock`，同时将其置为NULL。
 销毁信号的整个操作，并不需要外部进行管理，全部由内部执行操作完成，让开发更专注于业务逻辑。
 销毁过程中，是通过手动+自动释放来共同进行内存释放管理。



在发送信号的三种执行方法实现中，`sendNext`方法没有实现`[self.disposable dispose]`，而`sendError`与`sendCompleted`方法却实现了。

```objc
///RACSubscriber.m
#pragma mark RACSubscriber

- (void)sendNext:(id)value {
	@synchronized (self) {
		void (^nextBlock)(id) = [self.next copy];
		if (nextBlock == nil) return;

		nextBlock(value);
	}
}

- (void)sendError:(NSError *)e {
	@synchronized (self) {
		void (^errorBlock)(NSError *) = [self.error copy];
		[self.disposable dispose];

		if (errorBlock == nil) return;
		errorBlock(e);
	}
}

- (void)sendCompleted {
	@synchronized (self) {
		void (^completedBlock)(void) = [self.completed copy];
		[self.disposable dispose];

		if (completedBlock == nil) return;
		completedBlock();
	}
}
```



在`dispose`方法中，会有`while(YES)`的死循环，用于不断寻找销毁对象，直到找到为止，并将其销毁置空掉。

而`sendNext`方法并不意味着创建信号的代码块已执行结束完成，当创建信号的代码块中所有代码都已执行完成，但未实现`[self.disposable dispose]`方法，那么就会去执行`dealloc`方法。



##### 扩展一下：

 在控制器创建销毁信号时，若创建了一个`RACDisposable`类的成员变量，将其放入销毁信号`return`中。因为持有该销毁信号对象的是当前类，在RAC信号销毁过程中内部方法无法对其进行销毁操作，最终会导致内存泄漏问题



销毁信号使用成员变量而非临时变量，导致的内存泄漏：

![img](https://upload-images.jianshu.io/upload_images/1243805-dd6d61c5d94d7ded.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



---

【完】