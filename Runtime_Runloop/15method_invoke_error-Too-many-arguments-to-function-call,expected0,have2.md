新建一个工程，输入`method_invoke(receiver,method);`时报错：

[![img](https://univer2012.github.io/2017/05/15/15method-invoke-error-Too-many-arguments-to-function-call-expected0-have2/pic1.png)](https://univer2012.github.io/2017/05/15/15method-invoke-error-Too-many-arguments-to-function-call-expected0-have2/pic1.png)
百度之，找到来原因：

> 原来在 LLVM 6.0 中增加了一个 `OBJC_OLD_DISPATCH_PROTOTYPES`，默认配置在 Apple LLVM 6.0 - Preprocessing 中的`Enable Strict Checking of objc_msgSend Calls`中为`Yes`，所以就会出这个错误了。

在`Build Settings`中找到`Enable Strict Checking of objc_msgSend Calls`，设置为`No`后编译，就不会报错了。

参考自：[id objc_msgSend(id self, SEL op, …)build error](http://blog.sina.com.cn/s/blog_9075354e0102v2kc.html)