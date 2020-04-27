参考：
1. [[iOS][OC] NSNotificationCenter 进阶及自定义（附源代码）](https://www.jianshu.com/p/f163c471b452)


## 1、并不总是需要移除观察者
==自 iOS 9  开始==（见 [release notes](https://developer.apple.com/library/content/releasenotes/Foundation/RN-FoundationOlderNotes/index.html#10_11NotificationCenter) ），==`Foundation` 调整了 `NSNotificationCenter` 对观察者的引用方式（ `zeroing weak reference`），不再给已释放的观察者发送通知，因此以往在 `dealloc` 时移除观察者的做法可以省去。==

如果是需要适配 iOS 8，那么 `UIViewController`及其子类可以省去移除通知的过程（亲测有效），而其他对象则需要在 `dealloc` 前移除观察者。

> 感谢 Ace 同学第一时间的测试发现

## 2、控制器添加和移除观察者的良好实践
控制器对象对于通知的监听通常是在生命周期的 `viewDidLoad` 方法处理，也就是说，在 `viewDidLoad` 之前，还未添加观察者，对应地在在移除通知通知时可以做是否加载了视图的判断如下：
```
- (void)dealloc {
    if (self.isViewLoaded) {
        [[NSNotificationCenter defaultCenter] removeObserver:self];
    }
}
```

==这一点 `isViewLoaded` 的判断，对于 `NSNotification` 的监听来说不是必要的，因为在未监听通知的情况下，调用 `removeObserver:` 方法是仍旧是安全的==，而 `KVO ( key-value observing`，则不然。==因为 KVO 在未监听的情况下移除观察者是不安全的，所以如果是在 `viewDidLoad` 监听KVO ，则 KVO 的移除就需要执行判断==：
```
- (void)dealloc {
    if (self.isViewLoaded) {
        [self removeObserver:someObj forKeyPath:@"someKeyPath"];
    }
}
```

此外，很多时候控制器的视图还未加载，也需要监听特定的通知，此时通知的监听适合在构造方法 `initWithNibName:bundle:` 方法中监听，此构造方法在代码或者 `Interface Builder` 构建实例时都会调用：
```
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self) {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onNotification:) name:@"kNotificationName" object:nil];
    }
    return self;
}
```

## 3、系统 `NSNotificationCenter` 是支持 block 手法的

自 `iOS 4` 开始通知中心即支持 block 回调，其 API 如下：
```
- (id <NSObject>)addObserverForName:(nullable NSNotificationName)name
                             object:(nullable id)obj
                              queue:(nullable NSOperationQueue *)queue
                         usingBlock:(void (^)(NSNotification *note))block
                                    NS_AVAILABLE(10_6, 4_0);
```

回调可以指定操作队列，并返回一个观察者对象。调用示例：
```
- (void)observeUsingBlock {
    NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
    observee = [center addObserverForName:@"kNotificationName"
                                   object:nil
                                    queue:[NSOperationQueue mainQueue]
                               usingBlock:^(NSNotification * _Nonnull note) {
                                   NSLog(@"got the note %@", note);
                               }];
}

- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:observee];
}
```

其中，有几点值得注意：

1. 方法返回一个 `id<NSObject>` 监听者对象，其实是系统的私有类的实例，因为没必要暴露其具体类型和接口，所以用一个 `id<NSObject>` 对象指明用途，从中可见协议的又一个应用场景。

2. 这个返回值对象是充当了原来的 `target-action` 的封装实现，在其内部触发了 `action` 后调用起初传入的 `block` 参数。

3. ==返回的观察者和 `block` 都会被通知中心所持有，因此使用者有义务在必要的时候调用 `removeObserver:` 方法，将此监听移除，否则监听者和 block及其所捕获的变量都不会释放，从而导致内存泄露==。此处详细的说明和解决方案可以参考 SwiftGG翻译组的翻译文章 [Block 形式的通知中心观察者是否需要手动注销](https://juejin.im/post/5b596722e51d4517c564accf)

## 4、在必要时提前拦截通知的发送
通知的使用在跨层和面向多个对象通信时十分便利，也因此而导致难以管理的问题颇受诟病，发送通知时可能需要统一做一些工作，此时对通知进行拦截是必要的。==`NSNotificationCenter` 是 `CFNotificationCenter` 的封装，有使用类似 `NSArray` 的类簇设计，并采用了单例模式返回共享实例 `defaultCenter`==。==通过直接继承的方式进行发送通知的拦截是不可行的，因为获得的是始终是静态的单例对象==，从 Telegram 公司的[开源项目工程](https://github.com/peter-iakovlev/Telegram)中可以看到：通过借鉴 `KVO` 的实现原理，将单例对象的类修改为特定的子类，从而实现通知的拦截。

第一步，修改通知中心单例的类：
```
interface GSNoteCenter : NSNotificationCenter

@end


/// 修改单例的类为一个子类的类型
void hack() {
    id center = [NSNotificationCenter defaultCenter];
    object_setClass(center, GSNoteCenter.class);
}
```

第二步，拦截通知的发送事件：

利用继承多态特性，在发送通知的前后进行拦截：
```
@implementation GSNoteCenter

- (void)postNotificationName:(NSNotificationName)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo
{
    // do something before post
    [super postNotificationName:aName object:anObject userInfo:aUserInfo];
    // do something after post
}
@end
```

PS：拦截之后可以发现系统发送通知的数量和频率真高，从这个侧面看发送通知的性能问题不用太过顾忌。