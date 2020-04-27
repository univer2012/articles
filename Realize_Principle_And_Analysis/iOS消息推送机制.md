参考：
1.[iOS消息推送机制](https://www.cnblogs.com/luoxiaofu/p/8574096.html)

推送通知跟NSNotification不同
1. NSNotification是抽象的,不可见的
2. 推送通知是可见的

### iOS中提供了2中推送通知
1. 本地推送通知(Local Notification)
2. 远程推送通知(Remote Notification)

推送的作用:可以让不在前台运行的app,告知客户app内部发生的事情.(QQ消息推送,微信消息推送等等)

推送通知的呈现效果:

1. 在屏幕顶部显示的一条横幅
2. 在屏幕中间弹出一个UIAlertView
3. 在锁屏界面显示一块横幅
4. 跟新app图标的数字
5. 播放音效

# 本地通知

1. 不需要服务器支持(无需联网)就能发出的推送通知

2. 使用场景: 定时类任务(闹钟,简单的游戏等等)

本地通知推送的实现很简单:
### 1. 创建本地推送通知对象

`[[UILocalNotification alloc] init]`创建一个本地通知

### 2.设置本地通知的相关属性

必须设置的属性

```
//2.1. 推送通知的触发时间(何时发出推送通知)
@property(nonatomic,copy) NSDate *fireDate

//2.2.推送通知的具体内容
@property(nonatomic,copy) NSString *alertBody

//2.3.在锁屏时显示的动作标题(完整测标题:"滑动来" + alertAction)
@property(nonatomic,copy) NSString *alertAction

//2.4.设置锁屏界面alertAction是否有效
localNote.hasAction = YES;

//2.5.app图标数字
@property(nonatomic,assign) NSInteger applicationIconBadgeNumber

//2.6.调度本地推送通知(调度完毕后,推动通知会在特定时间fireDate发出)
[[UIApplication shareApplication] scheduleLocalNotification:ln]
可以进行设置的设置

//2.7.设置通知中心通知的标题
localNote.alertTitle = @"222222222222";

//2.8.设置音效(如果不设置就是系统默认的音效, 设置的话会在mainBundle中查找)
localNote.soundName = @"buyao.wav";

//2.9.每隔多久重复发一次推送通知
@property(nonatomic) NSCalendarUnit repeatInterval

//2.10.点击推送通知打开app时显示的启动图片(mainBundle 中提取图片)
@property(nonatomic,copy) NSSring *alertLaunchImage

//2.11.附加的额外信息
@property(nonatomic,copy) NSDictionary *userInfo

//2.12.时区
@property(nonatomic,copy) NSTimeZone *timeZone
(一般设置为[NSTimeZone defaultTimeZone],跟随手机的时区)
```
![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%9C%BA%E5%88%B6_1.gif)

--代码实现过程:

```
/*
 @property(nonatomic,copy) NSDate *fireDate;
 @property(nonatomic,copy) NSTimeZone *timeZone; 时区
 
 @property(nonatomic) NSCalendarUnit repeatInterval; 重复间隔(枚举)
 @property(nonatomic,copy) NSCalendar *repeatCalendar; 重复日期(NSCalendar)
 
 @property(nonatomic,copy) CLRegion *region 设置区域(设置当进入某一个区域时,发出一个通知)
 
 @property(nonatomic,assign) BOOL regionTriggersOnce YES,只会在第一次进入某一个区域时发出通知.NO,每次进入该区域都会发通知
 
 @property(nonatomic,copy) NSString *alertBody;      
 
 @property(nonatomic) BOOL hasAction;                是否隐藏锁屏界面设置的alertAction
 @property(nonatomic,copy) NSString *alertAction;    设置锁屏界面一个文字
 
 @property(nonatomic,copy) NSString *alertLaunchImage;   启动图片
 @property(nonatomic,copy) NSString *alertTitle
 
 @property(nonatomic,copy) NSString *soundName;
 
 @property(nonatomic) NSInteger applicationIconBadgeNumber;
 
 @property(nonatomic,copy) NSDictionary *userInfo; // 设置通知的额外的数据
 */

- (IBAction)addLocalNote:(id)sender {
    // 1.创建一个本地通知
    UILocalNotification *localNote = [[UILocalNotification alloc] init];
    
    // 2.设置本地通知的一些属性(通知发出的时间/通知的内容)
    // 2.1.设置通知发出的时间
    localNote.fireDate = [NSDate dateWithTimeIntervalSinceNow:5.0];
    // 2.2.设置通知的内容
    localNote.alertBody = @"吃饭了吗?";
    // 2.3.设置锁屏界面的文字
    localNote.alertAction = @"查看具体的消息";
    // 2.4.设置锁屏界面alertAction是否有效
    localNote.hasAction = YES;
    // 2.5.设置通过点击通知打开APP的时候的启动图片(无论字符串设置成什么内容,都是显示应用程序的启动图片)
    localNote.alertLaunchImage = @"111";
    // 2.6.设置通知中心通知的标题
    localNote.alertTitle = @"222222222222";
    // 2.7.设置音效
    localNote.soundName = @"buyao.wav";
    // 2.8.设置应用程序图标右上角的数字
    localNote.applicationIconBadgeNumber = 1;
    // 2.9.设置通知之后的属性
    localNote.userInfo = @{@"name" : @"张三", @"toName" : @"李四"};
    
    // 3.调度通知
    [[UIApplication sharedApplication] scheduleLocalNotification:localNote];
}
```
当消息被推送过来时,我们需要点击推送消息,来完成一些特定的任务.例如更新界面什么的(监听本地推送通知的点击)

当用户点击本地推送通知的时候,会自动打开app,这里有2种情况

#### 1.app没有关闭,只是一直隐藏在后台

让app进入前台,并会调用AppDelegate的下面的方法(并非重新启动app)

![2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%9C%BA%E5%88%B6_2.gif)

----代码实现
```
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
{
    // 在这里写跳转代码
    // 如果是应用程序在前台,依然会收到通知,但是收到通知之后不应该跳转
    if (application.applicationState == UIApplicationStateActive) return;
    
    if (application.applicationState == UIApplicationStateInactive) {
        // 当应用在后台收到本地通知时执行的跳转代码
        [self jumpToSession];
    }
    
    NSLog(@"%@", notification);
}

- (void)jumpToSession
{
    UILabel *redView = [[UILabel alloc] init];
    redView.backgroundColor = [UIColor redColor];
    redView.frame = CGRectMake(0, 100, 300, 400);
    redView.numberOfLines = 0;
    // redView.text = [NSString stringWithFormat:@"%@", launchOptions];
    [self.window.rootViewController.view addSubview:redView];
}
```


#### 2.app已经被关闭(进程被杀死)

![3](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%9C%BA%E5%88%B6_3.gif)

启动app,启动完毕会调用AppDelegate的下面的方法
```
- (BOOL)application:(UIApplication *)application didFinishLaunchWithOptions:(NSDictionary *)launchOptions;
```

`launchOptions`参数通过`UIApplicationLaunchOptionsLocalNotificationKey`取出本地推送通知对象

需要特别注意的是:在iOS8.0以后本地通知有了一些变化,如果要使用本地通知,需要得到用户的许可.
在`didFinishLaunchWithOptions`方法中添加如下代码: 

```

    #define IS_iOS8 ([[UIDevice currentDevice].systemVersion floatValue] >= 8.0)

    if (IS_iOS8) {
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge | UIUserNotificationTypeAlert | UIUserNotificationTypeSound categories:nil];
        [application registerUserNotificationSettings:settings];
    }
```

-----代码实现相关操作
```
#define IS_iOS8 ([[UIDevice currentDevice].systemVersion floatValue] >= 8.0)

@interface AppDelegate ()

@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    /*
     UIUserNotificationTypeNone    = 0,      不发出通知
     UIUserNotificationTypeBadge   = 1 << 0, 改变应用程序图标右上角的数字
     UIUserNotificationTypeSound   = 1 << 1, 播放音效
     UIUserNotificationTypeAlert   = 1 << 2, 是否运行显示横幅
     */
    
    [application setApplicationIconBadgeNumber:0];
    
    if (IS_iOS8) {
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge | UIUserNotificationTypeAlert | UIUserNotificationTypeSound categories:nil];
        [application registerUserNotificationSettings:settings];
    }
    
    // 如果是正常启动应用程序,那么launchOptions参数是null
    // 如果是通过其它方式启动应用程序,那么launchOptions就值
    if (launchOptions[UIApplicationLaunchOptionsLocalNotificationKey]) {
        // 当被杀死状态收到本地通知时执行的跳转代码
        // [self jumpToSession];
        UILabel *redView = [[UILabel alloc] init];
        redView.backgroundColor = [UIColor redColor];
        redView.frame = CGRectMake(0, 100, 300, 400);
        redView.numberOfLines = 0;
        redView.text = [NSString stringWithFormat:@"%@", launchOptions];
        [self.window.rootViewController.view addSubview:redView];
    }
    return YES;
}
```

# 远程推送(Remote Notification)

1. 从远程服务器推送给客户端的通知(需要联网)
2. 远程推送服务, 苹果起名为:APNS (Apple Push Notification Services)


### 解决问题:
只要联网了, 就能够接收到服务器推送的远程通知

### 使用须知:
所有的苹果设备,在联网状态下,都会与苹果服务器建立长连接.
##### 长连接:一直连接,客户端与服务器
##### 长连接作用:
1. 事件校准
2. 系统升级
3. 查找我的iPhone等....

##### 长连接的好处
1. 数据传输速度快
2. 数据保持最新状态


### 官方长连接的使用

#### 1.获得deviceToken的过程
![4](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%9C%BA%E5%88%B6_4.png)

![5](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%9C%BA%E5%88%B6_5.png)

1. 客户端向苹果服务APNS,发送设备的UDID和英文的Bundle Identifier.
2. 经苹果服务器加密生成一个deviceToken
3. 将当前用户的deviceToken(用户标识),发送给自己应用的服务器
4. 自己的服务器,将得到的deviceToken,进行保存

#### 2.利用deviceToken进行数据传输,推送通知

![6](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%9C%BA%E5%88%B6_6.png)
5. 需要推送的时候,将消息和deviceToken一起发送给APNS,苹果服务器,再通过deviceToken找到用户,并将消息发给用户

![7](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E6%B6%88%E6%81%AF%E6%8E%A8%E9%80%81%E6%9C%BA%E5%88%B6_7.png)

这里不再演示关于证书的配置, 简单的只进行说明步骤:
1. 创建明确的AppID,只有明确的AppID才能进行一些特殊的操作
2. 真机调试的APNS SSL证书
3. 发布程序的APNS SSL证书
4. 生成描述文件

[依次安装证书, 再装描述]


### 注册远程推送通知:
##### 1. 客户端如果想要接收APNs的远程推送通知,必须先进行注册(得到用户授权)

一般在APP启动完毕后就马上进行注册
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    if ([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0) {
        // 1.注册UserNotification,以获取推送通知的权限
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeSound | UIUserNotificationTypeAlert | UIUserNotificationTypeBadge categories:nil];
        [application registerUserNotificationSettings:settings];
        
        // 2.注册远程推送
        [application registerForRemoteNotifications];
    } else {
        [application registerForRemoteNotificationTypes:UIRemoteNotificationTypeNewsstandContentAvailability | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
    }
    
    return YES;
}
```

##### 2. 注册成功后, 调用AppDelegate的方法,获取到用户的deviceToken
```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    // <32e7cf5f 8af9a8d4 2a3aaa76 7f3e9f8e 1f7ea8ff 39f50a2a e383528d 7ee9a4ea>
    // <32e7cf5f 8af9a8d4 2a3aaa76 7f3e9f8e 1f7ea8ff 39f50a2a e383528d 7ee9a4ea>
    NSLog(@"%@", deviceToken.description);
}
```

##### 3. 点击推送通知,和本地一样有两种状况.

1. app没有关闭,只是一直隐藏在后台

让app进入前台, 并调用下面的方法(app没有重新启动)

过期的方法:
```
// 当接受到远程退职时会执行该方法(当进入前台或者应用程序在前台)
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
    NSLog(@"%@", userInfo);
    
    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    redView.frame = CGRectMake(100, 100, 100, 100);
    [self.window.rootViewController.view addSubview:redView];
}
```
苹果系统建议使用下面的方法:
```
/*
 1.开启后台模式
 2.调用completionHandler,告诉系统你现在是否有新的数据更新
 3.userInfo添加一个字段:"content-available" : "1" : 只要添加了该字段,接受到通知都会在后台运行
 */
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
    NSLog(@"%@", userInfo);
    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    redView.frame = CGRectMake(100, 100, 100, 100);
    [self.window.rootViewController.view addSubview:redView];
    
    completionHandler(UIBackgroundFetchResultNewData);
}
```

2. app已经关闭,需要重新开启,---基本实现方法和本地通知一致。