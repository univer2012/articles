来自：[2019 iOS面试题-----Block原理、Block变量截获、Block的三种形式、__block](https://juejin.im/post/5e13126b6fb9a04811666925)



- **什么是Block？**
- **Block变量截获**
- **Block的几种形式**

#### 一、什么是Block？

- ###### Block是将函数及其执行上下文封装起来的对象。

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

block内部有isa指针，所以说其本质也是OC对象。

block内部则为:

```c++
static NSInteger __WYTest__blockTest_block_func_0(struct __WYTest__blockTest_block_impl_0 *__cself, NSInteger n) {
  NSInteger num = __cself->num; // bound by copy

        return n * num;
    }
```

所以说 Block是将函数及其执行上下文封装起来的对象。

既然block内部封装了函数，那么它同样也有参数和返回值。



#### 二、Block变量截获

###### 1、局部变量截获 是值截获。 比如:

```objectivec
- (void)blockTest {
    
    DKImagePickerWithSwiftImages(R.image.multi_question_D(), R.image.multi_question_L())
    
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