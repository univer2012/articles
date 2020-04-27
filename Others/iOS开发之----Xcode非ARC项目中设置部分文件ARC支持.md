来自：[IOS开发之----Xcode非ARC项目中设置部分文件ARC支持](http://blog.csdn.net/jeffasd/article/details/50059247)

# 1 ARC是什么
**ARC是iOS 5推出的新功能，全称叫 ARC(Automatic Reference Counting)**。简单地说，就是代码中自动加入了retain/release，原先需要手动添加的用来处理内存管理的引用计数的代码可以自动地由编译器完成了。该机制在 iOS 5/ Mac OS X 10.7 开始导入，利用 **Xcode4.2 可以使用该机制**。简单地理解ARC，就是通过指定的语法，让编译器(LLVM 3.0)在编译代码时，自动生成实例的引用计数管理部分代码。有一点，**ARC并不是GC，它只是一种代码静态分析（Static Analyzer）工具。**

# 2 非ARC项目中设置部分文件ARC支持

 那么在xCode中经常需要导入一些外来的代码文件，如果导入的文件使用了ARC机制而你的当前项目没有使用ARC，那么xCode会给出警告，或者报错。我们该如何处理这些问题呢：

点击项目导航文件--> 选中Targets--> 选择 Build Phases --> 展开Compile Sources ：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Xcode%E9%9D%9EARC%E9%A1%B9%E7%9B%AE%E4%B8%AD%E8%AE%BE%E7%BD%AE%E9%83%A8%E5%88%86%E6%96%87%E4%BB%B6ARC%E6%94%AF%E6%8C%81_1.png)

这个时候，我们看到第二列的名称为：`Compiler Flags`。
 双击你所要使用ARC的文件，并输入 `-fobjc-arc`：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Xcode%E9%9D%9EARC%E9%A1%B9%E7%9B%AE%E4%B8%AD%E8%AE%BE%E7%BD%AE%E9%83%A8%E5%88%86%E6%96%87%E4%BB%B6ARC%E6%94%AF%E6%8C%81_2.png)

那么现在这个文件就可以在编译时使用ARC机制进行编译了。

 同上，如果想让使用ARC机制的代码不使用ARC机制，只需要输入 `-fno-objc-arc`。