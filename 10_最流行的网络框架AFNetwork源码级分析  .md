网络可以分为**即时** 和 **非即时** 。

在AFN流行之前，还有一些比较流行的框架，比如`ASIHTTPRequest`、`MKNetwork`



1. `ASIHTTPRequest`是基于CFNetwork框架来写的，这个框架是基于C的。

2. AFNetwork是基于`NSURLConnection`和 `NSURLSession`  来实现的。这2个控件其实是封装了CFNetwork框架。



AFNetwork在3.0之后，移除了`NSURLConnection`，只剩下`NSURLSession`，因为系统已经停止对`NSURLConnection`的更新维护。



`NSURLConnection`既有同步，也有异步。`NSURLSession`没有提供同步实现。



`NSURLSession`提供2种回调方式：block和delegate。block只在乎结果，过程是看不到的。如果需要过程，必须用delegate模式。









