来自：[2019 iOS面试题-----Block原理、Block变量截获、Block的三种形式、__block](https://juejin.im/post/5e13126b6fb9a04811666925)



- **什么是Block？**
- **Block变量截获**
- **Block的几种形式**

#### 一、什么是Block？

- ###### Block是将函数及其执行上下文封装起来的对象。

比如：

```objectivec

///WYTest.m文件中执行 `clang -rewrite-objc WYTest.m`
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

通过clang -rewrite-objc WYTest.m命令编译该.m文件，发现该block被编译成这个形式:

```

```

