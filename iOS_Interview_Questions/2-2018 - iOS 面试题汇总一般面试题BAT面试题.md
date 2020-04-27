来自：[2018 - iOS 面试题汇总一般面试题BAT面试题](https://cloud.tencent.com/developer/article/1144991)

[【2018年最新】iOS面试题及答案](https://yq.aliyun.com/articles/637724)


# 一般面试题

### 1.堆和栈的区别
#### 1.内存管理范围
只有oc对象需要进行内存管理

非oc对象类型比如基本数据类型不需要进行内存管理


#### 2.内存管理本质
因为：Objective-C的对象在内存中是以堆的方式分配空间的,并且堆内存是由你释放的，就是release

OC对象存放于堆里面(堆内存要程序员手动回收)

非OC对象一般放在栈里面(栈内存会被系统自动回收) 

堆里面的内存是动态分配的，所以也就需要程序员手动的去添加内存、回收内存

#### 3.内存分配以及管理方式

按分配方式分:

* 堆是动态分配和回收内存的，没有静态分配的堆 
* 栈有两种分配方式：静态分配和动态分配    
    * 静态分配是系统编译器完成的，比如局部变量的分配
    * 动态分配是有alloc函数进行分配的，但是栈的动态分配和堆是不同的，它的动态分配也由系统编译器进行释放，不需要程序员手动管理

按管理方式分:
对于栈来讲，是由系统编译器自动管理，不需要程序员手动管理 

对于堆来讲，释放工作由程序员手动管理，不及时回收容易产生内存泄露
* 堆：是大家共有的空间，分全局堆和局部堆。**全局堆**就是所有没有分配的空间，**局部堆**就是用户分配的空间。堆在操作系统对进程初始化的时候分配，运行过程中也可以向系统要额外的堆，但是记得用完了要还给操作系统，要不然就是内存泄漏。<br>&emsp;&emsp;堆里面一般 放的是静态数据，比如static的数据和字符串常量等，资源加载后一般也放在堆里面。一个进程的所有线程共有这些堆 ，所以对堆的操作要考虑同步和互斥的问题。程序里面编译后的数据段都是堆的一部分。
* 栈：是个线程独有的，保存其运行状态和局部自动变量的。栈在线程开始的时候初始化，每个线程的栈互相独立，因此 ，栈是　thread safe的。每个c++对象的数据成员也存在在栈中，每个函数都有自己的栈，栈被用来在函数之间传递参数。<br>&emsp;&emsp;操作系统在切换线程的时候会自动的切换栈，就是切换ss/esp寄存器。栈空间不需要在高级语言里面显式的分配 和释放。 

一句话总结就是：
* 堆：由程序员分配和释放,如果不释放可能会引起内存泄漏
* 栈：由编译器自动分配和释放，一般存放参数值，局部变量

### 2.KVO和KVC

KVC，即是指 `NSKeyValueCoding`(键值编码),提供一种机制来间接访问对象的属性。KVC 就是基于KVO技术来实现的。 

KVO，提供了一种观察者的机制，通过对某个对象的某个属性添加观察者，当该属性改变，就会调用`observeValueForKeyPath:`方法。

### 3.OC中创建线程的方法是什么?如果在主线程中执行代码， 方法是什么?如果想延时执行代码、方法是什么?
1、线程创建有三种方法：使用`NSThread`创建、使用GCD的dispatch、使用子类化的`NSOperation`,然后将其加入`NSOperationQueue`; 

2、在主线程执行代码，方法是`performSelectorOnMainThread:withObject:waitUntilDone:`; 

3、如果想延时执行代码可以用`performSelector:onThread:withObject:waitUntilDone:`；

### 4.指针与数组名的关系?

```
int arrayName[4] = {10, 20, 30, 40}; 
int *p = (int *)(&arrayName + 1); 
NSLog(@"%d", (&arrayName  - 1));  
```

1. `(&arrayName + 1)`：`&arrayName`是数组的地址（等价于指向arrayName数组的指针）  
2. 增加 1 会往后移动16个字节，开始是4个字节的位置，移动后就是16个字节后面的位置（也就是目前位置是20个字节）  
3. 最后又赋值给，int类型的指针p（int类型占4个字节） 
4. 所以(p - 1)就是减去4个字节，变成为16个字节的位置，输出的(p - 1)值为40 <br>`int *p = (int *)(&arrayName + 1);` <br>` NSLog(@"%d", *(p - 1));` <br> `//输出结果为 40`

### 5.#import 和#include有 么区别?@class呢?#import <>和 #import"" 有 么区别?

import是OC导入头文件的关键字,#include是C/C++导入头文件的关键字 

使用#import只导入一次不会重复导入,相当于#include和#pragma once;(使用#include可能会出现循环引用,使用#pragma once消除这种可能)

@class一般用于声明某个字符串作为类名使用,它只是声明了一个类名,没有导入`.h`文件中的内容,不会引起交叉编译问题 

`import< >`代表导入系统自带的框架 

`import" "`代表导入我们自己创建的文件,导入的是我们`.h`文件,也就是头文件 

### 6.属性readwrite.readonly,assign,retain,copy,nonatomic 什么作用 ? 在哪种情况下 ?

readwrite 是可读可写特性；需要生成getter方法和setter方法时（补充：默认属性，将生成不带额外参数的getter和setter方法（setter方法只有一个参数））

readonly 是只读特性 只会生成getter方法 不会生成setter方法 ;不希望属性在类外改变

<u>**assign 是赋值特性，setter方法将传入参数赋值给实例变量；仅设置变量时；**</u>

retain 表示持有特性，setter方法将传入参数先保留，再赋值，传入参数的retaincount会+1; 

copy 表示拷贝特性，setter方法将传入对象复制一份；需要完全一份新的变量时。 

nonatomic 非原子操作，决定编译器生成的setter getter是否是原子操作，atomic表示多线程安全，一般使用nonatomic

### 7.IBOutlet 连出来的视图属性为什么可以被设置成weak?

在 storyboard 中添加一个控件引用关系是这样的(以 UIbutton 为例): UIviewController -> UIview -> UIbutton 

此时 UIviewController 强引用着 UIview , UIview 强引用着 UIbutton , IBoutlet 连线到控制器的. m 或者. h 中作为视图的属性时用 weak 修饰就可以了, (觉得用 strong 修饰也可以但是没有必要)

添加到子控件也是强引用: UIbutton 就是添加到了 UIviewController 的 view 上

### 8 预处理指令define 声明一个常数， 以表明一年中有多少秒(忽略闰年)。

```
#define SECONDS_PER_YEAR (60 * 60 * 24 * 365)UL
```
### 9.重写-个NSString类型的，retain 式声明name属性的 setter和getter 法(MRC)

属性的三大特性:语义特性,原子特性,读写特性. 

同时重写setter和getter方法,`@@synthesize name = _name;`,关联属性和实例变量 如 
```

- (void)setName:(NSString *)name {
    if (_name != name) {
        [_name retain];
        [_name release];
        _name = name;
    }
}
- (NSString *)name {
    return [[_name retain] autorelease];
}
```

### 10.分析json、xml 的区别? json、xml 解析 时的底层是如何让处理的


 (一)JSON与XML的区别：

 （1）可读性方面：基本相同，XML的可读性比较好；

 （2）可扩展性方面：都具有良好的扩展性；

 （3）编码难度方面：相对而言，JSON的编码比较容易；

 （4）解码难度：JSON的解码难度基本为零，XML需要考虑子节点和父节点；

 （5）数据体积方面：JSON相对于XML来讲，数据体积小，传递的速度比较快；

 （6）数据交互方面：JSON与javascript的交互更加方便，更容易解析处理，更好的数据交互；

 （7）数据描述方面：XML对数据描述性比较好

 （8）传输速度方面：JSON的速度远远快于XML。

 （二）JSON与XML底层实现原理：

 （1）JSON底层原理：遍历字符串中的字符，最终根据格式规定的特殊字符，比如{}、[]、：等进行区分，{}号表示字典，[]号表示数组，：号是字典的键和值的分水岭，最终仍是将JSON转化为字典，只不过字典中的值可能是“字典、数组或者字符串而已”。

 （2）XML底层原理：XML解析常用的解析方法有两种：DOM解析和SAX解析；DOM采用的是树形结构的方式访问XML文档，而SAX采用的是事件模型；DOM解析把XML文档转化为一个包含其内容的树，并可以对树进行遍历，使用DOM解析器的时候需要处理整个XML文档，所以对内存和性能的要求比较高；SAX在解析XML文档的时候可以触发一系列的事件，当发现给定的tag的时候，他可以激活一个回调方法，告诉该方法指定的标签已经找到，SAX对内存的要求通常会比较低，因为他让开发人员自己来决定所要处理的tag，特别是当开发人员只需要处理文档中所包含部分数据时，SAX这种扩展能力得到了更好的体现。

### 11.对程序性能的优化你有什么建议?

1. 使用复用机制 
2. 尽可能设置 View 为不透明 
3. 避免臃肿的 XIB 文件
4. 不要阻塞主线程 
5. 图片尺寸匹配 UIImageView 
6. 选择合适的容器 
7. 启用 GZIP 数据压缩 
8. View 的复用和懒加载机制 
9. 缓存 <br>服务器的响应信息（response）。 <br>图片。 <br>计算值。比如：UITableView 的 row heights。 
10. 关于图形绘制 
11. 处理 Memory Warnings<br> 
    * 在 AppDelegate 中实现 `- [AppDelegate applicationDidReceiveMemoryWarning:]` 代理方法。 
    * 在 UIViewController 中重载 didReceiveMemoryWarning 方法。 
    * 监听 UIApplicationDidReceiveMemoryWarningNotification 通知。 
12. 复用高开销的对象
13. 减少离屏渲染(设置圆角和阴影的时候可以选用绘制的方法) 
14. 优化 UITableView 
    * 通过正确的设置 reuseIdentifier 来重用 Cell。
    * 尽量减少不必要的透明 View。 
    * 尽量避免渐变效果、图片拉伸和离屏渲染。 
    * 当不同的行的高度不一样时，尽量缓存它们的高度值。
    * 如果 Cell展示的内容来自网络，确保用异步加载的方式来获取数据，并且缓存服务器的 response。 
    * 使用 shadowPath 来设置阴影效果。 
    * 尽量减少subview的数量，对于subview较多并且样式多变的 Cell，可以考虑用异步绘制或重写 drawRect。 
    * 尽量优化 `- [UITableView tableView:cellForRowAtIndexPath:]`方法中的处理逻辑，如果确实要做一些处理，可以考虑做一次，缓存结果。 
    * 选择合适的数据结构来承载数据，不同的数据结构对不同操作的开销是存在差异的。 
    * 对于 `rowHeight`、`sectionFooterHeight`、`sectionHeaderHeight` 尽量使用常量。
15. 选择合适的数据存储方式 
    <br>在 iOS 中可以用来进行数据持有化的方案包括：
    * NSUserDefaults。只适合用来存小数据。XML、JSON、Plist 等文件。JSON 和 XML 文件的差异在「选择正确的数据格式」已经说过了。 使用 NSCoding 来存档。
    * NSCoding同样是对文件进行读写，所以它也会面临必须加载整个文件才能继续的问题。 
    * 使用 SQLite 数据库。可以配合 FMDB 使用。数据的相对文件来说还是好处很多的，比如可以按需取数据、不用暴力查找等等。 
    * 使用 CoreData。也是数据库技术，跟 SQLite 的性能差异比较小。但是 CoreData 是一个对象图谱模型，显得更面向对象；SQLite 就是常规的 DBMS。 
16. 减少应用启动时间         
    <br>快速启动应用对于用户来说可以留下很好的印象。尤其是第一次使用时。 <br>保证应用快速启动的指导原则： 
    * 尽量将启动过程中的处理分拆成各个异步处理流，比如：网络请求、数据库访问、数据解析等等。 
    * 避免臃肿的XIB文件，因为它们会在你的主线程中进行加载。重申：Storyboard 没这个问题，放心使用。 
    
    > 注意：在测试程序启动性能的时候，最好用与 Xcode 断开连接的设备进行测试。因为 watchdog 在使用 Xcode 进行调试的时候是不会启动的。 
17. 使用 Autorelease Pool （内存释放池） 
18. imageNamed 和 imageWithContentsOfFile


### 12.runloop 和线程有什么关系?

runloop与线程是一一对应的，一个runloop对应一个核心的线程，为什么说是核心的，是因为runloop是可以嵌套的，但是核心的只能有一个，他们的关系保存在一个全局的字典里。 runloop是来管理线程的，当线程的runloop被开启后，线程会在执行完任务后进入休眠状态，有了任务就会被唤醒去执行任务。

runloop在第一次获取时被创建，在线程结束时被销毁。 对于主线程来说，runloop在程序一启动就默认创建好了。 对于子线程来说，runloop是懒加载的，只有当我们使用的时候才会创建，所以在子线程用定时器要注意：确保子线程的runloop被创建，不然定时器不会回调。

### 13 介绍下layoutSubview和drawRect

layoutSubviews调用情况 
* init初始化UIView不会触发调用 
* addSubview会触发调用 
* 改变view的width和height的时候回触发调用 
* 一个UIScrollView滚动会触发调用 
* 旋转screen会触发调用 
* 改变一个UIView大小的时候会触发superView的layoutSubviews事件 
* 直接调用setLayoutSubviews会触发调用 
* `-(void)viewWillAppear:(BOOL)animated`会触发一次调用 
* `-(void)viewDidAppear:(BOOL)animated`看情况，可能有调用

drawRect调用情况: 
* 如果UIView没有设置frame大小，直接导致drawRect不能被自动调用。 
* drawRect在loadView和viewDidLoad这两个方法之后调用 
* 调用sizeToFit后自动调用drawRect 
* 通过设置contentMode值为UIViewContentModeRedraw。那么每次设置或者更改frame自动调用drawRect。 
* 直接调用setNeedsDisplay或者setNeedsDisplayInRect会触发调用

### 14.写个“标准“宏MIN，这个宏输两个参数并返回较小的那个
```
#define MIN(A,B) ((A) <= (B) ? (A) : (B))
```

### 15.类别有什么作用
1. 扩展已有的类（添加方法） 
2. 可以通过runtime添加属性


### 16.什么是method swizzing?讲讲你的使用场景以及使用时的注意事项
1. 给扩展添加属性 
2. 替换系统方法的的实现

### 17.讲讲iOS事件响应链的原理

1、响应者链通常是由视图（UIView）构成的；

2、一个视图的下一个响应者是它视图控制器（UIViewController）（如果有的话），然后再转给它的父视图（Super View）； 

3、视图控制器（如果有的话）的下一个响应者为其管理的视图的父视图； 

4、单例的窗口（UIWindow）的内容视图将指向窗口本身作为它的下一个响应者。需要指出的是，Cocoa Touch应用不像Cocoa应用，它只有一个UIWindow对象，因此整个响应者链要简单一点； 

5、单例的应用（UIApplication）是一个响应者链的终点，它的下一个响应者指向nil，以结束整个循环。

### 18.你在什么场景下会选择使 Category

1.扩展已有的类（添加方法）

### 19.UIview和CALayer是什么关系?你对CALayer做过什么?

1. 首先UIView可以响应事件，Layer不可以.
2. UIView是CALayer的delegate  
3. UIView主要处理事件，CALayer负责绘制就更好 
4. 每个 UIView 内部都有一个 CALayer 在背后提供内容的绘制和显示，并且 UIView 的尺寸样式都由内部的 Layer 所提供。
5. 两者都有树状层级结构，layer 内部有 SubLayers，View 内部有 SubViews.但是 Layer 比 View 多了个AnchorPoint
  

创建隐式动画、绘制边框圆角

### 20.如何处理UITableVier 中Cell 动态计算高度的问题，都有哪些方案?

一、你的Cell要使用AutoLayout来布局约束这是必须的； 

设置tableview的estimatedRowHeight为一个非零值，这个属性是设置一个预估的高度值，不用太精确。 

设置tableview的rowHeight属性为`UITableViewAutomaticDimension` 

二.第三方 UITableView+FDTemplateLayoutCell


### 21.AutoLayout 中的优先级是什么? UIScrollView 中使用Autolayout 会出现什么问题?

代码计算frame -> autoreszing(父控件和子控件的关系) -> autolayout(任何控件都可以产生关系) -> sizeclass 

可以设置两个看似有冲突的约束，但设置不同的优先级之后就不会有冲突了，当其中一个约束失效之后，另一个优先级比较低的约束就会起作用

### 22.NSURLConnection 和NSURLSession 的区别是什么? NSURLProtocol是做什么的?

1.下载 

`NSURLConnection`下载文件时，先是将整个文件下载到内存，然后再写入到沙盒，如果文件比较大，就会出现内存暴涨的情况。

而使用`NSURLSessionUploadTask`下载文件，会默认下载到沙盒中的tem文件中，不会出现内存暴涨的情况，但是在下载完成后会把tem中的临时文件删除，需要在初始化任务方法时，在`completionHandler`回调中增加保存文件的代码

2.请求方法的控制

`NSURLConnection`实例化对象，实例化开始，默认请求就发送(同步发送),不需要调用start方法。而cancel可以停止请求的发送，停止后不能继续访问，需要创建新的请求。

`NSURLSession`有三个控制方法，取消(cancel)、暂停(suspend)、继续(resume)，暂停以后可以通过继续恢复当前的请求任务。

3.使用NSURLSession进行断点下载更加便捷.

`NSURLSession`的构造方法（`sessionWithConfiguration:delegate:delegateQueue`）中有一个`NSURLSessionConfiguration`类的参数可以设置配置信息，其决定了cookie，安全和高速缓存策略，最大主机连接数，资源管理，网络超时等配置。

`NSURLConnection`不能进行这个配置，相比较与`NSURLConnection`依赖与一个全局的配置对象，灵活性而言，`NSURLSession`有很大的改进


### 23.怎么高效的实现控件的圆角效果

```
绘制圆角 
-(UIImageView *)roundedRectImageViewWithCornerRadius:(CGFloat)cornerRadius { 
    UIBezierPath *bezierPath = [UIBezierPath bezierPathWithRoundedRect:self.bounds cornerRadius:cornerRadius]; 
    CAShapeLayer *layer = [CAShapeLayer layer]; 
    layer.path = bezierPath.CGPath;
    self.layer.mask = layer; 
    return self; 
}
```

### 24.说说你了解weak属性?

weak关键字在OC中属于比较基础的知识。此特性表明该属性定义了一种关系“非拥有关系”（nonowning relationship）。为这种属性设置新的值时，设置方法既不保留新值，也不释放旧值。此特性同assign类似，然后在属性所指的对象遭到摧毁时，属性值也会清空（nil out）。

弱引用，不决定对象的存亡。即使一个对象被持有无数个弱引用，只要没有强引用指向它，那么还是会被清除。

### 25.假如Controller太臃肿，如何优化?

1. 将网络请求抽象到单独的类中
* 方便在基类中处理公共逻辑；  
* 方便在基类中处理缓存逻辑，以及其它一些公共逻辑； 
* 方便做对象的持久化。 
2. 将界面的封装抽象到专门的类中 
    * 构造专门的UIView的子类，来负责这些控件的拼装。这是最彻底和优雅的方式，不过稍微麻烦一些的是，你需要把这些控件的事件回调先接管，再都一一暴露回 Controller。 
3. 构造 ViewModel 借鉴MVVM。
    * 具体做法就是将 ViewController 给 View 传递数据这个过程，抽象成构造 ViewModel 的过程。 
4. 专门构造存储类 
    * 专门来处理本地数据的存取。 
5. 整合常量

### 26.项目中网络层如何做安全处理?

1. 判断API的调用请求是否来自于经过授权的APP。如若不是则拒绝请求访问
2. 在数据请求的过程中进行URL加密处理：防止反编译，接口信息被静态分析。
3. 数据传输加密：对客户端传输数据提供有效的加密方案，以防止网络接口的拦截。
4. 如果可以尽量使用HTTPS，可以有效的避免接口数据在传输中被攻击。

### 27.main()之前的过程有哪些?

在iOS中 main.m是我们所熟悉的程序入口。但是在在此之前其实程序以及做了很多事了。如系统会获取dyld的路径，并加载。加载程序中的依赖库。调用所有的+ load方法，并返回main函数地址。