使用运行时分析发现保留环(retain cycle)的iOS库。

## 有关
保留环是创建内存泄露最常见的方法之一。创建一个保留环非常容易，但很难发现它。FBRetainCycleDetector的目标是在运行中帮助找到保留环。这个项目的特征受到Circle的影响。

## 安装
#### Carthage

到你的Cartfile添加
```
github "facebook/FBRetainCycleDetector"
```
FBRetainCycleDetector建立在非调试版本，所以当你想测试它，使用：
```
carthage update --configuration Debug
```
#### CocoaPods
到你的podspec添加：
```
pod 'FBRetainCycleDetector'
```
你完全可以只在调试版本使用FBRetainCycleDetector，这是由编译标志控制的，它可以提供给构建以使其在其他配置中工作。

## 使用示例
让我们快速开动起来：
```
#import <FBRetainCycleDetector/FBRetainCycleDetector.h>
```
```
FBRetainCycleDetector *detector = [FBRetainCycleDetector new];
[detector addCandidate:myObject];
NSSet *retainCycles = [detector findRetainCycles];
NSLog(@"%@", retainCycles);
```
`- (NSSet<NSArray<FBObjectiveCGraphElement *> *> *)findRetainCycles` 将返回一个包装对象数组的集合。起初很难看，但让我们来看看它。该集合中的每个数组将表示一个保留环。这个数组中的每个元素都是这个保留循环中一个对象周围的包装器。检查FBObjectiveCGraphElement。

示例输出可以如下所示：
```
{(
    (
        "-> MyObject ",
        "-> _someObject -> __NSArrayI "
    )
)}
```

FBRetainCycleDetector寻找这些环不超过10个对象。我们可以把它变大（尽管将会变慢！）
```
FBRetainCycleDetector *detector = [FBRetainCycleDetector new];
[detector addCandidate:myObject];
NSSet *retainCycles = [detector findRetainCyclesWithMaxCycleLength:100];
```

### Filters
也有一些我们想忽略的保留环。因为不是每个保留环都是漏洞，我们可能想要过滤它们。为此我们需要制定过滤器：
```
NSMutableArray *filters = @[
  FBFilterBlockWithObjectIvarRelation([UIView class], @"_subviewCache"),
];

// Configuration object can describe filters as well as some options
FBObjectGraphConfiguration *configuration =
[[FBObjectGraphConfiguration alloc] initWithFilterBlocks:filters
                                     shouldInspectTimers:YES];
FBRetainCycleDetector *detector = [[FBRetainCycleDetector alloc] initWithConfiguration:configuration];
[detector addCandidate:myObject];
NSSet *retainCycles = [detector findRetainCycles];
```
每个过滤器是一个block，该block有2个可以沟通的FBObjectiveCGraphElement对象，如果它们的关系是有效的。
查看FBStandardGraphEdgeFilters学习更多有关如何使用过滤器。

### NSTimer
NSTimer比较麻烦因为它会保留它的target。这通常意味着一个保留环。FBRetainCycleDetector能够检测到这些，但如果你想跳过它们，你可以在配置中指定，在你路过的FBRetainCycleDetector时。
```
FBObjectGraphConfiguration *configuration =
[[FBObjectGraphConfiguration alloc] initWithFilterBlocks:someFilters
                                     shouldInspectTimers:NO];
FBRetainCycleDetector *detector = [[FBRetainCycleDetector alloc] initWithConfiguration:configuration];
```

### 关联
Objective-C让我们能使用`objc_setAssociatedObject`为每个对象设置关联对象。这些关联对象能携带保留环，如果我们使用固定的策略，像`OBJC_ASSOCIATION_RETAIN_NONATOMIC`。`FBRetainCycleDetector`能够捕捉这类环，但我们需要这样设置。在应用程序的生命周期的早期，最好是在main.m中我们可以添加这个：
```
#import <FBRetainCycleDetector/FBAssociationManager.h>

int main(int argc, char * argv[]) {
  @autoreleasepool {
    [FBAssociationManager hook];
    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
}
```
代码中有关`[FBAssociationManager hook]`将使用鱼钩插入函数`objc_setAssociatedObject`和`objc_resetAssociatedObjects`来跟踪关联，在他们产生之前。