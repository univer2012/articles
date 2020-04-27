来自：[RAC(ReactiveCocoa)介绍（十一）——RAC宏定义](https://www.jianshu.com/p/7086e090069d)



在编程领域里的宏是一种抽象（Abstraction），它根据一系列预定义的规则替换一定的文本模式。解释器在遇到宏时会自动进行这一模式替换。绝大多数情况下，“宏”这个词的使用暗示着将小命令或动作转化为一系列指令。

 在RAC框架中，其宏定义的功能强大能帮助开发者更加快速、便捷地进行开发工作。常用的比如：打破循环引用、以及KVO方法的属性监听等等。



打破实例变量的循环引用：

```objc
- (void)p_define {
    @weakify(self);         //备注：
    [[self.testTextField.rac_textSignal filter:^BOOL(NSString * _Nullable value) {
        //过滤判断条件
        @strongify(self)    //备注：
        if (self.testTextField.text.length >= 6) {
            self.testTextField.text = [self.testTextField.text substringToIndex:6];
            self.testLabel.text = @"已经到6位了";
            self.testLabel.textColor = UIColor.redColor;
        }
        return value.length <= 6;
        
    }] subscribeNext:^(NSString * _Nullable x) {
        //订阅逻辑区域
        NSLog(@"filter过滤后的订阅内容：%@",x);
    }];
}
```

KVO属性监听：

```objc
- (void)p_observe {
    [RACObserve(self.testLabel, text) subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x);
    }];
}
```



这一篇主要探究RAC中的宏定义强大之处究竟在哪。

 首先来看下最常用的`@weakify(self)`

```objc
///RACEXTScope.h
#define weakify(...) \
    rac_keywordify \
    metamacro_foreach_cxt(rac_weakify_,, __weak, __VA_ARGS__)
```



此处注意，反斜杠`\`的作用是作为连接符使用，将代码进行连接。即使用`weakify(...)`宏定义时，将先后执行 `rac_keywordify` 与 `metamacro_foreach_cxt(rac_weakify_,, __weak, __VA_ARGS__)` 代码。

 先来看下`rac_keywordify`代码的作用：

```cpp
///RACEXTScope.h
#if DEBUG
#define rac_keywordify autoreleasepool {}
#else
#define rac_keywordify try {} @catch (...) {}
#endif
```

在debug环境下，只有一句`autoreleasepool {}`，此代码是增强代码的编译能力，至于为何要如此使用？

在经常使用的宏定义`RACObserve(TARGET, KEYPATH)`观察KVO属性时，能够在KEYPATH中，代码预提示出指定TARGET中的属性：



![RACObserve能够提示出当前self中存在的实例变量](https://upload-images.jianshu.io/upload_images/1243805-e17775d66fd0bae2.png?imageMogr2/auto-orient/strip|imageView2/2/w/960)



`metamacro_foreach_cxt(rac_weakify_,, __weak, __VA_ARGS__)`代码实现：

```cpp
///RACmetamacros.h
#define metamacro_foreach_cxt(MACRO, SEP, CONTEXT, ...) \
        metamacro_concat(metamacro_foreach_cxt, metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)
        //此函数中，分别将`rac_weakify_`传入`MACRO`参数，(空格)传入`SEP`，`__weak`传入`CONTEXT`，`__VA_ARGS__`(可变参数)传入`...`
```



在`metamacro_foreach_cxt(MACRO, SEP, CONTEXT, ...)`函数中，实现了
```cpp
metamacro_concat(metamacro_foreach_cxt, metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)
```



首先来看看第二个参数`metamacro_argcount(...)`，这是在预编译时用来获取传入参数的个数。

 通过查看层层宏定义封装，依次可找到下列宏：

```objc
///RACmetamacros.h
#define metamacro_at(N, ...) \
        metamacro_concat(metamacro_at, N)(__VA_ARGS__)

#define metamacro_argcount(...) \
        metamacro_at(20, __VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)
///备注：此处将20传入N参数中，其余的作为可变参数传入 metamacro_concat(metamacro_at, N)(__VA_ARGS__)

#define metamacro_concat(A, B) \
        metamacro_concat_(A, B)

#define metamacro_concat_(A, B) A ## B
```



在Objective-C语言中方法调用底层实际上是对C的消息转发，Objective-C语言最终要编译成C函数语言，接触过runtime之后会更加明白。比如在runtime中最常用的`objc_msgSend`方法，Objective-C函数调用都要通过它来进行消息发送实现，`objc_msgSend(id self, SEL _cmd, arg)`，在Objective-C中，该消息发送方法默认实现了id类型的self以及方法选择器`_cmd`，arg参数为开发者自定义的内容。



宏定义在Objective-C中使用`#define`，在C中使用`define`，而#是Objective-C区别于C语言的存在。

那么 `#define metamacro_concat_(A, B) A ## B` 从Objective-C环境编译为C语言时，最终实现的是AB，**也就意味着将A、B拼接到一起。上述宏定义最终是将`metamacro_at`参数与20参数拼接到一起，组成`metamacro_at20(__VA_ARGS__)`。**拼接完成之后可以找到`metamacro_at20`的宏定义：

```objc
#define metamacro_at20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...) metamacro_head(__VA_ARGS__)
```



而且此处会发现，从0-20都已存在宏定义：

```objc
// metamacro_at expansions
#define metamacro_at0(...) metamacro_head(__VA_ARGS__)
#define metamacro_at1(_0, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at2(_0, _1, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at3(_0, _1, _2, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at4(_0, _1, _2, _3, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at5(_0, _1, _2, _3, _4, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at6(_0, _1, _2, _3, _4, _5, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at7(_0, _1, _2, _3, _4, _5, _6, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at8(_0, _1, _2, _3, _4, _5, _6, _7, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at9(_0, _1, _2, _3, _4, _5, _6, _7, _8, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at10(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at11(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at12(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at13(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at14(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at15(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at16(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at17(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at18(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at19(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...) metamacro_head(__VA_ARGS__)
```



<u>**`metamacro_atN`的宏定义，意思为截取掉宏定义中前N个元素，保留剩下的元素传入至`metamacro_head(__VA_ARGS__)`中。**</u>

 展开`metamacro_head(__VA_ARGS__)`，可以发现下列宏定义

```objc
#define metamacro_head(...) \
        metamacro_head_(__VA_ARGS__, 0)
        
#define metamacro_head_(FIRST, ...) FIRST        
```



意味着要取截取后剩下元素中的第一个元素，而这个元素的值也就是metamacro_argcount(...)宏返回出来的元素个数。
 其实现原理为20 -（ 20 - n ）= n，`metamacro_argcount(...)` 宏就是这样在预编译时期获取到参数个数的。



为了更方便理解`metamacro_argcount(...)` 宏实现的过程，举个例子：
 `metamacro_argcount(self,(NSString *)str)`计算出个数为2的过程。

 `metamacro_argcount(...)`宏展开后变为：

```objc
//宏里的可变参数个数为22个
metamacro_at(20, self, str, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)
//合并后变为metamacro_at20(...)宏
metamacro_at20(self, str, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)

```

而`metamacro_at20`的宏定义为

```objc
#define metamacro_at20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...) metamacro_head(__VA_ARGS__)
//metamacro_head(...)的可变参数即为截取后剩下的2个元素(2,1)
metamacro_head(2,1)
```



在`metamacro_at20`宏中，前20个元素位置已被预设好的元素占用，那么`metamacro_head(...)`的可变参数即为截取后剩下的2个元素(2,1)。通过`metamacro_head`宏取出第一个元素的值并返回，最后得到的数值为2，传入参数的个数为2。这也就是在预编译时如何获取传参个数的全过程。



这时，再回到`metamacro_concat(metamacro_foreach_cxt, metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)`宏，现在已经知道`metamacro_concat`是用来生成一个新的宏定义，此处就变为`metamacro_foreach_cxtN(MACRO, SEP, CONTEXT, __VA_ARGS__)`。



> 注：此处cxtN中的N为`metamacro_argcount(...)`宏返回的个数。



`metamacro_foreach_cxtN(MACRO, SEP, CONTEXT, __VA_ARGS__)`宏的实现，N同样为0-20：

```objc
// metamacro_foreach_cxt expansions
#define metamacro_foreach_cxt0(MACRO, SEP, CONTEXT)
#define metamacro_foreach_cxt1(MACRO, SEP, CONTEXT, _0) MACRO(0, CONTEXT, _0)

#define metamacro_foreach_cxt2(MACRO, SEP, CONTEXT, _0, _1) \
    metamacro_foreach_cxt1(MACRO, SEP, CONTEXT, _0) \
    SEP \
    MACRO(1, CONTEXT, _1)

#define metamacro_foreach_cxt3(MACRO, SEP, CONTEXT, _0, _1, _2) \
    metamacro_foreach_cxt2(MACRO, SEP, CONTEXT, _0, _1) \
    SEP \
    MACRO(2, CONTEXT, _2)

#define metamacro_foreach_cxt4(MACRO, SEP, CONTEXT, _0, _1, _2, _3) \
    metamacro_foreach_cxt3(MACRO, SEP, CONTEXT, _0, _1, _2) \
    SEP \
    MACRO(3, CONTEXT, _3)

#define metamacro_foreach_cxt5(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4) \
    metamacro_foreach_cxt4(MACRO, SEP, CONTEXT, _0, _1, _2, _3) \
    SEP \
    MACRO(4, CONTEXT, _4)

#define metamacro_foreach_cxt6(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5) \
    metamacro_foreach_cxt5(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4) \
    SEP \
    MACRO(5, CONTEXT, _5)

#define metamacro_foreach_cxt7(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6) \
    metamacro_foreach_cxt6(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5) \
    SEP \
    MACRO(6, CONTEXT, _6)
    ///... ...
```



继续拿上面的例子来说，当返回为2个元素个数之后
 在宏最外部分别将rac_weakify_传入MACRO参数，(空格)传入SEP，__weak传入CONTEXT

```objectivec
#define metamacro_foreach_cxt2(MACRO, SEP, CONTEXT, _0, _1) \
    metamacro_foreach_cxt1(MACRO, SEP, CONTEXT, _0) \
    SEP \
    MACRO(1, CONTEXT, _1)

    //展开后变为
    rac_weakify_(0, __weak, self)  \
    rac_weakify_(1, __weak, str)
```



此时，得到了一个rac_weakify_(...)宏，那么来看下这个宏什么作用

```objc
///RACEXTScope.h
#define rac_weakify_(INDEX, CONTEXT, VAR) \
    CONTEXT __typeof__(VAR) metamacro_concat(VAR, _weak_) = (VAR);
```



展开后

```objc

__weak __typeof__(self)  self_weak_ = (self);
__weak __typeof__(str)  str_weak_ = (str);
```

。。。。。。。。。分析了这么一大圈，就是为了一句弱引用？？？



不过这样也有好处，就是可以最多传入20个对象全部弱引用



既然已经分析出了`@weakify(...)`宏定义的作用，那么`@strongify(...)`宏定义作用也就显而易见了：将对象变为强引用。

```objc
#define rac_strongify_(INDEX, VAR) \
    __strong __typeof__(VAR) VAR = metamacro_concat(VAR, _weak_);

//最后的展开结果
__weak __typeof__(self_weak_)  self = (self_weak_);
__weak __typeof__(str_weak_)  str = (str_weak_);

```



这也说明了在RAC中@weakify(...)宏定义与@strongify(...)一定要搭配使用的原因。
 为什么要在这里加一个@符号？
 Objective-C源于C语言，输入字符串时，C语言用""来表示，而Objective-C是用@""来表示。此处要加@符号，是把C语言的结构包装成Objective-C。添加@符号，作用为预编译，会从底层C语言中找相应的函数，寻找TARGET相应的KEYPATH路径。



接下来，来分析一下RAC中观察属性KVO宏定义`RACObserve(TARGET, KEYPATH)`：

```objc
///NSObject+RACPropertySubscribing.h
#if __clang__ && (__clang_major__ >= 8)
#define RACObserve(TARGET, KEYPATH) _RACObserve(TARGET, KEYPATH)
#else
#define RACObserve(TARGET, KEYPATH) \
({ \
	_Pragma("clang diagnostic push") \
	_Pragma("clang diagnostic ignored \"-Wreceiver-is-weak\"") \
	_RACObserve(TARGET, KEYPATH) \
	_Pragma("clang diagnostic pop") \
})
#endif

///NSObject+RACPropertySubscribing.h
#define _RACObserve(TARGET, KEYPATH) \
({ \
	__weak id target_ = (TARGET); \
	[target_ rac_valuesForKeyPath:@keypath(TARGET, KEYPATH) observer:self]; \
})
```



将TARGET弱引用生成一个target_对象，下面主要来分析下一句代码
 先查看`@keypath(TARGET, KEYPATH)`

```objc
#define keypath(...) \
//判断可变参数个数是否为1
//若为1则执行 (keypath1(__VA_ARGS__)
//否则执行(keypath2(__VA_ARGS__)
    metamacro_if_eq(1, metamacro_argcount(__VA_ARGS__))(keypath1(__VA_ARGS__))(keypath2(__VA_ARGS__))

#define keypath1(PATH) \
    (((void)(NO && ((void)PATH, NO)), strchr(# PATH, '.') + 1))

#define keypath2(OBJ, PATH) \
    (((void)(NO && ((void)OBJ.PATH, NO)), # PATH))
```



此处`@keypath(TARGET, KEYPATH)`一定要添加@符号，**就是为了能预编译出TARGET中所有的KEYPATH属性。**



##### 何为预编译？

 在main函数执行之前，执行预编译处理。把整个类加载进内存中，在编程过程中，会去匹配TARGET类的类型，当匹配到对应类之后，会去编译查找对应的属性表property list、成员表IRG list、方法表method list。所以这里执行了预编译处理后，就可以提示出TARGET所有的示例变量、属性以及方法。



---

【完】