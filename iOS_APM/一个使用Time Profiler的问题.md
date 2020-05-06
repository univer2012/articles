项目经理反馈了个app启动后操作卡顿的问题，首先想到的是主线程卡住了，于是就想用`Instruments`的`Time Profiler`来查看下哪个方法把主线程卡住了。

真机安装上app后，`Time Profiler`中开始记录，总是如下框：
![pic1.png](https://upload-images.jianshu.io/upload_images/843214-091f64ae04c6b76c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

百思不得其解。于是百度一下。找到了[这篇博客](http://blog.csdn.net/reims2046/article/details/40857601)，说是在`Edit Scheme...`的`Profile`中，`Build Configuration`没有设置成`Debug`。如图：
![pic2.png](https://upload-images.jianshu.io/upload_images/843214-6b9c402337482aec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

后来按照如上设置，真机果然可以记录调试了。

