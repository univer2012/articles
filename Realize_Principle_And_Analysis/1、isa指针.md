![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/isa%E6%8C%87%E9%92%88_1.png)
可以看出: Class 是一个 objc_class 结构类型的指针, id是一个 objc_object 结构类型的指针.
我们再来看看 objc_class 的定义：

![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/isa%E6%8C%87%E9%92%88_2.png)

isa：是一个Class 类型的指针。每个实例对象都有个isa指针,他指向对象的类，而Class里也有个isa的指针, 指向meteClass(元类)。元类保存了类方法的列表。当类方法被调用时，先从本身查找类方法的实现，如果没有，元类会向他父类查找该方法。同时注意的是：元类（meteClass）也是类，它也是对象。元类也有isa指针,它的isa指针最终指向一个根元类(root meteClass)。根元类的isa指针指向本身，这样形成了一个封闭的内循环。

super_class：父类，如果该类已经是最顶层的根类,那么它为NULL。

version：类的版本信息,默认为0

info：供运行期使用的一些位标识。

instance_size：该类的实例变量大小

ivars：成员变量的数组

再来看看各个类实例变量的继承关系：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/isa%E6%8C%87%E9%92%88_3.png)

## 二.runtime 机制
runtime：指一个程序在运行（或者在被执行）的状态。也就是说，当你打开一个程序使它在电脑上运行的时候，那个程序就是处于运行时刻。在一些编程语言中，把某些可以重用的程序或者实例打包或者重建成为“运行库"。这些实例可以在它们运行的时候被连接或者被任何程序调用。

objective-c中runtime：是一套比较底层的纯C语言API, 属于1个C语言库, 包含了很多底层的C语言API。 在我们平时编写的OC代码中, 程序运行过程时, 其实最终都是转成了runtime的C语言代码。
    
runtime的应用：
1. 动态创建一个类(比如KVO的底层实现)
2. 动态地为某个类添加属性\方法,修改属性值\方法
3. 遍历一个类的所有成员变量(属性)\所有方法

实质上，以上的是通过相关方法来获取对象或者类的isa指针来实现的。


#### 相关函数

1.  增加

增加函数:class_addMethod

增加实例变量:class_addIvar

增加属性:@dynamic标签，或者class_addMethod，因为属性其实就是由getter和setter函数组成

增加Protocol:class_addProtocol (说实话我真不知道动态增加一个protocol有什么用,-_-!!)

2.  获取

获取函数列表及每个函数的信息(函数指针、函数名等等):`class_getClassMethod` `method_getName` ...

获取属性列表及每个属性的信息:`class_copyPropertyList` `property_getName`

获取类本身的信息,如类名等：`class_getName class_getInstanceSize`

获取变量列表及变量信息：`class_copyIvarList`

获取变量的值

3.    替换

将实例替换成另一个类：`object_setClass`

替换类方法的定义：`class_replaceMethod`

4.其他常用方法：

交换两个方法的实现：`method_exchangeImplementations`.

设置一个方法的实现：`method_setImplementation`.