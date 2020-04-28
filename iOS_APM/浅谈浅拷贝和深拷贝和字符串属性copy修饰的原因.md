一、深拷贝和浅拷贝
 在对象进行copy操作的时候。如果只复制了对象的指针，那么拷贝就属于浅拷贝。如果对对象的内容进行了拷贝。那么拷贝过程就属于深拷贝了。下面分别对NSString、NSArray、NSDictionary什么时候进行浅拷贝什么时候进行深拷贝进行分析。
 1.NSString:

```objc
    NSString *baseStr = @"默认的字符串";
    NSString *copyStr = [baseStr copy];
    NSString *mutableCopyStr = [baseStr mutableCopy];
    NSLog(@"%p---%p---%p",baseStr,copyStr,mutableCopyStr);

```

控制台输出信息为



![img](https://upload-images.jianshu.io/upload_images/5102699-f15f6566bc4432b3.png?imageMogr2/auto-orient/strip|imageView2/2/w/382)

image.png



2.NSMutableString:

```objc
NSMutableString *mutableStr = [NSMutableString stringWithString:@"默认的字符串"];
    NSString *baseStr = [mutableStr copy];
    NSMutableString *mutableBaseStr = [mutableStr mutableCopy];
    NSLog(@"%p---%p---%p",mutableStr,baseStr,mutableBaseStr);
```

控制台输出信息为



![img](https:////upload-images.jianshu.io/upload_images/5102699-d92e8c41c0ad951f.png?imageMogr2/auto-orient/strip|imageView2/2/w/369)

image.png



NSString结论:
 通过分析可得对NSString进行copy操作进行的是浅拷贝、mutableCopy操作进行的是深拷贝。
 对于NSMutableString进行的copy和mutableCopy都是深拷贝。
 3.NSArray

```objc
    NSArray *array = @[@"baseStr",@"baseStr2",@"baseStr3"];
    NSArray *copyArray = [array copy];
    NSArray *mutableCopy = [array mutableCopy];
    NSLog(@"%p---%p---%p",array,copyArray,mutableCopy);
```

控制台输出为



![img](https://upload-images.jianshu.io/upload_images/5102699-4c82cdc4699cb2c4.png?imageMogr2/auto-orient/strip|imageView2/2/w/367)

image.png



最终确认NSArray和NSDictionary进行copy
 4.NSMutableArray

```objc
    NSMutableArray *mutableArray = [[NSMutableArray alloc]initWithArray:@[@"aaa",@"bbb",@"ccc"]];
    NSArray *copyArray = [mutableArray copy];
    NSArray *mutableCopy = [mutableArray mutableCopy];
    NSLog(@"%p---%p---%p",mutableArray,copyArray,mutableCopy);
```

控制台输出为



![img](https:////upload-images.jianshu.io/upload_images/5102699-2829dfec0e6ea932.png?imageMogr2/auto-orient/strip|imageView2/2/w/435)

image.png



结论：**经验证NSString、NSArray、NSDictionary进行copy都是浅拷贝(指针拷贝）、mutableCopy是深拷贝(内存拷贝)**

### 二：@property使用copy和strong的区别
 property使用copy和strong的区别
 我们以字符串的处理为例进行讲解
 NSString的情况下进行判断
 （1）使用copy的情况

```objc
@interface ViewController ()

@property (nonatomic, copy) NSString *defaultCopyString;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    NSMutableString *mutableString = [NSMutableString stringWithString:@"baseString"];
    self.defaultCopyString = mutableString;
    [mutableString appendString:@"123"];
    NSLog(@"%@",self.defaultCopyString);
}
```

控制台输出为：

```

```

分析可得通过copy修饰的defaultCopyString字符串在进行赋值的时候，是将可变字符串进行了copy操作。对进行了内存的深拷贝。重新占用了一块内存地址。所以当对可变字符串进行处理的时候。可以发现defaultCopyString并没有发生改变。
 （2）使用strong修饰的情况

```objc
#import "ViewController.h"

@interface ViewController ()

@property (nonatomic, strong) NSString *defaultCopyString;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    NSMutableString *mutableString = [NSMutableString stringWithString:@"baseString"];
    self.defaultCopyString = mutableString;
    [mutableString appendString:@"123"];
    NSLog(@"%@",self.defaultCopyString);
}
```

控制台输出为：

```

```



通过分析可得当修饰符为strong的时候。并没有进行拷贝操作。只是将可变字符串mutableString的引用计数进行+1。内存地址并没有发生改变。当对可变字符串进行处理的时候defaultCopyString也同步发生了变化。



 经验证NSArray和NSDictionary具有跟NSString相同的特性。如果涉及到可变数组和可变字典之间的相互转换并且涉及到数组和字典的变化时。最好使用copy来修饰。

---

【完】