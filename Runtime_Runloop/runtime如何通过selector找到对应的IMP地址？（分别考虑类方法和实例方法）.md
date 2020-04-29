来自：[runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）](https://blog.csdn.net/dp948080952/article/details/52437451)

---

最近在看《招聘一个靠谱的iOS》，这是其中的一个题目，看着别人的解答不是很详细，于是就想弄清楚一些，通过查找了一些资料并且自己写了一些测试的代码，在这里做个总结！
概述

类对象中有类方法和实例方法的列表，列表中记录着方法的名词、参数和实现，而selector本质就是方法名称，runtime通过这个方法名称就可以在列表中找到该方法对应的实现。

```objc
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        
    const char *name                                         
    long version                                             
    long info                                                
    long instance_size                                       
    struct objc_ivar_list *ivars                             
    struct objc_method_list **methodLists                    
    struct objc_cache *cache                                 
    struct objc_protocol_list *protocols                     
#endif

} OBJC2_UNAVAILABLE;
```

这里声明了一个指向struct objc_method_list指针的指针，可以包含类方法列表和实例方法列表

## 具体实现

在寻找IMP的地址时，runtime提供了两种方法

```objc
IMP class_getMethodImplementation(Class cls, SEL name);
IMP method_getImplementation(Method m)
```

而根据官方描述，第一种方法可能会更快一些

> > @note \c class_getMethodImplementation may be faster than \c method_getImplementation(class_getInstanceMethod(cls, name)).



## 第1种方法
对于第一种方法而言，类方法和实例方法实际上都是通过调用class_getMethodImplementation()来寻找IMP地址的，不同之处在于传入的第一个参数不同

#### 类方法（假设有一个类A）

```objc
class_getMethodImplementation(objc_getMetaClass("A"),@selector(methodName));
```

#### 实例方法

```objc
class_getMethodImplementation([A class],@selector(methodName));
```

通过该传入的参数不同，找到不同的方法列表，方法列表中保存着下面方法的结构体，结构体中包含这方法的实现，selector本质就是方法的名称，通过该方法名称，即可在结构体中找到相应的实现。

```objc
struct objc_method {
    SEL method_name                                      
    char *method_types                                       
    IMP method_imp                                           
}
```



而对于第二种方法而言，传入的参数只有method，区分类方法和实例方法在于封装method的函数

### 类方法

```objc
Method class_getClassMethod(Class cls, SEL name)
```

### 实例方法

```objc
Method class_getInstanceMethod(Class cls, SEL name)
```

最后调用`IMP method_getImplementation(Method m)` 获取IMP地址

### 实验

![](https://img-blog.csdn.net/20160915111655601)

这里有一个叫Test的类，在初始化方法里，调用了两次getIMPFromSelector:方法，第一个aaa方法是不存在的，test1和test2分别为实例方法和类方法

![](https://img-blog.csdn.net/20160915105003219)

然后我同时实例化了两个Test的对象，打印信息如下

![](https://img-blog.csdn.net/20160915112001680)



大家注意图中红色标注的地址出现了8次：0x1102db280，这个是在调用`class_getMethodImplementation()`方法时，无法找到对应实现时返回的相同的一个地址，无论该方法是在实例方法或类方法，无论是否对一个实例调用该方法，返回的地址都是相同的，但是每次运行该程序时返回的地址并不相同，而对于另一种方法，如果找不到对应的实现，则返回0，在图中我做了蓝色标记。



还有一点有趣的是`class_getClassMethod()`的第一个参数无论传入`objc_getClass()`还是`objc_getMetaClass()`，最终调用`method_getImplementation()`都可以成功的找到类方法的实现。
而`class_getInstanceMethod()`的第一个参数如果传入`objc_getMetaClass()`，再调用`method_getImplementation()`时无法找到实例方法的实现却可以找到类方法的实现。



---



题目：

#### 问1：runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）

答：

类对象中有类方法和实例方法的列表，列表中记录着方法的名称、参数和实现，而selector本质就是方法名称，runtime通过这个方法名称就可以在列表中找到该方法对应的实现。



在结构体`objc_class`中声明了一个指向`struct objc_method_list`指针的指针，可以包含类方法列表和实例方法列表。

在寻找IMP的地址时，runtime提供了两种方法

```objc
IMP class_getMethodImplementation(Class cls, SEL name);
IMP method_getImplementation(Method m)
```
通过该传入的参数不同，找到不同的方法列表，方法列表中保存着下面方法的结构体，结构体中包含这方法的实现，selector本质就是方法的名称，通过该方法名称，即可在结构体中找到相应的实现。

```objc
struct objc_method {
    SEL method_name                                      
    char *method_types                                       
    IMP method_imp                                           
}
```



---

