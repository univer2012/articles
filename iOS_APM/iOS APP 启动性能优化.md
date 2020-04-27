来自：[iOS APP 启动性能优化](https://www.jianshu.com/p/5d4fcc5b534d)

---

本文介绍App启动性能优化，共分六个部分：

第一部分App启动过程

第二部分pre-main阶段的过程和可优化项

第三部分main()阶段可优化项

第四部分启动耗时的测量

第五部分总结我们app需要做的启动性能优化

# 【第一部分】App启动过程

------

iOS应用的启动可分为pre-main阶段和main()阶段，其中系统做的事情依次是：

![img](https:////upload-images.jianshu.io/upload_images/2578906-a2ec659ef03bf717.png?imageMogr2/auto-orient/strip|imageView2/2/w/475)

无论对于系统的动态链接库还是对于App本身的可执行文件而言，他们都算是image（镜像），而每个App都是以image( 镜像)为单位进行加载的

#### 什么是image

1、Executable： 应用的主要二进制（比如.o文件）

2、Dylib： [动态链接库](https://www.jianshu.com/p/1de663f64c05)（dynamic library，又称 DSO 或 DLL）

3、Bundle： 资源文件，不能被链接的 Dylib，只能在运行时使用 dlopen() 加载

#### 1. pre-main阶段

1.1. 加载应用的可执行文件（自身App的所有.o文件的集合）

1.2. 加载动态链接器dyld（dynamic loader，是一个专门用来加载动态链接库的库）

1.3. dyld递归加载应用所有依赖的动态链接库dylib

#### 2. main()阶段

2.1. dyld调用main() 

2.2. 调用UIApplicationMain() 

2.3. 调用applicationWillFinishLaunching

2.4. 调用didFinishLaunchingWithOptions

# 【第二部分】pre-main阶段的过程和可优化项

------

要对pre-main阶段的耗时做优化，需要再学习下dyld加载的过程，根据Apple在2016[WWDC](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.apple.com%2Fvideos%2Fplay%2Fwwdc2016%2F406%2F)上的介绍，dyld的加载主要分为4步：

#### 1. Load dylibs

![img](https:////upload-images.jianshu.io/upload_images/2578906-d2f04a6bb6a3d688.png?imageMogr2/auto-orient/strip|imageView2/2/w/773)

这一阶段dyld会分析应用依赖的dylib（xcode7以后.dylib已改为名.tbd），找到其mach-o文件，打开和读取这些文件并验证其有效性，接着会找到代码签名注册到内核，最后对dylib的每一个segment调用mmap()。

一般情况下，iOS应用会加载100-400个dylibs，其中大部分是系统库，这部分dylib的加载系统已经做了优化。

所以，依赖的dylib越少越好。在这一步，我们可以做的优化有：

1.1、尽量不使用内嵌（embedded）的dylib，加载内嵌dylib性能开销较大

1.2、合并已有的dylib和使用静态库（static archives），减少dylib的使用个数

1.3、懒加载dylib，但是要注意dlopen()可能造成一些问题，且实际上懒加载做的工作会更多

#### 2. Rebase/Bind

![img](https:////upload-images.jianshu.io/upload_images/2578906-9e60539d400485d8.png?imageMogr2/auto-orient/strip|imageView2/2/w/703)

在dylib的加载过程中，系统为了安全考虑，引入了ASLR（Address Space Layout Randomization）技术和代码签名。由于ASLR的存在，镜像（Image，包括可执行文件、dylib和bundle）会在随机的地址上加载，和之前指针指向的地址（preferred_address）会有一个偏差（slide），dyld需要修正这个偏差，来指向正确的地址。

Rebase在前，Bind在后，Rebase做的是将镜像读入内存，修正镜像内部的指针，性能消耗主要在IO。Bind做的是查询符号表，设置指向镜像外部的指针，性能消耗主要在CPU计算。

所以，指针数量越少越好。在这一步，我们可以做的优化有：

2.1、减少ObjC类（class）、方法（selector）、分类（category）的数量

2.2、减少[C++虚函数](https://link.jianshu.com?t=https%3A%2F%2Fblog.csdn.net%2Fu011702002%2Farticle%2Fdetails%2F77434297)的的数量（创建虚函数表有开销）

2.3、使用Swift structs（内部做了优化，符号数量更少）

#### 3. Objc setup

![img](https:////upload-images.jianshu.io/upload_images/2578906-d897284fbaf7c9ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/682)

大部分ObjC初始化工作已经在Rebase/Bind阶段做完了，这一步dyld会注册所有声明过的ObjC类，将分类插入到类的方法列表里，再检查每个selector的唯一性。

在这一步倒没什么优化可做的，Rebase/Bind阶段优化好了，这一步的耗时也会减少。

#### 4. Initializers

![img](https:////upload-images.jianshu.io/upload_images/2578906-95ebbcc392b872ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/686)

到了这一阶段，dyld开始运行程序的初始化函数，调用每个Objc类和分类的+load方法，调用C/C++ 中的构造器函数（用**attribute**((constructor))修饰的函数），和创建非基本类型的C++静态全局变量（通常是类或结构体）。Initializers阶段执行完后，dyld开始调用main()函数。

Objc的load函数和C++的静态构造函数采用由底向上的方式执行，来保证每个执行的方法，都可以找到所依赖的动态库。例：

![img](https:////upload-images.jianshu.io/upload_images/2578906-e7934dd4890d97f7.png?imageMogr2/auto-orient/strip|imageView2/2/w/336)

在这一步，我们可以做的优化有：

4.1、少在类的+load方法里做事情，尽量把这些事情推迟到+initiailize

4.2、减少构造器函数个数，在构造器函数里少做些事情

4.3、减少C++静态全局变量的个数

# 【第三部分】main()阶段的可优化项

------

这一阶段的优化主要是减少didFinishLaunchingWithOptions方法里的工作，在didFinishLaunchingWithOptions方法里，我们会创建应用的window，指定其rootViewController，调用window的makeKeyAndVisible方法让其可见。由于业务需要，我们会初始化各个二方/三方库，设置系统UI风格，检查是否需要显示引导页、是否需要登录、是否有新版本等，由于历史原因，这里的代码容易变得比较庞大，启动耗时难以控制。

所以，满足业务需要的前提下，didFinishLaunchingWithOptions在主线程里做的事情越少越好。在这一步，我们可以做的优化有：

1、梳理各个二方/三方库，找到可以延迟加载的库，做延迟加载处理，比如放到首页控制器的viewDidAppear方法里。

2、梳理业务逻辑，把可以延迟执行的逻辑，做延迟执行处理。比如检查新版本、注册推送通知等逻辑。

3、避免复杂/多余的计算。

4、采用性能更好的API。

5、避免在首页控制器的viewDidLoad和viewWillAppear做太多事情，这2个方法执行完，首页控制器才能显示，部分可以延迟创建的视图应做延迟创建/懒加载处理。

6、首页控制器用纯代码方式来构建。

# 【第四部分】启动耗时的测量

------

在进行优化之前，我们首先应该能测量各阶段的耗时。

#### 1. pre-main阶段测量

对于pre-main阶段，Xcode9之后，Apple提供了一种测量方法，在 Xcode 中 Edit scheme -> Run -> Auguments 将环境变量DYLD_PRINT_STATISTICS 设为1 ：

![img](https:////upload-images.jianshu.io/upload_images/2578906-4cc7ffd370c83554.png?imageMogr2/auto-orient/strip|imageView2/2/w/895)

设置好后把程序跑起来，控制台会有如下输出，pre-main阶段各过程的耗时一览无余

![img](https:////upload-images.jianshu.io/upload_images/2578906-a2181ddd8479abf2.png?imageMogr2/auto-orient/strip|imageView2/2/w/804)

#### 如何解读

1、pre-main阶段（main()函数之前）总共使用了2.1s（5s，i0S11.1测试）

2、在2.1s中，加载动态库用了1.2s，指针重定位使用了222.07ms，ObjC类初始化使用了174.56ms，各种初始化使用了521.02ms。

3、在初始化耗费的521.02ms中，用时最多的四个初始化是libSystem.B.dylib、libMainThreadChecker.dylib、libglInterpose.dilib以及teacher。

#### 2. main()阶段测量

对于main()阶段，主要是测量main()函数开始执行到didFinishLaunchingWithOptions执行结束的耗时，就需要自己插入代码到工程中了。先在main()函数里用变量StartTime记录当前时间：

![img](https:////upload-images.jianshu.io/upload_images/2578906-d3f55d536f2fb436.png?imageMogr2/auto-orient/strip|imageView2/2/w/625)

再在AppDelegate.m文件中用extern声明全局变量StartTime

![img](https:////upload-images.jianshu.io/upload_images/2578906-13e0118dc378c5ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/628)

最后在didFinishLaunchingWithOptions里，再获取一下当前时间，与StartTime的差值即是main()阶段运行耗时。

![img](https:////upload-images.jianshu.io/upload_images/2578906-c7d167ca5e41a8f0.png?imageMogr2/auto-orient/strip|imageView2/2/w/632)

# 【第五部分】总结我们app需要做的启动性能优化

------

#### 1. pre-main阶段的优化

顺便先说一下， pre-main阶段优化到什么范围内比较好呢，苹果给出的建议最好是400ms之内，但这个肯定要按照项目的实际情况有所取舍。

![img](https:////upload-images.jianshu.io/upload_images/2578906-1a654f4b88d95b18.png?imageMogr2/auto-orient/strip|imageView2/2/w/474)

1.1、排查无用的dylib（不确定的可以先删除，在编译下项目试试），减少dylib的数目

1.2、检查 framework应当设为optional和required，如果该framework在当前App支持的所有iOS系统版本都存在，那么就设为required，否则就设为optional

1.3、减少ObjC类（项目中不适用的的库，废弃的代码等）、方法（selector）、分类（category）的数量、无用的库、非基本类型的C++静态全局变量（通常是类或结构体）

1.4、压缩资源图片，删除无用的图片（IO操作）

1.4、少在类的+load方法里做事情，尽量把这些事情推迟到+initiailize

1.5、使用Swift structs（这是长期工作，可以考虑未来新页面用swift写）

#### 2. main()阶段的优化

2.1、可使用instruments的[Time Profiler](https://www.jianshu.com/p/0fd670547235)先分析启动时哪些地方比较耗时，是否可以做优化

2.2、梳理各个二方/三方库，找到可以延迟加载的库，做延迟加载处理，比如放到首页控制器或tabBar控制器的viewDidAppear方法里，并且保证只执行一次（按项目结构，放在合适的地方）

2.3、梳理业务逻辑，把可以延迟执行的逻辑，做延迟执行处理。比如检查新版本、注册推送通知等逻辑。

2.4、避免复杂/多余的计算

2.5、每次用NSLog方式打印会隐式的创建一个Calendar，因此需要删减启动时各业务方打的log

2.6、避免在用户看到的第一个界面（首页控制器或注册登录页面）的viewDidLoad和viewWillAppear做太多事情，这2个方法执行完，第一个页面才能显示，部分可以延迟创建的视图应做延迟创建/懒加载处理

2.7、首页控制器或注册登录页面用纯代码方式来构建

2.8、AppServerConfig的配置未来肯定要拆一下

2.9、持久化数据的读取到内存中的时间也可以评估一下

