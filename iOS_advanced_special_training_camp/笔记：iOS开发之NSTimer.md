

### 第1种解决方案：

```objc
#import "SGH0425Leaks6ViewController.h"

@interface SGH0425Leaks6ViewController ()

@property (nonatomic, strong)NSTimer *timer;

@end

@implementation SGH0425Leaks6ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(fire) userInfo:nil repeats:YES];
    
    [[NSRunLoop currentRunLoop]addTimer:self.timer forMode:NSDefaultRunLoopMode];
  
}
- (void)didMoveToParentViewController:(UIViewController *)parent {
    //if (parent == nil) { 
    if (parent.presentingViewController == nil) {
        [self.timer invalidate];
        self.timer = nil;
    }
}

- (void)fire {
    NSLog(@"fire.....");
}

- (void)dealloc
{
    NSLog(@"SGH0425Leaks6ViewController Dealloc");
}

@end
```





### 第2种解决方案：

不让NSTimer持有self，采用「消息转发」来解决循环引用的问题。

代码如下：

```objc
#import "SGH0425Leaks6ViewController.h"

@interface SGH0425Leaks6ViewController ()

@property (nonatomic, strong)NSTimer *timer;

@property (nonatomic, strong) NSObject *target;

@end

@implementation SGH0425Leaks6ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _target = [NSObject new];
    class_addMethod([_target class], @selector(fire), (IMP)fireImp, "v@:");
    
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:_target selector:@selector(fire) userInfo:nil repeats:YES];
}

void fireImp(id self, SEL _cmd) {
    NSLog(@"fireIMP --- fire.....");
}

- (void)fire {
    NSLog(@"fire.....");
}

- (void)dealloc
{
    NSLog(@"SGH0425Leaks6ViewController Dealloc");
    [self.timer invalidate];
    self.timer = nil;
}

@end
```



内存分析如图：

![second_issue@2x.png](https://upload-images.jianshu.io/upload_images/843214-359ff46ed727cec8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





### 第3种解决方案

使用中间变量NSProxy，弱持有self，来打破self与NSTimer中间的循环。

代码如下：

```objc
//=======SGHTimerProxy.h

@interface SGHTimerProxy : NSProxy

@property (nonatomic, weak) id target;

//lgProxy.target = self;

@end
```

```objc
//======SGHTimerProxy.m

#import "SGHTimerProxy.h"
#import <objc/runtime.h> 

@implementation SGHTimerProxy

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    return [self.target methodSignatureForSelector:sel];
}

- (void)forwardInvocation:(NSInvocation *)invocation {
    [invocation invokeWithTarget:self.target];
}

@end
```

```objc
#import "SGH0425Leaks6ViewController.h"
#import "SGHTimerProxy.h"

@interface SGH0425Leaks6ViewController ()

@property (nonatomic, strong)NSTimer *timer;

@property (nonatomic, strong) SGHTimerProxy* lgProxy;

@end

@implementation SGH0425Leaks6ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    //NSProxy
    //运用到了「消息转发」机制
    //处理的是第3点，慢速转发，
    
    self.flagThree = YES;
    
    _lgProxy = [SGHTimerProxy alloc]; //NSProxy只有alloc方法
    _lgProxy.target = self;
    
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:_lgProxy selector:@selector(fire) userInfo:nil repeats:YES];
}

- (void)fire {
    NSLog(@"fire.....");
}

- (void)dealloc
{
    NSLog(@"SGH0425Leaks6ViewController Dealloc");
    [self.timer invalidate];
    self.timer = nil;
}

@end
```



内存分析如图：

![third_issue@2x.png](https://upload-images.jianshu.io/upload_images/843214-5b09408cce985953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)