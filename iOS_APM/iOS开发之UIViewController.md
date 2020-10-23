来自：[iOS开发之UIViewController](https://www.jianshu.com/p/99f37dac2e8c)



---



对于初学者来说往往无法弄清iOS中各种各样的Controller之间的关系和使用场景。这篇文章将尝试整理各个Controller的用法以及注意事项。

> 主要介绍的Controller有：

- UIViewController
- UINavigationController
- UITabBarController
- ModalViewController
- ChildViewController



## UIXXXController之间的关系

对于各个Controller之间的关系我想从下图开始讲解：

![img](http://lisi1987-blog-images.qiniudn.com/blog_uiviewcontroller_001_v004.png)

img

程序的入口是main()这个大家都知道，iOS也不例外也是在main 里面：

```objectivec
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

对于这个main里面干了什么事情与这篇文章的主题不太符合，初学者可能也不太能理解，感兴趣的可以去了解一下`runloop`，我们只要知道在这之后，我们的APP将要执行的类就到了AppDelegate，并且我们的代码也是从这里开始。

与我们今天主题最相关的就是下面一段了：

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    self.rootViewController = [[UIViewController alloc] init];
    [self makeKeyAndVisible];
    return YES;
}
```

这段代码设置就是iOS程序里启动后显示的第一个ViewController，接下来的流程就全由你自己控制了。

> PS：在Xcode5 之后默认使用storyboard不需要在这里写任何代码就会有显示默认的ViewController，那时因为在info.plist中选择使用Main.storyboard的initialViewController作为初始ViewController。

对于一个iOS的APP来讲第一个UIViewController的类型是非常重要的，它将决定这个APP的UI架构和层级。

我们就从Xcode自带的3个典型模板工程去分析和讲解。

![img](http://lisi1987-blog-images.qiniudn.com/blog_uiviewcontroller_002.png)

img

如图，Xcode创建工程时有几组模板工程可以选择：`Master-Detail Application`、`Single View Application`、`Tabbed Application`。其中创建后对应的rootViewController分别为：

- Master-Detail Application ---> UINavigationController
- Single View Application ---> UIViewController
- Tabbed Application ---> UITabbarController

> PS：Page-Based Application在今天的主题中暂不讨论

## UINavigationController

### 使用方法

```objectivec
//初始化
UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:[CustomViewController new]];

//在vc中使用
[self.navigationController pushViewController:vc animated:YES];
```

### 视图跳转

UINavigationController实际可以理解为UIViewController的一个栈，有着先进后出的特点，操作过程中我们一般也是使用的push和pop进行操作。

```objectivec
pushViewController:animated:

popViewControllerAnimated:

popToRootViewControllerAnimated:

popToViewController:animated:
```

在这一块需要注意的是我们使用的self.navigationController，在实际使用中我们都应该是在UIViewController（UINavigationController栈里面的）来调用该方法，这个属性值最初就是在setRootController时系统设置的，然后在每一次push过程中传递。然后在push和pop这一块没什么好说的了。

### 视图区域

![img](http://lisi1987-blog-images.qiniudn.com/blog_uiviewcontroller_003.jpeg)

img

如图，红色线框区域为 `navigationController.view` 的范围区域，蓝色区域为navigationController当前栈顶的UIViewController的视图区域，黄色区域就是后面我们要讲的NavigationBar的区域。

由图可以看出UIViewController是处于UINavigationController内，并覆盖在navigationController.view上的，在我们push和pop操作过程中实际上更改的内容仅仅是UIViewController区域的内容，所以你在任何栈内的viewController使用navigationController.view操作，之后的结果是不会随着push和pop改变的。

在这里你还需要注意UINavigationController的几个属性:

```objectivec
topViewController   //返回UINavigationController栈顶的viewController

visibleViewController //返回UINavigationController可见到的viewController 包括ModalViewController

viewControllers //返回UINavigationController栈里面的所有viewController，以NSArray形式返回
```

> `topViewController` 与 `visibleViewController` 区别在于，如果当前栈顶的是viewController1，然后在这个Controller中使用presentViewController以Modal方式弹出viewController2。topViewController返回的是viewController1，visibleViewController返回的则是viewController2。



## UITabbarController

对于UITabbarController这就没什么好说的了，使用就真的是简单。

```objectivc
UITabBarController *tabBarViewController = [[UITabBarController alloc]init];
[self.window setRootViewController:tabBarViewController];    
FirstViewController* first = [[FirstViewController alloc]init];
SecondViewController* second = [[SecondViewController alloc]init];
tabBarViewController.viewControllers = [NSArray arrayWithObjects:first, second,nil];
```

更详细信息见：[官方文档](https://link.jianshu.com?t=https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITabBarController_Class/index.html#//apple_ref/occ/cl/UITabBarController)

## ChildViewController

今天的重点在于这一块了，首先我们抛出一个问题：

> 如何在一个ViewController中创建和管理多个复杂的子View？



在许多刚入门或者是初学者来说对于这种情况的处理方法就是`addSubView`，需要多少个子视图不停添加进去就对了。那么问题来了，

> - 产生代码量庞大而且逻辑复杂的ViewController，看着一个上千行代码的ViewController是不是想死的心都有了？
> - 产生的大量 `<nonatomic,strong>UIView` 占据的高内存如何处理？



处理`addSubView`这个方法，还有一种误用就是：

```objectivec
[self.view addSubView:self.vc.view];
```

直接添加ViewController的view到当前ViewController的view中，这种方法倒是可以解决代码的高耦合的问题，但是这种方法会产生一系列更加严重的问题：

> - 直接add进去的SubView不在 `ViewController` 的 view hierarchy 内，事件不会正常传递，如：旋转、触摸等，属于危险操作；
> - 违背CocoaTouch开发的设计MVC原则，ViewController应该且只应该管理一个view hierarchy 。
>
> 
>
> hierarchy [ˈhaɪərɑːki]  n. 层级；等级制度

这也不行那也不可以，我们到底需要怎么来用呢？`addChildViewController`才是我们需要的。

`addChildViewController`是在iOS5之后出现的，在这之前人们一直都在忍受着上面我们讲的种种阵痛。首先我们看看具体用法：

```objectivec
//添加newVC
[self addChildViewController:newVC];
//[newVC willMoveToParentViewController:self];
[self.view addSubview:newVC.view];
[newVC didMoveToParentViewController:self];

//移除oldVC
[oldVC willMoveToParentViewController:nil];
[oldVC.view removeFromSuperview];
[oldVC removeFromParentViewController]; 
//[oldVC didMoveToParentViewController:nil];
```

上面代码中写出了添加和移除ChildViewController的具体写法，**添加过程**：

1. 通过`addChildViewController:`添加子控制器；
2. 这一步为隐式调用，系统在`addChildViewController:`方法后会自动调用方法`willMoveToParentViewController:` ；
3. 将子控制器视图添加进主视图；
4. 通知子控制器 `childViewController` 添加完成，这一步需要手动显示调用。



移除过程与之相反只是调用的几个方法名不一样。



在Apple官方文档上明确表示了必须要调用 `didMoveToParentViewController` 和 `willMoveToParentViewController` 方法来确认完成过程执行完毕，初学者需要特别注意和几个方法的调用顺序，使用不当会导致`UIViewControllerHierarchyInconsistency`的警告。



介绍了使用方法，在来介绍一下使用它的好处，好处就是规避了上面提到的两种误用产生的后果，具体就是：

1. 解决了代码的高耦合；
2. 系统在收到内存警告的时候会回收一些并未显示的view，释放内存；
3. 这种方式add进入的view是属于当前view hierarchy内，可以正常传递各种事件。

在使用 `addChildViewController:` 还有一个比较高级的特性，就是可以由自己选择控制 `childViewController` 的 Appearance callbacks。

```objectivec
//该方法返回NO则childViewController不会自动viewWillAppear和viewWillDisappear对应的方法
- (BOOL)shouldAutomaticallyForwardAppearanceMethods
{
    return NO;
}
//viewWillAppear调用设置为YES，viewWillDisappear调用设置为NO
[self.customChildViewController beginAppearanceTransition:YES animated:animated];
//对应的DidAppear调用需要成对出现
[self.customChildViewController endAppearanceTransition];
```

这一篇关于UIViewController的介绍就写到这里了就结束了。



---

【完】