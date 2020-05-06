#### 1、手动整理

##### (1) 手动整理图片资源
之前对图片的管理很混乱，不仅每个控制器的文件夹的统计目录都有images文件夹，每个子模块的控制器同级目录下也有iamges文件夹。同时由于管理理念的变迁，以前的`/TKApp/images`和`/TKApp/resource/images`目录下还存放着图片。右键点击`show in Finder`查看，还可以看到有些图片没有引入工程但是在目录下还存在着。
![工程目录与真实目录下的文件1](https://upload-images.jianshu.io/upload_images/843214-f1b99f9bebcc71d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![pic2.png](https://upload-images.jianshu.io/upload_images/843214-f35a34ad97b3ef0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![pic3.png](https://upload-images.jianshu.io/upload_images/843214-a5b3361e8d58d4f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####(2)把没有使用的代码文件删除

有一部分代码是之前的模块文件，但是现在没有在用这些代码了。所以也要把他们找出来并删除掉。特别注意的是，有些模块可能没有引入到工程，所以真实目录下也要检查。
![没有用的子模块](https://upload-images.jianshu.io/upload_images/843214-d1b17b3c3a9c4bfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![没有用的控制器类](https://upload-images.jianshu.io/upload_images/843214-8166902874b71279.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![没有用的文件类](https://upload-images.jianshu.io/upload_images/843214-cb76428e5d5d483b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####2、工具分析来整理图片资源

对于不能立即找到在工程中没有用的图片，我们可以用【LSUnusedResources】开源软件(下载地址：[传送门](https://raw.githubusercontent.com/tinymind/LSUnusedResources/master/Release/LSUnusedResources.app.zip))来快速找出没被使用的资源文件。
使用方法如下：

- 输入`Xcode project path`;
- 在`Code Suffix`把工程用存在的文件格式打上勾，同时取消`Ignore`右旁的勾；
- 点击右下方的`Search`开始查找；
- 查找完成后，选中要删除的资源后，点击右下方的`Delete`可以删除对应目录下的该资源文件；

查找到的结果截图如下：
![LSUnusedResources查找结果-pic7.png](https://upload-images.jianshu.io/upload_images/843214-9600f88a16930f7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由于工程是由各负责人开发完成的，虽然代码在这里，可以每张全局搜索来查看有没有使用，但是未免工作量太大。同时也不是真正清楚这些找到的图片有没有间接使用到。还有就是源头修改才是正道，不然下次工程覆盖后，删除的图片就又回来了。

所以找到这些没有使用的资源文件后，把各个模块的资源图片整理出来，然后考虑告知经理，让各个模块的负责人去检查各自的代码，把没有使用的图片删除之，有间接使用的图片，说明一下。

下图是删除了行情、ygt、trade、open的无用图片后的安装包对比，减小了0.4MB:
![pic17_delete_hq_trade_ygt_open.png](https://upload-images.jianshu.io/upload_images/843214-241a1e37a2fd8f60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

具体整理的文件可以看《华安没有用到的图片统计》

####3、对图片进行无损压缩

清理完没有使用的图片资源后，可以进一步地，对工程中使用到的图片进行无损压缩，减小图片的总大小。

3.1、使用【ImageOptim】

【ImageOptim】的下载地址为：[传送门](https://imageoptim.com/ImageOptim.tbz2)

具体使用，就是直接把要压缩的图片拖到【ImageOptim】窗口里面，ImageOptim机会对图片进行压缩。截图如下：
![pic8.png](https://upload-images.jianshu.io/upload_images/843214-9854210f6f9628ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对拖进去先后坐下对比，图片确实变小了：
![压缩前后对比(实例)-pic9.png](https://upload-images.jianshu.io/upload_images/843214-309e0a90a55454d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.2、使用【tinyPNG】网站在线无损压缩(或者使用tinyPNG的开元软件【TinyPNG4Mac】进行无损压缩)

【tinyPNG】网站的地址为([传送门](https://tinypng.com/))

【TinyPNG4Mac】的下载地址在:[传送门](https://github.com/kyleduo/TinyPNG4Mac)。这个

鉴于【LaunchImage】的图片用【ImageOptim】无损压缩不理想，于是把这组图片都拖到【TinyPNG】网址去压缩，压缩效果如下：
![pic26_tinyPNG_optimize.png](https://upload-images.jianshu.io/upload_images/843214-6d0acdbcff9289e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打包后的前后对比如下：
![pic25_tinyPNG_compare.png](https://upload-images.jianshu.io/upload_images/843214-4be0571acd4b60a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

\####4、使用.xcassets来管理图片以减小安装包大小
为了说明使用`.xcassets`来管理可以有效减小安装包大小，我备份了一份【华安徽赢】的工程，主工程保持不变，备份工程把所有的`.png`图片都放在`Images.xcassets`下
![pic10_all_in_xcassets.png](https://upload-images.jianshu.io/upload_images/843214-3370ec65d45ee7f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![pic11_not_all_in_xcassets.png](https://upload-images.jianshu.io/upload_images/843214-06575bb832ba55ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![pic12_xcassets_compare.png](https://upload-images.jianshu.io/upload_images/843214-a5cf7fcee5745b3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上一张图可以看出，苹果确实会为放在`.xcassets`里面的图片进行压缩，安装包减小了0.2MB。

####5、编译器的优化
1、`Build Settngs`中的配置项

`Build Settings`->`Optimization Level`有几个编译优化选项，release版应该选择Fastest, Smalllest，这个选项会开启那些不增加代码大小的全部优化，并让可执行文件尽可能小。
![pic13_Optimization_Level.png](https://upload-images.jianshu.io/upload_images/843214-4c7248279d678aef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
项目已经设置了这个选项。

2、去除符号信息

`Build Settings`中的`Strip Linked Product`、`Strip Debug Symbols During Copy` 和 `Symbols Hidden by Default`在release版本应该设为`YES`，可以去除不必要的调试符号。`Symbols Hidden by Default`会把所有符号都定义成”private extern”。这些选项目前都是XCode里release的默认选项，但旧版XCode生成的项目可能不是，可以检查一下。
![pic14_symbols.png](https://upload-images.jianshu.io/upload_images/843214-e0be058dab2f634d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
项目之前已经设置`Strip Linked Product`、`Strip Debug Symbols During Copy`为`Yes`了。设置`Symbols Hidden by Default`为`Yes`解压ipa包后可以减小0.1MB：
![pic18_Symbols Hidden by Default.png](https://upload-images.jianshu.io/upload_images/843214-c4bd18fa3184ee01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3、配置App Thinning为包瘦身

App Thinning是伴随iOS9一起出现的，因此该功能也仅对iOS9以上设备生效。App Thinning主要包含以下3个方面：App Slicing(应用程序的划分)、On Demand Resources(按需加载资源) 和 Bitcode。

3.1、App Slicing

根据苹果官方文献的描述「Slicing  是为应用捆绑包创建、分发不同变体以适应不同目标设备的过程。一个变体只包含针对某个目标设备的可执行架构与资源。」 换句话说，App  Slicing仅向设备传送与之相关的资源（取决于屏幕分辨率，架构等等）。事实上，App Slicing 负责处理 App Thinning  的主要流程。

当你准备好提交 app 时，通常会（但必须使用 Xcode7，因为它包含支持 App Thinning 的 iOS9 SDK）向  iTtunes Connect 上传 .IPA 或 .App 文件。然后，应用商店分割该应用，创建特定的变体以适应性能不同设备。

3.2、On Demand Resources

按需加载资源是在 app 第一次安装后可下载的文件。举例说明，当玩家解锁游戏的特定关卡后可以下载新关卡（和这个关卡相关的特定内容）。此外，玩家已经通过的关卡可以被移除以便节约设备上的存储空间。

开启按需加载资源功能涉及改变 Xcode 中的设置：在`Build Settings`下，将`Enable On Demand Resources`设置为`Yes`。
![On_Demand_Resources的设置-pic15_On_Demand_Resources.png](https://upload-images.jianshu.io/upload_images/843214-9b70a30e3bd63b7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.3、Bitcode

本质上Bitcode也是苹果在用户下载前优化app的新方式。Bitcode 使得 app 无论在何设备上都能快速高效地运行。Bitcode 使用最新的编译器自动编译app并且针对特定架构进行优化。（例如，针对 iPhone 6s和 iPad Air 2等 64 位处理器的  arm64）

Bitcode 不会下载应用针对不同架构的优化，而仅下载与特定设备相关的优化，使得下载量更小，同时与前文所述的 App Thinning 技术紧密合作。

Bitcode 是 iOS 上较新的功能，对于新的项目需要手动开启。在`Build Settings`下，将`Enable Bitcode`设置为`Yes`来完成。
![pic16_bitcode.png](https://upload-images.jianshu.io/upload_images/843214-d1f950698169eb9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置后打包，报错，提示IM的`XMPPRoomHybridStorage`类还支持bitcode：
![pic19_bitcode.png](https://upload-images.jianshu.io/upload_images/843214-8e213ab8c679e2be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、设置`Make Strings Read-Only`为`Yes`

工程已设置。
![pic20_make_Strings_Read_Only.png](https://upload-images.jianshu.io/upload_images/843214-4e96b6b8b868cff4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5、去掉异常支持，`Enable C++ Exceptions` 和 `Enable Objective-C Exceptions`设为NO，并且`Other C Flags`添加`-fno-exceptions`。
![pic21_Enable_C++:OC_Exceptions.png](https://upload-images.jianshu.io/upload_images/843214-a349d6c9e2588114.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![pic22_other_linker_flags.png](https://upload-images.jianshu.io/upload_images/843214-21df2d97a3c589ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`Archive`后报错，原因是关闭`Enable Objective-C Exceptions`后`@try{//...} @catch() {//...}`不能使用了。
![pic23_Enable_OC_Exception_try.png](https://upload-images.jianshu.io/upload_images/843214-bd12b1e1d34c5b82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`Enable Objective-C Exceptions`设置回去后，打包，发现包大小没有变化：
![pic24_compare_Exceptions_NO.png](https://upload-images.jianshu.io/upload_images/843214-b7cd965ed41ab755.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####6、代码优化



参考资料：[iOS9 App Thinning(应用瘦身)功能介绍](http://www.jianshu.com/p/c74e532d2f36)