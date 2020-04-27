KVO监听探究，代码如下：

```objc
#import "ViewController.h"

#import "Person.h"

@interface ViewController ()

@property(nonatomic, strong)Person *p1;

@property(nonatomic, strong)Person *p2;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    _p1 = [Person new];
    _p2 = [Person new];
    
    _p2.name = @"Kody";
    
    [_p1 addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
    
    _p1.name = @"Tom";
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"change === %@",change);
}


@end
```

分别在`_p2.name = @"Kody";`和`_p1.name = @"Tom";`打上断点，运行。在第一个断点处，终端输入：

```
(lldb) po object_getClassName(_p1)
"Person"

"Person"

(lldb) po object_getClassName(_p2)
"Person"
```

运行到第二个断点处，终端输入：

```
(lldb) po object_getClassName(_p1)
"NSKVONotifying_Person"
```





再查看下`_p1`和`_p2`的地址，代码如下：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _p1 = [Person new];
    _p2 = [Person new];
    
    _p2.name = @"Kody";
    
    NSLog(@"监听之前 --- p1:%p,p2:%p", [_p1 methodForSelector:@selector(setName:)], [_p2 methodForSelector:@selector(setName:)]);
    
    [_p1 addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
    
    NSLog(@"监听之后 --- p1:%p,p2:%p", [_p1 methodForSelector:@selector(setName:)], [_p2 methodForSelector:@selector(setName:)]);
    
    _p1.name = @"Tom";
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"change === %@",change);
}
```

终端打印如下：

```
2020-04-25 10:20:26.142328+0800 004-CustomKVO[23187:2155801] 监听之前 --- p1:0x1013e6f00,p2:0x1013e6f00
2020-04-25 10:20:26.143450+0800 004-CustomKVO[23187:2155801] 监听之后 --- p1:0x10176bc8a,p2:0x1013e6f00
2020-04-25 10:20:26.143907+0800 004-CustomKVO[23187:2155801] change === {
    kind = 1;
    new = Tom;
}
```

发现p1的地址发生了变化，不是同一个实例变量了。





#### 注意：

在编写`objc_msgSendSuper(&superClass, **_cmd**, name);`时会报错，需要在`Build Settings`中的`Enalbe Stric Checking of objc_msgSend Calls`设置为`NO`。