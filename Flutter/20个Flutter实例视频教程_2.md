

【继续】

---



## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第10节-一个不简单的搜索条-1)第10节: 一个不简单的搜索条-1

搜索这个功能，大部分APP都会存在，这节课我们就学习一下，如何做一个有提示功能，而且交互很好的搜索条。需要提前提示的是，这节课的内容可能稍微有些难度，需要你慢慢消化，多作两遍。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_07.gif)

### 主入口文件

这个还是继承`StatelessWidget`,然后在home属性中加入`SearchBarDemo`，这是一个自定义的Widget，主要代码都在这个文件中。

`main.dart`文件的代码如下：

```text
import 'package:flutter/material.dart';
import 'search_bar_demo.dart';

void main() =>runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      theme: ThemeData.light(),
      home: SearchBarDemo()
    );
  }
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#数据文件-asset-dart)数据文件 asset.dart

`asset.dart`相当于数据文件，工作中这些数据是后台传递给我们，或者写成配置文件的，这里我们就以List的方式代替了。我们在这个文件中定义了两个List：

- searchList : 这个相当于数据库中的数据，我们要在这里进行搜索。
- recentSuggest ： 目前的推荐数据，就是搜索时，自动为我们进行推荐。

整体代码如下 ：

```text
const searchList = [
  "jiejie-大长腿",
  "jiejie-水蛇腰",
  "gege1-帅气欧巴",
  "gege2-小鲜肉"
];

const recentSuggest = [
  "推荐-1",
  "推荐-2"
];
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#appbar的样式制作)AppBar的样式制作

这节课我们先把第一个搜索界面布好，下节课我们主要作搜索的交互效果。看下面的代码：

```text
import 'package:flutter/material.dart';
import 'asset.dart';


class SearchBarDemo extends StatefulWidget {
  _SearchBarDemoState createState() => _SearchBarDemoState();
}

class _SearchBarDemoState extends State<SearchBarDemo> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar:AppBar(
        title:Text('SearchBarDemo'),
        actions:<Widget>[
          IconButton(
            icon:Icon(Icons.search),
            onPressed: (){
               print('开始搜索');
            }
          ),
        ]
      )
    );

  }
}
```

这时候就可以在虚拟机中进行预览了，但是这时候点击搜索按钮还没有任何反应。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第11节-一个不简单的搜索条-2)第11节: 一个不简单的搜索条-2

上节课已经作好了搜索条的基本布局，现在就是要在点击搜索图标后，变成搜索条的样式，并且有一定都交互效果。

在点击图标时执行`searchBarDelegate`类，这个类继承与`SearchDelegate`类，继承后要重写里边的四个方法。

### 重写buildActions方法：

`buildActions`方法时搜索条右侧的按钮执行方法，我们在这里方法里放入一个clear图标。 当点击图片时，清空搜索的内容。

代码如下:

```text
  @override
  List<Widget> buildActions(BuildContext context){
    return [
      IconButton(
        icon:Icon(Icons.clear),
        onPressed: ()=>query = "",)
      ];
  }
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#buildleading-方法重写)buildLeading 方法重写

这个时搜索栏左侧的图标和功能的编写，这里我们才用`AnimatedIcon`，然后在点击时关闭整个搜索页面，代码如下。

```text
  @override
  Widget buildLeading(BuildContext context) {
    return IconButton(
        icon: AnimatedIcon(
            icon: AnimatedIcons.menu_arrow, progress: transitionAnimation),
        onPressed: () => close(context, null));
  }
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#buildresults方法重写)buildResults方法重写

`buildResults`方法，是搜到到内容后的展现，因为我们的数据都是模拟的，所以我这里就使用最简单的`Container`+`Card`组件进行演示了，不做过多的花式修饰了。

```text
 @override
  Widget buildResults(BuildContext context) {
    return Container(
      width: 100.0,
      height: 100.0,
      child: Card(
        color: Colors.redAccent,
        child: Center(
          child: Text(query),
        ),
      ),
    );
  }
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#buildsuggestions方法重写)buildSuggestions方法重写

这个方法主要的作用就是设置推荐，就是我们输入一个字，然后自动为我们推送相关的搜索结果，这样的体验是非常好的。(具体代码的解释，请观看视频)

具体代码如下：

```text
 @override
  Widget buildSuggestions(BuildContext context) {
    final suggestionList = query.isEmpty
        ? recentSuggest
        : searchList.where((input) => input.startsWith(query)).toList();
    return ListView.builder(
        itemCount: suggestionList.length,
        itemBuilder: (context, index) => ListTile(
              title: RichText(
                  text: TextSpan(
                      text: suggestionList[index].substring(0, query.length),
                      style: TextStyle(
                          color: Colors.black, fontWeight: FontWeight.bold),
                      children: [
                    TextSpan(
                        text: suggestionList[index].substring(query.length),
                        style: TextStyle(color: Colors.grey))
                  ])),
            ));
  }


}
```

为了方便小伙伴们学习，给出所有`search_bar_demo.dart`文件的代码：

```text
import 'package:flutter/material.dart';
import 'asset.dart';


class SearchBarDemo extends StatefulWidget {
  _SearchBarDemoState createState() => _SearchBarDemoState();
}

class _SearchBarDemoState extends State<SearchBarDemo> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar:AppBar(
        title:Text('SearchBarDemo'),
        actions:<Widget>[
          IconButton(
            icon:Icon(Icons.search),
            onPressed: (){
               showSearch(context:context,delegate: searchBarDelegate());
            }
            // showSearch(context:context,delegate: searchBarDelegate()),
          ),
        ]
      )
    );

  }
}

class searchBarDelegate extends SearchDelegate<String>{
  @override
  List<Widget> buildActions(BuildContext context){
    return [
      IconButton(
        icon:Icon(Icons.clear),
        onPressed: ()=>query = "",)
      ];
  }
  
  @override
  Widget buildLeading(BuildContext context) {
    return IconButton(
        icon: AnimatedIcon(
            icon: AnimatedIcons.menu_arrow, progress: transitionAnimation),
        onPressed: () => close(context, null));
  }

  @override
  Widget buildResults(BuildContext context) {
    return Container(
      width: 100.0,
      height: 100.0,
      child: Card(
        color: Colors.redAccent,
        child: Center(
          child: Text(query),
        ),
      ),
    );
  }

   @override
  Widget buildSuggestions(BuildContext context) {
    final suggestionList = query.isEmpty
        ? recentSuggest
        : searchList.where((input) => input.startsWith(query)).toList();
    return ListView.builder(
        itemCount: suggestionList.length,
        itemBuilder: (context, index) => ListTile(
              title: RichText(
                  text: TextSpan(
                      text: suggestionList[index].substring(0, query.length),
                      style: TextStyle(
                          color: Colors.black, fontWeight: FontWeight.bold),
                      children: [
                    TextSpan(
                        text: suggestionList[index].substring(query.length),
                        style: TextStyle(color: Colors.grey))
                  ])),
            ));
  }


}
```

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第12节-流式布局-模拟添加照片效果)第12节: 流式布局 模拟添加照片效果

这节已一个模拟添加多张照片的小实例，主要学习一下流式布局在Flutter里的应用。如果你作为一个前端开发者，那这节课的内容将非常容易。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_08.gif)

### mediaQuery 媒体查询

使用`meidaQuery`可以很容易的得到屏幕的宽和高，得到宽和高的代码如下：

```text
final width = MediaQuery.of(context).size.width;
final height = MediaQuery.of(context).size.height;
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#wrap流式布局)Wrap流式布局

Flutter中流式布局大概有三种常用方法，这节课先学一下Wrap的流式布局。有的小伙伴会说Wrap中的流式布局可以用Flow很轻松的实现出来，但是Wrap更多的式在使用了Flex中的一些概念，某种意义上说式跟Row、Column更加相似的。

单行的Wrap跟Row表现几乎一致，单列的Wrap则跟Column表现几乎一致。但Row与Column都是单行单列的，Wrap则突破了这个限制，mainAxis上空间不足时，则向crossAxis上去扩展显示。

从效率上讲，Flow肯定会比Wrap高，但Wrap使用起来会更方便一些。

这个会在实例中用到，所以，我在实例中会讲解这个代码。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#gesturedetector-手势操作)GestureDetector 手势操作

`GestureDetector`它式一个Widget，但没有任何的显示功能，而只是一个手势操作，用来触发事件的。虽然很多Button组件是有触发事件的，比如点击，但是也有一些组件是没有触发事件的，比如：Padding、Container、Center这时候我们想让它有触发事件就需要再它们的外层增加一个`GestureDetector`，比如我们让Padding有触发事件，代码如下：

```text
Widget buildAddButton(){
    return  GestureDetector(
      onTap:(){
        if(list.length<9){
          setState(() {
                list.insert(list.length-1,buildPhoto());
          });
        }
      },
      child: Padding(
        padding:const EdgeInsets.all(8.0),
        child: Container(
          width: 80.0,
          height: 80.0,
          color: Colors.black54,
          child: Icon(Icons.add),
        ),
      ),
    );
  }
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#入口文件)入口文件

入口文件很简单，就是引用了`warp_demo.dart`文件，然后再home属性中使用了`WarpDemo`，代码如下：

```text
import 'package:flutter/material.dart';
import 'warp_demo.dart';

void main()=>runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: new ThemeData.dark(),
      home:WarpDemo()
    );
  }
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#warp-demo-dart)warp_demo.dart

主要的文件代码我就列在下面了，方便小伙伴们学习。讲解还式主要看视频吧。

```text
import 'package:flutter/material.dart';


//继承与动态组件
class WarpDemo extends StatefulWidget {
  _WarpDemoState createState() => _WarpDemoState();
}

class _WarpDemoState extends State<WarpDemo> {
  List<Widget> list;  //声明一个list数组

  @override
  //初始化状态，给list添加值，这时候调用了一个自定义方法`buildAddButton`
  void initState() { 
    super.initState();
    list = List<Widget>()..add(buildAddButton());
  }


  @override
  Widget build(BuildContext context) {
    //得到屏幕的高度和宽度，用来设置Container的宽和高
    final width = MediaQuery.of(context).size.width;
    final height = MediaQuery.of(context).size.height;
    


    return Scaffold(
      appBar: AppBar(
        title: Text('Wrap流式布局'),
      ),
      body:Center(
        child: Opacity(
          opacity: 0.8,
          child: Container(
            width: width,
            height: height/2,
            color: Colors.grey,
            child: Wrap(    //流式布局，
              children: list,
              spacing: 26.0,  //设置间距
            ),
          ),
        ),
      )
    );
  }

  Widget buildAddButton(){
    //返回一个手势Widget，只用用于显示事件
    return  GestureDetector(
      onTap:(){
        if(list.length<9){
          setState(() {
                list.insert(list.length-1,buildPhoto());
          });
        }
      },
      child: Padding(
        padding:const EdgeInsets.all(8.0),
        child: Container(
          width: 80.0,
          height: 80.0,
          color: Colors.black54,
          child: Icon(Icons.add),
        ),
      ),
    );
  }


  Widget buildPhoto(){
    return Padding(
        padding: const EdgeInsets.all(8.0),
        child: Container(
          width: 80.0,
          height: 80.0,
          color: Colors.amber,
          child: Center(
            child: Text('照片'),
          ),
        ),
    );
  }

}
```

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第13节-展开闭合案例)第13节: 展开闭合案例

手机的屏幕本身就很小，所以要合理利用空间，把主要的元素展示出来，次要或者不重要的元素等用户向看的时候再给用户展示。这类操作最常见的交互就是展开和闭合了。这节课我们主要学习一下`ExpansionTile`组件的使用。



![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_09.gif)

### ExpansionTile组件

`ExpansionTile Widget`就是一个可以展开闭合的组件，常用的属性有如下几个。

- title:闭合时显示的标题，这个部分经常使用`Text Widget`。
- leading:标题左侧图标，多是用来修饰，让界面显得美观。
- backgroundColor: 展开时的背景颜色，当然也是有过度动画的，效果非常好。
- children: 子元素，是一个数组，可以放入多个元素。
- trailing ： 右侧的箭头，你可以自行替换但是我觉的很少替换，因为谷歌已经表现的很完美了。
- initiallyExpanded: 初始状态是否展开，为true时，是展开，默认为false，是不展开。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#main-dart入口文件)main.dart入口文件

```text
import 'package:flutter/material.dart';
import 'expansion_tile.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      theme: new ThemeData.dark(),
      home:ExpansionTileDemo()
    );
  }
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#expansion-tile-dart-扩展组件)expansion_tile.dart 扩展组件

这个文件是我们的主要学习文件，但是并不复杂，就是一个可展开组件。全部代码如下:

```text
import 'package:flutter/material.dart';

class ExpansionTileDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title:Text('expansion tile demo')),
      body:Center(
        child: ExpansionTile(
          title:Text('Expansion Tile'),
          leading:Icon(Icons.ac_unit),
          backgroundColor: Colors.white12,
          children: <Widget>[
            ListTile(
              title:Text('list tile'),
              subtitle:Text('subtitle')
            )
          ],
          initiallyExpanded: true,
        )
      )
    );
  }
}
```

这时候就可以预览了，效果也应该出现了。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第14节-展开闭合列表案例)第14节: 展开闭合列表案例

上节课学的只是一个单个的展开闭合组件，你当然可以把这个组件作为List元素，组成一个数组，形成列表。不过Flutter也很贴心的为提供了一个ExpansionPanelList Widget，它可以实现展开闭合的列表功能。

需要注意的是这个列表必须放在可滑动组件中使用，否则会报错。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_10.gif)



### ExpansionPanelList 常用属性

- expansionCallback:点击和交互的回掉事件，有两个参数，第一个是触发动作的索引，第二个是布尔类型的触发值。
- children:列表的子元素，里边多是一个List数组。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#expandstatebean-自定义类)ExpandStateBean 自定义类

为了方便管理制作了一个`ExpandStateBean`类，里边就是两个状态，一个是是否展开`isOpen`,另一个索引值。代码如下:

```text
class ExpandStateBean{
  var isOpen;
  var index;
  ExpandStateBean(this.index,this.isOpen);
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#expansion-panel-list-dart)expansion_panel_list.dart

这个文件我就直接上代码了，讲解我会在视频里说明，代码中我也进行了详细的注释。

```text
import 'package:flutter/material.dart';

class ExpansionPanelListDemo extends StatefulWidget {
  _ExpansionPanelListDemoState createState() => _ExpansionPanelListDemoState();
}

class _ExpansionPanelListDemoState extends State<ExpansionPanelListDemo> {
  var currentPanelIndex = -1;
  List<int> mList;   //组成一个int类型数组，用来控制索引
  List<ExpandStateBean> expandStateList;    //开展开的状态列表， ExpandStateBean是自定义的类
  //构造方法，调用这个类的时候自动执行
  _ExpansionPanelListDemoState(){
    mList = new List(); 
    expandStateList = new List();
    //便利为两个List进行赋值
    for(int i=0;i<10;i++){
      mList.add(i);
      expandStateList.add(ExpandStateBean(i,false));
    }
  }
   //修改展开与闭合的内部方法
  _setCurrentIndex(int index, isExpand){
    setState(() {
          //遍历可展开状态列表
          expandStateList.forEach((item){
            if(item.index==index){
              //取反，经典取反方法
              item.isOpen = !isExpand;
            }
          });
        });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar:AppBar(
        title:Text("expansion panel list")
      ),
      //加入可滚动组件
      body:SingleChildScrollView(
        child: ExpansionPanelList(
          //交互回掉属性，里边是个匿名函数
          expansionCallback: (index,bol){
            //调用内部方法
            _setCurrentIndex(index, bol);
          },
          children: mList.map((index){//进行map操作，然后用toList再次组成List
            return ExpansionPanel(
              headerBuilder: (context,isExpanded){
                return ListTile(
                  title:Text('This is No. $index')
                );
              },
              body:ListTile(
                title:Text('expansion no.$index')
              ),
              isExpanded: expandStateList[index].isOpen
            );
          }).toList(),
        ),
      )

    );
  }
}
//自定义扩展状态类
class ExpandStateBean{
  var isOpen;
  var index;
  ExpandStateBean(this.index,this.isOpen);
}
```

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第15节-贝塞尔曲线切割)第15节: 贝塞尔曲线切割

现在人们对于网站的美感要求是越来越高了，所以很多布局需要优美的曲线设计。当然最简单的办法是作一个PNG的透明图片，然后外边放一个`Container`.但其内容如果本身就不是图片，只是容器，这种放入图片的做法会让包体变大。其实我们完全可以使用贝塞尔曲线进行切割。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_11.png)



### 去掉DeBug图标

在讲正式内容之前，先回答小伙伴们的一个问题，就是如何去掉DeBug图标。在我们进行编写代码预览时，有一Debug的图标一直在屏幕上，确实不太美观，其实只要语句代码就可以去掉的。

```text
 debugShowCheckedModeBanner: false,
```

这个代码要配置在主入口文件里，全部代码如下“

```text
import 'package:flutter/material.dart';
import 'custom_clipper.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      debugShowCheckedModeBanner: false,
      theme:ThemeData(
        primarySwatch: Colors.blue,
      ),
      home:HomePage()
    );
  }
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#clippath-路径裁切控件)ClipPath 路径裁切控件

`clipPath`控件可以把其内部的子控件切割，它有两个主要属性（参数）:

- child :要切割的元素，可以是容器，图片.....
- clipper : 切割的路径，这个要和CustomClipper对象配合使用。
- 

```text
import 'package:flutter/material.dart';

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body:Column(
        children: <Widget>[
          ClipPath(
            clipper:BottomClipper(),
            child: Container(
              color:Colors.deepPurpleAccent,
              height: 200.0,
            ),
          )
        ],
      )
    );
  }
}
```

在`Scaffold`里放置了一个列容器，然后把`ClipPath`控件放到了里边，`ClipPath`的子元素是一个容器控件`Container`。`BootomClipper`是我们自定义的一个对象，里边主要就是切割的路径。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#customclipper-裁切路径)CustomClipper 裁切路径

我们主要的贝塞尔曲线路径就写在`getClip`方法里，它返回一段路径。

一个二阶的贝塞尔曲线是需要控制点和终点的，控制点就像一块磁铁，把直线吸引过去，形成一个完美的弧度，这个弧度就是贝塞尔曲线了。

我们先来熟悉一下裁切路径和贝塞尔曲线，作一个最简单的贝塞尔曲线出来。

全部代码如下：

```text
import 'package:flutter/material.dart';

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body:Column(
        children: <Widget>[
          ClipPath(
            clipper:BottomClipper(),
            child: Container(
              color:Colors.deepPurpleAccent,
              height: 200.0,
            ),
          )
        ],
      )
    );
  }
}

class BottomClipperTest extends CustomClipper<Path>{
  @override
    Path getClip(Size size) {
      // TODO: implement getClip
      var path = Path();
      path.lineTo(0, 0);
      path.lineTo(0, size.height-30);
      var firstControlPoint =Offset(size.width/2,size.height);
      var firstEndPoint = Offset(size.width,size.height-30);

      path.quadraticBezierTo(firstControlPoint.dx, firstControlPoint.dy, firstEndPoint.dx, firstEndPoint.dy);

      path.lineTo(size.width, size.height-30);
      path.lineTo(size.width, 0);
    
      return path;

    }
    @override
      bool shouldReclip(CustomClipper<Path> oldClipper) {
        // TODO: implement shouldReclip
        return false;
      }

}
```

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第16节-波浪形式的贝塞尔曲线)第16节: 波浪形式的贝塞尔曲线

上节课已经对知识点有了了解，这节课我们主要就是加大一些难度，作个更复杂的贝塞尔裁切出来。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_12.png)

这节课主要改造上节课的代码，作一个波浪形的贝塞尔裁切。波浪形式的只要把裁切变成两个对称的贝塞尔曲线就可以实现了。代码如下：（具体方法视频）

```text
class BottomClipper extends CustomClipper<Path>{
  @override
    Path getClip(Size size) {
      // TODO: implement getClip
      var path = Path();
      path.lineTo(0, size.height-20);
      var firstControlPoint =Offset(size.width/4,size.height);
      var firstEndPoint = Offset(size.width/2.25,size.height-30);

      path.quadraticBezierTo(firstControlPoint.dx, firstControlPoint.dy, firstEndPoint.dx, firstEndPoint.dy);

      var secondControlPoint = Offset(size.width/4*3,size.height-80);
      var secondEndPoint = Offset(size.width,size.height-40);

      path.quadraticBezierTo(secondControlPoint.dx, secondControlPoint.dy, secondEndPoint.dx, secondEndPoint.dy);

      path.lineTo(size.width, size.height-40);
      path.lineTo(size.width, 0);

      return path;

    }
    @override
      bool shouldReclip(CustomClipper<Path> oldClipper) {
        // TODO: implement shouldReclip
        return false;
      }

}
```

这两节课的主要内容就是如何裁切和贝塞尔曲线的原理，其实裁切还有圆形裁切、圆角裁切和矩形裁切，因为都比较容易，我就不再讲解了。感兴趣的小伙伴可以自学一下。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第17节：打开应用的闪屏动画案例)第17节：打开应用的闪屏动画案例

打开一个APP，经常会看到精美的启动页，这种启动页也称为闪屏动画。它是从无到有有一个透明度的渐变动画的。图像展示完事后，才跳转到用户可操作的页面。这节课主要学习一下闪屏动画的制作。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_13.gif)

### AnimationController

`AnimationController`是`Animation`的一个子类，它可以控制`Animation`, 也就是说它是来控制动画的，比如说控制动画的执行时间。

我们这里有了两个参数 ：

- `vsync:this`:垂直同步设置，使用this就可以了。
- `duration`: 动画持续时间，这个可以使用`seconds`秒，也可以使用`milliseconds`毫秒，工作中经常使用毫秒，因为秒还是太粗糙了。

来看一段代码，这段代码就是控制动画的一个典型应用。

```text
 _controller = AnimationController(vsync:this,duration:Duration(milliseconds:3000));
_animation = Tween(begin: 0.0,end:1.0).animate(_controller);
```

这段代码的意思是，设置一个动画控制器，这个动画控制器控制动画执行时间是3000毫秒。然后我们设置了一段动画，动画使用了动画控制器的约定。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#animation-addstatuslistener)animation.addStatusListener

`animation.addStatusListener`动画事件监听器，它可以监听到动画的执行状态，我们这里只监听动画是否结束，如果结束则执行页面跳转动作。

```text
_animation.addStatusListener((status){
  if(status == AnimationStatus.completed){
    Navigator.of(context).pushAndRemoveUntil(
      MaterialPageRoute(builder: (context)=>MyHomePage()), 
      (route)=> route==null);
  }
});
```

- `AnimationStatus.completed`:表示动画已经执行完毕。
- `pushAndRemoveUntil`:跳转页面，并销毁当前控件。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#案例重要代码)案例重要代码

我们会了上边的知识点，做出案例就没什么问题了。我把复杂的代码都作了注释，小伙伴们应该可以看懂。

main.dart文件

```text
import 'package:flutter/material.dart';
import 'splash_screen.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch:Colors.blue,
      ),
      home:SplashScreen()
    );
  }
}
```

splash_screen.dart文件

```text
import 'package:flutter/material.dart';
import 'home_page.dart';

class SplashScreen extends StatefulWidget {
  _SplashScreenState createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> with SingleTickerProviderStateMixin {

  AnimationController _controller;
  Animation _animation;

  void initState() { 
    super.initState();
    _controller = AnimationController(vsync:this,duration:Duration(milliseconds:3000));
    _animation = Tween(begin: 0.0,end:1.0).animate(_controller);


    /*动画事件监听器，
    它可以监听到动画的执行状态，
    我们这里只监听动画是否结束，
    如果结束则执行页面跳转动作。 */
    _animation.addStatusListener((status){
      if(status == AnimationStatus.completed){
        Navigator.of(context).pushAndRemoveUntil(
          MaterialPageRoute(builder: (context)=>MyHomePage()), 
          (route)=> route==null);
      }
    });
    //播放动画
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }


  @override
  Widget build(BuildContext context) {
    return FadeTransition( //透明度动画组件
      opacity: _animation,  //执行动画
      child: Image.network(  //网络图片
        'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1546851657199&di=fdd278c2029f7826790191d59279dbbe&imgtype=0&src=http%3A%2F%2Fimg.zcool.cn%2Fcommunity%2F0112cb554438090000019ae93094f1.jpg%401280w_1l_2o_100sh.jpg',
        scale: 2.0,  //进行缩放
        fit:BoxFit.cover  // 充满父容器
      ),
    );
  }
}
```

home_page.dart文件

```text
import 'package:flutter/material.dart';

class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title:Text('首页')),
      body:Center(
        child: Text('我是首页')
      )
    );
  }
}
```

希望小伙伴都能练习一下，做出这个动态图片的效果。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第18节：右滑返回上一页案例)第18节：右滑返回上一页案例

在苹果手机上有一种才操作叫做右滑返回上一页，一些不错的应用都有类似的操作，随着苹果手机越来越多，这种操作开始普遍开来，在安卓手机上也开始使用。这节就来学习一下如何实现这种右滑返回上一页的操作。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_14.gif)

### Cupertino UI

其实早都知道Flutter有两套UI模板，一套是`material`,另一套就是`Cupertino`。`Cupertino`主要针对的的就是IOS系统的UI，所以用的右滑返回上一级就是在这个`Cupertino`里。

**Cupertino的引入方法**

直接使用import引入就可以了，代码如下:

```text
import 'package:flutter/cupertino.dart';
```

引入了`cupertino`的包之后，就可以使用皮肤和交互效果的特性了。要用的右滑返回上一页也是皮肤的交互特性，直接使用就可以了。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#cupertinopagescaffold)CupertinoPageScaffold

这个和以前使用`material`的`Scaffold`类似，不过他里边的参数是`child`，例如下面的代码.

```text
return CupertinoPageScaffold(
  child: Center(
    child: Container(
      height: 100.0,
      width:100.0,
      color: CupertinoColors.activeBlue,
      child: CupertinoButton(
        child: Icon(CupertinoIcons.add),
        onPressed: (){
          Navigator.of(context).push(
            CupertinoPageRoute(builder: (BuildContext context){
              return RightBackDemo();
            })
          );
        },
      ),
    ),
  ),
);
```

在`Cupertino`下也有很多`Widget`控件，他们都是以`Cupertino`开头的，这就让我们很好区分，当然两种皮肤是可以进行混用的。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#整个案例代码)整个案例代码

这个案例就两个主要文件，`main.dart`和`right_back_demo.dart`，具体解释可以看视频，代码如下:

**main.dart文件代码**

```text
import 'package:flutter/material.dart';
import 'right_back_demo.dart';

void main()=>runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      theme:ThemeData(primarySwatch: Colors.blue),
      home: RightBackDemo(),
    );
  }
}
```

**right_back_demo。dart文件**

```text
import 'package:flutter/cupertino.dart'; 

class RightBackDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return CupertinoPageScaffold(
      child: Center(
        child: Container(
          height: 100.0,
          width:100.0,
          color: CupertinoColors.activeBlue,
          child: CupertinoButton(
            child: Icon(CupertinoIcons.add),
            onPressed: (){
              Navigator.of(context).push(
                CupertinoPageRoute(builder: (BuildContext context){
                  return RightBackDemo();
                })
              );
            },
          ),
        ),
      ),
    );
  }
}
```

其实只要使用`CupertinoPageRoute`打开的子页面，就可以实现右滑返回上一级。所以整个案例并不难。希望小伙伴们有所收获。

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#第19节：tooltip控件实例)第19节：ToolTip控件实例

轻提示的效果在应用中式少不了的，其实Flutter已经准备好了轻提示控件，这就是`toolTip`。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_15.gif)

### 轻量级操作提示

其实Flutter中有很多提示控件,比如`Dialog`、`Snackbar`和`BottomSheet`这些操作都是比较重量级的，存在屏幕上的时间较长或者会直接打断用户的操作。

当然我并不是说这些控件不好，根据需求的不同，要有多种选择，所以才会给大家讲一下轻量级操作提示`Tooltip`

`Tooltip`是继承于`StatefulWidget`的一个Widget，它并不需要调出方法，当用户长按被`Tooltip`包裹的Widget时，会自动弹出相应的操作提示。

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#main-dart文件)main.dart文件

这节课就用最简单代码给大家做一个轻量级提示案例出来。入口文件代码如下：

```text
import 'package:flutter/material.dart';
import 'tool_tips_demo.dart';

void main()=> runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      theme:ThemeData.light(),
      home: ToolTipDemo(),
    );
  }
}
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#tool-tips-demo-dart文件)tool_tips_demo.dart文件

这个文件主要的用途就是承载轻提示控件，代码如下：

```text
import 'package:flutter/material.dart';

class ToolTipDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title:Text('tool tips demo')),
      body:Center(
        child: Tooltip(
          message: '不要碰我，我怕痒',
          child: Icon(Icons.all_inclusive),
        ),
      )
    );
  }
}
```

## [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#_20节draggable控件实例)20节Draggable控件实例

Flutter提供了强大的拖拽控件，可以灵活定制，并且非常简单。这节课就学习一下Flutter拖拽控件，并根据学到的知识作一个拖拽案例。

这节课也是我们这个文章的最后一节，一共讲了20节，这20节算是一个Flutter进阶，是入门和实战的一个桥梁，在课程中我尽量靠近了平时工作的开发环境，无论hi编辑工具还是区分模块，都以实际工作为出发点，我相信只要你用心学习了这20节，一定是有所收获的。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/FlutterDemo2_16.gif)

### Draggable Widget

`Draggable`控件负责就是拖拽，父层使用了`Draggable`，它的子元素就是可以拖动的，子元素可以实容器，可以是图片。用起来非常的灵活。

参数说明：

- `data`: 是要传递的参数，在`DragTarget`里，会接受到这个参数。当然要在拖拽控件推拽到`DragTarget`的时候。
- `child`:在这里放置你要推拽的元素，可以是容器，也可以是图片和文字。
- `feedback`: 常用于设置推拽元素时的样子，在案例中当推拽的时候，我们把它的颜色透明度变成了50%。当然你还可以改变它的大小。
- `onDraggableCanceled`:是当松开时的相应事件，经常用来改变推拽时到达的位置，改变时用`setState`来进行。

代码:

```text
Draggable(
  data:widget.widgetColor,
  child: Container(
    width: 100,
    height: 100,
    color:widget.widgetColor,
  ),
  feedback:Container(
    width: 100.0,
    height: 100.0,
    color: widget.widgetColor.withOpacity(0.5),
  ),
  onDraggableCanceled: (Velocity velocity, Offset offset){
    setState(() {
      this.offset = offset;
    });
  },
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#dragtarget-widget)DragTarget Widget

`DragTarget`是用来接收拖拽事件的控件，当把`Draggable`放到`DragTarget`里时，他会接收`Draggable`传递过来的值，然后用生成器改变组件状态。

- onAccept:当推拽到控件里时触发，经常在这里得到传递过来的值。
- builder: 构造器，里边进行修改child值。

```text
DragTarget(onAccept: (Color color) {
  _draggableColor = color;
}, builder: (context, candidateData, rejectedData) {
  return Container(
    width: 200.0,
    height: 200.0,
    color: _draggableColor,
  );
}),
```

### [#](https://jspang.com/posts/2019/02/22/flutterdemo.html#实例代码demo)实例代码DEMO

main.dart 文件：

```text
import 'package:flutter/material.dart';
import 'draggable_demo.dart';

void main() => runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title:'Flutter Demo',
      theme:ThemeData(
        primarySwatch: Colors.blue
      ),
      home:DraggableDemo()
    );
  }
}
```

draggable_demo.dart 文件

```text
import 'package:flutter/material.dart';

import 'draggable_widget.dart';

class DraggableDemo extends StatefulWidget {
  @override
  _DraggableDemoState createState() => _DraggableDemoState();
}

class _DraggableDemoState extends State<DraggableDemo> {
  Color _draggableColor = Colors.grey;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: Stack(
      children: <Widget>[
        DraggableWidget(
          offset: Offset(80.0, 80.0),
          widgetColor: Colors.tealAccent,
        ),
        DraggableWidget(
          offset: Offset(180.0, 80.0),
          widgetColor: Colors.redAccent,
        ),
        Center(
          child: DragTarget(onAccept: (Color color) {
            _draggableColor = color;
          }, builder: (context, candidateData, rejectedData) {
            return Container(
              width: 200.0,
              height: 200.0,
              color: _draggableColor,
            );
          }),
        )
      ],
    ));
  }
}
```

draggable_widget.dart 文件

```text
import 'package:flutter/material.dart';

class DraggableWidget extends StatefulWidget {
  final Offset offset;
  final Color widgetColor;
  const DraggableWidget({Key key, this.offset, this.widgetColor}):super(key:key);
  _DraggableWidgetState createState() => _DraggableWidgetState();
}

class _DraggableWidgetState extends State<DraggableWidget> {
  Offset offset = Offset(0.0,0.0);
  @override
  void initState() {
    super.initState();
    offset = widget.offset;
  }

  @override
  Widget build(BuildContext context) {
   return Positioned(
     left: offset.dx,
     top:offset.dy,
     child: Draggable(
       data:widget.widgetColor,
       child: Container(
         width: 100,
         height: 100,
         color:widget.widgetColor,
       ),
       feedback:Container(
         width: 100.0,
         height: 100.0,
         color: widget.widgetColor.withOpacity(0.5),
       ),
       onDraggableCanceled: (Velocity velocity, Offset offset){
         setState(() {
            this.offset = offset;
         });
       },
     ),
   );
  }
}
```

这个案例做完，我们这篇文章就算结束了，我也希望你能有所收获。但是看一遍肯定什么都学不会，关键的是自己动手作一下，才可以有所收获。我在文章和课程中反复强调，希望小伙伴们真的能学会。