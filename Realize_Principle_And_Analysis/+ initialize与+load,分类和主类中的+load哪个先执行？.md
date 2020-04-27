来自：[iOS - + initialize 与 +load](http://www.jianshu.com/p/9368ce9bb8f9)

来自：[NSObject的load和initialize方法](http://www.cocoachina.com/ios/20150104/10826.html)

> Objective-C 有两个神奇的方法：+load 和 +initialize，这两个方法在类被使用时会自动调用。但是两个方法的不同点会导致应用层面上性能的显著差异。

# 一、+ initialize 方法和+load 调用时机
首先说一下 + initialize 方法。苹果官方对这个方法有这样的一段描述：
> Initializes the class before it receives its first message.
> 在接收该类的第一个消息之前初始化该类。

也就是说，这个方法会在 **第一次初始化这个类之前** 被调用，我们用它来初始化静态变量。

 load 方法会在加载类的时候就被调用，也就是 **ios 应用启动的时候，就会加载所有的类，就会调用每个类的+load 方法。**

 之后我们结合代码来探究一下 + initialize 与 + load 两个方法的调用时机。

 ### 1.1.1 首先看+load ：
 ```
 //main.m
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
#import <Foundation/Foundation.h>
int main(int argc, char * argv[]) {
    NSLog(@"%s",__func__);
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

/*Person.m 
 Person.h文件在此忽略*/
#import "Person.h"

@implementation Person
+(void)load {
    NSLog(@"%s", __func__);
}
+(void)initialize {
    [super initialize];
    NSLog(@"%s %@", __func__, [self class]);
}
- (instancetype)init {
    self = [super init];
    if (self) {
        NSLog(@"%s", __func__);
    }
    return self;
}

/*Girl.m
Girl.h文件在此忽略*/
#import "Girl.h"
@implementation Girl
+(void)load {
    NSLog(@"%s", __func__);
}
+(void)initialize {
    [super initialize];
    NSLog(@"%s", __func__);
}
- (instancetype)init {
    self = [super init];
    if (self) {
        NSLog(@"%s", __func__);
    }
    return self;
}
@end
 ```

 运行程序，我们看一下输出日志：
 ```
 +[Person load]
 +[Girl load]
 main
 ```
 这说明，在我并<u><font color=#ff0000 size=3>没有对类做任何操作的情况下，+load 方法会被默认执行，并且是在 main 函数之前执行的。</font></u>

### 1.1.2 接下来我们来查看一下 + initialize 方法，
在苹果文档中有这段话：

> The superclass implementation may be called multiple times if subclasses do not implement `initialize()`—the runtime will call the inherited implementation—or if subclasses explicitly call `[super initialize]`. If you want to protect yourself from being run multiple times, you can structure your implementation along these lines:
```
+ (void)initialize {
 if (self == [ClassName self]) {
 // ... do the initialization ...
 }
}
```
> 父类的实现可以被多次调用，如果子类没有实现initialize() ——运行时将调用子类继承实现——或者子类显式调用 [super initialize] 。如果你想保护自己不被多次运行，你可以沿着这些路线构建你的实现：

也就是说，子类要不不重写 `+initialize` 方法，重写就要显示调用 `[super initialize]` 。  

 先在 ViewController 中创建 Person 和 Girl 对象：
 ```
 #import "ViewController.h"
#import "Person.h"
#import "Girl.h"
@interface ViewController ()
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    Person *a = [Person new];
    Person *b = [Person new];
    Girl *c = [Girl new];
    Girl *d = [Girl new];
}
@end
 ```
 输出如下：
 ```
 +[Person load]
 +[Girl load]
 main
 +[Person initialize] Person
 -[Person init]
 -[Person init]
 +[Person initialize] Girl
 +[Girl initialize]
 -[Person init]
 -[Girl init]
 -[Person init]
 -[Girl init]
 ```
如果`Girl`类中没有重写 `+initialize` 方法，则打印结果如下：

```
+[Person load]
 +[Girl load]
 main
 +[Person initialize] Person
 -[Person init]
 -[Person init]
 +[Person initialize] Girl
 -[Person init]
 -[Girl init]
 -[Person init]
 -[Girl init]
```

即**子类在没有重写 `+initialize` 方法时，会在运行时自动调用父类的  `+initialize` 。**

通过这个实验我们可以确定两点：
1. `+ initialize` 方法类似一个懒加载，如果没有使用这个类，那么系统默认不会去调用这个方法，且默认只加载一次；
2. `+ initialize` 的调用发生在 `+init` 方法之前。

### 1.1.3  + initialize 在父类与子类之间的关系
创建一个继承自 Person 类的 Son类，Son类不用实现任何代码：
```
#import "ViewController.h"
#import "Person.h"
#import "Girl.h"
#import "Son.h"
@interface ViewController ()
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    Person *a = [Person new];
    Person *b = [Person new];
    Son *z = [Son new];
}
@end
```
输出日志：
```
 +[Person load]
 +[Girl load]
 main
 +[Person initialize] Person
 -[Person init]
 -[Person init]
 +[Person initialize] Son
 -[Person init]
```
会看到， Person 类的 `+ initialize` 方法又被调用了，但是查看一下是子类 Son 调用的，也就是创建子类的时候，**子类会去调用父类的 + initialize 方法。**

# 二、总结
1. 类被加载时+ load会自动被调用，这个调用非常早。如果你实现了一个应用或框架的 + load，并且你的应用链接到这个框架上了，那么 ＋ load 会在 main() 函数之前被调用。如果你在一个可加载的 bundle 中实现了 + load，那么它会在 bundle 加载的过程中被调用。
2. `+initialize` 方法的调用看起来会更合理，通常在它里面写代码比在 + load 里写更好。+ initialize 很有趣，因为它是懒调用的，也有可能完全不被调用。
3. 类第一次被加载时，+ initialize 不会被调用。类接收消息时，运行时会先检查 + initialize 有没有被调用过。如果没有，会在消息被处理前调用。