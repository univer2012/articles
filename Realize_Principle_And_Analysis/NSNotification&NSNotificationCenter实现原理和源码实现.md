参考：
1. [NSNotification&NSNotificationCenter实现原理和源码实现](https://www.jianshu.com/p/a307587ac62c)

# 简述
在iOS中，NSNotification & NSNotificationCenter是使用观察者模式来实现的用于跨层传递消息。

# 观察者模式
定义：定义对象间的一种一对多的依赖关系。当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动更新。

应用场景：
1. 有两种抽象类型相互依赖。将它们封装在各自的对象中，就可以对它们单独进行改变和复用。
2. 对一个对象的改变需要同时改变其他对象，而不知道具体有多少对象有待改变。

3. 一个对象必须通知其他对象，而它又不需要知道其他对象是什么。

![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/NSNotification%26NSNotificationCenter%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E5%92%8C%E6%BA%90%E7%A0%81%E5%AE%9E%E7%8E%B0_1.png)

## NSNotification的使用
#### 1.向通知中心注册观察者
```
addObserver:selector:name:object:
（观察者接收到通知后执行任务的代码在发送通知的线程中执行）

addObserverForName:object:queue:usingBlock:
（观察者接收到通知后执行任务的代码在指定的操作队列中执行）
```
#### 2.通知中心向所有注册的观察者发送通知
```
postNotification:
postNotificationName:object:
postNotificationName:object:userInfo:
```
#### 3.观察者收到通知，执行相应代码

# 根据NSNotification&NSNotificationCenter接口给出实现代码
创建两个新类HHNotification,HHNotificationCenter，这两个类的接口和苹果提供的接口完全一样，我将根据接口提供的功能给出实现代码。

要点是通知中心是单例类，并且通知中心维护了一个包含所有注册的观察者的集合，这里我选择了动态数组来存储所有的观察者，源码如下：
```
+ (HHNotificationCenter *)defaultCenter
{
    static HHNotificationCenter *singleton;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        singleton = [[self alloc] initSingleton];
    });
    return singleton;
}

- (instancetype)initSingleton
{
    if ([super init]) {
        _observers = [NSMutableArray array];
    }
    return self;
}
```

还定义了一个观察者模型用于保存观察者，通知消息名，观察者收到通知后执行代码所在的操作队列和执行代码的回调，模型源码如下：
```
@interface HHObserverModel : NSObject

@property (nonatomic, strong) id observer;
@property (nonatomic, assign) SEL selector;
@property (nonatomic, copy) NSString *notificationName;
@property (nonatomic, strong) id object;
@property (nonatomic, strong) NSOperationQueue *operationQueue;
@property (nonatomic, copy) OperationBlock block;

@end
```
向通知中心注册观察者，源码如下：
```
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject
{
    HHObserverModel *observerModel = [[HHObserverModel alloc] init];
    observerModel.observer = observer;
    observerModel.selector = aSelector;
    observerModel.notificationName = aName;
    observerModel.object = anObject;
    [self.observers addObject:observerModel];
}
- (id <NSObject>)addObserverForName:(NSString *)name object:(id)obj queue:(NSOperationQueue *)queue usingBlock:(void (^)(HHNotification * _Nonnull))block
{
    HHObserverModel *observerModel = [[HHObserverModel alloc] init];
    observerModel.notificationName = name;
    observerModel.object = obj;
    observerModel.operationQueue = queue;
    observerModel.block = block;
    [self.observers addObject:observerModel];
    return nil;
}
```
发送通知有三种方式，最终都是调用`- (void)postNotification:(HHNotification *)notification`，源码如下：

```
- (void)postNotification:(HHNotification *)notification
{
    [self.observers enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        HHObserverModel *observerModel = obj;
        id observer = observerModel.observer;
        SEL selector = observerModel.selector;
        if (!observerModel.operationQueue) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            [observer performSelector:selector withObject:notification];
#pragma clang diagnostic pop
        } else {
            NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
                observerModel.block(notification);
            }];
            NSOperationQueue *operationQueue = observerModel.operationQueue;
            [operationQueue addOperation:operation];
        }
    }];
}
```

总结：iOS中不仅通知中心使用了观察者，KVO也是基于观察者模式实现的。整个项目的源码已经放在了GitHub上[项目源码](https://link.jianshu.com/?t=https://github.com/zongmumask/NSNotifcationCracking/tree/master)。如源码中有错误，欢迎指正。