n来自：[iOS-Block本质](https://www.jianshu.com/p/4e79e9a0dd82)



---



## 第一部分：Block本质

#### Q：什么是Block，Block的本质是什么？

- block本质上也是一个OC对象，它<u>内部也有个isa指针</u>
- block是**封装了<u>函数调用</u>以及<u>函数调用环境</u>的OC对象**
- block是<u>封装函数及其上下文的OC对象</u>

![img](https:////upload-images.jianshu.io/upload_images/1915113-71d81fe282e2a41d.png?imageMogr2/auto-orient/strip|imageView2/2/w/541)

block底层结构图

查看block源码：

> reserved [rɪˈzɜːvd] adj. 保留的，预订的；缄默的，冷淡的，
>
> descriptor [dɪˈskrɪptə(r)]  n.描述符号
>
> variables ['vɛrɪəbl] n. 【数】变量

```objc
struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  // 构造函数（类似于OC的init方法），返回结构体对象
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

// 封装了block执行逻辑的函数
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {

            NSLog((NSString *)&__NSConstantStringImpl__var_folders_2r__m13fp2x2n9dvlr8d68yry500000gn_T_main_c60393_mi_0);
        }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool;
        // 定义block变量
        void (*block)(void) = &__main_block_impl_0(
                                                   __main_block_func_0,
                                                   &__main_block_desc_0_DATA
                                                   );

        // 执行block内部的代码
        block->FuncPtr(block);
    }
    return 0;
}
```



![](https://upload-images.jianshu.io/upload_images/1915113-10f5856516bb4699.png?imageMogr2/auto-orient/strip|imageView2/2/w/937)

- FuncPtr：指向调用函数的地址
- __main_block_desc_0 ：block描述信息
- Block_size：block的大小



## 第二部分：Block捕获变量

#### Q：下述代码输出值为多少？

```objc
int age=10;
void (^Block)(void) = ^{
    NSLog(@"age:%d",age);
};
age = 20;
Block();
```

输出值为 age：10
 原因：创建block的时候，已经把age的值存储在里面了。

#### Q：下列代码输出值分别为多少？

```objc
auto int age = 10;
static int num = 25;
void (^Block)(void) = ^{
    NSLog(@"age:%d,num:%d",age,num);
};
age = 20;
num = 11;
Block();
```

输出结果为：age:10,num:11
 愿意：**auto变量<u>block访问方式是</u>值传递，static变量<u>block访问方式是</u>指针传递**
 源码证明：

```objc
int age = __cself->age; // bound by copy
int *num = __cself->num; // bound by copy

NSLog((NSString *)&__NSConstantStringImpl__var_folders_2r__m13fp2x2n9dvlr8d68yry500000gn_T_main_d2875b_mi_0, age, (*num));

int age = 10;
static int num = 25;

block = ((void (*)())&__test_block_impl_0((void *)__test_block_func_0, &__test_block_desc_0_DATA, age, &num)); //一个直接用`age`，一个是 用`&num`，&是取地址符

age = 20;
num = 11;
```

上述代码可查看 static修饰的变量，是根据指针访问的

#### Q：为什么block对auto和static变量捕获有差异？

**auto变量可能会被销毁，内存可能会被回收，所以<u>不采用指针访问</u>**；<u>static变量一直保存在内存中，指针访问即可</u>

#### Q：block对全局变量的捕获方式是？

block不需要对全局变量捕获，都是直接采用取值的

#### Q：为什么局部变量需要捕获？

考虑作用域的问题，需要跨函数访问，就需要捕获

#### Q：block的变量捕获（capture）

为了保证block内部能够正常访问外部的变量，block有个变量捕获机制



![img](https:////upload-images.jianshu.io/upload_images/1915113-b684175082c9a822.png?imageMogr2/auto-orient/strip|imageView2/2/w/792)

block的变量捕获

#### Q：block里访问self是否会捕获？

会，self是当调用block函数的参数，参数是局部变量，self指向调用者

#### Q：block里访问成员变量是否会捕获？

会，**成员变量的访问其实是`self->xx`，先捕获self，再通过self访问里面的成员变量**

## 第三部分 Block类型

#### Q：block有哪几种类型？

block的类型，取决于isa指针，可以通过调用class方法或者isa指针查看具体类型，最终都是继承自NSBlock类型

- __NSGlobalBlock __ （ _NSConcreteGlobalBlock ）
- __NSStackBlock __ （ _NSConcreteStackBlock ）
- __NSMallocBlock __ （ _NSConcreteMallocBlock ）

代码示例

```objc
void (^block1)(void) = ^{
    NSLog(@"block1");
};
NSLog(@"%@",[block1 class]);
NSLog(@"%@",[[block1 class] superclass]);
NSLog(@"%@",[[[block1 class] superclass] superclass]);
NSLog(@"%@",[[[[block1 class] superclass] superclass] superclass]);
NSLog(@"%@",[[[[[block1 class] superclass] superclass] superclass] superclass]);
```

输出结果：
```
08:35:59.670147+0800 ObjectiveCDemo20200321[10472:516399] __NSGlobalBlock__
08:35:59.670546+0800 ObjectiveCDemo20200321[10472:516399] __NSGlobalBlock
08:35:59.670748+0800 ObjectiveCDemo20200321[10472:516399] NSBlock
08:35:59.670899+0800 ObjectiveCDemo20200321[10472:516399] NSObject
08:35:59.671065+0800 ObjectiveCDemo20200321[10472:516399] (null)
```

上述代码输出了**block1的类型**，也**证实了block是对象，最终继承NSObject**

代码展示block的三种类型：

```objc
int age = 1;
void (^block1)(void) = ^{
    NSLog(@"block1");
};

void (^block2)(void) = ^{
    NSLog(@"block2:%d",age);
};

NSLog(@"%@/%@/%@",[block1 class],[block2 class],[^{
    NSLog(@"block3:%d",age);
} class]);
```

输出结果：

```
 __NSGlobalBlock __/__NSMallocBlock __/__NSStackBlock __
```

#### Q：各类型的block在内存中如何分配的？

- __NSGlobalBlock __ 在数据区
- __NSMallocBlock __ 在堆区
- __NSStackBlock __ 在栈区
- 堆：动态分配内存，需要程序员自己申请，程序员自己管理
- 栈：自动分配内存，自动销毁，先入后出，栈上的内容存在自动销毁的情况

![img](https:////upload-images.jianshu.io/upload_images/1915113-b06d523ee624ab7a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1001)

block内存分配

#### Q：如何判断block是哪种类型？

- 没有访问auto变量的block是`__NSGlobalBlock __ `，放在数据段
- MRC模式下，访问了auto变量的block是`__NSStackBlock __` ；ARC模式下，访问了auto变量的block是`__NSMallocBlock __` ；
- `[__NSStackBlock __ copy]`操作就变成了`__NSMallocBlock __` 。

#### Q：对每种类型block调用copy操作后是什么结果？

- ~~`__NSGlobalBlock __` 调用copy操作后，什么也不做~~

- ~~`__NSStackBlock __` 调用copy操作后，复制效果是：从栈复制到堆；副本存储位置是**堆**~~

- ~~`__NSMallocBlock __` 调用copy操作后，复制效果是：引用计数增加；副本存储位置是**堆**~~

  更新：

  1. `__NSGlobalBlock__` 调用copy操作后，复制效果是：copy后的对象，指针指向原GlobalBlock，引用计数不变。
  2. `__NSStackBlock__`  调用copy操作后，复制效果是：从栈复制到堆；副本存储位置是「堆」。
  3. `__NSMallocBlock__` 调用copy操作后，复制效果是：copy后的对象，指针指向原MallocBlock，引用计数不变。效果跟NSString的copy操作效果一样。

#### Q：在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上的几种情况？「备注：没法验证」

- 1.block作为函数返回值时
- 2.将block赋值给__strong指针时
- 3.block作为Cocoa API中方法名含有usingBlock的方法参数时
- 4.block作为GCD API的方法参数时

**MRC下block属性的建议写法**
 `@property (copy, nonatomic) void (^block)(void);`

**ARC下block属性的建议写法**
 `@property (copy, nonatomic) void (^block)(void);`

## 第四部分：对象类型的auto变量

#### Q：~~ARC下述代码中Person对象是否会释放？~~「备注：代码有问题，无法得出正确答案」

示例代码：

```objc
@implementation SGHARCBlockViewController


typedef void(^XBTBlock)(void);
XBTBlock btBlock;

- (void)viewDidLoad {
    [super viewDidLoad];
  
    [self demo5_0];
}
- (void)demo5_0 {
    
    SGHBlockPerson *p = [[SGHBlockPerson alloc] init];
    p.age = 10;
    
    btBlock = ^{
        NSLog(@"======= %ld",(long)p.age);
    };
    
    NSLog(@"%@, p_retainCount: %ld",btBlock, (long)CFGetRetainCount((__bridge  CFTypeRef)p));
}

///Person.m
- (void)dealloc{
    NSLog(@"Person - dealloc");
}
```

输出结果：不会打印Person - dealloc
 转化C++代码后：

```objc
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  Person *person;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, MJPerson *__strong _person, int flags=0) : person(_person) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

上述block为堆block，block里面有一个Person指针，Person指针指向Person对象。只要block还在，Person就还在。block强引用了Person对象。

#### Q：~~上述代码改换成MRC，Person对象会释放么？~~「备注：代码有问题，无法得出正确答案」

示例代码：

```objc
@implementation SGHMRCBlockViewController

typedef void(^XBTBlock)(void);
XBTBlock btBlock2;

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self demo5_0];
}

- (void)demo5_0 {
    
    SGHBlockPerson *p = [[SGHBlockPerson alloc] init];
    p.age = 10;
    
    btBlock2 = ^{
        NSLog(@"======= %ld",(long)p.age);
    };
    
    NSLog(@"%@",btBlock2);
    
    [p release]; //手动释放p
}
```



答：会的！NSStackBlock 不会持有p对象，p对象需要手动 `release` 。因为p执行了 `release` 操作，所以会打印出 Person - dealloc 。





## 扩展疑问：下面代码在MRC和ARC的打印，为什么？

MRC:

```objc
@implementation SGHMRCBlockViewController


typedef void(^XBTBlock)(void);


- (void)viewDidLoad {
    [super viewDidLoad];
  	[self demo5];
}
- (void)demo5 {
    XBTBlock btBlock2;
    SGHBlockPerson *p = [[SGHBlockPerson alloc] init];
    p.age = 10;
    
    btBlock2 = ^{
        NSLog(@"======= %ld",(long)p.age);
    };
    btBlock2();
    NSLog(@"%@, p_retainCount: %ld",btBlock2,[p retainCount]);
}


//SGHBlockPerson.m
@implementation SGHBlockPerson

- (void)dealloc{
    NSLog(@"Person - dealloc");
}

@end
```

打印：

```
[12255:610753] ======= 10
[12255:610753] <__NSStackBlock__: 0x7ffee8c3f7b8>, p_retainCount: 1
```

解答：因为StackBlock不会持有对象p，所以p对象的引用计数一直是1。又因为在MRC下，p对象需要手动释放，所以不会调用 `dealloc` 打印 Person - dealloc 。



ARC:

```objc
@implementation SGHARCBlockViewController


typedef void(^XBTBlock)(void);


- (void)viewDidLoad {
    [super viewDidLoad];
  	[self demo5];
}
- (void)demo5 {
    XBTBlock btBlock;
    SGHBlockPerson *p = [[SGHBlockPerson alloc] init];
    p.age = 10;
    
    btBlock = ^{
        NSLog(@"======= %ld",(long)p.age);
    };
    btBlock();
    NSLog(@"%@, p_retainCount: %ld",btBlock, (long)CFGetRetainCount((__bridge  CFTypeRef)p));
}
```

打印：

```
[12255:610753] ======= 10
[12255:610753] <__NSMallocBlock__: 0x6000034b2070>, p_retainCount: 3
[12255:610753] Person - dealloc
```

解答：因为MallocBlock会持有对象p，且 MallocBlock对象 和 p对象 都是是局部变量。当这个方法执行完成后，MallocBlock对象会被释放掉，此时对象p也会跟着被释放掉。



#### Q：下列代码中Person是否会被释放？

```objc
@autoreleasepool {
        XBTBlock block;
        
        {
            Person *p = [[Person alloc] init];
            p.age = 10;
            __weak Person *weakPersn = p;
            block = ^{
                NSLog(@"======= %d",weakPersn.age);
            };
        }
        NSLog(@"--------------");
}
```

答案：会释放

### 小结

无论MRC还是ARC，<font color=#FF0000>**栈空间上的block，不会持有对象；堆空间的block，会持有对象。**</font>

#### 验证：

ARC代码：

```objc
- (void)demo5_2 {
    typedef void(^XBTBlock)(void);
    XBTBlock btBlock;
    
    SGHBlockPerson *p = [[SGHBlockPerson alloc] init];
    p.age = 10;
    
    NSLog(@"retainCount:%ld", (long)CFGetRetainCount((__bridge  CFTypeRef)p));  //retainCount:1
    btBlock = ^{
        NSLog(@"======= %ld",(long)p.age);
    };
    
    NSLog(@"%@",btBlock);
    
    //测试：MallocBlock 是否会持有p
    NSLog(@"retainCount:%ld", (long)CFGetRetainCount((__bridge  CFTypeRef)p));
    btBlock = nil;
    NSLog(@"p:%@, retainCount:%ld", p, (long)CFGetRetainCount((__bridge  CFTypeRef)p));
}
```

执行打印如下：

```
retainCount:1
<__NSMallocBlock__: 0x6000025435a0>
retainCount:3
p:<SGHBlockPerson: 0x6000029d1b10>, retainCount:2
Person - dealloc
```

结论：**说明 MallocBlock 会持有对象p。**



MRC代码：

```objc
- (void)demo5_2 {
    typedef void(^XBTBlock)(void);
    XBTBlock btBlock2;
    
    
    SGHBlockPerson *p = [[SGHBlockPerson alloc] init];
    p.age = 10;
    
    btBlock2 = ^{
        NSLog(@"======= %ld",(long)p.age);
    };
    NSLog(@"%@",btBlock2);
    
    
    //测试：StackBlock 是否会持有p
    NSLog(@"retainCount:%ld", [p retainCount]);
    btBlock2 = nil;
    NSLog(@"p:%@, retainCount:%ld", p, [p retainCount]);
}
```

执行打印如下：

```
<__NSStackBlock__: 0x7ffeeb8dc7b8>
retainCount:1
p:<SGHBlockPerson: 0x6000029d1c60>, retainCount:1
```

结论：**说明 StackBlock 不会持有对象p 。**



#### Q：当block内部访问了对象类型的auto变量时，是否会强引用？

答案：分情况讨论，分为栈block和堆block

##### 一、栈block
 如果block是在栈上，将不会对auto变量产生强引用。栈上的block随时会被销毁，也没必要去强引用其他对象。

##### 二、堆block
1. 如果block被拷贝到堆上：
    a) 会调用block内部的copy函数，
    b) copy函数内部会调用`_Block_object_assign`函数，
    c) `_Block_object_assign` 函数会根据auto变量的修饰符（`__strong、__weak、__unsafe_unretained`）做出相应的操作，形成强引用（retain）或者弱引用。

2. 如果block从堆上移除：
    a) 会调用block内部的dispose函数
    b) dispose函数内部会调用`_Block_object_dispose` 函数
    c) ` _Block_object_dispose` 函数会自动释放引用的auto变量（release）

正确答案：

- 如果block在`栈`空间，不管外部变量是强引用还是弱引用，block都会弱引用访问对象
- 如果block在`堆`空间，如果外部强引用，block内部也是强引用；如果外部弱引用，block内部也是弱引用



---



#### Q：__weak 在使用clang转换OC为C++代码时，可能会遇到以下问题`cannot create __weak reference in file using manual reference`

解决方案：支持ARC、指定运行时系统版本，比如
 `xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 main.m`

#### Q1：gcd的block中引用 Person对象什么时候销毁？

```objc
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    SGHBlockPerson *person = [[SGHBlockPerson alloc] init];
    person.age = 10;
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"age:%ld",(long)person.age);
    });
    
    NSLog(@"touchesBegan");
}
```

输出结果：

```
14:36:03.395120+0800 test[1032:330314] touchesBegan
14:36:05.395237+0800 test[1032:330314] age:10
14:36:05.395487+0800 test[1032:330314] Person-dealloc
```

原因：gcd的block默认会做copy操作，即dispatch_after的block是堆block，block会对Person强引用，block销毁时候Person才会被释放。

#### Q2：上述代码如果换成__weak，Person什么时候释放？

```objc
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    SGHBlockPerson *person = [[SGHBlockPerson alloc] init];
    person.age = 10;
    
    __weak SGHBlockPerson *weakPerson = person;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"age:%ld",(long)weakPerson.age);
    });
    
    NSLog(@"touchesBegan");
}
```

输出结果：

```
14:38:42.996990+0800 test[1104:347260] touchesBegan
14:38:42.997481+0800 test[1104:347260] Person-dealloc
14:38:44.997136+0800 test[1104:347260] age:0
```

原因：使用`__weak`修饰过后的对象，堆block会采用弱引用，无法延时Person的寿命，所以在touchesBegan函数结束后，Person就会被释放，gcd就无法捕捉到Person。

#### Q3：如果gcd内包含gcd，Person会什么时候释放？

```objc
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    SGHBlockPerson *person = [[SGHBlockPerson alloc] init];
    person.age = 10;
    
    __weak SGHBlockPerson *weakPerson = person;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4.0 * NSEC_PER_SEC)),
                   dispatch_get_main_queue(), ^{
                       
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"2-----age:%p",person);
        });
        NSLog(@"1-----age:%p",weakPerson);
    });

    NSLog(@"touchesBegan");
}
```

输出结果：

```
14:48:01.293818+0800 test[1199:403589] touchesBegan
14:48:05.294127+0800 test[1199:403589] 1-----age:0x604000015eb0
14:48:08.582807+0800 test[1199:403589] 2-----age:0x604000015eb0
14:48:08.583129+0800 test[1199:403589] Person-dealloc
```

原因：**gcd内部只要有强引用Person，Person就会等待执行完再销毁！**所以Person销毁时间为7秒。

#### Q4：如果gcd内部先强引用后弱引用，Person什么时候释放？

```objc
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    SGHBlockPerson *person = [[SGHBlockPerson alloc] init];
    person.age = 10;
    
    __weak SGHBlockPerson *weakPerson = person;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4.0 * NSEC_PER_SEC)),
                   dispatch_get_main_queue(), ^{
                       
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"2-----age:%p",weakPerson);
        });
        NSLog(@"1-----age:%p",person);
    });

    NSLog(@"touchesBegan");
}
```

输出结果

```
14:52:29.036878+0800 test[1249:431302] touchesBegan
14:52:33.417862+0800 test[1249:431302] 1-----age:0x6000000178d0
14:52:33.418178+0800 test[1249:431302] Person-dealloc
14:52:36.418204+0800 test[1249:431302] 2-----age:0x0
```

原因：Person会等待强引用执行完毕后释放，只要强引用执行完，就不会等待后执行的弱引用，会直接释放。所以Person释放时间为4秒。

## 第五部分：__block修饰符

#### Q：block在修改NSMutableArray，需不需要添加__block？

不需要。

#### Q：block能否修改变量值？

auto修饰变量，block无法修改，因为block使用的时候是内部创建了变量来保存外部的变量的值，block只有修改内部自己变量的权限，无法修改外部变量的权限。
 static修饰变量，block可以修改，因为block把外部static修饰变量的指针存入，block直接修改指针指向变量值，即可修改外部变量值。
 全局变量值，全局变量无论哪里都可以修改，当然block内部也可以修改。

#### Q：`__block` int age = 10，系统做了哪些？

答案：**编译器会将`__block`变量包装成一个对象。**
 查看c++源码：

```objc
struct __Block_byref_age_0 {
  void *__isa;
__Block_byref_age_0 *__forwarding;//age的地址
 int __flags;
 int __size;
 int age;//age 的值
};
```

#### Q：__block 修饰符作用？

- __block可以**用于解决block内部，无法修改auto变量值的问题。**
- __block不能修饰全局变量、静态变量（static）
- **编译器会将`__block`变量包装成一个对象。**
- `__block`修改变量：age -> `__forwarding`-> age
- `__Block_byref_age_0`结构体内部地址和外部变量age是同一地址。

![img](https:////upload-images.jianshu.io/upload_images/1915113-bb96bba3f22f9d57.png?imageMogr2/auto-orient/strip|imageView2/2/w/376)

__forwarding指针指向

#### Q：block可以向NSMutableArray添加元素么？

```objc
NSMutableArray *arr = [NSMutableArray array];

Block block = ^{
    [arr addObject:@"123"];
    [arr addObject:@"2345"];
};
```

答案：可以，因为是addObject是使用NSMutableArray变量，而不是通过指针改变NSMutableArray，如果是`arr = nil`，这就是改变了NSMutableArray变量，会报错。

#### 5.1  `__block`的内存管理

**当block在栈上时，并不会对`__block`变量产生强引用。**

#### Q：block的属性修饰词为什么是copy？

block一旦没有进行copy操作，就不会在堆上
 block在堆上，程序员就可以对block做内存管理等操作，可以控制block的生命周期

#### Q：当block被copy到堆时，对__block修饰的变量做了什么？

- 会调用block内部的copy函数
- copy函数内部会调用_Block_object_assign函数
- _Block_object_assign函数会对__block变量形成强引用（retain）
- 对于__block 修饰的变量 assign函数对其强引用；对于外部对象 assign函数根据外部如何引用而引用

![img](https:////upload-images.jianshu.io/upload_images/1915113-e2db39ad66205f5e.png?imageMogr2/auto-orient/strip|imageView2/2/w/414)

block0复制到堆上

![img](https:////upload-images.jianshu.io/upload_images/1915113-4a837bfbd7cbadf8.png?imageMogr2/auto-orient/strip|imageView2/2/w/430)

block1复制到堆上

#### Q：当block从堆中移除时，对__block修饰的变量做了什么？

- 会调用block内部的dispose函数
- dispose函数内部会调用_Block_object_dispose函数
- _Block_object_dispose函数会自动释放引用的__block变量（release）

![img](https:////upload-images.jianshu.io/upload_images/1915113-a7ba3f734774b27c.png?imageMogr2/auto-orient/strip|imageView2/2/w/388)

block被废弃

![img](https:////upload-images.jianshu.io/upload_images/1915113-996bce79978e0094.png?imageMogr2/auto-orient/strip|imageView2/2/w/512)

block1被废弃

#### Q：block对象类型的auto变量、__block变量的区别？

从几方面回答：

##### 1.当block在栈上时，对它们都不会产生强引用

##### 2.当block拷贝到堆上时

- 都会通过copy函数来处理它们
- 对于__block 修饰的变量 assign函数对其强引用；对于外部对象 assign函数根据外部如何引用而引用
   __block变量（假设变量名叫做a）

```objc
_Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);
```

对象类型的auto变量（假设变量名叫做p）

```objc
_Block_object_assign((void*)&dst->p, (void*)src->p, 3/*BLOCK_FIELD_IS_OBJECT*/);
```

#### 3.当block从堆上移除时

- 都会通过dispose函数来释放它们
   __block变量（假设变量名叫做a）

```objc
_Block_object_dispose((void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);
```

对象类型的auto变量（假设变量名叫做p）

```objc
_Block_object_dispose((void*)src->p, 3/*BLOCK_FIELD_IS_OBJECT*/);
```

#### 4.__block的__forwarding指针

- 栈上__block的__forwarding指向本身
- 栈上__block复制到堆上后，栈上block的__forwarding指向堆上的block，堆上block的__forwarding指向本身

![img](https:////upload-images.jianshu.io/upload_images/1915113-6f2aa39b308b3f4c.png?imageMogr2/auto-orient/strip|imageView2/2/w/659)

__forwarding指向

#### Q：被__block修饰的对象类型在block上如何操作的？

分几方面回答：

##### 1.当__block变量在栈上时，不会对指向的对象产生强引用

##### 2.当__block变量被copy到堆时

- 会调用__block变量内部的copy函数
- copy函数内部会调用_Block_object_assign函数
- _Block_object_assign函数会根据所指向对象的修饰符（__strong、__weak、__unsafe_unretained）做出相应的操作，形成强引用（retain）或者弱引用`（注意：这里仅限于ARC时会retain，MRC时不会retain）`
- MRC环境下不会根据对象的修饰符引用，都是弱引用

##### 3.如果__block变量从堆上移除

- 会调用__block变量内部的dispose函数
- dispose函数内部会调用_Block_object_dispose函数
   _Block_object_dispose函数会自动释放指向的对象（release）

## 第六部分 block循环引用

#### Q：ARC下如何解决block循环引用的问题？

三种方式：__weak、__unsafe_unretained、__block

##### 1.第一种方式：__weak

```objc
- (void)sec3demo1 {
    SGHBlockPerson *person = [[SGHBlockPerson alloc] init];
//    __weak SGHBlockPerson *weakPerson = person;
    __weak typeof(person) weakPerson = person;

    person.block = ^{
        NSLog(@"age is %ld", (long)weakPerson.age);
    };
    person.block();
}
```

##### 2.第二种方式：__unsafe_unretained

```objc
- (void)sec3demo2 {
    SGHBlockPerson *person = [[SGHBlockPerson alloc] init];
    __unsafe_unretained  typeof(person) tPerson = person;
    person.block = ^{
        NSLog(@"age is %ld", (long)tPerson.age);
    };
    person.block();
}
```

##### 3.第三种方式：__block

```objc
- (void)sec3demo3 {
    SGHBlockPerson *person = [[SGHBlockPerson alloc] init];
    __block typeof(person) blockPerson = person;
    person.block = ^{
        NSLog(@"age is %ld", (long)blockPerson.age);
        blockPerson = nil; //使用完blockPerson后，如果没有设置`blockPerson = nil;`，会出现循环引用
    };
    person.block();
}
```

##### 4.三种方法比较

- `__weak`：**不会产生强引用，指向的对象销毁时，会自动让指针置为nil 。**
- `__unsafe_unretained`：不会产生强引用，不安全，<font color=#FF0000>**指向的对象销毁时，指针存储的地址值不变**</font>。
- `__block`：必须把引用对象置为nil，并且要调用该block。

![img](https:////upload-images.jianshu.io/upload_images/1915113-c6530ce13f306da1.png?imageMogr2/auto-orient/strip|imageView2/2/w/249)

__weak 和 __unsafe_unretained 解决循环引用方式

![img](https:////upload-images.jianshu.io/upload_images/1915113-81c1649b2934c0ee.png?imageMogr2/auto-orient/strip|imageView2/2/w/733)

__block解决循环引用方式

#### Q：MRC下如何解决block循环引用的问题？

两种方式：`__unsafe_unretained`、`__block`

##### 1.第一种方式：`__unsafe_unretained`

```objc
- (void)sec3demo1 {
    SGHBlockPerson *person = [[[SGHBlockPerson alloc] init] autorelease];
    __unsafe_unretained  typeof(person) tPerson = person;
    person.block = ^{
        NSLog(@"age is %ld", (long)tPerson.age);
    };
    person.block();
}
```

##### 2.第二种方式：`__block`

```objc
- (void)sec3demo2 {
    SGHBlockPerson *person = [[[SGHBlockPerson alloc] init] autorelease];
    __block typeof(person) blockPerson = person;
    person.block = ^{
        NSLog(@"age is %ld", (long)blockPerson.age);
        blockPerson = nil; //使用完blockPerson后，如果没有设置`blockPerson = nil;`，会出现循环引用
    };
    person.block();
}
```



----

【完】