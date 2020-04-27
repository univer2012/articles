来自：[iOS检测内存泄漏的方法](https://blog.csdn.net/yst19910702/article/details/81199478)



这几天闲点，不想撸代码，那就做做内存优化吧！在网上看过几篇博客下面这篇是比较好的 分析了几种内存泄漏。

常用的内存调试技巧，包括以下几种：

- 启用Zombie Object进行悬挂指针的检测。
- 应用Product -> Analysis进行内存泄露的初步检测。
- 可以在xcode的build setting中打开implicit retain of ‘self’ within blocks，xcode编译器会给出警告，逐个排查警告。
- 应用Leak Instrument进行内存泄露查找。
- 在以上方法不奏效的情况下，通过查看`dealloc`是否调用查看某个class是否泄露的问题。

 

具体操作方法：

![](http://cc.cocimg.com/api/uploads/20160217/1455702106588624.png)

在往下看之前请下载实例[MemoryProblems](https://github.com/samlaudev/MemoryProblems)，我们将以这个工程展开如何检查和解决内存问题。

悬挂指针问题

悬挂指针([Dangling Pointer](https://en.wikipedia.org/wiki/Dangling_pointer))就是当指针指向的对象已经释放或回收后，但没有对指针做任何修改(一般来说，将它指向空指针)，而是仍然指向原来已经回收的地址。如果指针指向的对象已经释放，但仍然使用，那么就会导致程序crash。

当你运行MemoryProblems后，点击悬挂指针那个选项，就会出现EXC_BAD_ACCESS崩溃信息。

![166109-14751cda6424d749.png](http://cc.cocimg.com/api/uploads/20160217/1455702165562260.png)

我们看看这个NameListViewController是做什么的？它继承UITableViewController，主要显示多个名字的信息。它的实现文件如下：

```objc
#import "NameListViewController.h"
#import "ArrayDataSource.h"

static NSString *const kNameCellIdentifier = @"NameCell";

@interface NameListViewController ()

#pragma mark - Model
@property (strong, nonatomic) NSArray *nameList;

#pragma mark - Data source
@property (assign, nonatomic) ArrayDataSource *dataSource;

@end

@implementation NameListViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.tableView.dataSource = self.dataSource;
}

#pragma mark - Lazy initialization
- (NSArray *)nameList
{
    if (!_nameList) {
        _nameList = @[@"Sam", @"Mike", @"John", @"Paul", @"Jason"];
    }
    return _nameList;
}

- (ArrayDataSource *)dataSource
{
    if (!_dataSource) {
        _dataSource = [[ArrayDataSource alloc] initWithItems:self.nameList
                                              cellIdentifier:kNameCellIdentifier
                                              tableViewStyle:UITableViewCellStyleDefault
                                          configureCellBlock:^(UITableViewCell *cell, NSString *item, NSIndexPath *indexPath) {
            cell.textLabel.text = item;
        }];
    }
    return _dataSource;
}


@end
```



要想通过tableView显示数据，首先要实现UITableViewDataSource这个协议，为了瘦身controller和复用data  source，我将它分离到一个类`ArrayDataSource`来实现`UITableViewDataSource`这个协议。然后在`viewDidLoad`方法里面将dataSource赋值给`tableView.dataSource`。

解释完`NameListViewController`的职责后，接下来我们需要思考出现EXC_BAD_ACCESS错误的原因和位置信息。

一般来说，出现EXC_BAD_ACCESS错误的原因都是悬挂指针导致的，但具体是哪个指针是悬挂指针还不确定，因为控制台并没有给出具体crash信息。

### 1.启用NSZombieEnabled

要想得到更多的crash信息，你需要启动NSZombieEnabled。具体步骤如下：

1.选中Edit Scheme，并点击

<img src="http://cc.cocimg.com/api/uploads/20160217/1455702260463590.png" style="zoom:50%;" />

2.Run -> Diagnostics -> Enable Zombie Objects

![166109-ae4f6b55212b75a9.png](http://cc.cocimg.com/api/uploads/20160217/1455702297971826.png)

设置完之后，再次运行和点击悬挂指针，虽然会再次crash，但这次控制台打印了以下有用信息：

![166109-9fe90d621bf6ce06.png](http://cc.cocimg.com/api/uploads/20160217/1455702322987711.png)

信息`message sent to deallocated instance 0x7fe19b081760`大意是向一个已释放对象发送信息，也就是已释放对象还调用某个方法。现在我们大概知道什么原因导致程序会crash，但是具体哪个对象被释放还仍然使用呢？

点击上面红色框的`Continue program execution`按钮继续运行，截图如下：

![166109-654444b25d8c5155.png](http://cc.cocimg.com/api/uploads/20160217/1455702351989090.png)

留意上面的两个红色框，它们两个地址是一样，而且`ArrayDataSource`前面有个`_NSZombie_`修饰符，说明dataSource对象被释放还仍然使用。

再进一步看dataSource声明属性的修饰符是assign

```objc
#pragma mark - Data source
@property (assign, nonatomic) ArrayDataSource *dataSource;
```

而assign对应就是`__unsafe_unretained`，它跟`__weak`相似，被它修饰的变量都不持有对象的所有权，但当变量指向的对象的RC为0时，变量并不设置为nil，而是继续保存对象的地址。

因此，在`viewDidLoad`方法中

```objc
- (void)viewDidLoad {

    [super viewDidLoad];

     

    self.tableView.dataSource = self.dataSource;    

    /*  由于dataSource是被assign修饰，self.dataSource赋值后，它对象的对象就马上释放，
     *  而self.tableView.dataSource也不是strong，而是weak，此时仍然使用，所有会导致程序crash
     */

}
```

分析完原因和定位错误代码后，至于如何修改，我想大家都心知肚明了，如果还不知道的话，留言给我。



## 二、内存泄露问题

还记得上一篇iOS/OS X内存管理(一)：基本概念与原理的引用循环例子吗？它会导致内存泄露，上次只是文字描述，不怎么直观，这次我们尝试使用Instruments里面的子工具Leaks来检查内存泄露。

### 1.静态分析

一般来说，在程序未运行之前我们可以先通过Clang Static Analyzer(静态分析)来检查代码是否存在bug。比如，内存泄露、文件资源泄露或访问空指针的数据等。下面有个静态分析的例子来讲述如何启用静态分析以及静态分析能够查找哪些bugs。

启动程序后，点击`静态分析`，马上就出现crash

![166109-036d86de2b9e9424.png](http://cc.cocimg.com/api/uploads/20160217/1455702456834408.png)

此时，即使启用`NSZombieEnabled`，控制台也不能打印出更多有关bug的信息，具体原因是什么，等下会解释。

打开`StaticAnalysisViewController`，里面引用[Facebook Infer](http://fbinfer.com/)工具的代码例子，包含个人日常开发中会出现的bugs：

```objc
///=====StaticAnalysisViewController.m

#import "StaticAnalysisViewController.h"

@implementation StaticAnalysisViewController

#pragma mark - Lifecycle
- (void)viewDidLoad
{
    [super viewDidLoad];

    [self memoryLeakBug];
    [self resoureLeakBug];
    [self parameterNotNullCheckedBlockBug:nil];
    [self npeInArrayLiteralBug];
    [self prematureNilTerminationArgumentBug];
}

#pragma mark - Test methods from facebook infer iOS Hello examples
- (void)memoryLeakBug
{
     CGPathRef shadowPath = CGPathCreateWithRect(self.inputView.bounds, NULL);
}

- (void)resoureLeakBug
{
    FILE *fp;
    fp=fopen("info.plist", "r");
}

-(void) parameterNotNullCheckedBlockBug:(void (^)())callback {
    callback();
}

-(NSArray*) npeInArrayLiteralBug {
    NSString *str = nil;
    return @[@"horse", str, @"dolphin"];
}

-(NSArray*) prematureNilTerminationArgumentBug {
    NSString *str = nil;
    return [NSArray arrayWithObjects: @"horse", str, @"dolphin", nil];
}

@end
```



下面我们通过静态分析来检查代码是否存在bugs。有两个方式：

- 手动静态分析：每次都是通过点击菜单栏的Product -> Analyze或快捷键shift + command + b

![166109-a890797a4457159d.png](http://cc.cocimg.com/api/uploads/20160217/1455702602889252.png)

- 自动静态分析：在Build Settings启用Analyze During 'Build'，每次编译时都会自动静态分析

![166109-5c1dcdd871fcb891.png](http://cc.cocimg.com/api/uploads/20160217/1455702620911747.png)

静态分析结果如下：

![166109-6c032a57f0fef09b.png](http://cc.cocimg.com/api/uploads/20160217/1455702630773154.png)



通过静态分析结果，我们来分析一下为什么`NSZombieEnabled`不能定位`EXC_BAD_ACCESS`的错误代码位置。由于callback传入进来的是null指针，而`NSZombieEnabled`只能针对某个已经释放对象的地址，所以启动`NSZombieEnabled`是不能定位的，不过可以通过静态分析可得知。



### 2.启动Instruments

有时使用静态分析能够检查出一些内存泄露问题，但是有时只有运行时使用Instruments才能检查到，启动Instruments步骤如下：

1.点击Xcode的菜单栏的 Product -> Profile 启动Instruments

![166109-95b4ea305007d321.png](http://cc.cocimg.com/api/uploads/20160217/1455702712157999.png)

2.此时，出现Instruments的工具集，选中Leaks子工具点击

![166109-379b199e81584b16.png](http://cc.cocimg.com/api/uploads/20160217/1455702722661805.png)

3.打开Leaks工具之后，点击红色圆点按钮启动Leaks工具，在Leaks工具启动同时，模拟器或真机也跟着启动

![166109-03e04393903c0c6d.png](http://cc.cocimg.com/api/uploads/20160217/1455702752470838.png)

4.启动Leaks工具后，它会在程序运行时记录内存分配信息和检查是否发生内存泄露。当你点击引用循环进去那个页面后，再返回到主页，就会发生内存泄露

![166109-1148d40299015b5f.gif](http://cc.cocimg.com/api/uploads/20160217/1455702765336340.gif)

内存泄露.gif

![QQ截图20160217175300.png](http://cc.cocimg.com/api/uploads/20160217/1455702800976860.png)

如果发生内存泄露，我们怎么定位哪里发生和为什么会发生内存泄露？

### 4.定位内存泄露

借助Leaks能很快定位内存泄露问题，在这个例子中，步骤如下：

- 首先点击Leak Checks时间条那个红色叉

![45.png](http://cc.cocimg.com/api/uploads/20160217/1455702906136270.png)

- 然后双击某行内存泄露调用栈，会直接跳到内存泄露代码位置

![46.png](http://cc.cocimg.com/api/uploads/20160217/1455702899486042.png)



### 5.分析内存泄露原因

上面已经定位好内存泄露代码的位置，至于原因是什么？可以查看上一篇的[iOS/OS X内存管理(一)：基本概念与原理](http://www.jianshu.com/p/1928b54e1253)的循环引用例子，那里已经有详细的解释。（将Test里的stong 改为 weak）



对Leaks的`Call Tree`的可选项：

![](https://upload-images.jianshu.io/upload_images/843214-975d1350fee5e668.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 三、难以检测Block引用循环

大多数的内存问题都可以通过静态分析和Instrument  Leak工具检测出来，但是有种block引用循环是难以检测的，看我们这个Block内存泄露例子，跟上面的悬挂指针例子差不多，只是在`configureCellBlock`里面调用一个方法configureCell。



```objc
///=====BlockLeakViewController.m

- (ArrayDataSource *)dataSource
{
    if (!_dataSource) {
        _dataSource = [[ArrayDataSource alloc] initWithItems:self.nameList cellIdentifier:kNameCellIdentifier tableViewStyle:UITableViewCellStyleDefault configureCellBlock:^(UITableViewCell *cell, NSString *item, NSIndexPath *indexPath) {
            cell.textLabel.text = item;
            [self configureCell];
            
        }];
    }
    return _dataSource;
}

- (void)configureCell
{
    NSLog(@"Just for test");
}

- (void)dealloc
{
    NSLog(@"release BlockLeakViewController");
}
```



我们首先用静态分析来看看能不能检查出内存泄露：

![166109-c9f8a4c970462eb6.png](http://cc.cocimg.com/api/uploads/20160217/1455703034961910.png)

结果是没有任何内存泄露的提示，我们再用Instrument Leak工具在运行时看看能不能检查出：

![166109-68e795cea155fd8e.gif](http://cc.cocimg.com/api/uploads/20160217/1455703044559251.gif)

**结果跟使用静态分析一样，还是没有任何内存泄露信息的提示。**



那么我们怎么知道这个`BlockLeakViewController`发生了内存泄露呢？还是根据iOS/OS X内存管理机制的一个基本原理：当某个对象的引用计数为0时，它就会自动调用`- (void)dealloc`方法。

在这个例子中，如果BlockLeakViewController被navigationController  pop出去后，没有调用dealloc方法，说明它的某个属性对象仍然被持有，未被释放。而我在dealloc方法打印release  BlockLeakViewController信息：

```objc
- (void)dealloc {
    NSLog(@"release BlockLeakViewController");
}
```

在我点击返回按钮后，其并没有打印出来，因此这个BlockLeakViewController存在内存泄露问题的。至于如何解决block内存泄露这个问题，很多基本功扎实的同学都知道如何解决，不懂的话，自己查资料解决吧！

```objc
( __weak typeof(self) wSelf = self; //防止循环引用 将block 里的self 换成 wSelf)
```





Retain cycle的补充说明：

### 在leak页面，选择cycle &roots 查看：

![](https://upload-images.jianshu.io/upload_images/843214-57c743ef1dd8f30d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上文没有介绍的，也是比较麻烦的，就是leak instrument没法查出的内存泄露。最近在调试一个这样的问题，写点心得。  认识工具  参看[Leaks Instrument](https://developer.apple.com/library/ios/documentation/AnalysisTools/Reference/Instruments_User_Reference/LeaksInstrument/LeaksInstrument.html)和[Allocation Instrument](https://developer.apple.com/library/ios/documentation/AnalysisTools/Reference/Instruments_User_Reference/AllocationsInstrument/AllocationsInstrument.html#//apple_ref/doc/uid/TP40011355-CH40-SW1)的官方文档。 



###  补充：

- Leak Instrument有Cycles & Roots界面，见上。

- Persistent  Bytes和#Persistent。#Persistent是object的数量，也就是allocation的次数，而Persistent  Bytes是具体的内存大小。#Persistent是我们需要关注的，内存有没有泄露也是看这个值是不是只增不减。

- `Allocation Instrument`进行profile的时候，为`Launch Configuration for Heap Allocations`勾选`Record reference counts`。



### 1.编译参数设置

 为了保证看到代码，而不是一堆无意义的内存地址，参考[The .dSYM File in Ios Project](http://hongchaozhang.github.io/blog/2015/08/17/the-dSYM-file-in-ios-project/)进行xcode的设置。
 通过`Allocation Instrument`，我们可以得到内存使用情况。为了清楚地看出是哪部分最可能是内存泄露，可以使用Call Trees视图，然后在右边：

###  

- 勾选`Hide System Libraries`，排除系统影响。

- 勾选Invert Call Tree，使占用内存最多的分支在最前面。

通过以上方法，可以大概确定是哪部分内存泄露。然后看看该class是不是被`dealloc`了。

- 如果`dealloc`了，那不是本文要解决的问题。

- 如果`dealloc`没有调用到，继续往下看。

Retain Cycle导致`dealloc`没有被调用
 在ARC下，`dealloc`不能被调用，一般是因为存在Retain Cycle，而导致Retain Cycle的情况可能是下面几种可能(参考[iOS Retain Cycle in ARC](http://kkoval.blogspot.com/2014/03/ios-retain-cycle-in-arc.html)和[Dealloc not being called on ARC app](http://stackoverflow.com/questions/9219030/dealloc-not-being-called-on-arc-app)):
 \1. Blocks
 并不是所有在block中引用`self`都会带来retain cycle，比如下面的代码就不会有内存泄露：

### 

如果dealloc没有被调用：

```objc
- (void)testSelfInCocoaBlocks {
    NSArray *cats = @[@"Smily", @"Garfild", @"Other cat"];
    [cats enumerateObjectsUsingBlock:^(NSString * _Nonnull cat, NSUInteger idx, BOOL * _Nonnull stop) {
        [self doSomethingWithCat:cat];
    }];
}
- (void)doSomethingWithCat:(NSString *)cat {}
```



因为在上面的代码中，block  ratain了self，但是self中没有retain这个block。只有当block中引用了self，并且self又以某种方式（比如用一个具有strong属性的Property指向该block，或者将该block加入了self的一个具有strong属性的array中）强引用了该block，才会引起内存泄露，比如：

```objc
@interface BlockLeakViewController ()
@property (strong, nonatomic) void (^block)();

@end
  
- (void)testSelfInBlock {
    self.block = ^{
        [self doSomethingWithCat:@"Fat Cat"];
    };
}
```



有时候即使没有直接引用self，也可能导致self被retain，这叫做“implicit retain”。一种可能的情况就是在block中引用了self的实例变量，比如：

```objc
@interface BlockLeakViewController ()
@property (strong, nonatomic) NSString *aCat;

@end

- (void)testHiddenSelfInCocoaBlocks {
    NSArray *cats = @[@"Smily", @"Garfild", @"Other cat"];
    [cats enumerateObjectsUsingBlock:^(NSString * _Nonnull cat, NSUInteger idx, BOOL * _Nonnull stop) {
        _aCat = cat;
        *stop = YES;
    }];
}
```

这段code在block中引用了self的实例变量`_aCat`。

为了避免implicit retain，可以在xcode的build setting中打开implicit retain of ‘self’ within blocks，xcode编译器会给出警告。



### 2.NSTimer



### 3.Observers/NSNotificationCenter

如果在view controller中创建了`NSTimer`，在消失view controller的时候需要调用`invalidate`，否则会产生ratain cycle。

当我们在`NSNotificationCenter`的block中引用self的时候，也会产生retain cycle，比如：

```objc
- (void)demo4 {
    [[NSNotificationCenter defaultCenter] addObserverForName:@"not" object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification * _Nonnull note) {
        [self doSomethingWithCat:@"Noty cat"];
    }];
}
```

在不用的时候需要将self从NSNotificationCenter中移除。



### 4.Array contained reference



### 5.Delegate

dealloc没被调用的调试

- 勾选Record reference counts，记录retain，release和autorelease事件。

- 为Allocation Lifespan选择All Objects Created，如果你想看到已经被dealloc的实例，这个可以不选。

- 在Allocation List -> All Allocations 中可以搜索自己感兴趣的class。

- 在reference count页面，注意使用All、uppair等进行过滤。



## 四、关于ARC下的retainCount

 

比如在view controller中将self放在了一个array中，而这个array在view controller消失的时候不会被释放，view controller的`dealloc`就不会被调用。



delegate的属性应该为weak。



调试dealloc没有被调用的情况，参考[Instruments Allocations track alloc and dealloc of objects of user defined classes](http://stackoverflow.com/questions/14890402/instruments-allocations-track-alloc-and-dealloc-of-objects-of-user-defined-class/14891837#14891837)，可以看到对应实例在整个生命周期中发生的所有和内存有关的事件，包括malloc，ratain，release等和每次事件的call stack。注意其中的两项设置：

在ARC之前，我们可以使用`retainCount`得到一个Object被retain的次数。 引入ARC之后，这个方法不能在code中使用，可以使用下面的方法获得retain的次数：

```
NSLog(@"Retain count is %ld", CFGetRetainCount((__bridge CFTypeRef)myObject));
```

或者通过设置断点，在调试窗口输入如下命令：

```
po object.retainCount
```



> 我们什么时候使用retainCount？Never！



## 五、总结

一般来说，在创建工程的时候，我都会在Build Settings启用Analyze During  'Build'，每次编译时都会自动静态分析。这样的话，写完一小段代码之后，就马上知道是否存在内存泄露或其他bug问题，并且可以修bugs。而在运行过程中，如果出现EXC_BAD_ACCESS，启用`NSZombieEnabled`，看出现异常后，控制台能否打印出更多的提示信息。如果想在运行时查看是否存在内存泄露，使用Instrument Leak工具。但是有些内存泄露是很难检查出来，有时只有通过手动覆盖`dealloc`方法，看它最终有没有调用。



补充：Instrument Leak工具

![img](https://img-blog.csdn.net/20180725114005994?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lzdDE5OTEwNzAy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180725114025325?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lzdDE5OTEwNzAy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

//选择模拟器 或者真机 -------> 选择项目 点击红色的按钮 开始检测

 

![img](https://img-blog.csdn.net/20180725133858730?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lzdDE5OTEwNzAy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![img](https://img-blog.csdn.net/20180725133930549?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lzdDE5OTEwNzAy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180725133945454?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lzdDE5OTEwNzAy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

点击选中泄露的红点 选择 call Tree 下面除了`hide System Libraies`不选  然后双击![img](https://img-blog.csdn.net/20180725134835947?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lzdDE5OTEwNzAy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

跳转到代码出现问题的位置

![img](https://img-blog.csdn.net/20180725134026773?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lzdDE5OTEwNzAy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

跳到有内存泄漏的地方，查看代码内存泄漏的原因加以修改。一般出现内存泄漏的原因：僵尸对象，循环引用，block 代理强引用，定时器 kvo  通知没有移除，webView使用不当，多次数循环导致内存暴涨，ViewController不释放 。一般会从这几个面进行排除。





###  大次数循环内存暴涨问题

记得有道比较经典的面试题，查看如下代码有何问题：

```objc
for (int i = 0; i < 100000; i++) {
        NSString *string = @"Abc";
        string = [string lowercaseString];
        string = [string stringByAppendingString:@"xyz"];
        NSLog(@"%@", string);
}
```

该循环内产生大量的临时对象，直至循环结束才释放，可能导致内存泄漏，解决方法为在循环中创建自己的autoReleasePool，及时释放占用内存大的临时变量，减少内存占用峰值。

```objc
for (int i = 0; i < 100000; i++) {
    @autoreleasepool {
        NSString *string = @"Abc";
        string = [string lowercaseString];
        string = [string stringByAppendingString:@"xyz"];
        NSLog(@"%@", string);
    }
}
```



---

【完】

