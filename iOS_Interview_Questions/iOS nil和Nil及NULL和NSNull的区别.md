来自：[iOS nil和Nil及NULL和NSNull的区别](https://www.jianshu.com/p/8a40657c6085)



---



1. `nil` 是空对象。对它做 `retain` 操作会引起程序崩溃，例如 **字典添加键值** 或 **数组添加新元素** 等。

2. NSNull是 <font color=#038103>**“值为空的对象”**</font> 。如果你查阅开发文档你会发现 `NSNull` 这个类是继承NSObject，并且只有一个 `+ (NSNull *) null;` 类方法。这就说明 `NSNull` 对象**拥有一个有效的内存地址**，所以在程序中对它的任何引用都是不会导致程序崩溃的。
3. `nil` 和 `Nil` 在使用上是没有严格限定的，也就是说凡是使用nil的地方都可以用 `Nil` 来代替，反之亦然。只不过从编程人员的规约中，我们约定俗成地将 **`nil` 表示一个空对象**，**`Nil` 表示一个空类**。
4.  **`NULL` 就是典型C语言的语法，它表示一个空指针** 。

---



# 一、nil

我们给对象赋值时一般会使用object = nil，表示我想把这个对象释放掉；
 或者对象由于某种原因，经过多次release，于是对象引用计数器为0了，系统将这块内存释放掉，这个时候这个对象为nil，我称它为 <font color=#038103>“**空对象**”</font>。（注意：我这里强调的是“空对象”，下面我会拿它和“值为空的对象”作对比！！！）

所以对于这种空对象，所有关于retain的操作都会引起程序崩溃，例如 **字典添加键值** 或 **数组添加新元素** 等，具体可参考如下代码：



![img](https:////upload-images.jianshu.io/upload_images/3334822-3037205cb81bc127.png?imageMogr2/auto-orient/strip|imageView2/2/w/490)

image.png

# 二、NSNull

NSNull和nil的区别在于，nil是一个空对象，已经完全从内存中消失了，而如果我们想表达“我们需要有这样一个容器，但这个容器里什么也没有”的观念时，我们就用到`NSNull` ，我称它为 <font color=#038103>**“值为空的对象”**</font> 。如果你查阅开发文档你会发现NSNull这个类是继承NSObject，并且只有一个 `+ (NSNull *) null;` 类方法。这就说明NSNull对象拥有一个有效的内存地址，所以在程序中对它的任何引用都是不会导致程序崩溃的。参考代码如下：



![img](https:////upload-images.jianshu.io/upload_images/3334822-b065231092ca93f9.png?imageMogr2/auto-orient/strip|imageView2/2/w/490)

image.png

# 三、Nil

nil和Nil在使用上是没有严格限定的，也就是说凡是使用nil的地方都可以用Nil来代替，反之亦然。只不过从编程人员的规约中，我们约定俗成地将 **`nil` 表示一个空对象**，**`Nil` 表示一个空类**。参考代码如下：



![img](https:////upload-images.jianshu.io/upload_images/3334822-a2d2c232a0a0de3a.png?imageMogr2/auto-orient/strip|imageView2/2/w/490)

image.png

# 四、NULL

我们知道Object-C来源于C、支持于C，当然也有别于C。而 **`NULL` 就是典型C语言的语法，它表示一个空指针** 。参考代码如下：

```c
int *ponit = NULL;
```



---

【完】