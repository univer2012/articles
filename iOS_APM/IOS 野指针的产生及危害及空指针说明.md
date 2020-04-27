参考：
1. [IOS 野指针的产生及危害 及空指针说明](http://blog.sina.com.cn/s/blog_945590aa0102vnun.html)


# 一、什么是空指针和野指针
## 1.空指针
1> 没有存储任何内存地址的指针就称为**空指针(NULL指针)**。

2> 空指针就是被赋值为nil的指针，在没有被具体初始化之前，为nil。

## 2.野指针
**"野指针"不是nil指针，是指向"垃圾"内存（不可用内存）的指针。野指针是非常危险的。**

例如：
```
//MRC状态
Student *stu = [[Student alloc] init];
[stu setAge:10];
[stu release];  //这里已经释放内存
[stu setAge:10];  //---》报错
```

### 3.接下来分析一下报错原因

1) 执行完第1行代码后，内存中有个指针变量stu，指向了Student对象
```
Student *stu = [[Student alloc] init];
```
![野指针的产生及危害及空指针说明1](https://upload-images.jianshu.io/upload_images/843214-00d4f55591d04e9d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假设Student对象的地址为0xff43，指针变量stu的地址为0xee45，stu中存储的是Student对象的地址0xff43。即指针变量stu指向了这个Student对象。

2)接下来是第3行代码
```
[stu setAge:10];
```

这行代码的意思是：给stu所指向的Student对象发送一条setAge:消息，即调用这个Student对象的setAge:方法。目前来说，这个Student对象仍存在于内存中，所以这句代码没有任何问题。

3) 接下来是第5行代码
```
[stu release];
```

 这行代码的意思是：给stu指向的Student对象发送一条release消息。在这里，Student对象接收到release消息后，会马上被销毁，所占用的内存会被回收。

(release的具体用法会放到OC内存管理中详细讨论)
![野指针的产生及危害及空指针说明2](https://upload-images.jianshu.io/upload_images/843214-dc4b5e7fa5c8f630.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Student对象被销毁了，地址为0xff43的内存就变成了"垃圾内存"，然而，指针变量stu仍然指向这一块内存，这时候，stu就称为了野指针！

4)最后执行了第7行代码
```
[stu setAge:10];
```
这句代码的意思仍然是： 给stu所指向的Student对象发送一条setAge:消息。但是在执行完第5行代码后，Student对象已经被销毁了，它所占用的内存已经是垃圾内存，如果你还去访问这一块内存，那就会报野指针错误。这块内存已经不可用了，也不属于你了，你还去访问它，肯定是不合法的。所以，这行代码报错了！


### 4.如果改动一下代码，就不会报错

```
//MRC状态
Student *stu = [[Student alloc] init];
[stu setAge:10];
[stu release];  
stu = nil;
[stu setAge:10]; 
```
注意第7行代码，stu变成了空指针，stu就不再指向任何内存了

![野指针的产生及危害及空指针说明3](https://upload-images.jianshu.io/upload_images/843214-20be45e2f0ef234a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为stu是个空指针，没有指向任何对象，因此第9行的setAge:消息是发不出去的，不会造成任何影响。当然，肯定也不会报错。

### 5.总结

1)利用野指针发消息是很危险的，会报错。也就是说，如果一个对象已经被回收了，就不要再去操作它，不要再尝试给它发消息。

2)**利用空指针发消息是没有任何问题的**，也就是说下面的代码是没有错误的：
```
 [nil setAge:10]; 
```

