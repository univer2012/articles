参考：
1. [浅谈 iOS Notification](https://www.jianshu.com/p/2503e3e5fc64)

我们在开发程序的时候，程序内不同对象间的通信是不可避免的，iOS中主要有以下这些通信方式：

![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E6%B5%85%E8%B0%88%20iOS%20Notification_1.png)

图中按照耦合度的强弱和通信的形式(一对一还是一对多)进行了划分，这篇文章我们主要说一下Notifications。

通知机制想必大家都很熟悉，平常的开发中或多或少的应该都用过。它是Cocoa中一个非常重要的机制，能把一个事件发送给多个监听该事件的对象，而消息的发送者不需要知道消息接收者的任何信息，消息的接受者也只是向通知中心（`NSNotificationCenter`）注册监听自己感兴趣的通知即可，任何对象都可以监听或者发送通知，这在很大程度上降低了消息发送者和接受者之间的耦合度。这也是iOS中观察者模式的一种实现方式。

# Notification（通知）

当我们发通知时，我们发送的是什么？答案是Notification，一个Notification对象封装了通知发送者想要传递给监听的的信息，它有3个属性：
```
@property (readonly, copy) NSString *name;  // 通知的名称，用来标示一个通知，一般为字符串
@property (nullable, readonly, retain) id object;   // 任意想要携带的对象，通常为发送者自己
@property (nullable, readonly, copy) NSDictionary *userInfo;    // 附加信息
```
通知就是以`Notification`的形式从通知发送者发出，到通知中心，然后再分发给所有监听该通知的对象的，通知监听者们接收到通知之后，可以获取到传递过来的`Notification`对象，从而获取里面封装的一些信息，做相应的处理。

# NSNotificationCenter（通知中心）

通知中心是整个通知机制的关键所在，它管理着监听者的注册和注销，通知的发送和接收。通知中心维护着一个通知的分发表，把所有通知发送者发送的通知，转发给对应的监听者们。每一个iOS程序都有一个唯一的通知中心，你不必自己去创建一个，它是一个单例，通过`[NSNotificationCenter defaultCenter]`方法获取。

注册监听者方法:
```
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(nullable NSString *)aName object:(nullable id)anObject;

- (id <NSObject>)addObserverForName:(nullable NSString *)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block;
```
第一个方法是大家常用的方法，不用多说。

第二个方法带了一个block，这个block就是通知被触发时要执行的block，这个block带有一个notification参数；**该方法还有一个queue参数，可以指定这个block在哪个队列上执行，如果传nil，这个block将会在发送通知的线程中同步执行**。然后注意到，这个方法有一个id类型的返回值，这个返回值是一个不透明的中间值，用来充当监听者，使用时，我们需要将这个返回的监听者保存起来，在后面移除监听者的时候用到。

移除监听者方法:
```
- (void)removeObserver:(id)observer;
- (void)removeObserver:(id)observer name:(nullable NSString *)aName object:(nullable id)anObject;
```

在监听对象销毁前，记得把改对象监听的通知移除掉。

发送通知方法:
```
- (void)postNotification:(NSNotification *)notification;
- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject;
- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject userInfo:(nullable NSDictionary *)aUserInfo;
```

还有2点需要注意的是：

1. **通知中心默认是以同步的方式发送通知的**，也就是说，当一个对象发送了一个通知，==**只有当该通知的所有接受者都接受到了通知中心分发的通知消息并且处理完成后，发送通知的对象才能继续执行接下来的方法**==。异步发送通知的方法下面会说到。
2. 在一个多线程的程序中，发送方发送通知的线程通常就是监听者接受通知的线程，这可能和监听者注册时的线程不一样。

> Cocoa中有2种通知中心，一种是`NSNotificationCenter`，它只能处理一个程序内部的通知，另一种是`NSDistributedNotificationCenter`（mas OS上才有）,它能处理一台机器上不同程序之间的通知，由于本篇主要讲iOS的通知，所以在这就不赘述，感兴趣的同学可以去[苹果官方文档](https://link.jianshu.com/?t=https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Notifications/Articles/NotificationCenters.html#//apple_ref/doc/uid/20000216-BAJGDAFC)里详细了解。

# NSNotificationQueue（通知队列）

通知队列，顾名思义，就是放通知（Notification对象）的队列。一般的发送通知方式，通知中心收到发送者发出的通知后，会立刻分发给监听者。==但是如果把通知放在通知队列中，通知就可以等到某些特定时刻再发出，比如等到之前发出的通知在runloop中处理完，或者runloop空闲的时候==。它就像通知中心的缓冲池，把一些不着急发出的通知存在通知队列中。

![2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E6%B5%85%E8%B0%88%20iOS%20Notification_2.png)

==这些存储在通知队列中的通知会以先进先出的方法发出（FIFO==），放一个通知到达队列的头部，它将被通知队列转发给通知中心，然后通知中心再分发给相应的监听者们。

==每个线程有一个默认的通知队列，它和通知中心关联着==，你也可以自己为线程或者通知中心创建多个通知队列。

通知队列给通知机制提供了2个重要的特性：==通知合并==和==异步发送通知==

# 通知合并
有时候，对一个可能会发生不止一次的事件，你想发送一个通知去通知某些对象做一些事，但当这个事件重复发生时，你又不想再发送同样的通知了。

你可能会这样做，设置一个flag来决定是否还需要发送通知，当第一个通知发出去时，把这个flag设置为不在需要发送通知，那么当相同的事件再发生时，就不会发送相同的通知了，看起来很美好，不过这样是不能达到我们的目的的，还是那个问题，因为普通的通知发送方式默认是同步的，通知的发送者需要等到所有的监听者都接收并处理完消息才能接着处理接下来的业务逻辑，也就是说当第一个通知发出的时候，可能还没回来，第二个通知已经发出去了，在你改变flag的值的时候，可能已经发出去若干个通知了...

这个时候，就需要用到通知队列的通知合并功能了。使用`NSNotificationQueue`的`enqueueNotification:postingStyle:coalesceMask:forModes:`方法，设置第三个参数coalesceMask的值，来指定不同的合并规则，coalesceMask有3个给定的值：

```
typedef NS_OPTIONS(NSUInteger, NSNotificationCoalescing) {
    NSNotificationNoCoalescing = 0,
    NSNotificationCoalescingOnName = 1,
    NSNotificationCoalescingOnSender = 2
};
```
分别是==不合并==，==按通知的名字合并==，和==按通知的发送者合并==。

设置合并规则后再加入到通知队列中，通知队列会按照给定的合并规则，在之前入队的通知中查找，然后移除符合合并规则的通知，这样就达到了只发送一个通知的目的。

合并规则还可以用 `|` 符号连接，指定多个：

```
NSNotification *myNotification = [NSNotification notificationWithName:@"MyNotificationName" object:nil];
[[NSNotificationQueue defaultQueue] enqueueNotification:myNotification
                                          postingStyle:NSPostWhenIdle
                                          coalesceMask:NSNotificationCoalescingOnName | NSNotificationCoalescingOnSender
                                              forModes:nil];
```

# 异步发送通知
使用通知队列的下面2个方法，将通知加到通知队列中，就可以将一个通知异步的发送到当前的线程，==这些方法调用后会立即返回，不用再等待通知的所有监听者都接收并处理完==。
```
- (void)enqueueNotification:(NSNotification *)notification postingStyle:(NSPostingStyle)postingStyle;

- (void)enqueueNotification:(NSNotification *)notification postingStyle:(NSPostingStyle)postingStyle coalesceMask:(NSNotificationCoalescing)coalesceMask forModes:(nullable NSArray<NSString *> *)modes;
```
> 注意：如果通知入队的线程在该通知被通知队列发送到通知中心之前结束了，那么这个通知将不会被发送了。

注意到上面第二个方法中，有一个`modes`参数，当指定了某种特定runloop mode后，该通知值有在当前runloop为指定mode的下，才会被发出。

通知队列发送通知有3种类型，也就是上面方法中的`postingStyle`参数，它有3种取值：
```
typedef NS_ENUM(NSUInteger, NSPostingStyle) {
    NSPostWhenIdle = 1,
    NSPostASAP = 2,
    NSPostNow = 3
};
```
#### NSPostASAP （尽快发送 Posting As Soon As Possible）
- 以`NSPostASAP`风格进入队列的通知会在运行循环的当前迭代完成时被发送给通知中心，如果当前运行循环模式和请求的模式相匹配的话（如果请求的模式和当前模式不同，则通知在进入请求的模式时被发出）。

- 由于运行循环在每个迭代过程中可能进行多个调用分支（callout），所以在当前调用分支退出及控制权返回运行循环时，通知可能被分发，也可能不被分发。其它的调用分支可能先发生，比如定时器或由其它源触发了事件，或者其它异步的通知被分发了。

开发者通常可以将NSPostASAP风格用于开销昂贵的资源，比如显示服务器。如果在运行循环的一个调用分支过程中有很多客户代码在窗口缓冲区中进行描画，在每次描画之后将缓冲区的内容刷新到显示服务器的开销是很昂贵的。在这种情况下，每个draw...方法都会将诸如“FlushTheServer” 这样的通知排入队列，并指定按名称和对象进行合并，以及使用NSPostASAP风格。结果，在运行循环的最后，那些通知中只有一个被派发，而窗口缓冲区也只被刷新一次。

#### NSPostWhenIdle（空闲时发送）
以NSPostWhenIdle风格进入队列的通知只在运行循环处于等待状态时才被发出。在这种状态下，运行循环的输入通道中没有任何事件，包括定时器和异步事件。以NSPostWhenIdle风格进入队列的一个典型的例子是当用户键入文本、而程序的其它地方需要显示文本字节长度的时候。在用户输入每一个字符后都对文本输入框的尺寸进行更新的开销是很大的（而且不是特别有用），特别是当用户快速输入的时候。在这种情况下，Cocoa会在每个字符键入之后，将诸如“ChangeTheDisplayedSize”这样的通知进行排队，同时把合并开关打开，并使用NSPostWhenIdle风格。当用户停止输入的时候，队列中只有一个“ChangeTheDisplayedSize”通知（由于合并的原因）会在运行循环进入等待状态时被发出，显示部分也因此被刷新。请注意，运行循环即将退出（当所有的输入通道都过时的时候，会发生这种情况）时并不处于等待状态，因此也不会发出通知。

#### NSPostNow（立即发送）

以NSPostNow风格进入队列的通知会在合并之后，立即发送到通知中心。开发者可以在不需要异步调用行为的时候 使用NSPostNow风格（或者通过NSNotificationCenter的postNotification:方法来发送）。在很多编程环境下，我们不仅允许同步的行为，而且希望使用这种行为：即开发者希望通知中心在通知派发之后返回，以便确定观察者对象收到通知并进行了处理。当然，当开发者希望通过合并移除队列中类似的通知时，应该用enqueueNotification...方法，且使用NSPostNow风格，而不是使用`postNotification:`方法。


# 发送通知到指定线程
通知中心分发通知的线程一般就是通知的发出者发送通知的线程。但是有时候，你可能想自己决定通知发出的线程，而不是由通知中心来决定。举个栗子，在后台线程中有一个对象监听者主线程中界面上的一些变化，比如一个window的关闭或者一个按钮的点击，这时通知是在主线程中发出的，通常来说只能在主线程中接受，但是你会希望这个对象能在后台线程中接到通知，而不是主线程中。这时你就需要在这些通知本来在的线程中抓住它们，然后将它们重定向到你想要指定的线程。

一种实现思路就是实现自定义的通知队列（不是`NSNotificationQueue`）去保存那些通知，然后将他们重定向到你指定的线程，大致流程就是：照常用一个对象去监听一个通知，当这个通知被触发时，监听者接受到后，判断当前线程是否为处理该通知正确的线程，如果是则处理，否则，将改通知保存到我们自定义的通知队列中，然后给目标队列发送一个信号，表明这个通知需要在目标队列中处理，目标队列接受到信号后，从通知队列中取出通知并处理。
不过这个处理思路也是有一些局限性的，[苹果官方文档](https://link.jianshu.com/?t=https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Notifications/Articles/Threading.html#//apple_ref/doc/uid/20001289-CEGJFDFG)中给出了一些基本代码和思路，感兴趣的同学可以看看。

## 参考

1. [NSHipster](https://link.jianshu.com/?t=http://nshipster.com/nsnotification-and-nsnotificationcenter/)
2. [iOS Notification Center](https://www.jianshu.com/p/209ef870e131)
3. [苹果官方文档](https://link.jianshu.com/?t=https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Notifications/Introduction/introNotifications.html#//apple_ref/doc/uid/10000043-SW1)