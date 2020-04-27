参考：
1. [iOS Notification实现原理](https://blog.csdn.net/qq_18505715/article/details/76146575)

```
目录 
    一、通知的基本使用
    1、基本概念
    2、什么情况下使用通知
    3、如何使用通知
    4、使用通知需要注意哪些细节
    
    二、通知的实现原理
    1、概述
    2、实现
```

## 一、通知的基本使用
### 1、基本概念

`NSNotification` 是iOS中一个调度消息通知的类,采用单例模式设计,在程序中实现传值、回调等地方应用很广。在iOS中，`NSNotification` & `NSNotificationCenter`是使用观察者模式来实现的用于跨层传递消息。

### 2、什么情况下使用通知
观察者模式 ： 定义对象间的一种一对多的依赖关系。当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动更新。

应用场景 :
1. 对一个对象的改变需要同时改变其他对象，而不知道具体有多少对象有待改变。

2. 一个对象必须通知其他对象，而它又不需要知道其他对象是什么


### 3、如何使用通知
#### 1> 向观察者中心添加观察者(2种方式)
```
//观察者接收到通知后执行任务的代码在发送通知的线程中执行 
addObserver:selector:name:object: 

//观察者接收到通知后执行任务的代码在指定的操作队列中执行 
addObserverForName:object:queue:usingBlock:
```

#### 2> 通知中心向观察者发送消息
```
    postNotification:
    postNotificationName:object:
    postNotificationName:object:userInfo:
```
#### 3> 观察者接收到消息执行相应的行为

#### 4> 在通知中心移除观察者
```
removeObserver:
removeObserver:name:object:
```
### 4、使用通知需要注意哪些细节

1>. 通知一定要移除，在dealloc方法里面移除
2. 通知有同步通知和异步通知，只不过我们同步通知用得比较多。
3. 不能用`- (instancetype)init` 初始化一个通知

## 二、通知的实现原理

1、概述 ：

首先，信息的传递就依靠通知(NSNotification),也就是说，通知就是信息(执行的方法，观察者本身(self),参数)的包装。通知中心(NSNotificationCenter)是个单例，向通知中心注册观察者，也就是说，这个通知中心有个集合，这个集合存放着观察者。那么这个集合是什么样的数据类型 ？ 

可以这么思考： 发送通知需要name参数，添加观察者也有个name参数，这两个name一样的时候，当发送通知时候，观察者对象就能接受到信息，执行对应的操作。那么这个集合很容易想到就是NSDictionary!

==key就是name，value就是NSArray(存放数据模型)，里面存放观察者对象==。如下图

![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%20Notification%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86_1.png)

当发送通知时，在通知通知的字典，根据name找到value，这个value就是一数组，数组里面存放数据模型(observer、SEL)。即可执行对应的行为。

## 2、实现