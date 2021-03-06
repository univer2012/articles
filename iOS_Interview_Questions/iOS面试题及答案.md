[iOS面试题及答案](https://www.jianshu.com/p/2e1b3f54b4f3)



1、设计模式是什么？ 你知道哪些设计模式，并简要叙述？
```
设计模式是一种编码经验，就是用比较成熟的逻辑去处理某一种类型的事情。
1). MVC模式：Model View Control，把模型 视图 控制器 层进行解耦合编写。
2). MVVM模式：Model View ViewModel 把模型 视图 业务逻辑 层进行解耦和编写。
3). 单例模式：通过static关键词，声明全局变量。在整个进程运行期间只会被赋值一次。
4). 观察者模式：KVO是典型的通知模式，观察某个属性的状态，状态发生变化时通知观察者。
5). 委托模式：代理+协议的组合。实现1对1的反向传值操作。
6). 工厂模式：通过一个类方法，批量的根据已有模板生产对象。
```

2、MVC 和 MVVM 的区别
```
1). MVVM是对胖模型进行的拆分，其本质是给控制器减负，将一些弱业务逻辑放到VM中去处理。
2). MVC是一切设计的基础，所有新的设计模式都是基于MVC进行的改进。
```

3、#import跟 #include 有什么区别，@class呢，#import<> 跟 #import””有什么区别？
答：
```
1). #import是Objective-C导入头文件的关键字，#include是C/C++导入头文件的关键字，使用#import头文件会自动只导入一次，不会重复导入。
2). @class告诉编译器某个类的声明，当执行时，才去查看类的实现文件，可以解决头文件的相互包含。
3). #import<>用来包含系统的头文件，#import””用来包含用户头文件。
```

4、frame 和 bounds 有什么不同？
```
frame指的是：该view在父view坐标系统中的位置和大小。(参照点是父view的坐标系统)
bounds指的是：该view在本身坐标系统中的位置和大小。(参照点是本身坐标系统)
```

5、Objective-C的类可以多重继承么？可以实现多个接口么？Category是什么？重写一个类的方式用继承好还是分类好？为什么？
答：Objective-C的类不可以多重继承；可以实现多个接口（协议）；Category是类别；一般情况用分类好，用Category去重写类的方法，仅对本Category有效，不会影响到其他类与原有类的关系。

6、@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的
@property 的本质是什么？
    @property = ivar + getter + setter;
“属性” (property)有两大概念：ivar（实例变量）、getter+setter（存取方法）

“属性” (property)作为 Objective-C 的一项特性，主要的作用就在于封装对象中的数据。 Objective-C 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”(access method)来访问。其中，“获取方法” (getter)用于读取变量值，而“设置方法” (setter)用于写入变量值。

#### 7、@property中有哪些属性关键字？/ @property 后面可以有哪些修饰符？
属性可以拥有的特质分为四类:
1.原子性--- nonatomic 特质
2.读/写权限---readwrite(读写)、readonly (只读)
3.内存管理语义---assign、strong、 weak、unsafe_unretained、copy
4.方法名---getter=<name> 、setter=<name>
5.不常用的：nonnull,null_resettable,nullable



#### 8、属性关键字 readwrite，readonly，assign，retain，copy，nonatomic 各是什么作用，在那种情况下用？
答：
1). readwrite 是可读可写特性。需要生成getter方法和setter方法。
2). readonly 是只读特性。只会生成getter方法，不会生成setter方法，不希望属性在类外改变。
3). assign 是赋值特性。setter方法将传入参数赋值给实例变量;仅设置变量时,assign用于基本数据类型。
4). retain(MRC)/strong(ARC) 表示持有特性。setter方法将传入参数先保留，再赋值，传入参数的retaincount会+1。
5). copy 表示拷贝特性。setter方法将传入对象复制一份，需要完全一份新的变量时。
6). nonatomic 非原子操作。决定编译器生成的setter和getter方法是否是原子操作，atomic表示多线程安全，一般使用nonatomic，效率高。



#### 9、什么情况使用 weak 关键字，相比 assign 有什么不同？
1.在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性。
2.自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。

IBOutlet连出来的视图属性为什么可以被设置成weak?
    因为父控件的subViews数组已经对它有一个强引用。

不同点：
assign 可以用非 OC 对象，而 weak 必须用于 OC 对象。
weak 表明该属性定义了一种“非拥有关系”。在属性所指的对象销毁时，属性值会自动清空(nil)。

10、怎么用 copy 关键字？
 用途：
 1. NSString、NSArray、NSDictionary 等等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；
 2. block 也经常使用 copy 关键字。

 说明：
 block 使用 copy 是从 MRC 遗留下来的“传统”,在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，但写上 copy 也无伤大雅，还能时刻提醒我们：编译器自动对 block 进行了 copy 操作。如果不写 copy ，该类的调用者有可能会忘记或者根本不知道“编译器会自动对 block 进行了 copy 操作”，他们有可能会在调用之前自行拷贝属性值。这种操作多余而低效。

11、用@property声明的 NSString / NSArray / NSDictionary 经常使用 copy 关键字，为什么？如果改用strong关键字，可能造成什么问题？
答：用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作（就是把可变的赋值给不可变的），为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。

1. 因为父类指针可以指向子类对象,使用 copy 的目的是为了让本对象的属性不受外界影响,使用 copy 无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本。
2. 如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性。

//总结：使用copy的目的是，防止把可变类型的对象赋值给不可变类型的对象时，可变类型对象的值发送变化会无意间篡改不可变类型对象原来的值。

12、浅拷贝和深拷贝的区别？
答：
浅拷贝：只复制指向对象的指针，而不复制引用对象本身。
深拷贝：复制引用对象本身。内存中存在了两份独立对象本身，当修改A时，A_copy不变。

13、系统对象的 copy 与 mutableCopy 方法
不管是集合类对象（NSArray、NSDictionary、NSSet ... 之类的对象），还是非集合类对象（NSString, NSNumber ... 之类的对象），接收到copy和mutableCopy消息时，都遵循以下准则：
1. copy 返回的是不可变对象（immutableObject）；如果用copy返回值调用mutable对象的方法就会crash。
2. mutableCopy 返回的是可变对象（mutableObject）。

一、非集合类对象的copy与mutableCopy
  在非集合类对象中，对不可变对象进行copy操作，是指针复制，mutableCopy操作是内容复制；
  对可变对象进行copy和mutableCopy都是内容复制。用代码简单表示如下：
    NSString *str = @"hello word!";
    NSString *strCopy = [str copy] // 指针复制，strCopy与str的地址一样
    NSMutableString *strMCopy = [str mutableCopy] // 内容复制，strMCopy与str的地址不一样

    NSMutableString *mutableStr = [NSMutableString stringWithString: @"hello word!"];
    NSString *strCopy = [mutableStr copy] // 内容复制
    NSMutableString *strMCopy = [mutableStr mutableCopy] // 内容复制

二、集合类对象的copy与mutableCopy (同上)
  在集合类对象中，对不可变对象进行copy操作，是指针复制，mutableCopy操作是内容复制；
  对可变对象进行copy和mutableCopy都是内容复制。但是：集合对象的内容复制仅限于对象本身，对集合内的对象元素仍然是指针复制。(即单层内容复制)
    NSArray *arr = @[@[@"a", @"b"], @[@"c", @"d"];
    NSArray *copyArr = [arr copy]; // 指针复制
    NSMutableArray *mCopyArr = [arr mutableCopy]; //单层内容复制

    NSMutableArray *array = [NSMutableArray arrayWithObjects:[NSMutableString stringWithString:@"a"],@"b",@"c",nil];
    NSArray *copyArr = [mutableArr copy]; // 单层内容复制
    NSMutableArray *mCopyArr = [mutableArr mutableCopy]; // 单层内容复制

【总结一句话】：
    只有对不可变对象进行copy操作是指针复制（浅复制），其它情况都是内容复制（深复制）！

14、这个写法会出什么问题：@property (nonatomic, copy) NSMutableArray *arr;
问题：添加,删除,修改数组内的元素的时候,程序会因为找不到对应的方法而崩溃。
//如：-[__NSArrayI removeObjectAtIndex:]: unrecognized selector sent to instance 0x7fcd1bc30460
// copy后返回的是不可变对象（即 arr 是 NSArray 类型，NSArray 类型对象不能调用 NSMutableArray 类型对象的方法）
原因：是因为 copy 就是复制一个不可变 NSArray 的对象，不能对 NSArray 对象进行添加/修改。

15、如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？
若想令自己所写的对象具有拷贝功能，则需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议。
具体步骤：
    1. 需声明该类遵从 NSCopying 协议
    2. 实现 NSCopying 协议的方法。
        // 该协议只有一个方法: 
        - (id)copyWithZone:(NSZone *)zone;
        // 注意：使用 copy 修饰符，调用的是copy方法，其实真正需要实现的是 “copyWithZone” 方法。

16、写一个 setter 方法用于完成 @property (nonatomic, retain) NSString *name，写一个 setter 方法用于完成 @property (nonatomic, copy) NSString *name
答：
// retain
- (void)setName:(NSString *)str {
  [str retain];
  [_name release];
  _name = str;
}
// copy
- (void)setName:(NSString *)str {
  id t = [str copy];
  [_name release];
  _name = t;
}

17、@synthesize 和 @dynamic 分别有什么作用？
@property有两个对应的词，一个是@synthesize（合成实例变量），一个是@dynamic。
如果@synthesize和@dynamic都没有写，那么默认的就是 @synthesize var = _var;
// 在类的实现代码里通过 @synthesize 语法可以来指定实例变量的名字。(@synthesize var = _newVar;)
1. @synthesize 的语义是如果你没有手动实现setter方法和getter方法，那么编译器会自动为你加上这两个方法。
2. @dynamic 告诉编译器，属性的setter与getter方法由用户自己实现，不自动生成（如，@dynamic var）。

18、常见的 Objective-C 的数据类型有那些，和C的基本数据类型有什么区别？如：NSInteger和int
答：
Objective-C的数据类型有NSString，NSNumber，NSArray，NSMutableArray，NSData等等，这些都是class，创建后便是对象，而C语言的基本数据类型int，只是一定字节的内存空间，用于存放数值;NSInteger是基本数据类型，并不是NSNumber的子类，当然也不是NSObject的子类。NSInteger是基本数据类型Int或者Long的别名(NSInteger的定义typedef long NSInteger)，它的区别在于，NSInteger会根据系统是32位还是64位来决定是本身是int还是long。

19、id 声明的对象有什么特性？
答：id 声明的对象具有运行时的特性，即可以指向任意类型的Objcetive-C的对象。

20、Objective-C 如何对内存管理的，说说你的看法和解决方法？
答：Objective-C的内存管理主要有三种方式ARC(自动内存计数)、手动内存计数、内存池。
1). 自动内存计数ARC：由Xcode自动在App编译阶段，在代码中添加内存管理代码。
2). 手动内存计数MRC：遵循内存谁申请、谁释放；谁添加，谁释放的原则。
3). 内存释放池Release Pool：把需要释放的内存统一放在一个池子中，当池子被抽干后(drain)，池子中所有的内存空间也被自动释放掉。内存池的释放操作分为自动和手动。自动释放受runloop机制影响。

21、Objective-C 中创建线程的方法是什么？如果在主线程中执行代码，方法是什么？如果想延时执行代码、方法又是什么？
答：线程创建有三种方法：使用NSThread创建、使用GCD的dispatch、使用子类化的NSOperation,然后将其加入NSOperationQueue;在主线程执行代码，方法是performSelectorOnMainThread，如果想延时执行代码可以用performSelector:onThread:withObject:waitUntilDone:

22、Category（类别）、 Extension（扩展）和继承的区别
区别：
1. 分类有名字，类扩展没有分类名字，是一种特殊的分类。
2. 分类只能扩展方法（属性仅仅是声明，并没真正实现），类扩展可以扩展属性、成员变量和方法。
3. 继承可以增加，修改或者删除方法，并且可以增加属性。

23、我们说的OC是动态运行时语言是什么意思？
答：主要是将数据类型的确定由编译时，推迟到了运行时。简单来说, 运行时机制使我们直到运行时才去决定一个对象的类别,以及调用该类别对象指定方法。

24、为什么我们常见的delegate属性都用是week而不是retain/strong？
答：是为了防止delegate两端产生不必要的循环引用。
@property (nonatomic, weak) id<UITableViewDelegate> delegate;

25、什么时候用delete，什么时候用Notification？
Delegate(委托模式)：1对1的反向消息通知功能。
Notification(通知模式)：只想要把消息发送出去，告知某些状态的变化。但是并不关心谁想要知道这个。

26、什么是 KVO 和 KVC？
1). KVC(Key-Value-Coding)：键值编码 是一种通过字符串间接访问对象的方式（即给属性赋值）
    举例说明：
    stu.name = @"张三" // 点语法给属性赋值
    [stu setValue:@"张三" forKey:@"name"]; // 通过字符串使用KVC方式给属性赋值
    stu1.nameLabel.text = @"张三";
    [stu1 setValue:@"张三" forKey:@"nameLabel.text"]; // 跨层赋值
2). KVO(key-Value-Observing)：键值观察机制 他提供了观察某一属性变化的方法，极大的简化了代码。
     KVO只能被KVC触发，包括使用setValue:forKey:方法和点语法。
   // 通过下方方法为属性添加KVO观察
   - (void)addObserver:(NSObject *)observer
                     forKeyPath:(NSString *)keyPath
                     options:(NSKeyValueObservingOptions)options
                     context:(nullable void *)context;
      // 当被观察的属性发送变化时，会自动触发下方方法                   
   - (void)observeValueForKeyPath:(NSString *)keyPath
                              ofObject:(id)object
                                  change:(NSDictionary *)change
                                 context:(void *)context{}

KVC 和 KVO 的 keyPath 可以是属性、实例变量、成员变量。

27、KVC的底层实现？
当一个对象调用setValue方法时，方法内部会做以下操作：
1). 检查是否存在相应的key的set方法，如果存在，就调用set方法。
2). 如果set方法不存在，就会查找与key相同名称并且带下划线的成员变量，如果有，则直接给成员变量属性赋值。
3). 如果没有找到_key，就会查找相同名称的属性key，如果有就直接赋值。
4). 如果还没有找到，则调用valueForUndefinedKey:和setValue:forUndefinedKey:方法。
这些方法的默认实现都是抛出异常，我们可以根据需要重写它们。

28、KVO的底层实现？
KVO基于runtime机制实现。

29、ViewController生命周期
按照执行顺序排列：
1. initWithCoder：通过nib文件初始化时触发。
2. awakeFromNib：nib文件被加载的时候，会发生一个awakeFromNib的消息到nib文件中的每个对象。      
3. loadView：开始加载视图控制器自带的view。
4. viewDidLoad：视图控制器的view被加载完成。  
5. viewWillAppear：视图控制器的view将要显示在window上。
6. updateViewConstraints：视图控制器的view开始更新AutoLayout约束。
7. viewWillLayoutSubviews：视图控制器的view将要更新内容视图的位置。
8. viewDidLayoutSubviews：视图控制器的view已经更新视图的位置。
9. viewDidAppear：视图控制器的view已经展示到window上。 
10. viewWillDisappear：视图控制器的view将要从window上消失。
11. viewDidDisappear：视图控制器的view已经从window上消失。

30、方法和选择器有何不同？
selector是一个方法的名字，方法是一个组合体，包含了名字和实现。

31、你是否接触过OC中的反射机制？简单聊一下概念和使用
1). class反射
    通过类名的字符串形式实例化对象。
        Class class = NSClassFromString(@"student"); 
        Student *stu = [[class alloc] init];
    将类名变为字符串。
        Class class =[Student class];
        NSString *className = NSStringFromClass(class);
2). SEL的反射
    通过方法的字符串形式实例化方法。
        SEL selector = NSSelectorFromString(@"setName");  
        [stu performSelector:selector withObject:@"Mike"];
    将方法变成字符串。
        NSStringFromSelector(@selector*(setName:));

32、调用方法有两种方式：
1). 直接通过方法名来调用。[person show];
2). 间接的通过SEL数据来调用 SEL aaa = @selector(show); [person performSelector:aaa];  

33、如何对iOS设备进行性能测试？
答： Profile-> Instruments ->Time Profiler

34、开发项目时你是怎么检查内存泄露？
1). 静态分析 analyze。
2). instruments工具里面有个leak可以动态分析。

35、什么是懒加载？
答：懒加载就是只在用到的时候才去初始化。也可以理解成延时加载。
我觉得最好也最简单的一个例子就是tableView中图片的加载显示了, 一个延时加载, 避免内存过高,一个异步加载,避免线程堵塞提高用户体验。

36、类变量的 @public，@protected，@private，@package 声明各有什么含义？
@public 任何地方都能访问;
@protected 该类和子类中访问,是默认的;
@private 只能在本类中访问;
@package 本包内使用,跨包不可以。

37、什么是谓词？
谓词就是通过NSPredicate给定的逻辑条件作为约束条件,完成对数据的筛选。
//定义谓词对象,谓词对象中包含了过滤条件(过滤条件比较多)
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"age<%d",30];
//使用谓词条件过滤数组中的元素,过滤之后返回查询的结果
NSArray *array = [persons filteredArrayUsingPredicate:predicate];

38、isa指针问题
isa：是一个Class 类型的指针. 每个实例对象有个isa的指针,他指向对象的类,而Class里也有个isa的指针, 指向meteClass(元类)。元类保存了类方法的列表。当类方法被调 用时,先会从本身查找类方法的实现,如果没有,元类会向他父类查找该方法。同时注意的是:元类(meteClass)也是类,它也是对象。元类也有isa指针,它的isa指针最终指向的是一个根元类(root meteClass)。根元类的isa指针指向本身,这样形成了一个封闭的内循环。

39、如何访问并修改一个类的私有属性？
1). 一种是通过KVC获取。
2). 通过runtime访问并修改私有属性。

40、一个objc对象的isa的指针指向什么？有什么作用？
答：指向他的类对象,从而可以找到对象上的方法。

41、下面的代码输出什么？
@implementation Son : Father
- (id)init {
   if (self = [super init]) {
       NSLog(@"%@", NSStringFromClass([self class])); // Son
       NSLog(@"%@", NSStringFromClass([super class])); // Son
   }
   return self;
}
@end
// 解析：
self 是类的隐藏参数，指向当前调用方法的这个类的实例。
super是一个Magic Keyword，它本质是一个编译器标示符，和self是指向的同一个消息接收者。
不同的是：super会告诉编译器，调用class这个方法时，要去父类的方法，而不是本类里的。
上面的例子不管调用[self class]还是[super class]，接受消息的对象都是当前 Son *obj 这个对象。

42、写一个完整的代理，包括声明、实现
// 创建
@protocol MyDelagate
@required
-(void)eat:(NSString *)foodName; 
@optional
-(void)run;
@end

//  声明 .h
@interface person: NSObject<MyDelagate>

@end

//  实现 .m
@implementation person
- (void)eat:(NSString *)foodName { 
   NSLog(@"吃:%@!", foodName);
} 
- (void)run {
   NSLog(@"run!");
}

@end

43、isKindOfClass、isMemberOfClass、selector作用分别是什么
isKindOfClass：作用是某个对象属于某个类型或者继承自某类型。
isMemberOfClass：某个对象确切属于某个类型。
selector：通过方法名，获取在内存中的函数的入口地址。

44、delegate 和 notification 的区别
1). 二者都用于传递消息，不同之处主要在于一个是一对一的，另一个是一对多的。
2). notification通过维护一个array，实现一对多消息的转发。
3). delegate需要两者之间必须建立联系，不然没法调用代理的方法；notification不需要两者之间有联系。

45、什么是block？
闭包（block）：闭包就是获取其它函数局部变量的匿名函数。

46、block反向传值
在控制器间传值可以使用代理或者block，使用block相对来说简洁。

在前一个控制器的touchesBegan:方法内实现如下代码。

  // OneViewController.m
  TwoViewController *twoVC = [[TwoViewController alloc] init];
  twoVC.valueBlcok = ^(NSString *str) {
    NSLog(@"OneViewController拿到值：%@", str); 
  };
  [self presentViewController:twoVC animated:YES completion:nil];

  // TwoViewController.h   （在.h文件中声明一个block属性）
  @property (nonatomic ,strong) void(^valueBlcok)(NSString *str);

  // TwoViewController.m   （在.m文件中实现方法）
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 传值:调用block
    if (_valueBlcok) {
        _valueBlcok(@"123456");
    }
}

47、block的注意点
1). 在block内部使用外部指针且会造成循环引用情况下，需要用__week修饰外部指针：
    __weak typeof(self) weakSelf = self; 
2). 在block内部如果调用了延时函数还使用弱指针会取不到该指针，因为已经被销毁了，需要在block内部再将弱指针重新强引用一下。
    __strong typeof(self) strongSelf = weakSelf;
3). 如果需要在block内部改变外部栈区变量的话，需要在用__block修饰外部变量。

48、BAD_ACCESS在什么情况下出现？
答：这种问题在开发时经常遇到。原因是访问了野指针，比如访问已经释放对象的成员变量或者发消息、死循环等。

49、lldb（gdb）常用的控制台调试命令？
1). p 输出基本类型。是打印命令，需要指定类型。是print的简写
    p (int)[[[self view] subviews] count]
2). po 打印对象，会调用对象description方法。是print-object的简写
    po [self view]
3). expr 可以在调试时动态执行指定表达式，并将结果打印出来。常用于在调试过程中修改变量的值。
4). bt：打印调用堆栈，是thread backtrace的简写，加all可打印所有thread的堆栈
5). br l：是breakpoint list的简写

50、你一般是怎么用Instruments的？
Instruments里面工具很多，常用：
1). Time Profiler: 性能分析
2). Zombies：检查是否访问了僵尸对象，但是这个工具只能从上往下检查，不智能。
3). Allocations：用来检查内存，写算法的那批人也用这个来检查。
4). Leaks：检查内存，看是否有内存泄露。

51、iOS中常用的数据存储方式有哪些？
数据存储有四种方案：NSUserDefault、KeyChain、file、DB。
    其中File有三种方式：plist、Archive（归档）
    DB包括：SQLite、FMDB、CoreData

52、iOS的沙盒目录结构是怎样的？
沙盒结构：
1). Application：存放程序源文件，上架前经过数字签名，上架后不可修改。
2). Documents：常用目录，iCloud备份目录，存放数据。（这里不能存缓存文件，否则上架不被通过）
3). Library：
        Caches：存放体积大又不需要备份的数据。(常用的缓存路径)
        Preference：设置目录，iCloud会备份设置信息。
4). tmp：存放临时文件，不会被备份，而且这个文件下的数据有可能随时被清除的可能。

53、iOS多线程技术有哪几种方式？
答：pthread、NSThread、GCD、NSOperation

54、GCD 与 NSOperation 的区别：
GCD 和 NSOperation 都是用于实现多线程：
    GCD 基于C语言的底层API，GCD主要与block结合使用，代码简洁高效。
    NSOperation 属于Objective-C类，是基于GCD更高一层的封装。复杂任务一般用NSOperation实现。

55、写出使用GCD方式从子线程回到主线程的方法代码
答：dispatch_sync(dispatch_get_main_queue(), ^{ });

56、如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）
// 使用Dispatch Group追加block到Global Group Queue,这些block如果全部执行完毕，就会执行Main Dispatch Queue中的结束处理的block。
// 创建队列组
dispatch_group_t group = dispatch_group_create();
// 获取全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); 
// 当并发队列组中的任务执行完毕后才会执行这里的代码
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 合并图片
});

57、dispatch_barrier_async（栅栏函数）的作用是什么？
函数定义：dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
作用：
    1.在它前面的任务执行结束后它才执行，它后面的任务要等它执行完成后才会开始执行。
    2.避免数据竞争

// 1.创建并发队列
dispatch_queue_t queue = dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
// 2.向队列中添加任务
dispatch_async(queue, ^{  // 1.2是并行的
    NSLog(@"任务1, %@",[NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"任务2, %@",[NSThread currentThread]);
});

dispatch_barrier_async(queue, ^{
    NSLog(@"任务 barrier, %@", [NSThread currentThread]);
});

dispatch_async(queue, ^{   // 这两个是同时执行的
    NSLog(@"任务3, %@",[NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"任务4, %@",[NSThread currentThread]);
});

// 输出结果: 任务1 任务2 ——》 任务 barrier ——》任务3 任务4 
// 其中的任务1与任务2，任务3与任务4 由于是并行处理先后顺序不定。

58、以下代码运行结果如何？
- (void)viewDidLoad {
    [super viewDidLoad];
    NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}
// 只输出：1。（主线程死锁）

59、什么是 RunLoop
从字面上讲就是运行循环，它内部就是do-while循环，在这个循环内部不断地处理各种任务。
一个线程对应一个RunLoop，基本作用就是保持程序的持续运行，处理app中的各种事件。通过runloop，有事运行，没事就休息，可以节省cpu资源，提高程序性能。

主线程的run loop默认是启动的。iOS的应用程序里面，程序启动后会有一个如下的main()函数
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

60、什么是 Runtime
Runtime又叫运行时，是一套底层的C语言API，其为iOS内部的核心之一，我们平时编写的OC代码，底层都是基于它来实现的。

61、Runtime实现的机制是什么，怎么用，一般用于干嘛？
1). 使用时需要导入的头文件 <objc/message.h> <objc/runtime.h>
2). Runtime 运行时机制，它是一套C语言库。
3). 实际上我们编写的所有OC代码，最终都是转成了runtime库的东西。
    比如：
        类转成了 Runtime 库里面的结构体等数据类型，
        方法转成了 Runtime 库里面的C语言函数，
        平时调方法都是转成了 objc_msgSend 函数（所以说OC有个消息发送机制）
    // OC是动态语言，每个方法在运行时会被动态转为消息发送，即：objc_msgSend(receiver, selector)。
    // [stu show];  在objc动态编译时，会被转意为：objc_msgSend(stu, @selector(show));    
4). 因此，可以说 Runtime 是OC的底层实现，是OC的幕后执行者。

有了Runtime库，能做什么事情呢？
Runtime库里面包含了跟类、成员变量、方法相关的API。
比如：
（1）获取类里面的所有成员变量。
（2）为类动态添加成员变量。
（3）动态改变类的方法实现。
（4）为类动态添加新的方法等。
因此，有了Runtime，想怎么改就怎么改。
62、什么是 Method Swizzle（黑魔法），什么情况下会使用？
1). 在没有一个类的实现源码的情况下，想改变其中一个方法的实现，除了继承它重写、和借助类别重名方法暴力抢先之外，还有更加灵活的方法 Method Swizzle。
2). Method Swizzle 指的是改变一个已存在的选择器对应的实现的过程。OC中方法的调用能够在运行时通过改变，通过改变类的调度表中选择器到最终函数间的映射关系。
3). 在OC中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字。利用OC的动态特性，可以实现在运行时偷换selector对应的方法实现。
4). 每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系。IMP有点类似函数指针，指向具体的方法实现。
5). 我们可以利用 method_exchangeImplementations 来交换2个方法中的IMP。
6). 我们可以利用 class_replaceMethod 来修改类。
7). 我们可以利用 method_setImplementation 来直接设置某个方法的IMP。
8). 归根结底，都是偷换了selector的IMP。

63、_objc_msgForward 函数是做什么的，直接调用它将会发生什么？
答：_objc_msgForward是 IMP 类型，用于消息转发的：当向一个对象发送一条消息，但它并没有实现的时候，_objc_msgForward会尝试做消息转发。

64、什么是 TCP / UDP ?
TCP：传输控制协议。
UDP：用户数据协议。

TCP 是面向连接的，建立连接需要经历三次握手，是可靠的传输层协议。
UDP 是面向无连接的，数据传输是不可靠的，它只管发，不管收不收得到。
简单的说，TCP注重数据安全，而UDP数据传输快点，但安全性一般。

65、通信底层原理（OSI七层模型）
OSI采用了分层的结构化技术，共分七层：
    物理层、数据链路层、网络层、传输层、会话层、表示层、应用层。

66、介绍一下XMPP？
XMPP是一种以XML为基础的开放式实时通信协议。
简单的说，XMPP就是一种协议，一种规定。就是说，在网络上传东西，XMM就是规定你上传大小的格式。

67、OC中创建线程的方法是什么？如果在主线程中执行代码，方法是什么？
// 创建线程的方法
- [NSThread detachNewThreadSelector:nil toTarget:nil withObject:nil]
- [self performSelectorInBackground:nil withObject:nil];
- [[NSThread alloc] initWithTarget:nil selector:nil object:nil];
- dispatch_async(dispatch_get_global_queue(0, 0), ^{});
- [[NSOperationQueue new] addOperation:nil];

// 主线程中执行代码的方法
- [self performSelectorOnMainThread:nil withObject:nil waitUntilDone:YES];
- dispatch_async(dispatch_get_main_queue(), ^{});
- [[NSOperationQueue mainQueue] addOperation:nil];

68、tableView的重用机制？
答：UITableView 通过重用单元格来达到节省内存的目的: 通过为每个单元格指定一个重用标识符，即指定了单元格的种类,当屏幕上的单元格滑出屏幕时，系统会把这个单元格添加到重用队列中，等待被重用，当有新单元格从屏幕外滑入屏幕内时，从重用队列中找看有没有可以重用的单元格，如果有，就拿过来用，如果没有就创建一个来使用。

69、用伪代码写一个线程安全的单例模式
static id _instance;
+ (id)allocWithZone:(struct _NSZone *)zone {
   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
       _instance = [super allocWithZone:zone];
   });
   return _instance;
}

+ (instancetype)sharedData {
   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
       _instance = [[self alloc] init];
   });
   return _instance;
}

- (id)copyWithZone:(NSZone *)zone {
   return _instance;
}

70、如何实现视图的变形?
答：通过修改view的 transform 属性即可。

71、在手势对象基础类UIGestureRecognizer的常用子类手势类型中哪两个手势发生后，响应只会执行一次？
答：UITapGestureRecognizer,UISwipeGestureRecognizer是一次性手势,手势发生后,响应只会执行一次。

72、字符串常用方法：
NSString *str = @"abc*123";
NSArray *arr = [str componentsSeparatedByString:@"*"]; //以目标字符串把原字符串分割成两部分，存到数组中。@[@"abc", @"123"];

73、如何高性能的给 UIImageView 加个圆角?
不好的解决方案：使用下面的方式会强制Core Animation提前渲染屏幕的离屏绘制, 而离屏绘制就会给性能带来负面影响，会有卡顿的现象出现。
self.view.layer.cornerRadius = 5.0f;
self.view.layer.masksToBounds = YES;

正确的解决方案：使用绘图技术
- (UIImage *)circleImage {
    // NO代表透明
    UIGraphicsBeginImageContextWithOptions(self.size, NO, 0.0);
    // 获得上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    // 添加一个圆
    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);
    CGContextAddEllipseInRect(ctx, rect);
    // 裁剪
    CGContextClip(ctx);
    // 将图片画上去
    [self drawInRect:rect];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    // 关闭上下文
    UIGraphicsEndImageContext();
    return image;
}

还有一种方案：使用了贝塞尔曲线"切割"个这个图片, 给UIImageView 添加了的圆角，其实也是通过绘图技术来实现的。
UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
imageView.center = CGPointMake(200, 300);
UIImage *anotherImage = [UIImage imageNamed:@"image"];
UIGraphicsBeginImageContextWithOptions(imageView.bounds.size, NO, 1.0);
[[UIBezierPath bezierPathWithRoundedRect:imageView.bounds
                       cornerRadius:50] addClip];
[anotherImage drawInRect:imageView.bounds];
imageView.image = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
[self.view addSubview:imageView];

74、你是怎么封装一个view的
1）. 可以通过纯代码或者xib的方式来封装子控件
2）. 建立一个跟view相关的模型，然后将模型数据传给view，通过模型上的数据给view的子控件赋值

/**
 *  纯代码初始化控件时一定会走这个方法
 */
- (instancetype)initWithFrame:(CGRect)frame {
    if(self = [super initWithFrame:frame]) {
        [self setupUI];
    }
    return self;
}

/**
 *  通过xib初始化控件时一定会走这个方法
 */
- (id)initWithCoder:(NSCoder *)aDecoder {
    if(self = [super initWithCoder:aDecoder]) {
        [self setupUI];
    }
    return self;
}

- (void)setupUI {
    // 初始化代码
}

75、HTTP协议中 POST 方法和 GET 方法有那些区别?
1. GET用于向服务器请求数据，POST用于提交数据
2. GET请求，请求参数拼接形式暴露在地址栏，而POST请求参数则放在请求体里面，因此GET请求不适合用于验证密码等操作
3. GET请求的URL有长度限制，POST请求不会有长度限制

76、请简单的介绍下APNS发送系统消息的机制
APNS优势：杜绝了类似安卓那种为了接受通知不停在后台唤醒程序保持长连接的行为，由iOS系统和APNS进行长连接替代。
APNS的原理：
    1). 应用在通知中心注册，由iOS系统向APNS请求返回设备令牌(device Token)
    2). 应用程序接收到设备令牌并发送给自己的后台服务器
    3). 服务器把要推送的内容和设备发送给APNS
    4). APNS根据设备令牌找到设备，再由iOS根据APPID把推送内容展示


第三方框架
1、AFNetworking 底层原理分析
AFNetworking主要是对NSURLSession和NSURLConnection(iOS9.0废弃)的封装,其中主要有以下类:
1). AFHTTPRequestOperationManager：内部封装的是 NSURLConnection, 负责发送网络请求, 使用最多的一个类。(3.0废弃)
2). AFHTTPSessionManager：内部封装是 NSURLSession, 负责发送网络请求,使用最多的一个类。
3). AFNetworkReachabilityManager：实时监测网络状态的工具类。当前的网络环境发生改变之后,这个工具类就可以检测到。
4). AFSecurityPolicy：网络安全的工具类, 主要是针对 HTTPS 服务。

5). AFURLRequestSerialization：序列化工具类,基类。上传的数据转换成JSON格式
    (AFJSONRequestSerializer).使用不多。
6). AFURLResponseSerialization：反序列化工具类;基类.使用比较多:
7). AFJSONResponseSerializer; JSON解析器,默认的解析器.
8). AFHTTPResponseSerializer; 万能解析器; JSON和XML之外的数据类型,直接返回二进
制数据.对服务器返回的数据不做任何处理.
9). AFXMLParserResponseSerializer; XML解析器;

2、描述下SDWebImage里面给UIImageView加载图片的逻辑
SDWebImage 中为 UIImageView 提供了一个分类UIImageView+WebCache.h, 这个分类中有一个最常用的接口sd_setImageWithURL:placeholderImage:，会在真实图片出现前会先显示占位图片，当真实图片被加载出来后再替换占位图片。
    
加载图片的过程大致如下：
    1.首先会在 SDWebImageCache 中寻找图片是否有对应的缓存, 它会以url 作为数据的索引先在内存中寻找是否有对应的缓存
    2.如果缓存未找到就会利用通过MD5处理过的key来继续在磁盘中查询对应的数据, 如果找到了, 就会把磁盘中的数据加载到内存中，并将图片显示出来
    3.如果在内存和磁盘缓存中都没有找到，就会向远程服务器发送请求，开始下载图片
    4.下载后的图片会加入缓存中，并写入磁盘中
    5.整个获取图片的过程都是在子线程中执行，获取到图片后回到主线程将图片显示出来
    
SDWebImage原理：
调用类别的方法：
    1. 从内存（字典）中找图片（当这个图片在本次使用程序的过程中已经被加载过），找到直接使用。
    2. 从沙盒中找（当这个图片在之前使用程序的过程中被加载过），找到使用，缓存到内存中。
    3. 从网络上获取，使用，缓存到内存，缓存到沙盒。

3、友盟统计接口统计的所有功能
APP启动速度，APP停留页面时间等

算法
1、不用中间变量,用两种方法交换A和B的值
// 1.中间变量
void swap(int a, int b) {
   int temp = a;
   a = b;
   b = temp;
}

// 2.加法
void swap(int a, int b) {
   a = a + b;
   b = a - b;
   a = a - b;
}

// 3.异或（相同为0，不同为1. 可以理解为不进位加法）
void swap(int a, int b) {
   a = a ^ b;
   b = a ^ b;
   a = a ^ b;
}
```



2、求最大公约数
/** 1.直接遍历法 */
int maxCommonDivisor(int a, int b) {
    int max = 0;
    for (int i = 1; i <=b; i++) {
        if (a % i == 0 && b % i == 0) {
            max = i;
        }
    }
    return max;
}
/** 2.辗转相除法 */
int maxCommonDivisor(int a, int b) {
    int r;
    while(a % b > 0) {
        r = a % b;
        a = b;
        b = r;
    }
    return b;
}


// 扩展：最小公倍数 = (a * b)/最大公约数

3、模拟栈操作
 /**
 *  栈是一种数据结构，特点：先进后出
 *  练习：使用全局变量模拟栈的操作
 */
#include <stdio.h>
#include <stdbool.h>
#include <assert.h>
//保护全局变量：在全局变量前加static后，这个全局变量就只能在本文件中使用
static int data[1024];//栈最多能保存1024个数据
static int count = 0;//目前已经放了多少个数(相当于栈顶位置)

//数据入栈 push
void push(int x){
    assert(!full());//防止数组越界
    data[count++] = x;
}
//数据出栈 pop
int pop(){
    assert(!empty());
    return data[--count];
}
//查看栈顶元素 top
int top(){
    assert(!empty());
    return data[count-1];
}

//查询栈满 full
bool full() {
    if(count >= 1024) {
        return 1;
    }
     return 0; 
}

//查询栈空 empty
bool empty() {
    if(count <= 0) {
        return 1;
    }
    return 0;
}

int main(){
    //入栈
    for (int i = 1; i <= 10; i++) {
        push(i);
    }
  
    //出栈
    while(!empty()){
        printf("%d ", top()); //栈顶元素
        pop(); //出栈
    }
    printf("\n");
    
    return 0;
}

4、排序算法
选择排序、冒泡排序、插入排序三种排序算法可以总结为如下：
都将数组分为已排序部分和未排序部分。
1. 选择排序将已排序部分定义在左端，然后选择未排序部分的最小元素和未排序部分的第一个元素交换。
2. 冒泡排序将已排序部分定义在右端，在遍历未排序部分的过程执行交换，将最大元素交换到最右端。
3. 插入排序将已排序部分定义在左端，将未排序部分元的第一个元素插入到已排序部分合适的位置。

选择排序
/** 
 *  【选择排序】：最值出现在起始端
 *  
 *  第1趟：在n个数中找到最小(大)数与第一个数交换位置
 *  第2趟：在剩下n-1个数中找到最小(大)数与第二个数交换位置
 *  重复这样的操作...依次与第三个、第四个...数交换位置
 *  第n-1趟，最终可实现数据的升序（降序）排列。
 *
 */
void selectSort(int *arr, int length) {
    for (int i = 0; i < length - 1; i++) { //趟数
        for (int j = i + 1; j < length; j++) { //比较次数
            if (arr[i] > arr[j]) {
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
}

冒泡排序
/** 
 *  【冒泡排序】：相邻元素两两比较，比较完一趟，最值出现在末尾
 *  第1趟：依次比较相邻的两个数，不断交换（小数放前，大数放后）逐个推进，最值最后出现在第n个元素位置
 *  第2趟：依次比较相邻的两个数，不断交换（小数放前，大数放后）逐个推进，最值最后出现在第n-1个元素位置
 *   ……   ……
 *  第n-1趟：依次比较相邻的两个数，不断交换（小数放前，大数放后）逐个推进，最值最后出现在第2个元素位置 
 */
void bublleSort(int *arr, int length) {
    for(int i = 0; i < length - 1; i++) { //趟数
        for(int j = 0; j < length - i - 1; j++) { //比较次数
            if(arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        } 
    }
}

5、折半查找（二分查找）
/**
 *  折半查找：优化查找时间（不用遍历全部数据）
 *
 *  折半查找的原理：
 *   1> 数组必须是有序的
 *   2> 必须已知min和max（知道范围）
 *   3> 动态计算mid的值，取出mid对应的值进行比较
 *   4> 如果mid对应的值大于要查找的值，那么max要变小为mid-1
 *   5> 如果mid对应的值小于要查找的值，那么min要变大为mid+1
 *
 */ 

// 已知一个有序数组, 和一个key, 要求从数组中找到key对应的索引位置 
int findKey(int *arr, int length, int key) {
    int min = 0, max = length - 1, mid;
    while (min <= max) {
        mid = (min + max) / 2; //计算中间值
        if (key > arr[mid]) {
            min = mid + 1;
        } else if (key < arr[mid]) {
            max = mid - 1;
        } else {
            return mid;
        }
    }
    return -1;
}

```

编码格式（优化细节）
1、在 Objective-C 中，enum 建议使用 NS_ENUM 和 NS_OPTIONS 宏来定义枚举类型。
//定义一个枚举(比较严密)
typedef NS_ENUM(NSInteger, BRUserGender) {
    BRUserGenderUnknown,    // 未知
    BRUserGenderMale,       // 男性
    BRUserGenderFemale,     // 女性
    BRUserGenderNeuter      // 无性
};

@interface BRUser : NSObject<NSCopying>

@property (nonatomic, readonly, copy) NSString *name;
@property (nonatomic, readonly, assign) NSUInteger age;
@property (nonatomic, readonly, assign) BRUserGender gender;

- (instancetype)initWithName:(NSString *)name age:(NSUInteger)age gender:(BRUserGender)gender;

@end

//说明：
//既然该类中已经有一个“初始化方法” ，用于设置 name、age 和 gender 的初始值: 那么在设计对应 @property 时就应该尽量使用不可变的对象：其三个属性都应该设为“只读”。用初始化方法设置好属性值之后，就不能再改变了。
//属性的参数应该按照下面的顺序排列： （原子性，读写，内存管理）


2、避免使用C语言中的基本数据类型，建议使用 Foundation 数据类型，对应关系如下：
int -> NSInteger
unsigned -> NSUInteger
float -> CGFloat
动画时间 -> NSTimeInterval

```

#其它知识点
1、HomeKit，是苹果2014年发布的智能家居平台。

2、什么是 OpenGL、Quartz 2D？

Quatarz 2d 是Apple提供的基本图形工具库。只是适用于2D图形的绘制。
OpenGL，是一个跨平台的图形开发库。适用于2D和3D图形的绘制。

3、ffmpeg框架：ffmpeg 是音视频处理工具，既有音视频编码解码功能，又可以作为播放器使用。


4、谈谈 UITableView 的优化
1). 正确的复用cell。
2). 设计统一规格的Cell
3). 提前计算并缓存好高度（布局），因为heightForRowAtIndexPath:是调用最频繁的方法；
4). 异步绘制，遇到复杂界面，遇到性能瓶颈时，可能就是突破口；
4). 滑动时按需加载，这个在大量图片展示，网络加载的时候很管用！
5). 减少子视图的层级关系
6). 尽量使所有的视图不透明化以及做切圆操作。
7). 不要动态的add 或者 remove 子控件。最好在初始化时就添加完，然后通过hidden来控制是否显示。
8). 使用调试工具分析问题。

5、如何实行cell的动态的行高
如果希望每条数据显示自身的行高，必须设置两个属性，1.预估行高，2.自定义行高。
设置预估行高 tableView.estimatedRowHeight = 200。
设置定义行高 tableView.estimatedRowHeight = UITableViewAutomaticDimension。 
如果要让自定义行高有效，必须让容器视图有一个自下而上的约束。

6 如何让计时器调用一个类方法
计时器只能调用实例方法，但是可以在这个实例方法里面调用静态方法。
使用计时器需要注意，计时器一定要加入RunLoop中，并且选好model才能运行。scheduledTimerWithTimeInterval方法创建一个计时器并加入到RunLoop中所以可以直接使用。
如果计时器的repeats选择YES说明这个计时器会重复执行，一定要在合适的时机调用计时器的invalid。不能在dealloc中调用，因为一旦设置为repeats 为yes，计时器会强持有self，导致dealloc永远不会被调用，这个类就永远无法被释放。比如可以在viewDidDisappear中调用，这样当类需要被回收的时候就可以正常进入dealloc中了。

 [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(timerMethod) userInfo:nil repeats:YES];

-(void)timerMethod
{
//调用类方法
    [[self class] staticMethod];
}

-(void)invalid
{
    [timer invalid];
    timer = nil;
}

7 如何重写类方法
1、在子类中实现一个同基类名字一样的静态方法
2、在调用的时候不要使用类名调用，而是使用[self class]的方式调用。原理，用类名调用是早绑定，在编译期绑定，用[self class]是晚绑定，在运行时决定调用哪个方法。


8 NSTimer创建后，会在哪个线程运行。
用scheduledTimerWithTimeInterval创建的，在哪个线程创建就会被加入哪个线程的RunLoop中就运行在哪个线程
自己创建的Timer，加入到哪个线程的RunLoop中就运行在哪个线程。


9 id和NSObject＊的区别
id是一个 objc_object 结构体指针，定义是
typedef struct objc_object *id
id可以理解为指向对象的指针。所有oc的对象 id都可以指向，编译器不会做类型检查，id调用任何存在的方法都不会在编译阶段报错，当然如果这个id指向的对象没有这个方法，该崩溃还是会崩溃的。

NSObject *指向的必须是NSObject的子类，调用的也只能是NSObjec里面的方法否则就要做强制类型转换。

不是所有的OC对象都是NSObject的子类，还有一些继承自NSProxy。NSObject *可指向的类型是id的子集。

10.ios开发逆向传值的几种方法整理
第一种：代理传值
第二个控制器：
@protocol WJSecondViewControllerDelegate <NSObject>
- (void)changeText:(NSString*)text;
@end
 @property(nonatomic,assign)id<WJSecondViewControllerDelegate>delegate;

- (IBAction)buttonClick:(UIButton*)sender {
_str = sender.titleLabel.text;
[self.delegate changeText:sender.titleLabel.text];
[self.navigationController popViewControllerAnimated:YES];
}


第一个控制器:
- (IBAction)pushToSecond:(id)sender {
WJSecondViewController *svc = [[WJSecondViewController alloc]initWithNibName:@"WJSecondViewController" bundle:nil];
svc.delegate = self;
svc.str = self.navigationItem.title;
[self.navigationController pushViewController:svc animated:YES];
[svc release];
}
- (void)changeText:(NSString *)text{
self.navigationItem.title = text;
}

第二种：通知传值
第一个控制器：
 //注册监听通知
 [[NSNotificationCenter defaultCenter] addObserver:self         selector:@selector(limitDataForModel:) name:@"NOV" object:nil];
- (void)limitDataForModel:(NSNotification *)noti{
self.gamesInfoArray = noti.object;
}

第二个控制器：
//发送通知
  [[NSNotificationCenter defaultCenter]     postNotificationName:@"NOV" object:gameArray];

第三种：单例传值
Single是一个单例类，并且有一个字符串类型的属性titleName
在第二个控制器：
- (IBAction)buttonClick:(UIButton*)sender {
Single *single = [Single sharedSingle];
single.titleName = sender.titleLabel.text;
[self.navigationController popViewControllerAnimated:YES];
}

第一个控制器：
- (void)viewWillAppear:(BOOL)animated{
[super viewWillAppear:animated];
Single *single = [Single sharedSingle];
self.navigationItem.title = single.titleName;
}

第四种：block传值
第二个控制器：
@property (nonatomic,copy) void (^changeText_block)(NSString*);
- (IBAction)buttonClick:(UIButton*)sender {
_str = sender.titleLabel.text;
self.changeText_block(sender.titleLabel.text);
[self.navigationController popViewControllerAnimated:YES];
}

第一个控制器：
- (IBAction)pushToSecond:(id)sender {
WJSecondViewController *svc = [[WJSecondViewController alloc]initWithNibName:@"WJSecondViewController" bundle:nil];
svc.str = self.navigationItem.title;
[svc setChangeText_block:^(NSString *str) {
    >self.navigationItem.title = str;
}]；
[self.navigationController pushViewController:svc animated:YES];
}

第五种：extern传值
第二个控制器：
 extern NSString *btn;
- (IBAction)buttonClick:(UIButton*)sender {
btn = sender.titleLabel.text;
[self.navigationController popViewControllerAnimated:YES];
}

第一个控制器:
NSString *btn = nil;
- (void)viewWillAppear:(BOOL)animated{
[super viewWillAppear:animated];
self.navigationItem.title = btn;
}

第六种：KVO传值
第一个控制器:
- (void)viewDidLoad {
[super viewDidLoad];
 _vc =[[SecondViewController alloc]init];
//self监听vc里的textValue属性
[_vc addObserver:self forKeyPath:@"textValue" options:0 context:nil];   
}

第二个控制器:
- (IBAction)buttonClicked:(id)sender {
self.textValue = self.textField.text;
[self.navigationController popViewControllerAnimated:YES];
}

11.浅谈iOS开发中方法延迟执行的几种方式
Method1. performSelector方法

Method2. NSTimer定时器

Method3. NSThread线程的sleep

Method4. GCD


12.NSPersistentStoreCoordinator  ,   NSManaged0bjectContext 和NSManaged0bject中的那些需要在线程中创建或者传递
 答：NSPersistentStoreCoordinator是持久化存储协调者，主要用于协调
托管对象上下文和持久化存储区之间的关系。NSManagedObjectContext使用协调者的托管对象模型将数据保存到数
据库，或查询数据。

13.您是否做过一部的网络处理和通讯方面的工作？如果有，能具体介绍一下实现策略么
答：使用NSOperation发送异步网络请求，使用NSOperationQueue管理
线程数目及优先级，底层是用NSURLConnetion，

14.你使用过Objective-C的运行时编程（Runtime Programming）么？如果使用过，你用它做了什么？你还能记得你所使用的相关的头文件或者某些方法的名称吗？
 答：Objecitve-C的重要特性是Runtime（运行时）,在#import <objc/runtime.h> 下能看到相关的方法，用过objc_getClass()和class_copyMethodList()获取过私有API;使用  
objective-c
Method method1 = class_getInstanceMethod(cls, sel1);
Method method2 = class_getInstanceMethod(cls, sel2);
method_exchangeImplementations(method1, method2);  

代码交换两个方法，在写unit test时使用到。
15.Core开头的系列的内容。是否使用过CoreAnimation和CoreGraphics。UI框架和CA，CG框架的联系是什么？分别用CA和CG做过些什么动画或者图像上的内容。（有需要的话还可以涉及Quartz的一些内容）
答：UI框架的底层有CoreAnimation，CoreAnimation的底层有CoreGraphics。    
UIKit | 
------------ | 
Core Animation | 
Core Graphics |
Graphics Hardware|  
使用CA做过menu菜单的展开收起（太逊了）  

16.是否使用过CoreText或者CoreImage等？如果使用过，请谈谈你使用CoreText或者CoreImage的体验。
答：CoreText可以解决复杂文字内容排版问题。CoreImage可以处理图
片，为其添加各种效果。体验是很强大，挺复杂的。

17.NSNotification和KVO的区别和用法是什么？什么时候应该使用通知，什么时候应该使用KVO，它们的实现上有什么区别吗？如果用protocol和delegate（或者delegate的Array）来实现类似的功能可能吗？如果可能，会有什么潜在的问题？如果不能，为什么？（虽然protocol和delegate这种东西面试已经面烂了…）
答：NSNotification是通知模式在iOS的实现，KVO的全称是键值观察
(Key-value observing),其是基于KVC（key-value coding）的，KVC是一
个通过属性名访问属性变量的机制。例如将Module层的变化，通知到多
个Controller对象时，可以使用NSNotification；如果是只需要观察某个
对象的某个属性，可以使用KVO。
对于委托模式，在设计模式中是对象适配器模式，其是delegate是指向
某个对象的，这是一对一的关系，而在通知模式中，往往是一对多的关
系。委托模式，从技术上可以现在改变delegate指向的对象，但不建议
这样做，会让人迷惑，如果一个delegate对象不断改变，指向不同的对
象。

18.你用过NSOperationQueue么？如果用过或者了解的话，你为什么要使用NSOperationQueue，实现了什么？请描述它和G.C.D的区别和类似的地方（提示：可以从两者的实现机制和适用范围来描述）。
答：使用NSOperationQueue用来管理子类化的NSOperation对象，控制
其线程并发数目。GCD和NSOperation都可以实现对线程的管理，区别
是 NSOperation和NSOperationQueue是多线程的面向对象抽象。项目中
使用NSOperation的优点是NSOperation是对线程的高度抽象，在项目中
使用它，会使项目的程序结构更好，子类化NSOperation的设计思路，
是具有面向对象的优点（复用、封装），使得实现是多线程支持，而接
口简单，建议在复杂项目中使用。
项目中使用GCD的优点是GCD本身非常简单、易用，对于不复杂的多线
程操作，会节省代码量，而Block参数的使用，会是代码更为易读，建议
在简单项目中使用。

19.既然提到G.C.D，那么问一下在使用G.C.D以及block时要注意些什么？它们两是一回事儿么？block在ARC中和传统的MRC中的行为和用法有没有什么区别，需要注意些什么？
答：使用block是要注意，若将block做函数参数时，需要把它放到最
后，GCD是Grand Central Dispatch，是一个对线程开源类库，而Block
是闭包，是能够读取其他函数内部变量的函数。


对于Objective-C，你认为它最大的优点和最大的不足是什么？对于不足之处，现在有没有可用的方法绕过这些不足来实现需求。如果可以的话，你有没有考虑或者实践过重新实现OC的一些功能，如果有，具体会如何做？

答：最大的优点是它的运行时特性，不足是没有命名空间，对于命名冲
 突，可以使用长命名法或特殊前缀解决，如果是引入的第三方库之间的
命名冲突，可以使用link命令及flag解决冲突。


你实现过一个框架或者库以供别人使用么？如果有，请谈一谈构建框架或者库时候的经验；如果没有，请设想和设计框架的public的API，并指出大概需要如何做、需要注意一些什么方面，来使别人容易地使用你的框架。

答：抽象和封装，方便使用。首先是对问题有充分的了解，比如构建一
个文件解压压缩框架，从使用者的角度出发，只需关注发送给框架一个
解压请求，框架完成复杂文件的解压操作，并且在适当的时候通知给是
哦难过者，如解压完成、解压出错等。在框架内部去构建对象的关系，
通过抽象让其更为健壮、便于更改。其次是API的说明文档。
```