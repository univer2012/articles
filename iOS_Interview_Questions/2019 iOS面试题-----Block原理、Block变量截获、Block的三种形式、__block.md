来自：[2019 iOS面试题-----Block原理、Block变量截获、Block的三种形式、__block](https://juejin.im/post/5e13126b6fb9a04811666925)



- **什么是Block？**
- **Block变量截获**
- **Block的几种形式**

### 一、什么是Block？

- #### Block是将函数及其执行上下文封装起来的对象。

比如：

```objectivec
///WYTest.m文件中执行 `clang -rewrite-objc WYTest.m`


#import <Foundation/Foundation.h>
@interface WYTest : NSObject

@end

@implementation WYTest

- (void)blockTest {
    NSInteger num = 3;
    NSInteger(^block)(NSInteger) = ^NSInteger(NSInteger n){
        return n * num;
    };
    
    block(2);
}

@end
```

通过 `clang -rewrite-objc WYTest.m` 命令编译该.m文件，发现该block被编译成这个形式:

```c++
static void _I_WYTest_blockTest(WYTest * self, SEL _cmd) {
    NSInteger num = 3;
    NSInteger(*block)(NSInteger) = ((NSInteger (*)(NSInteger))&__WYTest__blockTest_block_impl_0((void *)__WYTest__blockTest_block_func_0, &__WYTest__blockTest_block_desc_0_DATA, num));

    ((NSInteger (*)(__block_impl *, NSInteger))((__block_impl *)block)->FuncPtr)((__block_impl *)block, 2);
}
```

其中 `WYTest` 是文件名，` blockTest` 是方法名，这些可以忽略。
 其中 `__WYTest__blockTest_block_impl_0` 结构体为

```c++
struct __WYTest__blockTest_block_impl_0 {
  struct __block_impl impl;
  struct __WYTest__blockTest_block_desc_0* Desc;
  NSInteger num;
  __WYTest__blockTest_block_impl_0(void *fp, struct __WYTest__blockTest_block_desc_0 *desc, NSInteger _num, int flags=0) : num(_num) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```



`__block_impl` 结构体为

```c++
struct __block_impl {
  void *isa;        //isa指针，所以说Block是对象
  int Flags;
  int Reserved;
  void *FuncPtr;    //函数指针
};
```

**block内部有isa指针，所以说其本质也是OC对象。**

block内部则为:

```c++
static NSInteger __WYTest__blockTest_block_func_0(struct __WYTest__blockTest_block_impl_0 *__cself, NSInteger n) {
  NSInteger num = __cself->num; // bound by copy

        return n * num;
    }
```

所以说 **Block是将函数及其执行上下文封装起来的对象。**

既然block内部封装了函数，那么它同样也有参数和返回值。



### 二、Block变量截获

#### 1、局部变量截获 是值截获。 比如:

```objectivec
- (void)demo1 {
    
    NSInteger num = 3;
    
    NSInteger(^block)(NSInteger) = ^NSInteger(NSInteger n) {
        
        return n * num;
        
    };
    
    num = 1;
    
    NSLog(@"%zd", block(2));
}
```

这里的输出是6而不是2，原因就是对局部变量num的截获是值截获。

同样，在block里如果修改变量num，也是无效的，甚至编译器会报错。

```objectivec
- (void)demo2 {
    NSMutableArray * arr = [NSMutableArray arrayWithObjects:@"1",@"2", nil];
    
    void(^block)(void) = ^{
        
        NSLog(@"%@",arr);//局部变量
        [arr addObject:@"4"];
    };
    
    [arr addObject:@"3"];
    
    arr = nil;
    
    block();
}
```

打印为： 1，2，3

局部对象变量也是一样，截获的是值，而不是指针。在外部将其置为nil，对block没有影响，而该对象调用方法会影响。

#### 2、局部静态变量截获  是指针截获。

```objectivec
- (void)demo3 {
    static  NSInteger num = 3;
    
    NSInteger(^block)(NSInteger) = ^NSInteger(NSInteger n){
        
        return n*num;
    };
    
    num = 1;
    
    NSLog(@"%zd",block(2));
}
```

输出为2，意味着num = 1这里的修改num值是有效的，即是指针截获。
 同样，在block里去修改变量m，也是有效的。

#### 3、全局变量，静态全局变量截获：不截获,直接取值。

我们同样用clang编译看下结果。

```objectivec
static NSInteger num3 = 300;

NSInteger num4 = 3000;

- (void)blockTest {
    NSInteger num = 30;
    
    static NSInteger num2 = 3;
    
    __block NSInteger num5 = 30000;
    
    void(^block)(void) = ^{
        
        NSLog(@"%zd",num);//局部变量
        
        NSLog(@"%zd",num2);//静态变量
        
        NSLog(@"%zd",num3);//全局变量
        
        NSLog(@"%zd",num4);//全局静态变量
        
        NSLog(@"%zd",num5);//__block修饰变量
    };
    
    block();
}
```

编译后

```c++
struct __WYTest__blockTest_block_impl_0 {
  struct __block_impl impl;
  struct __WYTest__blockTest_block_desc_0* Desc;
  NSInteger num;                        //局部变量
  NSInteger *num2;                      //静态变量
  __Block_byref_num5_0 *num5; // by ref //__block修饰变量
  __WYTest__blockTest_block_impl_0(void *fp, struct __WYTest__blockTest_block_desc_0 *desc, NSInteger _num, NSInteger *_num2, __Block_byref_num5_0 *_num5, int flags=0) : num(_num), num2(_num2), num5(_num5->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

（`impl.isa = &_NSConcreteStackBlock;` 这里注意到这一句，即说明该block是栈block）

可以看到**局部变量被编译成值形式**，而**静态变量被编成指针形式，全局变量并未截获**。而  **`__block` 修饰的变量也是以指针形式截获的，并且生成了一个新的结构体 对象**：

```c++
struct __Block_byref_num5_0 {
  void *__isa;
__Block_byref_num5_0 *__forwarding;
 int __flags;
 int __size;
 NSInteger num5;
};
```

该对象有个属性：num5，即我们用` __block` 修饰的变量。
 **这里 `__forwarding` 是指向自身的(栈block)。**

一般情况下，如果我们要**对block截获的<u>局部变量</u>进行赋值操作需添加 `__block`  修饰符，而对全局变量、静态变量是不需要添加 `__block` 修饰符的。**

另外，block里访问self或成员变量都会去截获self。



### 三、Block的几种形式

- 分为全局Block(`_NSConcreteGlobalBlock`)、栈Block( `_NSConcreteStackBlock`)、堆Block(`_NSConcreteMallocBlock`)三种形式

- 其中栈Block存储在栈(stack)区，堆Block存储在堆(heap)区，全局Block存储在已初始化数据(.data)区

#### 1、不使用外部变量的block是全局block

比如：

```objectivec
    NSLog(@"%@",[^{
        NSLog(@"globalBlock");
    } class]);
```

输出：

```
__NSGlobalBlock__
```

#### 2、使用外部变量并且未进行copy操作的block是栈block

比如:

```objectivec
  NSInteger num = 10;
    NSLog(@"%@",[^{
        NSLog(@"stackBlock:%zd",num);
    } class]);
```

输出：

```
__NSStackBlock__
```



日常开发常用于这种情况:

```objectivec
[self testWithBlock:^{
    NSLog(@"%@",self);
}];

- (void)testWithBlock:(dispatch_block_t)block {
    block();

    NSLog(@"%@",[block class]);
}
```

#### 3、对栈block进行copy操作，就是堆block，而对全局block进行copy，仍是全局block

- 比如堆1中的全局进行copy操作，即赋值：

```objectivec
void (^globalBlock)(void) = ^{
        NSLog(@"globalBlock");
    };

 NSLog(@"%@",[globalBlock class]);
```

输出：

```
__NSGlobalBlock__
```

仍是全局block

- 而对2中的栈block进行赋值操作：

```objectivec
NSInteger num = 10;

void (^mallocBlock)(void) = ^{

        NSLog(@"stackBlock:%zd",num);
    };

NSLog(@"%@",[mallocBlock class]);
```

输出：

```
__NSMallocBlock__
```

对栈blockcopy之后，并不代表着栈block就消失了，左边的mallock是堆block，右边被copy的仍是栈block
 比如:

```objectivec
- (void)sec2demo3_3 {
    [self test2WithBlock:^{
        NSLog(@"%@",self);
    }];
}

- (void)test2WithBlock:(dispatch_block_t)block {
    block();
    
    dispatch_block_t tempBlock = block;
    
    NSLog(@"%@,%@",[block class],[tempBlock class]);
}
```

输出：

```
__NSStackBlock__,__NSMallocBlock__
```

- #### 即如果对栈Block进行copy，将会copy到堆区，对堆Block进行copy，将会增加引用计数，对全局Block进行copy，因为是已经初始化的，所以什么也不做。

另外，`__block` 变量在copy时，由于 `__forwarding` 的存在，栈上的 `__forwarding`指针会指向堆上的 `__forwarding`变量，而堆上的 `__forwarding` 指针指向其自身，所以，如果对 `__block` 的修改，实际上是在修改堆上的 `__block` 变量。

#### 即__forwarding指针存在的意义就是，无论在任何内存位置，  都可以顺利地访问同一个__block变量。

- 另外由于block捕获的 `__block` 修饰的变量会去持有变量，那么如果用 `__block` 修饰self，且self持有block，并且block内部使用到 `__block` 修饰的self时，就会造成多循环引用。即self持有block，block 持有 `__block` 变量，而 `__block` 变量持有self，造成内存泄漏。
   比如：

```objectivec
  __block typeof(self) weakSelf = self;
    
    _testBlock = ^{
        
        NSLog(@"%@",weakSelf);
    };
    
    _testBlock();
```

如果要解决这种循环引用，可以主动断开 `__block` 变量对self的持有，即在block内部使用完weakself后，将其置为nil。但这种方式有个问题，**如果block一直不被调用，那么循环引用将一直存在。**
 所以，我们最好还是用`__weak`来修饰self 。

