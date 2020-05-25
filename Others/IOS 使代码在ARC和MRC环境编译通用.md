来自：[IOS 使代码在ARC和MRC环境编译通用](https://blog.csdn.net/weixin_30279751/article/details/97799105)



---



虽然现在使用MRC的项目已经很少了，但是如果维护一些老的项目还是需要使用MRC的。 
 但是如果一个项目的编译环境会在ARC和MRC来回切换的话会让程序员很头疼， 
 比如一些变量的release操作，retain操作在两种环境切换时会报错，如果手动修改的话会很麻烦。 
 但是有没有什么解决办法能使你的代码在两种环境下都好用呢？

答案当然是有的。

很简单，见下面代码：

```objectivec
#if __has_feature(objc_arc)
    // 下面写ARC代码
#else
    // 下面写MRC代码
    [tool1 release];
#endif
```

ARC和MRC要执行的代码写在相应的位置就好。



---



【完】