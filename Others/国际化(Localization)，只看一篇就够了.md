参考：
1. [iOS开发技巧-国际化(Localization)，只看一篇就够了](https://www.jianshu.com/p/f8edd7b7a217)


本文主要涉及iOS的国际化，网上虽然有很多相关的文章，但是仔细阅读下来感觉都不太全面，因此重开一篇总结，记录项目中遇到的所有要点，demo见最下方链接。
1. App名称国际化
2. 图片、文字国际化
3. 强制默认显示某种语言
4. 启动图国际化
5. iOS10所需的权限配置国际化
6. xib/storyboard国际化
7. 总结

--------

# 1.App名称国际化
非常简单地按步骤修改就可以了。

`PROJECT` -> `Info` -> `Localizations`中点击下方的小“+”，添加需要添加的语言，本文中以简体中文和英文为例。（国际化的所有操作，都需要这一步作为前提。）

![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/1.png)

添加以`InfoPlist.string`为名称的`String File`。==查到的资料都说需要名称一模一样才能使用，没试过其他的名字==。

![2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/2.png)

![3](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/3.png)

选中新建好的`InfoPlist.string`，点击Localize按钮，添加语言。

![4](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/4.png)

完成上一步骤后在右边勾选所需要语言，Xcode会自动创建对应的string文件。
![5](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/5.png)

分别在对应的string文件中填写App名称就可以了。
![6](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/6.png)

![7](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/7.png)

![8](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/8.png)

![9](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/9.png)

#### 关于Bundle name和Bundle display name:

[stackoverflow.com/questions/9667582/bundle-name-and-bundle-display-name](https://link.jianshu.com/?t=https://stackoverflow.com/questions/9667582/bundle-name-and-bundle-display-name)

就显示来说，==`Bundle display name`关系着icon下方的App名称文本显示==，**而`Bundle name`则作为存储App的文件夹名称，并没有什么影响。**


#### 用以上方法修改App名称的一个问题：

如果系统为有对应string文件的语言时，可以正常显示。

如果系统为无对应string文件的语言时，删除App重装后会跟随设定的开发语言显示；直接修改系统语言时会跟随上一次有对应string文件时的语言显示。

![10设定开发语言](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/10.png)

# 2.图片、文字

仍是非常简单地按照步骤修改即可。

首先创建一个string文件(`String File`)，名称为`Localizable.string`。

![11](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/11.png)

选中`Localizable.string`，点击右边的`Localize`按钮，在弹框的下拉菜单中随便选一个需要添加string文件的语言，确认。（操作同`InfoPlist.string`的一样）

![12](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/12.png)

右边的小勾要点上，勾选了之后Xcode才会自动创建对应的string文件。

![13](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/13.png)

在对应的语言的`Localizable.string`文件中添加对应的图片名称和文本内容。
```
"mainImage" = "mainImage_cn";//等号左边为代码需要调用的key，右边为对应的中文图片名称value。

"mainText" = "Chinese";//等号左边为代码中需要调用的key，右边为对应的中文文本value。
```
![14](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/14.png)

![15](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/15.png)

最后，只要在代码中需要显示图片和文字的部分使用Foundation框架中的`NSLocalizedString(key, comment)`调用即可。

```
// 程序将根据第一个参数去对应语言的文件中取对应的值，第二个参数将转化为字符串文件里的注释，可以传nil，也可以传空字符串@""。
//#defineNSLocalizedString(key,comment) [[NSBundle mainBundle] localizedStringForKey:(key)value:@""table:nil]

//图片调用
UIImageView *imageView = [[UIImageView alloc]initWithImage:[UIImage imageNamed:NSLocalizedString(@"mainImage", nil) ]];

//文本调用
textLabel.text = NSLocalizedString(@"mainText", nil);
```

修改语言后的显示：

![16](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/16.png)

![17](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/17.png)

#### 以上的操作后，代码中文字和图片的国际化已经完成了，但是在更换系统语言后会遇到一些问题。

我的项目中只做了简体中文和英文的语言设置，**如果系统语言为日文时安装App，App中语言会根据我设置的开发语言显示为英文**。**如果在App已安装后更改系统语言为日文，则会跟随App上一次的语言设置作相同的显示**。这样并不符合我的项目需求，因此我加入了下方的第三部分。

# 3.强制默认显示某种语言

我的项目需求：在系统语言选定为中文时，App语言为中文，其他情况下全部作英文显示。

因此需要添加系统语言读取，经过判断后直接调用我需要的某种语言对应的值来显示。

```
//在Appdelegate.m中添加系统语言检测与赋值
NSArray *languages = [NSLocale preferredLanguages];
NSString *language = [languages objectAtIndex:0];
if ([language hasPrefix:@"zh"]) {//检测开头匹配，是否为中文
    [[NSUserDefaults standardUserDefaults] setObject:@"zh-Hans" forKey:@"appLanguage"];//App语言设置为中文
}else{//其他语言
    [[NSUserDefaults standardUserDefaults] setObject:@"en" forKey:@"appLanguage"];//App语言设置为英文
}
```

```
//在需要的部分添加手动选取语言的宏，并调用得到对应的值
//宏
#define Localized(key)  [[NSBundle bundleWithPath:[[NSBundle mainBundle] pathForResource:[NSString stringWithFormat:@"%@",[[NSUserDefaults standardUserDefaults] objectForKey:@"appLanguage"]] ofType:@"lproj"]] localizedStringForKey:(key) value:nil table:@"Localizable"]

//调用
forceLabel.text = Localized(@"forceText");
```

选定系统语言为中文，安装App并运行（显示结果为图1），之后修改系统语言为日文（显示结果为图2），使用`NSLocalizedString`国际化的文字显示为中文，使用自定义宏`Localized`国际化的文字显示为英文。

![18图1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/18.png)

![19图2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/19.png)


# 4.启动图
启动图国际化的方法有两种，一种是在`Info.plist`中添加图片（需要导入的图片较多），一种是添加不同的storyboard分别调用（需要导入的图片少些）。操作很简单，但是我在实际使用中遇到了两个问题花费了一些时间，问题将附于下方做一些简述。

## 4.1 图片+Info.plist使启动图国际化

这个方法同样可以用作App中图片的国际化，但是我比较习惯把图片导入`Assets.xcassets`中使用，因此在图片国际化中没有做介绍。这个方法主要是在`Info.plist`中直接配置启动页的参数来展示启动图，下方附上官方的文档。

[developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html)

首先取消默认的启动图，将`TARGET` -> `General` -> `Launch Screen File`中默认的`LaunchScreen`去掉，留空。

![20](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/20.png)

导入准备好的启动图，这里我做了3.5、4、4.7、5.5英寸的。之后选中图片，点击`Localize`按钮，这个步骤和string文件的操作一样，四个图片都要做。

![21](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/21.png)

右键点击设置好的图片，选中`Show in Finder`。下图中，`en.lproj`中保存的是英文版的启动图，`zh-Hans.lproj`保存的是中文的启动图。把准备好的对应的图片修改名称后拖入就行了，图片名称一定要和工程中导入的一样。

![22](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/22.png)

图片准备完毕之后，将`Info.plist`以源码的形式打开，或者是直接在属性列表中点击“+”添加对应的属性。

![23](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/23.png)

Source Code中对应的代码：（demo的Sourcecode Codes中有）

![24](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/24.png)

Property List中对应的列表：

![25](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/25.png)

删除原先安装的App，重新安装后即可看到效果。

![26](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/26.png)

![27](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/27.png)

**运行后如果看不到启动图更换效果，请删除App后重新运行。**

## 4.2 `storyboard` + `InfoPlist.string`使启动图国际化

首先创建作为启动页的storyboard，分别命名区分。我设置的名字是`LaunchImage_En`和`LaunchImage_Ch`，名字自己分得清就可以啦。

![28](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/28.png)

`storyboard`中添加`ViewController`，勾选`Is Initial View Controller`（中英文的启动页都需要勾选），添加启动图，添加约束占满屏。

![29](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/29.png)

![30](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/30.png)

==设置好后在`Info.plist`中添加启动图名称。值我填了项目默认的`LaunchScreen`，但是并不用它作为启动图，后面还要手动赋值。==

![31](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/31.png)

右键点击`Show Raw Keys/Values`看一下key。并在`InfoPlist.string`中手动将一开始创建的两个中英文启动图的storyboard值赋进。

![32](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/32.png)

![33](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/33.png)

![34](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/34.png)

![35](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/35.png)


storyboard更改启动图国际化完成。


### 下面是一开始做国际化时遇到的关于启动图的两个问题。
#### 1. 想要看到启动图的效果，必须删除原有App后重新安装。
查找了一些相关的资料后发现，启动图的资源只会保留一份，在已有的情况下不会重新生成，根据苹果的用户交互指引,该页面是在程序加载时显示的,不建议动态修改。在比较了一些其他的App，例如QQ、DJI GO、淘宝、微博等，有些启动图使用的是可以在不同语言系统下共用的图片，其他的也并没有做动态修改。

#### 2. 使用storyboard修改启动图时一定要记得在`Info.plist`中添加`Launch screen interface file base name`，不然无法显示启动图，运行后会看到App的icon变成了启动图，App的视图大小也会有问题。

# 5. iOS10所需的权限配置

同storyboard设置启动图国际化。

首先在`Info.plist`中添加好权限后右键，选择`Show Raw Keys/Values`看一下key。

![36](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/36.png)

![37](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/37.png)

把key复制进`InfoPlist.string`中分别写好对应的中英文描述即可。

![38](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/38.png)

![39](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%BD%E9%99%85%E5%8C%96(Localization)%EF%BC%8C%E5%8F%AA%E7%9C%8B%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/39.png)

# 6. xib/storyboard国际化

官方文档中有图有真相地描述过啦！

[developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html)

# 7. 总结
iOS的项目国际化中并没有什么难点，主要是找到的一些描述比较含糊或者不全。

国际化其实就是为各个语言单独创建一份资源，通过名称为`*.lproj`文件夹来保存。在勾选需要的语言后，Xcode会自动创建对应的文件，修改其中的值即可。

demo地址：github.com/Linciay/TYLocalization