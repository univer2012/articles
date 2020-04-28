#### 问：定义了一个block，它在内存的那一块？

答：

block没有引用外部变量，MRC和ARC模式下，是`__NSGlobalBlock__`。

block有引用外部变量，在MRC下是`__NSStackBlock__`，在ARC下是`__NSMallocBlock__`。

---



# 1 Block基础和retain cycle(循环引用)

### 1.1 blcok简介

Block 是从c语言的扩展来的，并不是什么高新技术，和swift语言的闭包需要注意的是由于 Objective-C在iOS中不支持GC机制。

错误的内存管理
* 要么导致return cycle内存泄漏
* 要么内存被提前释放导致crash。


Block的使用很像函数指针，**不过与函数最大的不同是：Block可以访问函数以外、词法作用域以内的外部变量的值**。换句话说，Block不仅实现函数的功能，还能携带函数的执行环境。

## 1.2 block基本语法
### 1.2.1如何定义block变量
```
//第一个block是一个int类型的返回值，并且有2个参数
    //第二个block是没有返回值，没有参数的block
    int (^sumBlock)(int, int);
    void (^myBlock)();
```
### 1.2.2 如何使用block来封装代码
```
//最基本的用法
    int (^sumBlock)(int, int) = ^(int a, int b) {
        return a - b;
    };
```
```
//1.宏定义一个block
    typedef int (^MyBlock)(int, int);
    //2.利用宏定义来定义变量
    MyBlock sumBlock;
    //3.定义一个block变量来实现两个参数相加
    sumBlock = ^(int a, int b) {
        return a + b;
    };
    //4.定义一个block变量来实现两个参数相减
    MyBlock minusBlock = ^(int a, int b) {
        return a - b;
    };
    //5.定义一个block变量来实现两个参数相乘
    MyBlock multiplyBlock = ^(int a, int b) {
        return a * b;
    };
```

### 1.2.3 如何调用block
```
NSLog(@"%d - %d - %d", multiplyBlock(2, 4), sumBlock(10, 9), minusBlock(10,8));
    //这个依次输出是 8,19,2
```
### 1.2.4 block可以访问外部变量
```
int a = 10;
    //给局部变量加上__block之后就可以改变b局部变量的值,将取变量此刻运行时的值
    __block int b = 2;
    //定义一个block
    void (^block)();

    block = ^{
        /*默认情况下，block内部不能修改外面的局部变量。所以下面一句会报错：
         Variable is not assignable(missing __block type specifier)  */
        //a = 20;
        //给局部变量加上__block关键字，这个局部变量就可以在block内部修改
        b = 25;
    };
    block();
    NSLog(@"%d, %d", a, b);
    /*输出： 10, 25*/
```

## 1.3 block基本理解

1. Block执行的代码其实在编译的时候就已经准备好了，就等着我们调用
一个包含Block执行时需要的所有外部变量值的数据结构。 
2. Block将使用到的、作用域附近变量的值建立一份快照拷贝到栈上。

## 1.4 block在内存中的分析

block内存中的三个位置 `NSGlobalBlock`，`NSStackBlock`, `NSMallocBlock`
* `NSGlobalBlock` :和函数类似，位于text代码段
* `NSStackBlock` : 栈内存，函数返回后Block将无效
* `NSMallocBlock` : 堆内存

```
//宏定义一个block
        typedef long (^BlockSum)(int, int);
        BlockSum block1 =^long(int a,int b){
            return a + b;
        };
        NSLog(@"%@",block1);
        //<__NSGlobalBlock__: 0x1000020b0>

        int base = 100;
        BlockSum block2 = ^long(int a, int b) {
            return base + a + b;
        };
        NSLog(@"%@",block2);
        /*arc和非arc所在的内存位置不同:
         mrc:
         <__NSStackBlock__: 0x7fff5698b070>
         arc:
         <__NSMallocBlock__: 0x10051cf60>  */
        BlockSum block3 = [block2 copy];
        NSLog(@"%@", block3);
        //<__NSMallocBlock__: 0x10051cf60>
```

上述中为什么block1在NSGlobalBlock中，block2在NSStackBlock(mrc)，NSMallocBlock(arc)中？因为block2用到了外部变量base，需要建立局部变量的快照，所以在(定义，不是运行)局部变量被拷贝到栈上(mrc)，堆（arc）上。

```
typedef long (^BlockSum)(int, int);
        int base = 2;
        base += 2;
        BlockSum sum = ^ long (int a,int b){
            return base + a + b;
        };
        base ++ ;
        NSLog(@"%ld",sum(1,2));
        //7
```
分析上述代码，**因为有局部变量拷贝到栈里或者堆里，所以不会用运行时的变量base，而是拷贝base**，所以输出的结果为 7，不是8。

## 1.5 Block的copy，retain，release操作

1. 对block retain操作并不会改变引用计数器,retainCount ,始终为1；
2. **<font color=#ff0000 size=3>NSGlobalBlock：retain、copy、release操作都无效；</font>**
3. Block_copy与copy等效，Block_release与release等效；
4. NSStackBlock：<font color=#ff0000 size=3>retain、release操作无效</font>，必须注意的是，<font color=#ff0000 size=3>NSStackBlock在函数返回后，Block内存将被回收。即使retain也没用</font>。容易犯的错误是 `[[mutableAarry addObject:stackBlock]`，在函数出栈后，从mutableAarry中取到的stackBlock已经被回收，变成了野指针。正确的做法是先将stackBlock copy到堆上，然后加入数组：`[mutableAarry addObject:[[stackBlock copy] autorelease]]`。**支持copy，copy之后生成新的NSMallocBlock类型对象。**
5. NSMallocBlock支持retain、release，虽然retainCount始终是1，但内存管理器中仍然会增加、减少计数。copy之后不会生成新的对象，只是增加了一次引用，类似retain；
6. **尽量不要对Block使用retain操作。**


测试代码如下：
```
//宏定义一个block
        typedef long (^BlockSum)(int, int);
        //<__NSGlobalBlock__: 0x1000020b0>
        BlockSum block1 =^long(int a,int b){
            return a + b;
        };
        NSLog(@"block1: %lu", (unsigned long)[block1 retainCount]);
        BlockSum tempBlock1 = [block1 retain];
        NSLog(@"block1_retain: %lu", (unsigned long)[block1 retainCount]);
        BlockSum tempBlock12 = [block1 copy];
        NSLog(@"block1_copy: %lu", (unsigned long)[block1 retainCount]);
        [block1 release];
        NSLog(@"block1_release: %lu", (unsigned long)[block1 retainCount]);
        NSLog(@"NSGlobalBlock: %@, retain: %@, copy: %@",block1, tempBlock1, tempBlock12);

        //MRC: <__NSStackBlock__: 0x7fff5698b070>
        int base = 100;
        BlockSum block2 = ^long(int a, int b) {
            return base + a + b;
        };
        NSLog(@"block2: %lu", (unsigned long)[block2 retainCount]);
        BlockSum tempBlock2 = [block2 retain];
        NSLog(@"block2_retain: %lu", (unsigned long)[block2 retainCount]);
        [block2 release];
        NSLog(@"block2_release: %lu", (unsigned long)[block2 retainCount]);



        //<__NSMallocBlock__: 0x10051cf60>
        BlockSum block3 = [block2 copy];
        NSLog(@"NSStackBlock: %@, retain: %@, copy: %@", block2, tempBlock2, block3);

        NSLog(@"block3: %lu", (unsigned long)[block3 retainCount]);
        BlockSum tempBlock3 = [block3 retain];
        NSLog(@"block3_retain: %lu", (unsigned long)[block3 retainCount]);
        BlockSum tempBlock31 = [block3 copy];
        NSLog(@"block3_copy: %lu", (unsigned long)[block3 retainCount]);
        [block3 release];
        NSLog(@"block3_release: %lu", (unsigned long)[block3 retainCount]);
        NSLog(@"NSMallocBlock: %@, retain: %@, copy: %@", block3, tempBlock3, tempBlock31);
```

输出如下：
```
block1: 1
 block1_retain: 1
 block1_copy: 1
 block1_release: 1
 NSGlobalBlock: <__NSGlobalBlock__: 0x10a1d3118>, retain: <__NSGlobalBlock__: 0x10a1d3118>, copy: <__NSGlobalBlock__: 0x10a1d3118>
 block2: 1
 block2_retain: 1
 block2_release: 1
 NSStackBlock: <__NSStackBlock__: 0x7fff55a30060>, retain: <__NSStackBlock__: 0x7fff55a30060>, copy: <__NSMallocBlock__: 0x604000248370>
 block3: 1
 block3_retain: 1
 block3_copy: 1
 block3_release: 1
 NSMallocBlock: <__NSMallocBlock__: 0x604000248370>, retain: <__NSMallocBlock__: 0x604000248370>, copy: <__NSMallocBlock__: 0x604000248370>
```

## 1.6 Block不同类型的变量
### 1.6.1 static 和基本数据类型
```
//宏定义一个block
        typedef long (^BlockSum)(int, int);
        //1.static int
        static int base = 100;
        BlockSum sum = ^long(int a, int b){
            return a + b + base;
        };
        base = 0;
        NSLog(@"%ld",sum(1, 2));

        //2.int
        int base1 = 100;
        BlockSum sum1 = ^long(int a, int b){
            return a + b + base1;
        };
        base1 = 0;
        NSLog(@"%ld",sum1(1, 2));
        /*输出分别为：
        3  
        103*/
```
如果是**static时，外部可以改变base变量，因为一直是一个内存地址，并没有建立局部变量的快照，不是在定义时copy的常量。**

如果是**基本类型，会建立一个拷贝，不是同一个地址，所以值不会改变。**

所以static输出的是 3 ，基本数据类型是 103

### 1.6.2 static变量 如果block中也有变量的时候
```
//宏定义一个block
        typedef long (^BlockSum)(int, int);
        //1.static int
        static int base = 10;
        BlockSum sum = ^long(int a, int b){
            base ++;
            return a + b + base;
        };
        base = 0;
        NSLog(@"%d ,\n %ld,\n %d", base, sum(1, 2),base);
        /*输出为：
        0,
        4,
        1*/
```
这段代码说明block内部对外部static修饰的变量可以在内部进行修改，如果不加static或者block的会报错。

* Block变量：被__block修饰的变量，称作block变量, 基本类型的Block变量等效于全局变量、或静态变量；
* BlockA被BlockB使用时，BlockB被copy到堆上，被使用的BlockA也会被copy。但作为参数的Block是不会发生copy的；
* ARC的block所有的都在堆上。
* MRC的看下边的实例：
```
//宏定义一个block
typedef long (^BlockSum)(int, int);

void bar(BlockSum block2){
    NSLog(@"%@",block2);
    ///<__NSStackBlock__: 0x7fff5bfa0078>

    void (^block3) (BlockSum) = ^(BlockSum sum){
        NSLog(@"%@",sum);
        NSLog(@"%@",block2);
    };

    block3(block2);
     ///<__NSStackBlock__: 0x7fff5bfa0078>
     ///<__NSStackBlock__: 0x7fff5bfa0078>
    NSLog(@"block3: %@",block3);    //block3: <__NSStackBlock__: 0x7fff5670e028>
    block3 = [block3 copy];
    NSLog(@"block3_copy: %@",block3);   //block3_copy: <__NSMallocBlock__: 0x60000005b030>

    block3(block2);
     ///<__NSStackBlock__: 0x7fff5bfa0078>
     ///<__NSMallocBlock__: 0x604000246ab0>
}

int main(int argc, char * argv[]) {
    @autoreleasepool {
        int base = 10;
        BlockSum block1 = ^long(int a, int b){
            return a + b + base;
        };
        NSLog(@"%@", block1);
        ///<__NSStackBlock__: 0x7fff5bfa0078>
        bar(block1);
    }
}
```

### 1.6.3 ObjC对象，不同于基本类型，Block会引起对象的引用计数变化
```
@interface MyClass:NSObject {
    NSObject *_instanceObj;
}
@end
@implementation MyClass
NSObject *__globalObj = nil;
-(id)init {
    if (self = [super init]) {
        _instanceObj = [[NSObject alloc] init];
    }
    return self;
}
-(void)test {
    static NSObject *__staticObj = nil;
    __globalObj = [[NSObject alloc] init];
    __staticObj = [[NSObject alloc] init];

    NSObject *localObj = [[NSObject alloc] init];
    __block NSObject* blockObj = [[NSObject alloc] init];

    typedef void (^MyBlock)(void) ;
    MyBlock aBlock = ^{
        NSLog(@"%@", __globalObj);
        NSLog(@"%@", __staticObj);
        NSLog(@"%@", _instanceObj);
        NSLog(@"%@", localObj);
        NSLog(@"%@", blockObj);
    };
    aBlock = [[aBlock copy] autorelease];
    aBlock();

    NSLog(@"%lu", (unsigned long)[__globalObj retainCount]);
    NSLog(@"%lu", (unsigned long)[__staticObj retainCount]);
    NSLog(@"%lu", (unsigned long)[_instanceObj retainCount]);
    NSLog(@"%lu", (unsigned long)[localObj retainCount]);
    NSLog(@"%lu", (unsigned long)[blockObj retainCount]);
}
@end


int main(int argc, char * argv[]) {
    @autoreleasepool {
        MyClass *obj = [[[MyClass alloc] init] autorelease];
        [obj test];
        return 0;
    }
}
```
输出：
```
<NSObject: 0x604000015140>
 <NSObject: 0x604000015150>
 <NSObject: 0x604000015130>
 <NSObject: 0x604000015160>
 <NSObject: 0x604000015170>
 1
 1
 1
 2
 1
```
__globalObj和__staticObj在内存中的位置是确定的，所以Block copy时不会retain对象。

**_instanceObj在Block copy时也没有直接retain _instanceObj对象本身，但会retain self。所以在Block中可以直接读写_instanceObj变量。**

localObj在Block copy时，系统自动retain对象，增加其引用计数。

blockObj在Block copy时也不会retain。


### 1.6.4 **非ObjC对象**，如GCD队列dispatch_queue_t。**Block copy时并不会自动增加他的引用计数**，这点要非常小心。

### 1.6.5 Block中使用的ObjC对象的行为
```
@property (nonatomic, copy) void(^myBlock)(void);
//...
MyClass* obj = [[[MyClass alloc] init] autorelease];
self.myBlock = ^ {
    [obj doSomething];
}; 
```

对象obj在Block被copy到堆上的时候自动retain了一次。**因为Block不知道obj什么时候被释放，为了不在Block使用obj前被释放，Block retain了obj一次。在Block被释放的时候，obj被release一次。**

## 1.7 retain cycle(循环引用的问题)
```
ASIHTTPRequest *request = [ASIHTTPRequest  requestWithURL:url];
[request setCompletionBlock:^{
    NSString* string = [request responseString];
}];
```
在上边这个实例中request和Block循环引用，所以我们只需要打断其中的循环即可。

解决这个问题的办法是使用弱引用打断retain cycle：
```
__block ASIHTTPRequest *request = [ASIHTTPRequest requestWithURL:url];
[request setCompletionBlock:^{
    NSString* string = [request responseString];
}];
```
request被持有者释放后，request 的retainCount变成0，request被dealloc，request释放持有的Block，导致Block的retainCount变成0，也被销毁。这样这两个对象内存都被回收。

与上面情况类似的是：
```
@interface ViewController ()
//报错: This block declaration is not a prototype
//@property(nonatomic)void(^myBlock)();
@property(nonatomic)void(^myBlock)(void);

@property(nonatomic,retain)NSString *someVar;

@end

@implementation ViewController


- (void)viewDidLoad {
    [super viewDidLoad];
    //=====1.
    //self和block循环引用解决办法同上
    self.myBlock = ^{
        [self doSomething];
    };
    //=====2.
    self.myBlock = ^{
        NSLog(@"%@", _someVar);
    };

    //=====3.
    NSString *str = _someVar;
    self.myBlock = ^{
        NSLog(@"%@",str);
    };
}
-(void)doSomething {
}
@end
```

### 1.7.1 retain cycle不只发生在两个对象之间，也可能发生在多个对象之间，这样问题更复杂，更难发现。
```
@interface ClassA : NSObject
@property(nonatomic, copy)void(^myBlock)(void);
@end
@implementation ClassA

@end

@interface ViewController ()
@property(nonatomic)ClassA *objA;
@end
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    ClassA *objA = [[[ClassA alloc] init] autorelease];
    objA.myBlock = ^{
        [self doSomething];
    };
    self.objA = objA;
}
-(void)doSomething {
}
@end
```

解决办法同样是用__block打破循环引用：
```
ClassA *objA = [[[ClassA alloc] init] autorelease];
    /*Cannot create __weak reference in file using manual reference counting
     在使用MRC的文件中，不能创建__weak引用计数 */
    //__weak typeof(self) *weakSelf = self;
    __block typeof(self) blockSelf = self; 
    objA.myBlock = ^{
        [self doSomething];
    };
    self.objA = objA;
```

对上边的进行分析 self(retain 1) －---> objA(retain 1) ---->Block (retain 1)---->self 循环引用。

### ~~1.7.2 注意：MRC中__block是不会引起retain；但在ARC中__block则会引起retain。ARC中应该使用__weak或__unsafe_unretained弱引用。__weak只能在iOS5以后使用。~~
```
//ARC模式下 
//'retainCount' is unavailable: not available in automatic reference counting mode
NSLog(@"%@", [self retainCount]);
```

## 1.8 block对象被提前释放
看下面例子，有这种情况，如果不只是request持有了Block，另一个对象也持有了Block(下边的等号是一条虚线一条实线，block指向request 的是虚线)
--->request =======>Block<---- ObjA

这时**request已被完全释放，但Block仍被objA持有，没有释放，如果这时触发了Block，在Block中将访问已经销毁的request，这将导致程序crash**。为了避免这种情况，开发者必须要注意对象和Block的生命周期。

另一个常见错误使用是，开发者担心retain cycle错误的使用__block。比如
```objc
__block kkProducView* weakSelf = self;
dispatch_async(dispatch_get_main_queue(), ^{
    weakSelf.xx = xx;
});
```

将Block作为参数传给dispatch_async时，系统会将Block拷贝到堆上，如果Block中使用了实例变量，还将retain self，因为dispatch_async并不知道self会在什么时候被释放，**为了确保系统调度执行Block中的任务时self没有被意外释放掉，dispatch_async必须自己retain一次self，任务完成后再release self**。但这里使用__block，使dispatch_async没有增加self的引用计数，这使得在系统在调度执行Block之前，self可能已被销毁，但系统并不知道这个情况，导致Block被调度执行时self已经被释放导致crash。

```objc
//MRC模式下运行
@interface MyClass : NSObject
@property(nonatomic, copy)void(^myBlock)(void);
@end
@implementation MyClass
-(void)test {
    __block MyClass *weakSelf = self;
    double delayInseconds = 10.0;
    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayInseconds * NSEC_PER_SEC));
    dispatch_after(popTime, dispatch_get_main_queue(), ^{
        NSLog(@"%@",weakSelf);
    });
}
@end

@interface ViewController ()
@property(nonatomic)MyClass *objA;

@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    MyClass *obj = [[[MyClass alloc] init] autorelease];
    [obj test];
}
@end
```

这里用dispatch_after模拟了一个异步任务，10秒后执行Block。**但执行Block的时候`MyClass* obj`已经被释放了，导致crash。解决办法是不要使用`__block`**。


![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/1516154909446_2.png)