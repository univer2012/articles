# Flutter 的疑问与解答

### 1、Flutter打包iOS

来自：[flutter 打包iOS](https://www.jianshu.com/p/93a261ebdf67)

---

先在项目目录下运行

```ruby
flutter build ios --release
```

再到xcode下进行打包

如果不进行build命令，则在xcode下会报错：

```
flutter Could not find an option named "track-widget-creation".
```

具体打包方法：
 [https://github.com/bingoogolapple/bingoogolapple.github.io/issues/46](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fbingoogolapple%2Fbingoogolapple.github.io%2Fissues%2F46)



#### 1.1、安卓打包

在项目的顶目录下，运行：

```
flutter build apk
```





### 2、Cupertino风格 push时隐藏底部TabBar

 来自[Flutter Cupertino风格 push时隐藏底部TabBar](https://www.jianshu.com/p/1b02cbb32bdc?utm_source=desktop&utm_medium=timeline)

---

方法一：push的时候，添加`rootNavigato`参数并将值设置为`true`

```dart
Navigator.of(context,rootNavigator: true).push(
                CupertinoPageRoute(
                  builder: (BuildContext context) {
                    return DetailPage();
                  }
                )
              )
```

方法二： 修改创建tabbar时的代码

```dart
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return CupertinoTabScaffold(
        tabBar: CupertinoTabBar(
          items: [
            BottomNavigationBarItem(title: Text("首页"), icon: Icon(Icons.menu)),
            BottomNavigationBarItem(title: Text("项目"), icon: Icon(Icons.business)),
            BottomNavigationBarItem(title: Text("我的"), icon: Icon(Icons.account_box)),
          ],
        ),
        tabBuilder: (BuildContext context, int index) {
          // return CupertinoTabView(
          //   builder: (context) {
            switch (index) {
              case 0:
                return FirstPage();
                break;
              case 1:
                return SecondPage();
                break;
              case 2:
                return ThirdPage();
                break;
              }
            },
        //   );

        // },
    );
  }
}
```

每个`page`不再用`CupertinoTabView`包裹

可以看到点击第一个tab操作时显示正常，第二个tab时竟然黑屏了

**怎么解决？**

在`FirstPage`、`SecondPage`、`ThirdPage`中，
 `navigationBar`属性添加`transitionBetweenRoutes: false`。

```dart
navigationBar: CupertinoNavigationBar(
            middle: Text("首页"),
            transitionBetweenRoutes: false,
        )
```

之后便可正常显示。



### 3、修改底部导航栏选中item的图片颜色和文字颜色

来自：[Flutter底部导航栏BottomNavigationBar](https://blog.csdn.net/yuzhiqiang_1993/article/details/88118902)

---

一般情况下，我们底部导航栏不会弄得这么花哨，所以一般都是使用fixed模式，此时，导航栏的图标和标题颜色会使用**fixedColor**指定的颜色，如果没有指定fixedColor，则使用默认的主题色**primaryColor**

代码示例：

```dart
@override
  Widget build(BuildContext context) {
    return Scaffold(
       body: pages[currentIndex],
       bottomNavigationBar: BottomNavigationBar(
         items: bottomNavItems,
         onTap: (int index){
           setState(() {
             _changePage(index);
           });
         },
         currentIndex: currentIndex,
         type: BottomNavigationBarType.fixed,
       ),
    );
  }

  /*切换页面*/
  void _changePage(int index) {
    /*如果点击的导航项不是当前项  切换 */
    if (index != currentIndex) {
      setState(() {
        currentIndex = index;
      });
    }
  }
```



---

来自：[Flutter笔记(一)：BottomNavigationBar常见问题](https://www.jianshu.com/p/7274bad9f7ec)

```dart
BottomNavigationBar({
    Key key,
    @required this.items,  //必须有的item
    this.onTap,  //点击事件
    this.currentIndex = 0,  //当前选中
    this.elevation = 8.0,  //高度
    BottomNavigationBarType type,  //排列方式
    Color fixedColor,    //'Either selectedItemColor or fixedColor can be specified, but not both'
    this.backgroundColor,  //背景
    this.iconSize = 24.0,  //icon大小
    Color selectedItemColor,  //选中颜色
    this.unselectedItemColor,  //未选中颜色
    this.selectedIconTheme = const IconThemeData(),
    this.unselectedIconTheme = const IconThemeData(),
    this.selectedFontSize = 14.0,  //选中文字大小
    this.unselectedFontSize = 12.0,  //未选中文字大小
    this.selectedLabelStyle,  
    this.unselectedLabelStyle,
    this.showSelectedLabels = true, //是否显示选中的Item的文字
    bool showUnselectedLabels,  //是否显示未选中的Item的问题
  })
```



### 4、出现`Waiting for another flutter command to release the startup lock...`异常的解决办法

来自：[Waiting for another flutter command to release the startup lock... 异常解决](https://blog.csdn.net/qq_26287435/article/details/89537153?fps=1&locationNum=2)

---



在VS Code中执行`Get packages`、`Run Flutter`等操作时，先在终端执行一下`flutter doctor`，甄别一下flutter是否能正常使用。因为有时会出现下面提示的问题：

```shell
Waiting for another flutter command to release the startup lock...
```

一般出现这问题，那不管是`flutter create xxx_flutter`(创建flutter项目)、还是`flutter packages get`(拉取flutter packages)，还是`flutter run`(运行flutter项目)，都会一直在等待，没有下文。



这时就需要终端运行下`flutter doctor`，看是不是出现了上面的提示。



出现上面的提示，可以有2种解决办法：

**办法一：**关闭项目，重启IDE，但这些操作都无效，除非你重启电脑。

**办法二：**

- 1. 进入到你的flutter sdk目录中，然后找到`bin/cache/lockfile`文件，删除它即可。

- 2. 删除之后你再运行`flutter doctor`，你会发现错误已经解决了。



### 5、flutter之依赖版本冲突:

```
E:\Flutter_Sdk\flutter\bin\flutter.bat --no-color packages get
Running "flutter packages get" in LearnFlutter...
The current Dart SDK version is 2.1.0-dev.4.0.flutter-cd9a42239f.

Because flutterdemo depends on build_runner >=0.1.1 <=0.9.1 which requires SDK version >=1.9.1 <2.0.0-∞, version solving failed.
pub get failed (1)
Process finished with exit code 1
```



来自：[flutter之依赖版本冲突](https://www.jianshu.com/p/cb9f2af0d3da)



解决方法：

###### 1.打开pubspec.yaml找到对应的build_runner

```
dev_dependencies:
 build_runner: ^0.8.0
```

可以看到报错时候依赖的是哪一个版本，然后当我们又不知道目前到底要依赖那个版本才算是合适的正确的时候我们可以这样做

```
dev_dependencies:
 build_runner: any
```

any可以自动的调用pub的版本分析器来寻找最合适的能够避免冲突的依赖版本并下载，这个时候就没问题了。

### 6、运行Flutter项目时，`DEBUG CONSOLE`报错：

```
Launching lib/main.dart on iPhone 11 Pro Max in debug mode...
ProcessException: No such file or directory
  Command: /usr/local/bin/pod install --verbose
Exited (sigterm)
```

解决办法：

1、