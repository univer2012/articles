随着iOS10发布的临近,大家的App都需要适配iOS10,下面是我总结的一些关于iOS10适配方面的问题,如果有错误,欢迎指出.

### 一、系统判断方法失效

来到iOS10时代后，很多文章都说原来的系统判断方法不能用了。自己一直没注意过这个问题，所以看到这部分的时候，还是一脸懵逼。

不过看到过一篇文章，说`[[[UIDevice currentDevice] systemVersion] floatValue]`是有问题的，具体在我的[开发中的一些小技巧](https://univer2012.github.io/2017/05/11/9some-tips-in-development/)有说明。但是想不到还有很多实现方法，下面为大家一一介绍：

方案一：

```objc
//等于
#define SYSTEM_VERSION_EQUAL_TO(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedSame)
//大于
#define SYSTEM_VERSION_GREATER_THAN(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedDescending)
//大于等于
#define SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)
//小于
#define SYSTEM_VERSION_LESS_THAN(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)
//小于等于
#define SYSTEM_VERSION_LESS_THAN_OR_EQUAL_TO(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedDescending)
//... ...
if (SYSTEM_VERSION_EQUAL_TO(@"10.3")) {
    NSLog(@"当前系统是iOS 10.3");
}
if (SYSTEM_VERSION_GREATER_THAN(@"10.3")) {
    NSLog(@"当前系统大于iOS 10.3");
}
if (SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(@"10.3")) {
    NSLog(@"当前系统大于或等于iOS 10.3");
}
if (SYSTEM_VERSION_LESS_THAN(@"10.3")) {
    NSLog(@"当前系统小于iOS 10.3");
}
if (SYSTEM_VERSION_LESS_THAN_OR_EQUAL_TO(@"10.3")) {
    NSLog(@"当前系统小于或等于iOS 10.3");
}
```

方案二：

```objc
if ([[NSProcessInfo processInfo] isOperatingSystemAtLeastVersion:(NSOperatingSystemVersion){.majorVersion = 9, .minorVersion = 1, .patchVersion = 0}]) {
    NSLog(@"Hello from > iOS 9.1");
}
if ([NSProcessInfo.processInfo isOperatingSystemAtLeastVersion:(NSOperatingSystemVersion){9,3,0}]) { NSLog(@"Hello from > iOS 9.3");
}
```

方案三：

```objc
if (NSFoundationVersionNumber > NSFoundationVersionNumber_iOS_8_4) {
    // do stuff for iOS 9 and newer
    NSLog(@"当前系统大于iOS 8.4");
}
else {
    NSLog(@"当前系统为小于或登录iOS 8.4");
    // do stuff for older versions than iOS 9
}
```

对于方案三：

有时候会缺少一些常量,`NSFoundationVersionNumber`是在`NSObjCRuntime.h`中定义的,作为Xcode7.3.1的一部分,我们设定常熟范围从iPhone OS 2到`#define NSFoundationVersionNumber_iOS_8_4 1144.17`,在iOS 10(Xcode 8)中,苹果补充了缺少的数字,设置有未来的版本.

```objc
#define NSFoundationVersionNumber_iOS_8_x_Max 1199
#define NSFoundationVersionNumber_iOS_9_0 1240.1
#define NSFoundationVersionNumber_iOS_9_1 1241.14
#define NSFoundationVersionNumber_iOS_9_2 1242.12
#define NSFoundationVersionNumber_iOS_9_3 1242.12
#define NSFoundationVersionNumber_iOS_9_4 1280.25
#define NSFoundationVersionNumber_iOS_9_x_Max 1299
```

在Swift中，可以这样写：

```swift

```

阅读文章：

[iOS 10 的适配问题](http://www.jianshu.com/p/f8151d556930)