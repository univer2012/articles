# KVO原理探究



#### 课程内容

KVO原理解析与应用

1. 使用（参数的意义，哪些用到了KVO）
2. 开启方式（手动和自动）
3. 原理探析（系统做了些什么；是否监听地址变化等）
4. 容器类监听使用（RAC bug）
5. 多级路径属性管理（维护）



**用到KVO的例子**

1. NSOperation
2. NSOperationQueue
3.  RAC

对象有： 观察者、被观察的对象属性。

使用的最后一定要`移除观察者`，否则会导致崩溃；



添加观察者的方法：

```objc
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
```

其中`NSKeyValueObservingOptions`是一个枚举值：

```objc
typedef NS_OPTIONS(NSUInteger, NSKeyValueObservingOptions) {
    NSKeyValueObservingOptionNew = 0x01,	//返回新值
    NSKeyValueObservingOptionOld = 0x02,	//返回旧值

    NSKeyValueObservingOptionInitial API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0)) = 0x04,	//再注册的使用发送一次通知，在改变时也会发送通知
    NSKeyValueObservingOptionPrior API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0)) = 0x08		//改变之前，发送一次通知，改变之后再发送一次通知

};
```

> 单词：
>
> Prior [ˈpraɪə(r)] adj. （时间、顺序等）先前的；优先的
>
> Initial [ɪ'nɪʃəl]   adj. 最初的；字首的



监听值变化的方法：

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"change:%@", change);
}
```

其中`NSKeyValueChangeKey` 也有4个值；

```
typedef NS_ENUM(NSUInteger, NSKeyValueChange) {
    NSKeyValueChangeSetting = 1,
    NSKeyValueChangeInsertion = 2,
    NSKeyValueChangeRemoval = 3,
    NSKeyValueChangeReplacement = 4,
};
```

我们一般用`NSKeyValueChangeSetting`，下面的3个是针对容器的，后面会讲到容器的监听。



#### 怎么实现手动开启KVO

手动开启KVO的方式是，在被监听的类里面实现：

```objc
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
    if ([key isEqualToString:@"name"]) {
        return NO;
    }
    return YES;
}
```

这个方法是`NSKerValueObserving.h`文件中，`NSObject(NSKeyValueObservingCustomization)`的扩展方法。



如果该方法返回`NO`，就转化为自动的。



添加上面的方法到`EOCFamily.m`后，再次运行，发现没有执行`- (void)observeValueForKeyPath:ofObject:change: context:`了。

这时我们修改为：

```diff
+		[_eocFamily willChangeValueForKey:@"name"];
    _eocFamily.name = @"EOC_New";
+    [_eocFamily didChangeValueForKey:@"name"];
```

这样就触发了手动模式。自动方法中也是调用了`willChangeValueForKey:`和`didChangeValueForKey:`这2个方法。



那我们什么场景会调用这种手动模式呢？

手动模式就是我们自己来控制，在一定条件下触发通知，不是任何时候改变都会发通知。

比如条件是：

```objc
		NSString *name = @"eoc";
    if (name.length > 2) {
        [_eocFamily willChangeValueForKey:@"name"];
        _eocFamily.name = name;
        [_eocFamily didChangeValueForKey:@"name"];
    }
```



### 底层原理做了什么事情？

原理是：利用运行时，生成一个对象的子类，并生成子类对象，并替换原理对象的isa指针，并且重写了set方法。



我们打印一些日志，并运行。`SHKVOExploreViewController.m`的所有代码如下：

```objc
#import "SHKVOExploreViewController.h"
#import "EOCFamily.h"

@interface SHKVOExploreViewController () {
    EOCFamily *_eocFamily;
}

@end

@implementation SHKVOExploreViewController
/**
 NSOperation
 NSOperationQueue
 RAC
 
 KVO 观察者莫斯
 观察者，被观察的对象属性  移除观察者
 */
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _eocFamily = [EOCFamily new];
    
    _eocFamily.name = @"EOC_Old";
    
    NSLog(@"befor: %s", object_getClassName(_eocFamily));
    [_eocFamily addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
    NSLog(@"after: %s", object_getClassName(_eocFamily));
    
    NSString *name = @"eoc";
    if (name.length > 2) {
        [_eocFamily willChangeValueForKey:@"name"];
        _eocFamily.name = name;
        [_eocFamily didChangeValueForKey:@"name"];
    }
    
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    //NSLog(@"change:%@", change);
}
    
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
//    NSString *tempName = @"EOC";
//    _eocFamily.name = tempName;
//    NSLog(@"two:%p", _eocFamily.name);
}

@end
```

同时`EOCFamily.m`要注释掉`automaticallyNotifiesObserversForKey:`，因为手动的话就不会触发系统的KVO。所有代码如下：

```objc
#import "EOCFamily.h"

@implementation EOCFamily
    
- (instancetype)init
{
    self = [super init];
    if (self) {
        _person = [EOCPerson new];
        _eocAry = [NSMutableArray new];
    }
    return self;
}
    
//+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
//    if ([key isEqualToString:@"name"]) {
//        return NO;
//    }
//    return YES;
//}

@end
```

打印如下：

```objc
2019-11-03 14:27:46.284994+0800 ObjectiveCDemo160728[39039:895160] befor: EOCFamily
2019-11-03 14:27:46.285761+0800 ObjectiveCDemo160728[39039:895160] after: NSKVONotifying_EOCFamily
```

这样我们就验证了，替换了isa指针。那怎么验证设置了值了呢？

我们写一个方法，返回当前的子类对象：

```objc
+ (NSArray *)findSubClass:(Class)defaultClass {
    int count = objc_getClassList(NULL, 0);
    if (count <= 0) {
        return [NSArray array];
    }
    NSMutableArray *output = [NSMutableArray arrayWithObject:defaultClass];
    Class *classes = (Class *)malloc(sizeof(Class) * count);
    objc_getClassList(classes, count);
    for (int i = 0; i < count; i++) {
        if (defaultClass == class_getSuperclass(classes[i])) {
            [output addObject:classes[i]];
        }
    }
    free(classes);
    return output;
}
```



然后`viewDidLoad`方法改为：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _eocFamily = [EOCFamily new];
    
    _eocFamily.name = @"EOC_Old";
    
    //NSLog(@"befor: %s", object_getClassName(_eocFamily));
    NSLog(@"befor: %@",[SHKVOExploreViewController findSubClass:[_eocFamily class]]);
    [_eocFamily addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
    //NSLog(@"after: %s", object_getClassName(_eocFamily));
     NSLog(@"after: %@",[SHKVOExploreViewController findSubClass:[_eocFamily class]]);
    
    NSString *name = @"eoc";
    if (name.length > 2) {
        [_eocFamily willChangeValueForKey:@"name"];
        _eocFamily.name = name;
        [_eocFamily didChangeValueForKey:@"name"];
    }
    
}
```

打印如下：

```
2019-11-03 15:08:17.904078+0800 ObjectiveCDemo160728[41490:953422] befor: (
    EOCFamily
)
2019-11-03 15:08:17.915533+0800 ObjectiveCDemo160728[41490:953422] after: (
    EOCFamily,
    "NSKVONotifying_EOCFamily"
)
```



####我们探究的原理，是否是监听地址的变化？

探究代码如下：

```objc
#import "SHKVOExploreViewController.h"
#import "EOCFamily.h"

#import <objc/runtime.h>

@interface SHKVOExploreViewController () {
    EOCFamily *_eocFamily;
}

@end

@implementation SHKVOExploreViewController
/**
 NSOperation
 NSOperationQueue
 RAC
 
 KVO 观察者莫斯
 观察者，被观察的对象属性  移除观察者
 */
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _eocFamily = [EOCFamily new];
    
    _eocFamily.name = @"EOC_Old";
    
    [_eocFamily addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
    
    
    NSString *name = @"eoc";
    _eocFamily.name = name;
    NSLog(@"one:%p", _eocFamily.name);
    
}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    NSString *tempName = @"eoc";
    _eocFamily.name = tempName;
    NSLog(@"two:%p", _eocFamily.name);
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"change:%@", change);
}
    


+ (NSArray *)findSubClass:(Class)defaultClass {
    int count = objc_getClassList(NULL, 0);
    if (count <= 0) {
        return [NSArray array];
    }
    NSMutableArray *output = [NSMutableArray arrayWithObject:defaultClass];
    Class *classes = (Class *)malloc(sizeof(Class) * count);
    objc_getClassList(classes, count);
    for (int i = 0; i < count; i++) {
        if (defaultClass == class_getSuperclass(classes[i])) {
            [output addObject:classes[i]];
        }
    }
    free(classes);
    return output;
}
    
@end
```

打印如下：

```objc
2019-11-03 15:14:11.271920+0800 ObjectiveCDemo160728[41618:985424] change:{
    kind = 1;
    new = eoc;
}
2019-11-03 15:14:11.272120+0800 ObjectiveCDemo160728[41618:985424] one:0x100679130
2019-11-03 15:14:14.470084+0800 ObjectiveCDemo160728[41618:985424] change:{
    kind = 1;
    new = eoc;
}
2019-11-03 15:14:14.470416+0800 ObjectiveCDemo160728[41618:985424] two:0x100679130
```

可以看到，地址前后是一样的。可以得出结论：不是对地址进行监听的。那到底是怎么来触发通知的呢？



注意上面的【原理】中描述的内容：`并且重写了set方法。`也就是说，系统是在`setName:`方法，在赋值前后，分别加了`willChangeValueForKey:`和`didChangeValueForKey:`这2个方法。

代码如下：

```objc
- (void)setName:(NSString *)name {
    [self willChangeValueForKey:name];		//系统添加的方法
    _name = @"name";
    [self didChangeValueForKey:name];			//系统添加的方法
}
```





### 4.容器类的监听使用

容器类包括`NSArray`、`NSMutableArray`、`NSDictionary`、`NDMutableDictionary`、`NSSet`、`NSMutableSet`等。



探究代码如下：

```objc
#import "SHKVOExploreViewController.h"
#import "EOCFamily.h"

#import <objc/runtime.h>

@interface SHKVOExploreViewController () {
    EOCFamily *_eocFamily;
}

@end

@implementation SHKVOExploreViewController
/**
 NSOperation
 NSOperationQueue
 RAC
 
 KVO 观察者莫斯
 观察者，被观察的对象属性  移除观察者
 */
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _eocFamily = [EOCFamily new];
    
    
    [_eocFamily addObserver:self forKeyPath:@"eocAry" options:NSKeyValueObservingOptionNew context:nil];
    
    [_eocFamily.eocAry addObject:@"one"];
    
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"change:%@", change);
}
    
@end
```



上面代码是没有触发KVO的通知的。因为上面的代码没有执行`set`方法。



把上面代码改为：

```diff
+   _eocFamily.eocAry = [NSMutableArray new];
-    [_eocFamily.eocAry addObject:@"one"];
```

那就会触发通知。因为这样就触发了`set`方法了。



要监听`eocAry`容器，需要这样写：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _eocFamily = [EOCFamily new];
    
    _eocFamily.name = @"EOC_Old";
    
    [_eocFamily addObserver:self forKeyPath:@"eocAry" options:NSKeyValueObservingOptionNew context:nil];
    
    [[_eocFamily mutableArrayValueForKeyPath:@"eocAry"] addObject:@"one"];
    
}
```

这样就可以监听到`eocAry`的元素的变化了：

```
2019-11-03 16:04:27.385007+0800 ObjectiveCDemo160728[42703:1136192] change:{
    indexes = "<_NSCachedIndexSet: 0x600003866cc0>[number of indexes: 1 (in 1 ranges), indexes: (0)]";
    kind = 2;
    new =     (
        one
    );
}
```



`- (NSMutableArray *)mutableArrayValueForKeyPath:`方法是在`Foundation` -->`NSKeyValueCode.h`中的一个方法，也就是说，该方法是属于KVC的方法。



#### 6.KVO为什么是基于KVC的？

所以说，KVO为什么是基于KVC的？ 可以这样说，KVC就是KVO的入口。我们做一下打印：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _eocFamily = [EOCFamily new];
    
    _eocFamily.name = @"EOC_Old";
    
    [_eocFamily addObserver:self forKeyPath:@"eocAry" options:NSKeyValueObservingOptionNew context:nil];
    
    NSMutableArray *tempAry = [_eocFamily mutableArrayValueForKeyPath:@"eocAry"];
    NSLog(@"tempAry:%s", object_getClassName(tempAry));
    NSLog(@"eocAry:%s", object_getClassName(_eocFamily.eocAry));
    
    [[_eocFamily mutableArrayValueForKeyPath:@"eocAry"] addObject:@"one"];
    
}
```

打印结果如下：

```objc
2019-11-03 16:10:54.259702+0800 ObjectiveCDemo160728[43047:1167705] tempAry:NSKeyValueNotifyingMutableArray
2019-11-03 16:10:54.259888+0800 ObjectiveCDemo160728[43047:1167705] eocAry:__NSArrayM
```

所以我们可以知道，`mutableArrayValueForKeyPath:`是通过KVC的方法来修改了数组的isa指针。



#### 5.多级路径属性管理

如果监听的是`_eocFramily.person`，代码如下：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _eocFamily = [EOCFamily new];
    
    _eocFamily.name = @"EOC_Old";
    
    [_eocFamily addObserver:self forKeyPath:@"person" options:NSKeyValueObservingOptionNew context:nil];
    _eocFamily.person.age = @"11";
}
```

如果监听子路径，是可以接收到通知的：

```objc
[_eocFamily addObserver:self forKeyPath:@"person.age" options:NSKeyValueObservingOptionNew context:nil];
```

但是本质上，只要一个`person`的任何一个属性发生了改变，那么这个人就是应该发生了变化。如果通过监听它的子路径去监听这个人的变化的话，那就会写一系列上面的代码方法。



这样是很麻烦的。我们可以通过在`person`属性所在类`EOCFamily.m`中添加 【子集路径关联】 方法：

```objc
+ (NSSet<NSString *> *)keyPathsForValuesAffectingValueForKey:(NSString *)key {
    NSSet *keySet = [super keyPathsForValuesAffectingValueForKey:key];
    
    if ([key isEqual:@"person"]) {
        NSSet *set = [NSSet setWithObject:@"_person.age"];
        keySet = [keySet setByAddingObjectsFromSet:set];
    }
    return keySet;
}
```

>  注意：
>
> `NSSet *set = [NSSet setWithObject:@"_person.age"];`的`_person.age`是带`_`开头的。如果没有这个`_`，程序会崩溃，找不到这个路径。



添加上面的【子集路径关联】 方法后，运行，会发现用[_eocFamily addObserver:self forKeyPath:@"person" options:NSKeyValueObservingOptionNew context:nil];`直接监听`person`是可以收到监听的。

```
2019-11-03 16:39:47.787107+0800 ObjectiveCDemo160728[43734:1284703] change:{
    kind = 1;
    new = "<EOCPerson: 0x60000027d680>";
}
```



