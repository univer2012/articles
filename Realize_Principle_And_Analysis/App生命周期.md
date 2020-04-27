参考：

1. [iOS App的生命周期](http://www.cocoachina.com/ios/20171219/21586.html)

### 1.iOS程序的启动执行顺序
程序启动顺序图

![iOS启动原理图](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/App%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F_iOS%E5%90%AF%E5%8A%A8%E5%8E%9F%E7%90%86%E5%9B%BE.png)

具体执行流程：
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    NSLog(@"--- %s ---",__func__);//__func__打印方法名
    return YES;
}
- (void)applicationWillResignActive:(UIApplication *)application {
     NSLog(@"--- %s ---",__func__);
}
- (void)applicationDidEnterBackground:(UIApplication *)application {
   NSLog(@"--- %s ---",__func__);
}
- (void)applicationWillEnterForeground:(UIApplication *)application {
   NSLog(@"--- %s ---",__func__);
}
- (void)applicationDidBecomeActive:(UIApplication *)application {
  NSLog(@"--- %s ---",__func__);
}
- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application {
     NSLog(@"--- %s ---",__func__);
}
- (void)applicationWillTerminate:(UIApplication *)application {
    NSLog(@"--- %s ---",__func__);
}
```

启动程序:
```
-[AppDelegate application:didFinishLaunchingWithOptions:] 
-[AppDelegate applicationDidBecomeActive:]
```
按下 `Command + H + SHIFT`：
```
-[AppDelegate applicationWillResignActive:]
-[AppDelegate applicationDidEnterBackground:]
```
重新点击 进入程序：
```
-[AppDelegate applicationWillEnterForeground:]
-[AppDelegate applicationDidBecomeActive:]
```
内存警告：
```
-[AppDelegate applicationDidReceiveMemoryWarning:]
```

### 2.UIViewController 的生命周期
```
// 非storyBoard(xib或非xib)都走这个方法
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    NSLog(@"%s", __FUNCTION__);
    if (self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil]) {
    }
    return self;
}
// storyBoard走这个方法
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
     NSLog(@"%s", __FUNCTION__);
    if (self = [super initWithCoder:aDecoder]) {
    }
    return self;
}
// xib 加载 完成
- (void)awakeFromNib {
    [super awakeFromNib];
     NSLog(@"%s", __FUNCTION__);
}
// 加载视图(默认从nib)
- (void)loadView {
    NSLog(@"%s", __FUNCTION__);
    self.view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
    self.view.backgroundColor = [UIColor redColor];
}
// 视图控制器中的视图加载完成，viewController自带的view加载完成
- (void)viewDidLoad {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidLoad];
}
// 视图将要出现
- (void)viewWillAppear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewWillAppear:animated];
}
// view 即将布局其 Subviews
- (void)viewWillLayoutSubviews {
    NSLog(@"%s", __FUNCTION__);
    [super viewWillLayoutSubviews];
}
// view 已经布局其 Subviews
- (void)viewDidLayoutSubviews {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidLayoutSubviews];
}
// 视图已经出现
- (void)viewDidAppear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidAppear:animated];
}
// 视图将要消失
- (void)viewWillDisappear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewWillDisappear:animated];
}
// 视图已经消失
- (void)viewDidDisappear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidDisappear:animated];
}
// 出现内存警告 
- (void)didReceiveMemoryWarning {
    NSLog(@"%s", __FUNCTION__);
    [super didReceiveMemoryWarning];
}
// 视图被销毁
- (void)dealloc {
    NSLog(@"%s", __FUNCTION__);
}
```

#### 分析

1. `initWithNibName:bundle:`

初始化UIViewController，执行关键数据初始化操作，非StoryBoard创建UIViewController都会调用这个方法。

**注意：不要在这里做View相关操作，View在loadView方法中才初始化。**

2. `initWithCoder:`
如果使用StoryBoard进行视图管理，程序不会直接初始化一个UIViewController，StoryBoard会自动初始化或在segue被触发时自动初始化，因此方法`initWithNibName:bundle:`不会被调用，但是`initWithCoder:`会被调用。

3. `awakeFromNib`
当`awakeFromNib`方法被调用时，所有视图的outlet和action已经连接，但还没有被确定，这个方法可以算作适合视图控制器的实例化配合一起使用的，因为有些需要根据用户喜好来进行设置的内容，无法存在storyBoard或xib中，所以可以在awakeFromNib方法中被加载进来。

4. `loadView`

当执行到loadView方法时，如果视图控制器是通过nib创建，那么视图控制器已经从nib文件中被解档并创建好了，接下来任务就是对view进行初始化。

loadView方法在UIViewController对象的view被访问且为空的时候调用。这是它与awakeFromNib方法的一个区别。

假设我们在处理内存警告时释放view属性：self.view = nil。因此loadView方法在视图控制器的生命周期内可能被调用多次。

loadView方法不应该直接被调用，而是由系统调用，它会加载或创建一个view并把它赋值给UIViewController的view属性。

在创建view的过程中，首先会根据nibName去找对应的nib文件然后加载。如果nibName为空或找不到对应的nib文件，则会创建一个空视图(这种情况一般是纯代码)

**注意:在重写loadView方法的时候，不要调用父类的方法。**

5. `viewDidLoad`
当loadView将view载入内存中，会进一步调用viewDidLoad方法来进行进一步设置。此时，视图层次已经放到内存中，通常，我们对于各种初始化数据的载入，初始设定、修改约束、移除视图等很多操作都可以这个方法中实现。

6. `viewWillAppear`
系统在载入所有的数据后，将会在屏幕上显示视图，这时会先调用这个方法，通常我们会在这个方法对即将显示的视图做进一步的设置。比如，设置设备不同方向时该如何显示；设置状态栏方向、设置视图显示样式等。

另一方面，当APP有多个视图时，上下级视图切换是也会调用这个方法，如果在调入视图时，需要对数据做更新，就只能在这个方法内实现。

7. `viewWillLayoutSubviews`
view即将布局其Subviews。 比如view的bounds改变了(例如:状态栏从不显示到显示,视图方向变化)，要调整Subviews的位置，在调整之前要做的工作可以放在该方法中实现

8. `viewDidLayoutSubviews`
view已经布局其Subviews，这里可以放置调整完成之后需要做的工作。

9. `viewDidAppear`
在view被添加到视图层级中以及多视图，上下级视图切换时调用这个方法，在这里可以对正在显示的视图做进一步的设置。

10. `viewWillDisappear`
在视图切换时，当前视图在即将被移除、或被覆盖是，会调用该方法，此时还没有调用removeFromSuperview。

11. `viewDidDisappear`
view已经消失或被覆盖，此时已经调用removeFromSuperView;

12. `dealloc`
视图被销毁，此次需要对你在init和viewDidLoad中创建的对象进行释放。

13. `didReceiveMemoryWarning`
在内存足够的情况下，app的视图通常会一直保存在内存中，但是如果内存不够，一些没有正在显示的viewController就会收到内存不足的警告，然后就会释放自己拥有的视图，以达到释放内存的目的。但是系统只会释放内存，并不会释放对象的所有权，所以通常我们需要在这里将不需要显示在内存中保留的对象释放它的所有权，将其指针置nil。