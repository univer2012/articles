来自：[block 捕获外部变量解析](https://blog.csdn.net/li15809284891/article/details/62896569)



---



面试题：block是怎么捕获外部变量的？



----



先看一张全图

![](https://img-blog.csdn.net/20170317180118270?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



在上面的图片中可以看到：

1. block 内部不可以修改自动变量的值，但是加上__block以后就可以；
2. block 内部可以修改对象属性的值，但是不可以修改对象的指向。



接下来会逐个分析。

## 1.全局变量

编译前
![](https://img-blog.csdn.net/20170317180651168?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
编译后
![](https://img-blog.csdn.net/20170317180706092?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以发现编译后是**直接复制的，没有特殊操作。**
原因：全局变量是存放在全局符号表里面的，在整个 app 生命周期都可以访问掉，所以不存在出了作用域会被销毁的问题，因此没有必要做特殊操作。



## 2.全局静态变量

编译前
![](https://img-blog.csdn.net/20170317181006688?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

编译后
![](https://img-blog.csdn.net/20170317181029860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
对比发现，**block 对于全局静态变量的处理和全局变量一样，直接赋值**，原因也同上。



## 3.局部静态变量

编译前
![](https://img-blog.csdn.net/20170317181435720?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



编译后
![](https://img-blog.csdn.net/20170317181501971?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

差别出来了：

1. block 语法**将其地址作为 block 语法的结构体的一部分，然后通过地址赋值，相当于两个函数之间调用了地址传递，所以可以完美修改**。
2. 原因：虽然说静态变量存储在静态变量区域，和程序有一样的生命周期，但是他局限在某个函数内部，所以别的函数访问不到，不像全局变量，函数都可以访问到，所以要通过保存他的地址来修改



## 4.自动变量

编译前
![](https://img-blog.csdn.net/20170317181924082?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



编译后
![](https://img-blog.csdn.net/20170317181942598?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1. 如第一张图所见，block 不能对自动变量的值进行修改，但是可以访问。

2. 不能修改原因：在上图标注的地方可以看到， **block 截获的自动变量被作为一个结构体成员变量放到了 block 语法的结构体中，除了值一样，没有任何联系，相当于两个函数之间的值传递，在一个函数中修改并不会影响另外一个函数中的值**，所以编译器干脆就禁止这样做

   

## 5. `__block` 修饰的自动变量

编译前
![](https://img-blog.csdn.net/20170317182621758?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



编译后
![](https://img-blog.csdn.net/20170317182642086?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过对比发现，加了`__block` 修饰符之后，编译后文件相比于其他的增加了好多，一个个来分析。

1. 对于第一处改变，会发现在 block 语法所对应的结构体中，增加了一个`__block_byref_block_val_0` 类型的结构体指针，对于这个结构体，它的结构如下

   ```c
   struct __Block_byref_block_val_0 {
     void *__isa;//isa 指针，说明他是一个类对象
   __Block_byref_block_val_0 *__forwarding;//指向自身
    int __flags;
    int __size;
    int block_val;//截获的外部变量的值
   };
   ```

2. 看了上面的大概知道了，**block 语法将带有`__block` 修饰符的自动变量包装成了一个对象**，（oc 是通过 isa 指针来区分不同的类的，由上面的 isa 可以知道他是一个对象，他有自己的类型，对于类型在后面会提到），**而对象又是在堆上的，所以就可以知道，`__block` 将其修饰的自动变量从栈上拷贝到了堆上，对于堆上的对象，是可以直接修改它的值的** 。那么它是通过什么方法复制的呢，看第二条

3. 在上面看到了最后两处被标记的增加的地方，分别为

   ```c
    static void __main_block_copy_0();
    static void __main_block_dispose_0();
   ```

   这两个函数被添加到了 `static struct __main_block_desc_0` 的结构体之中，因为在 oc 中是不允许对象作为 c 语言的结构体成员的。因为在 ARC 下，oc 对象的生命周期是通过编译器来管理的，但是对于 c 语言结构体来说，编译器无法分析出其成员的生命周期，所以不能做处理。
   
   幸好 oc 的运行时库能够准确把握 block 从栈上复制到堆上以及堆上的 block 被废弃的时机，于是就通过`static void __main_block_copy_0()` 将其从栈上拷贝到堆上，当 block 被销毁时通过 `static void __main_block_dispose_0()` 函数来销毁。

   

   在赋值时看到一个 `__forwarding` 指针，指向自身，这是为什么呢？
   
   因为当 block 从栈上拷贝到堆上之后，我们需要访问到堆上的对象，这个时候就需要通过栈上的 block 的 `__forwarding` 指针来找到堆上的 block，从而保证不管是在栈上还是堆上，都可以正确访问到 block，栈上的 `__forwarding` 指向堆上的 block，堆上的 block 指向自身。



## 6. block 捕获外部对象

编译前

![](https://img-blog.csdn.net/20170317191006295?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



编译后

![](https://img-blog.csdn.net/20170317191223686?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGkxNTgwOTI4NDg5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



发现**编译后捕获的外部对象，被作为 block 语法的结构体的一个成员**。因为是 ARC 下，所有结构体中的对象的默认修饰符也是  `strong`，同上面的原因。**ARC 下不能将对象作为结构体的成员，因为编译器不能分析其生命周期，所以同样需要上面两个函数来进行 拷贝和释放**。



---

【完】