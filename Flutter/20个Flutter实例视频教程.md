来自：[20个Flutter实例视频教程 让你轻松上手工作](https://jspang.com/posts/2019/02/22/flutterdemo.html)



---

这篇文章是Flutter基础的一个进阶教程，主要讲解的是在工作中一些常用的功能。学完这篇文章和视频，你可以轻松制作出80%以上工作中常用的Flutter复杂效果，对Flutter有更深刻的了解。

**本文是实战和基础的一个衔接总结。**在学完基础后，很多小伙伴都急于上手一个实战项目，觉的做完实战项目就算学会了Flutter，达到了工作水平一样。其实在实战和基础之间还需要有一个过度，人们喜欢称之为进阶。那这篇文章或者说视频就是进阶教程，它会以小实例的方式进行讲解。实例也会达到20个以上，每个小实例就是一个小的实战。



## 前置知识和说明

**特别说明**

最重要的放在最前面，这些实例有些是我原创的代码，有些是在Github上找到的，都是开源的，所以我和你都可以放心大胆的使用。（我不创造知识，我只是知识的搬运工。创造知识是尤大神和Google这样的人或公司才能做到的 ，我的级别还不行。我的目的就是减轻你的学习成本。）

视频的目的是为了节约大家的学习成本，让大家简单快速的学会知识。视频中的任何一段代码我都有详细的讲解，但是我觉既然是一个进阶教程，不光要学习零散的知识点，最重要的是理解Flutter的开发模式、习惯和一些面向对象方法。

那在学习这篇文章之前，你应该先学习一下基础知识，至少把下面这四篇文章，共25集视频，先看一边，作一下练习。(这些视频是完全免费的，大家直接可以在博客上收看)。

第一季Flutter视频教程地址：https://jspang.com/posts/2019/01/20/flutter-base.html

第二季Flutter视频教程地址：https://jspang.com/posts/2019/01/21/flutter-base2.html



第三季Flutter视频教程地址：https://jspang.com/posts/2019/01/28/flutter-base3.html



第四季Flutter视频教程地址：https://jspang.com/posts/2019/02/01/flutter-base4.html



## 案例效果查看

| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo01.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo02.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo03.gif) |
| ------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------ |
| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo04.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo05.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo06.gif) |
| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo07.png) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo08.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo09.gif) |
| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo10.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo11.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo12.gif) |
| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo13.png) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo14.png) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo15.gif) |
| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo16.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo17.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo18.gif) |

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第01节-底部导航栏制作-1)第01节: 底部导航栏制作-1

工作中最简单的一个APP也要具备一个功能，就是底部导航栏，你很难找出没有底部导航栏的应用。这么刚需的功能，那就从这里入手开始学习吧。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo1_01.gif)

### 主入口文件的编写

首先我们先写一个主入口文件，这个文件只是简单的APP通用结构，最主要的是要引入自定义的`BottomNavigationWidget`组件。

main.dart代码如下:

```dart
import 'package:flutter/material.dart';
import 'bottom_navigation_widget.dart';

void main()=> runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter bottomNavigationBar',
      theme:ThemeData.light(),
      home:BottomNavigationWidget()
    );
  }.
}
```

注意的是`BottomNaivgationWidget`这个组件还没有编写，所以现在会报错。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#statefulwidget-讲解)StatefulWidget 讲解

在编写`BottomNaivgationWidget`组件前，我们需要简单了解一下什么是`StatefulWidget`.

`StatefulWidget`具有可变状态(state)的窗口组件（widget）。使用这个要根据变化状态，调整State值。

在lib目录下，新建一个`bottom_navigation_widget.dart`文件。

它的初始化和以前使用的`StatelessWidget`不同，我们在VSCode中直接使用快捷方式生成代码（直接在VSCode中输入stful）：

```text
class name extends StatefulWidget {
  _nameState createState() => _nameState();
}

class _nameState extends State<name> {
  @override
  Widget build(BuildContext context) {
    return Container(
       child: child,
    );
  }
}
```

上面的代码可以清楚的看到，使用`StatefulWidget`分为两个部分，第一个部分是继承与`StatefullWidget`，第二个部分是继承于`State`.其实`State`部分才是我们的重点，主要的代码都会写在`State`中。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#bottomnaivgationwidget自定义)BottomNaivgationWidget自定义

接下来我们就要创建`BottomNaivgationWidget`这个Widget了，只是建立一个底部导航。

```dart
import 'package:flutter/material.dart';


class BottomNavigationWidget extends StatefulWidget {
  _BottomNavigationWidgetState createState() => _BottomNavigationWidgetState();
}

class _BottomNavigationWidgetState extends State<BottomNavigationWidget> {
  final _BottomNavigationColor = Colors.blue;

  @override
  Widget build(BuildContext context) {
     return Scaffold(
     
       bottomNavigationBar: BottomNavigationBar(
         items: [
           BottomNavigationBarItem(
             icon:Icon(
               Icons.home,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'Home',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
           BottomNavigationBarItem(
             icon:Icon(
               Icons.email,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'Email',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
           BottomNavigationBarItem(
             icon:Icon(
               Icons.pages,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'Pages',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
           BottomNavigationBarItem(
             icon:Icon(
               Icons.airplay,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'AipPlay',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
         ],
         type:BottomNavigationBarType.fixed
       ),
     );
  }
}
```

这时候我们就可以使用`flutter run`来进行查看代码了，效果已经出现，在APP的页面上已经出现了一个底部导航栏，只不过现在还点击还没有什么效果。

下节课我们继续学习，让点击后可以切换页面，完成我们第一个的小实例效果。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第02节-底部导航栏制作-2)第02节: 底部导航栏制作-2

这节课我们继续上节课的内容，把底部导航的完整效果做出来，我们先把几个要导航的页面创建出来。

### 子页面的编写

子页面我们就采用最简单的编写了，只放入一个`AppBar`和一个`Center`，然后用Text Widget表明即可。

先来写一个HomeScreen组件，新建一个pages目录，然后在目录下面新建`home_screen.dart`文件。

```dart
import 'package:flutter/material.dart';

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar:AppBar(
        title: Text('HOME'),
      ),
      body:Center(
        child: Text('HOME'),
      )
    );
  }
}
```

有了这个文件剩下的文件就可以复制粘贴，然后改少量的代码来完成了。

分别建立:

- email_screen.dart
- pages_screen.dart
- airplay_screen.dart

这些都是导航要用的子页面，有了这些页面，我们才能继续编写代码。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#重写initstate-方法)重写initState()方法

我们要重新`initState()`方法，把刚才做好的页面进行初始化到一个Widget数组中。有了数组就可以根据数组的索引来切换不同的页面了。这是现在几乎所有的APP采用的方式。

代码如下:

```dart
 List<Widget> list = List();
 @override
 void initState(){
    list
      ..add(HomeScreen())
      ..add(EmailScreen())
      ..add(PagesScreen())
      ..add(AirplayScreen());
    super.initState();
  }
```

这里的`..add()`是Dart语言的..语法，如果你学过编程模式，你一定听说过建造者模式，简单来说就是返回调用者本身。这里list后用了..add()，还会返回list，然后就一直使用..语法，能一直想list里增加widget元素。 最后我们调用了一些父类的`initState()`方法。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#bottomnavigationbar里的响应事件)BottomNavigationBar里的响应事件

`BottomNavigationBar`组件里提供了一个相应事件`onTap`，这个事件自带一个索引值`index`，通过索引值我们就可以和我们list里的索引值相对应了。

```dart
        onTap:(int index){
           setState((){
             _currentIndex= index;
           });
         },
```

现在给出全部的`bottom_navigation_widget.dart`的全部代码，讲解我会在视频中一行一行讲解。

```dart
import 'package:flutter/material.dart';
import 'pages/home_screen.dart';
import 'pages/email_screen.dart';
import 'pages/pages_screen.dart';
import 'pages/airplay_screen.dart';

class BottomNavigationWidget extends StatefulWidget {
  _BottomNavigationWidgetState createState() => _BottomNavigationWidgetState();
}

class _BottomNavigationWidgetState extends State<BottomNavigationWidget> {
  final _BottomNavigationColor = Colors.blue;
  int _currentIndex = 0;
  List<Widget> list = List();

  @override
  void initState(){
    list
      ..add(HomeScreen())
      ..add(EmailScreen())
      ..add(PagesScreen())
      ..add(AirplayScreen());
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
     return Scaffold(
       body: list[_currentIndex],
       bottomNavigationBar: BottomNavigationBar(
         items: [
           BottomNavigationBarItem(
             icon:Icon(
               Icons.home,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'Home',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
           BottomNavigationBarItem(
             icon:Icon(
               Icons.email,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'Email',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
           BottomNavigationBarItem(
             icon:Icon(
               Icons.pages,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'Pages',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
           BottomNavigationBarItem(
             icon:Icon(
               Icons.airplay,
               color:_BottomNavigationColor,
             ),
             title:Text(
               'AipPlay',
               style:TextStyle(color:_BottomNavigationColor)
             )
           ),
         ],
         currentIndex:_currentIndex,
         onTap:(int index){
           setState((){
             _currentIndex= index;
           });
         },
         type:BottomNavigationBarType.fixed
       ),
     );
  }
}
```

通过两节的学习，基本掌握了APP底部导航的编写方法，其实这些知识是很面向工作的，因为真是项目中，你也要如此划分你的代码结构。

我希望你一定要动手练习一下，因为不动手练习，你是真的学不会的，这不再是基础知识，而是稍显复杂的小案例，并且这些案例你练习后，以后工作中用到，是可以直接拷贝里边的代码的，绝对不亏的。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第03节-不规则底部工具栏制作-1)第03节: 不规则底部工具栏制作-1

大部分的底部导航都是中规中矩的，但有些时候也需要突出个性，比如在中间部位增加一个突出的按钮。这节课我们主要学习一下不规则的导航如何制作。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo1_02.gif)



### 自定义主题样本

Flutter支持自定义主题，如果使用自定义主题，设置的内容项是非常多的，这可能让初学者头疼，Flutter贴心的为给我们准备了主题样本。

> primarySwatch ：现在支持18种主题样本了。

具体代码如下：

```dart
theme: ThemeData(
  primarySwatch: Colors.lightBlue,
),
```

会了这个知识后，我们就可以先把我们的主入口文件编写一下了，具体代码如下：

```dart
import 'package:flutter/material.dart';
import 'bottom_appBar_demo.dart';

void main()=>runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
theme: ThemeData(
  primarySwatch: Colors.lightBlue,
),
      home:BottomAppBarDemo(),
    );
  }
}
```

这时候`bottom_appBar_demo.dart`这个文件还没有，也找不到，这个文件是我们的主要文件，我们的主要业务逻辑会写在这个文件里。

因为没有所以你写完之后会报错，那接下来我们就来编写这个文件。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#floatingactionbutton-widget)floatingActionButton Widget

`floatingActionButton`工作中我们通常简称它为“FAB”，也许只是我们公司这样称呼，从字面理解可以看出，它是“可交互的浮动按钮”,其实在Flutter默认生成的代码中就有这家伙，只是我们没有正式的接触。

一般来说，它是一个圆形，中间放着图标，会优先显示在其他Widget的前面。

下面我们来看看它的常用属性:

- onPressed ：点击相应事件，最常用的一个属性。
- tooltip：长按显示的提示文字，因为一般只放一个图标在上面，防止用户不知道，当我们点击长按时就会出现一段文字性解释。非常友好，不妨碍整体布局。
- child ：放置子元素，一般放置Icon Widget。

我们来看一下`floatingActionButton`的主要代码:

```dart
floatingActionButton: FloatingActionButton(
    onPressed: (){
      Navigator.of(context).push(MaterialPageRoute(builder:(BuildContext context){
        return EachView('New Page');
      }));
    },
    tooltip: 'Increment',
    child: Icon(
      Icons.add,
      color: Colors.white,
    ),
  ),
```

写完这些代码已经有了一个悬浮的按钮，但这个悬浮按钮还没有和低栏进行融合，这时候需要一个属性。

```dart
floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
```

这时候就可以和底栏进行融合了。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#bottomappbar-widget)BottomAppBar Widget

`BottomAppBar`是 底部工具栏的意思，这个要比`BottomNavigationBar`widget灵活很多，可以放置文字和图标，当然也可以放置容器。

`BottomAppBar`的常用属性:

- color:这个不用多说，底部工具栏的颜色。
- shape：设置底栏的形状，一般使用这个都是为了和`floatingActionButton`融合，所以使用的值都是CircularNotchedRectangle(),有缺口的圆形矩形。
- child ： 里边可以放置大部分Widget，让我们随心所欲的设计底栏。

这节课先来看看这个布局，下节课我们再添加交互效果。

本节课主要代码：

```dart
import 'package:flutter/material.dart';

class BottomAppBarDemo extends StatefulWidget {
  _BottomAppBarDemoState createState() => _BottomAppBarDemoState();
}

class _BottomAppBarDemoState extends State<BottomAppBarDemo> {

  @override
  Widget build(BuildContext context) {
     return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: (){
          
          },
          tooltip: 'Increment',
          child: Icon(
            Icons.add,
            color: Colors.white,
          ),
        ),
       floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
       bottomNavigationBar: BottomAppBar(
         color:Colors.lightBlue,
         shape:CircularNotchedRectangle(),
         child: Row(
           mainAxisSize: MainAxisSize.max,
           mainAxisAlignment: MainAxisAlignment.spaceAround,
           children: <Widget>[
             IconButton(
               icon:Icon(Icons.home),
               color:Colors.white,
               onPressed:(){
                
               }
             ),
             IconButton(
               icon:Icon(Icons.airport_shuttle),
               color:Colors.white,
               onPressed:(){
                 
               }
             ),
           ],
         ),
       )
        ,
     );
  }
}
```

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第04节-不规则底部工具栏制作-2)第04节: 不规则底部工具栏制作-2

上节课已经完成了基本的页面布局，但是还没有交互效果，这节主要制作一下交互效果。



### StatefulWidget子页面的制作

在前两节课的实例中我们使用了子页面，但子页面继承与`StatelessWidget`(不可变控件)，所以很麻烦的写了4个页面，其实完全可以写一个继承于`StatefulWidget`的控件，进行搞定。

新建一个`each_view.dart`文件，然后输入如下代码：

```dart
import 'package:flutter/material.dart';

class EachView extends StatefulWidget {
  String _title;
  EachView(this._title);
  @override
  _EachViewState createState() => _EachViewState();
}

class _EachViewState extends State<EachView> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar:AppBar(title:Text(widget._title)),
      body: Center(child:Text(widget._title)),
    );
  }
}
```

代码中设置了一个内部的`_title`变量，这个变量是从主页面传递过来的，然后根据传递过来的具体值显示在APP的标题栏和屏幕中间。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#按钮交互效果的制作)按钮交互效果的制作

这些效果都是在`bottom_appBar_demo.dart`页面完成的。首先我们需要引入新作的子页面`each_view.dart`。

```dart
import 'each_view.dart';
```

新建两个变量,主要作用是控制body中的试图，也就是显示不同的子页面。

```dart
  List<Widget> _eachView;  //创建视图数组
  int _index = 0;          //数组索引，通过改变索引值改变视图
```

下一步是为`_eachView`进行初始化赋值，我们可以直接重写初始化方法，具体代码如下：

```dart
  @override
  void initState() {
      // TODO: implement initState
      super.initState();
      _eachView = List();
      _eachView..add(EachView('Home'))..add(EachView('Me'));
    }
```

剩下的就是写个个按钮的交互事件，交互的动作分两种：

- 直接打开子导航，比如我们点击了中间的”+“按钮，我们直接开启子页面。

```dart
onPressed: (){
    Navigator.of(context).push(MaterialPageRoute(builder:(BuildContext context){
      return EachView('New Page');
    }));
  },
```

- 改变状态，通过改变状态，来切换页面，这样我们整体页面并没有被刷新。

```dart
onPressed:(){
  setState(() {
    _index=0;             
  });
}
```

为了方便小伙伴们学习，给出`bottom_appBar_demo.dart`所有代码，代码如下:

```dart
import 'package:flutter/material.dart';
import 'each_view.dart';

class BottomAppBarDemo extends StatefulWidget {
  _BottomAppBarDemoState createState() => _BottomAppBarDemoState();
}

class _BottomAppBarDemoState extends State<BottomAppBarDemo> {
  List<Widget> _eachView;  //创建视图数组
  int _index = 0;          //数组索引，通过改变索引值改变视图

  @override
  void initState() {
      // TODO: implement initState
      super.initState();
      _eachView = List();
      _eachView..add(EachView('Home'))..add(EachView('Me'));
    }

  @override
  Widget build(BuildContext context) {
     return Scaffold(
       body:_eachView[_index],
        floatingActionButton: FloatingActionButton(
        onPressed: (){
          Navigator.of(context).push(MaterialPageRoute(builder:(BuildContext context){
            return EachView('New Page');
          }));
        },
          tooltip: 'Increment',
          child: Icon(
            Icons.add,
            color: Colors.white,
          ),
        ),
       floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
       bottomNavigationBar: BottomAppBar(
         color:Colors.lightBlue,
         shape:CircularNotchedRectangle(),
         child: Row(
           mainAxisSize: MainAxisSize.max,
           mainAxisAlignment: MainAxisAlignment.spaceAround,
           children: <Widget>[
             IconButton(
               icon:Icon(Icons.home),
               color:Colors.white,
                onPressed:(){
                  setState(() {
                    _index=0;             
                  });
                }
             ),
             IconButton(
               icon:Icon(Icons.airport_shuttle),
               color:Colors.white,
               onPressed:(){
                 setState(() {
                    _index=1;             
                 });
               }
             ),
           ],
         ),
       )
        ,
     );
  }
}
```

我相信小伙伴都制作出了这个效果，这个在工作中也很常见。希望你们都能学会，特别是动态子页面的这个使用技巧。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第05节-酷炫的路由动画-1)第05节: 酷炫的路由动画-1

现在Flutter的路由效果已经非常不错了，能满足大部分App的需求，但是谁不希望自己的App更酷更炫那，学完这节课你就可以给自己的APP加上酷炫的路由动画了。

| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_01.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_02.gif) |
| ------------------------------------------------------ | ------------------------------------------------------ |
| ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_03.gif) | ![img](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_04.gif) |

其实路由动画的原理很简单，就是重写并继承`PageRouterBuilder`这个类里的`transitionsBuilder`方法。

不过这个方法还是有很多写法的，通过写法的不同，产生的动画效果也有所不同。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#主入口方法)主入口方法

先编写一个主入口方法，还是最简单的格式，只不过home属性，使用的`FirstPage`的组件是我们自定义的，需要我们再次编写。入口文件的代码如下：

```dart
import 'package:flutter/material.dart';
import 'pages.dart';

void main()=>runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      theme: new ThemeData(
        primarySwatch: Colors.blue,
      ),
      home:FirstPage()
    );
  }
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#pages-dart页面的编写)pages.dart页面的编写

主入口文件用`import`引入了`pages.dart`文件，这个文件就是生成了两个页面（Flutter里的页面也是Widget,这个你要跟网页区分开）。有了两个页面就可以实现路由跳转了。

`pages.dart`文件的代码如下，这里我们先用普通路由代替，看一看效果。

```dart
import 'package:flutter/material.dart';


class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.blue,
      appBar:AppBar(
        title:Text('FirstPage',style: TextStyle(fontSize: 36.0)),
        elevation: 0.0,
      ),
      body:Center(
        child: MaterialButton(
          child: Icon(
            Icons.navigate_next,
            color:Colors.white,
            size:64.0,
          ),
          onPressed: (){
              Navigator.of(context).push(
                MaterialPageRoute(
                  builder:(BuildContext context){
                     return SecondPage();
                  }));
          },
        ),
      )
    );
  }
}

class SecondPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.pinkAccent,
      appBar: AppBar(
        title: Text('SecondPage',style:TextStyle(fontSize:36.0),),
        backgroundColor: Colors.pinkAccent,
        leading:Container(),
        elevation: 0.0,
      ),
      body:Center(
        child: MaterialButton(
          child: Icon(
            Icons.navigate_before,
            color:Colors.white,
            size:64.0
          ),
          onPressed: ()=>Navigator.of(context).pop(),
        ),
      )
    );
  }
}
```

上面代码中有一个新知识点，需要学习一下：

1. **AppBar Widger的 elevation 属性：这个值是AppBar 滚动时的融合程度，一般有滚动时默认是4.0，现在我们设置成0.0，就是和也main完全融合了。**

写完这个页面代码后，已经可以进行最基本的导航了，但是并没有什么酷炫的动画。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#自定义customroute-widget)自定义CustomRoute Widget

新建一个`custome_router.dart`文件，这个就是要自定义的路由方法，自定义首先要继承于通用的路由的构造器类`PageRouterBuilder`。继承之后重写父类的`CustomRoute`构造方法。

构造方法可以简单理解为：只要以调用这个类或者说是Widget，构造方法里的所有代码就执行了。

`custome_router.dart`代码如下(详细解释视频中说):

```dart
import 'package:flutter/material.dart';
import 'custome_router.dart';

class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.blue,
      appBar:  AppBar(
        title: Text('FirstPage', style: TextStyle(fontSize: 36.0),),
        elevation: 0.0,
      ),
      body: Center(
        child: MaterialButton(
          child: Icon(
            Icons.navigate_next,
            color: Colors.white,
            size: 64.0,
          ),
          onPressed: (){
            Navigator.of(context).push(
              MaterialPageRoute(
                builder: (BuildContext context) {
                  return SecondPage();
                }
              )
            );
          },
        ),
      ),

    );
  }
}

class SecondPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.pinkAccent,
      appBar: AppBar(
        title: Text('SecondPage',style: TextStyle(fontSize: 36.0),),
        backgroundColor: Colors.pinkAccent,
        leading: Container(),
        elevation: 0.0,
      ),
      body: Center(
        child: MaterialButton(
          child: Icon(
            Icons.navigate_before,
            color: Colors.white,
            size: 64.0,
          ),
          onPressed: () => Navigator.of(context).pop(),
        ),
      ),
    );
  }
}
```

- FadeTransition:渐隐渐现过渡效果，主要设置opactiy（透明度）属性，值是0.0-1.0。
- animate :动画的样式，一般使用动画曲线组件（CurvedAnimation）。
- curve: 设置动画的节奏，也就是常说的曲线，Flutter准备了很多节奏，通过改变动画取消可以做出很多不同的效果。
- transitionDuration：设置动画持续的时间，建议再1和2之间。

`pages.dart`文件修改如下：

```dart
class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.blue,
      appBar:  AppBar(
        title: Text('FirstPage', style: TextStyle(fontSize: 36.0),),
        elevation: 0.0,
      ),
      body: Center(
        child: MaterialButton(
          child: Icon(
            Icons.navigate_next,
            color: Colors.white,
            size: 64.0,
          ),
          onPressed: (){
            /*Navigator.of(context).push(
              MaterialPageRoute(
                builder: (BuildContext context) {
                  return SecondPage();
                }
              )
            );*/
            Navigator.of(context).push(CustomRoute(SecondPage()));
          },
        ),
      ),

    );
  }
}
```



写完代码，我们已经可以看到在切换路由时有了动画效果，下节课我们再写三个常用动画效果。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第06节-酷炫的路由动画-2)第06节: 酷炫的路由动画-2

这节课我们接着上节课再作三个常用的动画效果，目的是让你更深刻的了解路由动画的使用方法。

### 缩放路由动画

```text
return ScaleTransition(
  scale:Tween(begin:0.0,end:1.0).animate(CurvedAnimation(
    parent:animation1,
    curve: Curves.fastOutSlowIn
    )),
    child:child
);
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#旋转-缩放路由动画)旋转+缩放路由动画

旋转+缩放的思路是在一个路由动画里的child属性里再加入一个动画，让两个动画同时执行。来看详细代码：

```text
 return RotationTransition(
  turns:Tween(begin:0.0,end:1.0)
  .animate(CurvedAnimation(
    parent: animation1,
    curve: Curves.fastOutSlowIn
  )),
  child:ScaleTransition(
    scale:Tween(begin: 0.0,end:1.0)
    .animate(CurvedAnimation(
        parent: animation1,
        curve:Curves.fastOutSlowIn
    )),
    child: child,
  )
);
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#左右滑动路由动画)左右滑动路由动画

其实用的做多的还是左右滑动路由动画，其实实现起来也是非常简单，直接使用`SlideTransition`就可以了。

```text
// 幻灯片路由动画
return SlideTransition(
  position: Tween<Offset>(
    begin: Offset(-1.0, 0.0),
    end:Offset(0.0, 0.0)
  )
  .animate(CurvedAnimation(
    parent: animation1,
    curve: Curves.fastOutSlowIn
  )),
  child: child,
);
```

总结:动画的使用会让我们的APP更加酷炫，也会让别人觉的你不是一个新手，再Flutter里使用动画是非常方便的，所以你可以把这些动画效果事先写好，在工作中直接使用。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第07节-毛玻璃效果制作)第07节: 毛玻璃效果制作

Flutter的Fliter Widget 也是非常强大的，它可以制作出你想要的神奇滤镜效果。这节我们就以实战的方式，制作一个毛玻璃效果，通过实例来学习Fitler 的用法。（制作效果如下图）

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_05.png)



### main.dart文件编写

这个和以前的写法都一样，所以就直接贴代码了。详细解释会在视频中解说，不过我相信不解释，小伙伴也一定可以看明白。

```dart
import 'package:flutter/material.dart';
import 'frosted_glass_emo.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      theme:ThemeData(
        primarySwatch: Colors.blue,
      ),
      home:Scaffold(
        body:FrostedGlassDemo(),
      )
    );
  }
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#backdropfilter-widget)BackdropFilter Widget

`BackdropFilter`就是背景滤镜组件，使用它可以给父元素增加滤镜效果，它里边最重要的一个属性是`filter`。`filter`属性中要添加一个滤镜组件，实例中我们添加了图片滤镜组件，并给了模糊效果。

> 图片地址：https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1545738629147&di=22e12a65bbc6c4123ae5596e24dbc5d3&imgtype=0&src=http%3A%2F%2Fpic30.photophoto.cn%2F20140309%2F0034034413812339_b.jpg

我们新建一个`frosted_glass_demo.dart`的文件，然后写入下面的代码，具体的解释已经写到了代码的注释中。

```dart
import 'package:flutter/material.dart';
import 'dart:ui';   //引入ui库，因为ImageFilter Widget在这个里边。

class FrostedGlassDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body:Stack(   //重叠的Stack Widget，实现重贴
        children: <Widget>[
          ConstrainedBox( //约束盒子组件，添加额外的限制条件到 child上。
            constraints: const BoxConstraints.expand(), //限制条件，可扩展的。
            child:Image.network('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1545738629147&di=22e12a65bbc6c4123ae5596e24dbc5d3&imgtype=0&src=http%3A%2F%2Fpic30.photophoto.cn%2F20140309%2F0034034413812339_b.jpg')
          ),
          Center(
            child: ClipRect(  //裁切长方形
              child: BackdropFilter(   //背景滤镜器
                filter: ImageFilter.blur(sigmaX: 5.0,sigmaY: 5.0), //图片模糊过滤，横向竖向都设置5.0
                child: Opacity( //透明控件
                  opacity: 0.5,
                  child: Container(// 容器组件
                    width: 500.0,
                    height: 700.0,
                    decoration: BoxDecoration(color:Colors.grey.shade200), //盒子装饰器，进行装饰，设置颜色为灰色
                    child: Center(
                      child: Text(
                        'JSPang',
                        style: Theme.of(context).textTheme.display3, //设置比较酷炫的字体
                      ),
                    ),
                  ),
                ),
              ),
            ),
          )
        ],
      )
    );
  }
}
```

这个代码嵌套很多，所以一定要注意你的代码层次，容易出错的地方就是嵌套错误。这个效果尽量少用，因为我测试了一下，它对系统的消耗还是比较大的。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第08节-保持页面状态)第08节: 保持页面状态

在工作中切换页面时，再切换回来，时要求页面状态不发生改变的。这能给APP浏览者最好的体验，机会所有的APP都有这个需求，属于一个大众需求。这节课我们就来看看这样的效果如何实现。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_06.gif)



### With 关键字的使用

with是dart的关键字，意思是混入的意思，就是说可以将一个或者多个类的功能添加到自己的类无需继承这些类， 避免多重继承导致的问题。

比如下面的代码：

```text
class _KeepAliveDemoState extends State<KeepAliveDemo> with SingleTickerProviderStateMixin {

}
```

需要注意的是with后边是Mixin，而不是普通的Widget，这个初学者比较爱犯错误。需要强调一下。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#tabbar-widget的使用)TabBar Widget的使用

`TabBar`是切换组件，它需要设置两个属性。

- controller: 控制器，后边跟的多是`TabController`组件。
- tabs：具体切换项，是一个数组，里边放的也是Tab Widget。

```dart
bottom:TabBar(
  controller: _controller,
  tabs:[
    Tab(icon:Icon(Icons.directions_car)),
    Tab(icon:Icon(Icons.directions_transit)),
    Tab(icon:Icon(Icons.directions_bike)),
  ],
)
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#基本页面布局)基本页面布局

我们先把入口页面布局好，下节课我们再让他保持状态。学了上面两个知识，你其实可以做出来布局了。全部代码如下：

```dart
import 'package:flutter/material.dart';

void main()=>runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme:ThemeData(
        primarySwatch: Colors.blue,
      ),
      home:KeepAliveDemo()
    );
  }
} 

class KeepAliveDemo extends StatefulWidget {
  _KeepAliveDemoState createState() => _KeepAliveDemoState();
}
/*
with是dart的关键字，意思是混入的意思，
就是说可以将一个或者多个类的功能添加到自己的类无需继承这些类，
避免多重继承导致的问题。
SingleTickerProviderStateMixin 主要是我们初始化TabController时，
需要用到vsync ，垂直属性，然后传递this
*/
class _KeepAliveDemoState extends State<KeepAliveDemo> with SingleTickerProviderStateMixin {
  TabController _controller;

  @override
  void initState(){
    super.initState();
    _controller = TabController(length:3, vsync: this);
  }

  //重写被释放的方法，只释放TabController
  @override
  void dispose(){
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar:AppBar(
        title:Text('Keep Alive Demo'),
        bottom:TabBar(
          controller: _controller,
          tabs:[
            Tab(icon:Icon(Icons.directions_car)),
            Tab(icon:Icon(Icons.directions_transit)),
            Tab(icon:Icon(Icons.directions_bike)),
          ],
        )
      ),
      body:TabBarView(
        controller: _controller,
        children: <Widget>[
         Text('1111'),
         Text('2222'),
         Text('3333')
        ],
      )
    );
  }
}
```

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第09节-保持页面状态-2)第09节: 保持页面状态-2

上节课已经做出了TabBar的效果，这节课主要任务是保持状态。那为了能看出是保持状态的，我们作一个按钮，然后点一下加一。就跟Flutter为我们生成的例子一样。

我这里就直接贴出代码了，知识点我已经在代码中写了注释，并且我会在视频中详细讲解。

如果小伙伴们对代码有什么不了解的，可以私信，我来进行解答。

```dart
import 'package:flutter/material.dart';

class MyHomePage extends StatefulWidget {

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

//混入AutomaticKeepAliveClientMixin，这是保持状态的关键
//然后重写wantKeppAlive 的值为true。
class _MyHomePageState extends State<MyHomePage> with AutomaticKeepAliveClientMixin {
  int _counter = 0;
  //重写keepAlive 为ture ，就是可以有记忆功能了。
  @override
  bool get wantKeepAlive => true;
  //声明一个内部方法，用来点击按钮后增加数量
  void _incrementCounter(){
    setState((){ _counter++;});
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body:Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('点一下加1，点一下加1:'),
            Text(
              '$_counter',
              style:Theme.of(context).textTheme.display1,
            )
          ],
        ),
      ),
      //增加一个悬浮按钮，点击时触犯_incrementCounter方法
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

写完这个Widget，然后在入口文件中引入。

```dart
import 'keep_alive_demo.dart';
```

然后把Body区域改成我们刚写的MyHomePage Widget 就可以了，注意是改三个。

修改之后的`KeepAliveDemo`代码为：

```dart
class KeepAliveDemo extends StatefulWidget {
  @override
  _KeepAliveDemoState createState() => _KeepAliveDemoState();
}

class _KeepAliveDemoState extends State<KeepAliveDemo> with SingleTickerProviderStateMixin {
  TabController _controller;

@override
  void initState() {
    super.initState();
    _controller = TabController(length: 3, vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Keep Alive Demo'),
        bottom: TabBar(
          controller: _controller,
          tabs: <Widget>[
            Tab(icon: Icon(Icons.directions_car)),
            Tab(icon: Icon(Icons.directions_transit)),
            Tab(icon: Icon(Icons.directions_bike)),
          ],
        ),
      ),
      body: TabBarView(
        controller: _controller,
        children: <Widget>[
          MyHomePage(),
          MyHomePage(),
          MyHomePage(),
        ],
      ),
    );
  }
}
```

