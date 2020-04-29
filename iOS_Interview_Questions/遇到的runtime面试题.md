参考：
1. [ios学习--你会遇到的runtime面试题（详）](https://blog.csdn.net/zzzzzdddddxxxxx/article/details/53542441)
2. [iOS-runtime 之面试题详解二](https://www.jianshu.com/p/de4ef8d027ac)
3. 


### 1.讲一下 OC 的消息机制

- OC中的方法调用其实都是转成了`objc_msgSend`函数的调用，给receiver（方法调用者）发送了一条消息（selector方法名）
- `objc_msgSend`底层有3大阶段：消息发送（当前类、父类中查找）、动态方法解析、消息转发

### 2.什么是Runtime？平时项目中有用过么？ 【要背】

答：
1、**OC是一门动态性比较强的编程语言，允许很多操作推迟到程序运行时再进行**

2、**OC的动态性就是由Runtime来支撑和实现的，Runtime是一套C语言的API，封装了很多动态性相关的函数**


3、平时编写的OC代码，底层都是转换成了Runtime API进行调用


### 3.runtime具体应用 【要背】


1.利用关联对象（AssociatedObject）给分类添加属性

2.遍历<u>类的</u>所有成员变量（修改textfield的占位文字颜色、字典转模型、自动归档解档）

3.交换方法实现（交换系统的方法）

4.利用消息转发机制，解决方法找不到的异常问题

# 二、解答
### 1. runtime怎么添加属性、方法等

- ivar表示成员变量
- class_addIvar
- class_addMethod
- class_addProperty
- class_addProtocol
- class_replaceProperty


### 2. runtime 如何实现 weak 属性


首先要搞清楚weak属性的特点:

**weak策略表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值，此特质同assign类似。然而在<u>属性所指的对象</u>遭到摧毁时，属性值也会清空(nil out)**

那么runtime如何实现weak变量的自动置nil？

**runtime对注册的类，会进行布局，会将 weak 对象放入一个 hash 表中。用 <u>weak 指向的对象内存地址</u>作为 key，当此对象的引用计数为0时，会调用对象的 dealloc 方法。假设 <u>weak 指向的对象内存地址</u>是a，那么就会以a为key，在这个 weak hash表中搜索，找到<u>所有以a为key的 weak 对象</u>，从而设置为 nil。**

weak属性需要在dealloc中置nil么?

在ARC环境无论是强指针还是弱指针都无需在 dealloc 设置为 nil ， ARC 会自动帮我们处理。

即便是编译器不帮我们做这些，weak也不需要在dealloc中置nil，
在属性所指的对象遭到摧毁时，属性值也会清空


```objc
// 模拟下weak的setter方法，大致如下
- (void)setObject:(NSObject *)object {         
    objc_setAssociatedObject(self, "object", object, OBJC_ASSOCIATION_ASSIGN); 
    [object cyl_runAtDealloc:^{ 
        _object = nil; 
    }];
}
```

### 3.runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）



1. **每个对象中都有一个对象方法列表（对象方法缓存）**
2. **类方法列表，是存放在isa指针指向的元类对象中（类方法缓存）**
3. **方法列表中，每个方法结构体中记录着方法名称，方法实现，以及参数类型，其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现.**
4. 当我们发送一个消息给一个NSObject对象时，这条消息会在对象的类对象方法列表里查找
5. 当我们发送一个消息给一个类时，这条消息会在类的Meta Class对象的方法列表里查找



**类对象中有类方法和实例方法的列表，列表中记录着方法的名称、参数和实现，而selector本质就是方法名称，runtime<u>通过这个方法名称</u>就可以在列表中找到<u>该方法对应的实现</u>。**

**在结构体`objc_class`中，声明了一个指向`struct objc_method_list`指针的指针，可以包含类方法列表和实例方法列表。**

**在寻找IMP的地址时，runtime提供了两种方法**

```objc
IMP class_getMethodImplementation(Class cls, SEL name);
IMP method_getImplementation(Method m)
```

通过传入参数的不同，找到不同的方法列表。方法列表中保存着下面的`objc_method`结构体，结构体中包含这方法的实现。selector本质就是方法的名称，通过该方法名称，即可在结构体中找到相应的实现。

```objc
struct objc_method {
    SEL method_name                                      
    char *method_types                                       
    IMP method_imp                                           
}
```



参考：

1. [runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）](https://blog.csdn.net/dp948080952/article/details/52437451)

2. [runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）](https://www.cnblogs.com/huangzs/p/7574254.html)

---

