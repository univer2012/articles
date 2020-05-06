来自：[iOS 全局禁止横屏，但UIWebView 全屏播放视频，横屏，解决办法](http://www.cnblogs.com/fengtengfei/p/4646562.html)

`UIWebview`在播放网页视频的时候我们需要进行**是否全屏状态**的监听。
一般的需求是在播放视频时候需要横屏，退出全屏的时候不能横屏，但是`UIWebview`没有给出响应的方法，

本问题解决的Demo工程https://github.com/darren90/iOS_Demo/tree/master/02-UIWebview

## 1、其他界面不支持横屏：

这个比较容易，我的思路是：

### 1.1 在`APPDelegate.h`文件中增加属性：是否支持横屏

```
/***  是否允许横屏的标记 */
@property (nonatomic,assign)BOOL allowRotation;
```

### 1.2 在`APPDelegate.m`文件中增加方法，控制全部不支持横屏

```
- (NSUInteger)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
    if (self.allowRotation) {
        return UIInterfaceOrientationMaskAll;
    }
    return UIInterfaceOrientationMaskPortrait;
}
```

这样在其他界面想要横屏的时候，我们只要控制`allowRotation`这个属性就可以控制其他界面进行横屏了。

```
//需在上面#import "AppDelegate.h"
AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
appDelegate.allowRotation = YES;
//不让横屏的时候 appDelegate.allowRotation = NO;即可
```



## 2 播放界面横屏

### 2.1 播放界面横屏：网上的解决方案

网上有人给出两种方法：

第一：但是iOS8下失效，没有用：

```
[[NSNotificationCenter defaultCenter] addObserver:self
selector:@selector(videoStarted:)
name:@"UIMoviePlayerControllerDidEnterFullscreenNotification"
object:nil];//
```



第二：通过js，但是没有找到详细的，可解决的方案。

所以也就是没有找到成熟的方案，所以就自己分析了。

### 2.2 播放界面横屏：问题分析

方法总会有的，我们要向监听`UIWebView`视频播放时候是否全屏，也就是我们要能拿到播放视频的view或者是viewcontroller，但是由于`UIWebView`没有比较直观的方法，所以只能从其他地方下手了

通过Reveal我们可以查看到view的一些层级关系，可以看出弹出播放的是AVPlayerView，在UIWindow上：
[![img](https://univer2012.github.io/2017/12/14/43iOS全局禁止横屏-但UIWebView全屏横屏播放视频的解决方法/140047380322614.png)](https://univer2012.github.io/2017/12/14/43iOS全局禁止横屏-但UIWebView全屏横屏播放视频的解决方法/140047380322614.png)

使用Xcode中的`Debug View Hierarchy`调试，看到的view的层级关系如下：
[![img](https://univer2012.github.io/2017/12/14/43iOS全局禁止横屏-但UIWebView全屏横屏播放视频的解决方法/QQ20171214-150133.png)](https://univer2012.github.io/2017/12/14/43iOS全局禁止横屏-但UIWebView全屏横屏播放视频的解决方法/QQ20171214-150133.png)

好了，问题到这已经很明晰了，我们要么能拿到`AVPlayerView`，要么拿到`UIWindow`才能控制播放界面，分析后发现`AVPlayerView`不好拿，但是`UIWindow`就很easy了。

### 2.3 播放界面横屏：问题解决

所以这里可以使用UIWindow的通知，就可以解决

```
//在加载`UIWebview`的viewController中
- (void)viewDidLoad {
    [super viewDidLoad];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(beginFullScreen) name:UIWindowDidBecomeVisibleNotification object:nil];//进入全屏
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(endFullScreen) name:UIWindowDidBecomeHiddenNotification object:nil];//退出全屏
}//sudo spctl --master-poster
```



在退出全屏时，增加逻辑让其强制编程竖屏，这样当全屏播放的时候，点击down(“完成”)时，就会自动变成竖屏了。

```
// 进入全屏
-(void)begainFullScreen
{
    AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
    appDelegate.allowRotation = YES;
}
// 退出全屏
-(void)endFullScreen
{
    AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
    appDelegate.allowRotation = NO;
    [[UIApplication sharedApplication] setStatusBarHidden:NO];
}
```



实例Demo工程https://github.com/darren90/iOS_Demo/tree/master/02-UIWebview