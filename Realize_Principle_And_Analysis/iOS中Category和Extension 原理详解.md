来自：[iOS中Category和Extension 原理详解](http://www.cocoachina.com/articles/19163)



#### 问1：为什么category不能添加成员变量？



Objective-C类是由`Class`类型来表示的，它实际上是一个指向`objc_class`结构体的指针。它的定义如下：

```objc
typedef struct objc_class *Class;
```

在`objc_class`结构体中，`ivars`是`objc_ivar_list`（成员变量列表）指针；`methodLists`是指向`objc_method_list`指针的指针。<font color=#FF0000>**在Runtime中，`objc_class`结构体大小是固定的，不可能往这个结构体中添加数据，只能修改。所以，`ivars`指向的是一个固定区域，只能修改成员变量值，不能增加成员变量个数**</font>。**`methodLists`是一个二维数组，可以修改`*methodLists`的值来增加成员方法。虽没办法扩展<u>methodLists指向的内存区域</u>，却可以改变<u>这个内存区域的值</u>（存储的是指针）**。因此，可以动态添加方法，不能添加成员变量。



#### 问2：category中能添加属性吗？

要从Category的结构体开始分析：

```objc
typedef struct category_t {
    const char *name;  //类的名字
    classref_t cls;  //类
    struct method_list_t *instanceMethods;  //category中所有给类添加的实例方法的列表
    struct method_list_t *classMethods;  //category中所有添加的类方法的列表
    struct protocol_list_t *protocols;  //category实现的所有协议的列表
    struct property_list_t *instanceProperties;  //category中添加的所有属性
} category_t;
```

从Category的定义也可以看出Category的可为（可以添加实例方法，类方法，甚至可以实现协议，添加属性）和不可为（无法添加实例变量）。

但是为什么网上很多人都说Category不能添加属性呢？

**Category实际上是允许添加属性的，同样可以使用`@property`。但是不会生成`_变量`（带下划线的成员变量），也不会生成添加属性的getter和setter方法的实现**。所以，<u>尽管添加了属性，也无法使用点语法调用getter和setter方法</u>（实际上，点语法是可以写的，只不过在运行时调用到这个方法时候会报方法找不到的错误，如下图）。但<font color=#038103>**实际上可以使用runtime去实现Category，为已有的类添加新的属性，并生成getter和setter方法**</font>。详细内容可以看峰哥之前的文章：[《iOS Runtime之四：关联对象》](http://www.imlifengfeng.com/blog/?p=397)



---



#（一）Category

### 1、什么是Category?**

category是Objective-C 2.0之后添加的语言特性，别人口中的分类、类别其实都是指的category。category的主要作用是为已经存在的类添加方法。除此之外，apple还推荐了category的另外两个使用场景。

可以把类的实现分开在几个不同的文件里面。这样做有几个显而易见的好处。

- 可以减少单个文件的体积
- 可以把不同的功能组织到不同的category里
- 可以由多个开发者共同完成一个类
- 可以按需加载想要的category
- 声明私有方法

apple 的SDK中就大面积的使用了category这一特性。比如UIKit中的UIView。apple把不同的功能API进行了分类，这些分类包括UIViewGeometry、UIViewHierarchy、UIViewRendering等。

不过除了apple推荐的使用场景，广大开发者脑洞大开，还衍生出了category的其他几个使用场景：

- 模拟多继承（另外可以模拟多继承的还有protocol）
- 把framework的私有方法公开

### 2、category特点

- category只能给某个已有的类扩充方法，不能扩充成员变量。
- **category中也可以添加属性，只不过`@property`只会生成setter和getter的声明，不会生成setter和getter的实现以及成员变量**。
- **如果category中的方法和类中原有方法同名，运行时会优先调用category中的方法**。也就是，category中的方法会覆盖掉类中原有的方法。所以开发中尽量保证不要让分类中的方法和原有类中的方法名相同。避免出现这种情况的解决方案是给分类的方法名统一添加前缀。比如category_。
- <font color=#038103>**如果多个category中存在同名的方法，运行时到底调用哪个方法由编译器决定，最后一个参与编译的方法会被调用**</font>。

如下图，给UIView添加了两个category（one 和 two），并且给这两个分类都添加了名为log的方法：

![1055199-1299217b7107f5e2.png](http://cc.cocimg.com/api/uploads/20170502/1493710588468886.png)

UIView+one

![1055199-5695720d7e83e504.png](http://cc.cocimg.com/api/uploads/20170502/1493710606382484.png)

UIView+two

在viewController中引入这两个category的.h文件。调用log方法：

![1055199-7688fb9db151b48b.png](http://cc.cocimg.com/api/uploads/20170502/1493710623915660.png)

调用category方法

当编译顺序如下图所示时，调用UIView + one.m的log方法，如下图：

![1055199-696fa0a7658c380e.png](http://cc.cocimg.com/api/uploads/20170502/1493710634767648.png)

编译顺序

![1055199-0b47ed86d61ff013.png](http://cc.cocimg.com/api/uploads/20170502/1493710651182373.png)

调用结果

将UIView + one.m移动到UIView + two.m上面，调用UIView + two.m的log方法。如下图：

![1055199-fd9f993454fd7d1f.png](http://cc.cocimg.com/api/uploads/20170502/1493710666613075.png)

编译顺序

![1055199-b8acd265f23c7fe9.png](http://cc.cocimg.com/api/uploads/20170502/1493710676233152.png)

调用结果

### 3、调用优先级

**分类(category) > 本类 > 父类**。即，优先调用cateory中的方法，然后调用本类方法，最后调用父类方法。

注意：category是在运行时加载的，不是在编译时。

### 4、为什么category不能添加成员变量？

Objective-C类是由Class类型来表示的，它实际上是一个指向objc_class结构体的指针。它的定义如下：

```objc
typedef struct objc_class *Class;
```

objc_class结构体的定义如下：

```objc
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                       OBJC2_UNAVAILABLE;  // 父类
    const char *name                        OBJC2_UNAVAILABLE;  // 类名
    long version                            OBJC2_UNAVAILABLE;  // 类的版本信息，默认为0
    long info                               OBJC2_UNAVAILABLE;  // 类信息，供运行期使用的一些位标识
    long instance_size                      OBJC2_UNAVAILABLE;  // 该类的实例变量大小
    struct objc_ivar_list *ivars            OBJC2_UNAVAILABLE;  // 该类的成员变量链表
    struct objc_method_list **methodLists   OBJC2_UNAVAILABLE;  // 方法定义的链表
    struct objc_cache *cache                OBJC2_UNAVAILABLE;  // 方法缓存
    struct objc_protocol_list *protocols    OBJC2_UNAVAILABLE;  // 协议链表
#endif
} OBJC2_UNAVAILABLE;
```

在上面的objc_class结构体中，`ivars`是`objc_ivar_list`（成员变量列表）指针；`methodLists`是指向`objc_method_list`指针的指针。<font color=#FF0000>**在Runtime中，`objc_class`结构体大小是固定的，不可能往这个结构体中添加数据，只能修改。所以，`ivars`指向的是一个固定区域，只能修改成员变量值，不能增加成员变量个数**</font>。**`methodLists`是一个二维数组，所以可以修改`*methodLists`的值来增加成员方法。虽没办法扩展methodLists指向的内存区域，却可以改变这个内存区域的值（存储的是指针）**。因此，可以动态添加方法，不能添加成员变量。

### 5、category中能添加属性吗？

Category不能添加成员变量（instance variables），那到底能不能添加属性（property）呢？

这个我们要从Category的结构体开始分析：

```objc
typedef struct category_t {
    const char *name;  //类的名字
    classref_t cls;  //类
    struct method_list_t *instanceMethods;  //category中所有给类添加的实例方法的列表
    struct method_list_t *classMethods;  //category中所有添加的类方法的列表
    struct protocol_list_t *protocols;  //category实现的所有协议的列表
    struct property_list_t *instanceProperties;  //category中添加的所有属性
} category_t;
```

从Category的定义也可以看出Category的可为（可以添加实例方法，类方法，甚至可以实现协议，添加属性）和不可为（无法添加实例变量）。

但是为什么网上很多人都说Category不能添加属性呢？

**Category实际上是允许添加属性的，同样可以使用`@property`，但是不会生成`_变量`（带下划线的成员变量），也不会生成添加属性的getter和setter方法的实现**。所以，<u>尽管添加了属性，也无法使用点语法调用getter和setter方法</u>（实际上，点语法是可以写的，只不过在运行时调用到这个方法时候会报方法找不到的错误，如下图）。但<font color=#038103>**实际上可以使用runtime去实现Category，为已有的类添加新的属性，并生成getter和setter方法**</font>。详细内容可以看峰哥之前的文章：[《iOS Runtime之四：关联对象》](http://www.imlifengfeng.com/blog/?p=397)

![1055199-f1b6fd566ad50692.png](http://cc.cocimg.com/api/uploads/20170502/1493710865826134.png)

给category添加property

![1055199-e47b578db44fd663.png](http://cc.cocimg.com/api/uploads/20170502/1493710878713669.png)

调用category中property的setter（报方法找不到的错误）

![1055199-d9d250fd1691d24f.png](http://cc.cocimg.com/api/uploads/20170502/1493710885730399.png)![1055199-d9d250fd1691d24f.png](http://cc.cocimg.com/api/uploads/20170502/1493710891419990.png)

调用category中property的getter（报方法找不到的错误）

需要注意的有两点：

- 1)、category的方法没有“完全替换掉”原来类已经有的方法，也就是说如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA。
- 2)、**category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这就是我们平常所说的，category的方法会“覆盖”掉原来类的同名方法。这是因为，运行时在查找方法的时候，是顺着<u>方法列表的顺序</u>查找的。它只要一找到对应名字的方法，就会罢休，殊不知后面可能还有一样名字的方法。**

# （二）Extension

### 1、 什么是extension

extension被开发者称之为扩展、延展、匿名分类。extension看起来很像一个匿名的category，但是extension和category几乎完全是两个东西。和category不同的是，**extension不但可以声明方法，还可以声明属性、成员变量。extension一般用于声明私有方法，私有属性，私有成员变量**。

### 2、 extension的存在形式

**category是拥有.h文件和.m文件的东西**。但是extension不然。**extension只存在于一个.h文件中，或者extension只能寄生于一个类的.m文件中**。比如，viewController.m文件中通常寄生这么个东西，其实这就是一个extension：

```objc
@interface ViewController ()
 
@end
```

当然我们也可以创建一个单独的extension文件，如下图：

![1055199-56cba3e296a4bf59.png](http://cc.cocimg.com/api/uploads/20170502/1493711045659045.png)

![1055199-d6e6a0b1e95e4e80.png](http://cc.cocimg.com/api/uploads/20170502/1493711059565240.png)



![1055199-753211c1b4ca69d6.png](http://cc.cocimg.com/api/uploads/20170502/1493711070288177.png)

UIView_extension.h中声明方法：

![1055199-fe05e6b5e89e4d15.png](http://cc.cocimg.com/api/uploads/20170502/1493711093398852.png)

导入UIView_extension.h文件进行使用：

![1055199-a392389b341fc944.png](http://cc.cocimg.com/api/uploads/20170502/1493711118866208.png)

注意：extension常用的形式并不是以一个单独的.h文件存在，而是寄生在类的.m文件中。

# （三）category和extension的区别

就category和extension的区别来看，我们可以推导出一个明显的事实，extension可以添加实例变量，而category是无法添加实例变量的（因为在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这对编译型语言来说是灾难性的）。

1. **extension在编译期决议，它就是类的一部分**，但是**category**则完全不一样，它是**在运行期决议**的。extension在编译期和头文件里的`@interface`以及实现文件里的`@implement`一起形成一个完整的类。extension伴随类的产生而产生，也随该类一起消亡。
2. extension一般用来隐藏类的私有信息，**你必须有一个类的源码才能为一个类添加extension，所以你无法为系统类比如`NSString`添加extension，除非创建子类再添加extension**。而category不需要有类的源码，我们可以给系统提供的类添加category。
3. **extension可以添加实例变量，而category不可以**。
4. **extension和category都可以添加属性，但是category的属性不能生成成员变量和getter、setter方法的实现**。





参考文章

[iOS Category详解](http://www.imlifengfeng.com/blog/?p=474)

[Objective-C的Category与关联对象实现原理](http://lib.csdn.net/article/ios/44323)