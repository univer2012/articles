来自：[Flutter免费视频第二季-常用组件](https://jspang.com/posts/2019/01/21/flutter-base2.html)

---

## 第01节：Text Widget 文本组件的使用

### TextAlign属性

TextAlign属性就是文本的对齐方式，它的属性值有如下几个（详细请看视频中讲解）：

- center: 文本以居中形式对齐,这个也算比较常用的了。
- left:左对齐，经常使用，让文本居左进行对齐，效果和start一样。
- right :右对齐，使用频率也不算高。
- start:以开始位置进行对齐，类似于左对齐。
- end: 以为本结尾处进行对齐，不常用。有点类似右对齐.

总结起来，也就算三个对齐方式，left(左对齐)、center（居中对齐）、right（右对齐）。我们来看一下具体代码：

```dart
child:Text(
  'Hello JSPang  ,非常喜欢前端，并且愿意为此奋斗一生。我希望可以出1000集免费教程。',
  textAlign:TextAlign.left,
)
```

### maxLines属性

设置最多显示的行数，比如我们现在只显示1行，类似一个新闻列表的题目。代码如下：

```dart
child:Text(
  'Hello JSPang  ,非常喜欢前端，并且愿意为此奋斗一生。我希望可以出1000集免费教程。',
  textAlign:TextAlign.left,
  maxLines: 1,
)
```

### overflow属性

overflow属性是用来设置文本溢出时，如何处理,它有下面几个常用的值供我们选择。

- clip：直接切断，剩下的文字就没有了，感觉不太友好，体验性不好。
- ellipsis:在后边显示省略号，体验性较好，这个在工作中经常使用。
- fade: 溢出的部分会进行一个渐变消失的效果，当然是上线的渐变，不是左右的哦。

### style属性

style属性的内容比较多，具体的你可以查一下API，我这里带作一个效果，方便大家快速学会Style的用法。

我们下面要作的效果为，字体大小为25.0,颜色为粉红色,并且有一个下划线。
```dart
child:Text(
  'Hello JSPang  ,非常喜欢前端，并且愿意为此奋斗一生。我希望可以出1000集免费教程。',
  textAlign:TextAlign.left,
  overflow:TextOverflow.ellipsis,
  maxLines: 1,
  style: TextStyle(
    fontSize:25.0,
    color:Color.fromARGB(255, 255, 150, 150),
    decoration:TextDecoration.underline,
    decorationStyle:TextDecorationStyle.solid,
  ),
)
```

更详细的属性资料可以参看这个网址：https://docs.flutter.io/flutter/painting/TextStyle-class.html

其实程序员最重要的一个能力就是查看文档的能力，我这里也只是讲些最常使用的，在实际工作中，还是要多查稳定。



## 第02节：Container容器组件的使用1
Container（容器控件）在Flutter是经常使用的控件，它就相当于我们HTML里的<div>标签，每个页面或者说每个视图都离不开它。那这节课我们就来学习一下。

### Alignment属性

其实容器的作用就是方便我们进行布局的，Flutter这点也作的很好，我们先来看容器属性中的`Alignment`。

这个属性**针对的是Container内child的对齐方式，也就是容器子内容的对齐方式，并不是容器本身的对齐方式。**

先作一个效果：建立一个容器，然后容器内加入一段文字`Hello JSPang`, 并让它居中对齐。

具体代码如下：

```dart
return MaterialApp(
  title:'Text widget',
  home:Scaffold(
    body:Center(
     child:Container(
       child:new Text('Hello JSPang',style: TextStyle(fontSize: 40.0),),
       alignment: Alignment.center,
     ),
    ),
  ),
);
```

这时候可以看见，我们的文本已经居中显示在手机屏幕上了。当然它的对齐方式还有如下几种：

- `bottomCenter`:下部居中对齐。
- `botomLeft`: 下部左对齐。
- `bottomRight`：下部右对齐。
- `center`：纵横双向居中对齐。
- `centerLeft`：纵向居中横向居左对齐。
- `centerRight`：纵向居中横向居右对齐。
- `topLeft`：顶部左侧对齐。
- `topCenter`：顶部居中对齐。
- `topRight`： 顶部居左对齐。



### 设置宽、高和颜色属性

设置宽、高和颜色属性是相对容易的，只要在属性名称后面加入浮点型数字就可以了，比如要设置宽是500，高是400，颜色为亮蓝色。代码如下：

```dart
child:Container(
  child:new Text('Hello JSPang',style: TextStyle(fontSize: 40.0),),
  alignment: Alignment.center,
  width:500.0,
  height:400.0,
  color: Colors.lightBlue,
),
```

实现的效果如图所示：

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/flutter_di2ji_1.png)



## 第03节：Container容器组件的使用2

上节已经简单的学习了一下Container容器组件的用法，这节我们继续学习，**主要讲解一下的`padding`,`margin`和`decoration`这几个属性**。我们先来看看Padding属性。

### padding属性

<font color=#FF0000>padding的属性就是一个内边距，它和你使用的前端技术CSS里的`padding`表现形式一样，指的是Container边缘和child内容的距离</font>。先来看一个内边距为10的例子。具体代码如下(我们还是接着上节课的代码来写)：

```dart
child:Container(
  child:new Text('Hello JSPang',style: TextStyle(fontSize: 40.0),),
  alignment: Alignment.topLeft,
  width:500.0,
  height:400.0,
  color: Colors.lightBlue,
  padding:const EdgeInsets.all(10.0),
),
```

上面主要的padding代码就一句。

```dart
padding:const EdgeInsets.all(10.0),
```

这句的意思是设置`Container`的内边距是10，左右上下全部为10，这看起来非常容易。那我们再加大一点难度。如果上边距为30，左边距为10，这时候`EdgeInsets.all()`就满足不了我们了。

**`EdgeInsets.fromLTRB(value1,value2,value3,value4)`**

我们用`EdgeInsets.fromLTRB(value1,value2,value3,value4)`可以满足我们的需求，**`LTRB`分别代表左、上、右、下。**

那我们设置上边距为30，左边距为10，就可以用下面的代码来编写。

```dart
padding:const EdgeInsets.fromLTRB(10.0, 30.0, 0.0, 0.0),
```

这时候我们的结果就变成了下图。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/flutter_di2ji_2.png)

有了这两个属性，基本就可以满足我们的工作需要了。



### margin属性

会了padding属性的设置，margin就变的非常容易了，因为方法基本上一样。<font color=#FF0000>不过margin是外边距，指的是container和外部元素的距离</font>。

现在要把container的外边距设置为10个单位，代码如下:

```dart
child:Container(
  child:new Text('Hello JSPang',style: TextStyle(fontSize: 40.0),),
  alignment: Alignment.topLeft,
  width:500.0,
  height:400.0,
  color: Colors.lightBlue,
  padding:const EdgeInsets.fromLTRB(10.0,30.0,0.0,0.0),
  margin: const EdgeInsets.all(10.0),
),
```

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/flutter_di2ji_3.png)

图中可以看出，已经有了明显的外边距。

当然你也可以分别设置不同的外边距，方法也是使用`fromLTRB`，我在视频中演示，这里就不累述了。



### decoration属性

<font color=#FF0000>`decoration`是 container 的修饰器，主要的功能是设置背景和边框。</font>

比如你需要给背景加入一个渐变，这时候需要使用BoxDecoration这个类，代码如下（需要注意的是如果你设置了decoration，就不要再设置color属性了，因为这样会冲突）。

```dart
child:Container(
  child:new Text('Hello JSPang',style: TextStyle(fontSize: 40.0),),
  alignment: Alignment.topLeft,
  width:500.0,
  height:400.0,
  padding:const EdgeInsets.fromLTRB(10.0,30.0,0.0,0.0),
  margin: const EdgeInsets.all(10.0),
+ decoration:new BoxDecoration(
+   gradient:const LinearGradient(
+     colors:[Colors.lightBlue,Colors.greenAccent,Colors.purple]
+   )
+ ),
),
```

上面的代码去掉了color的设置，这时候container的背景就变成了渐变颜色，如下图。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/flutter_di2ji_4.png)



**设置边框**

设置边框可以在decoration里设置border属性，比如你现在要设置一个红色边框，宽度为2。代码如下：

```dart
child:Container(
  child:new Text('Hello JSPang',style: TextStyle(fontSize: 40.0),),
  alignment: Alignment.topLeft,
  width:500.0,
  height:400.0,
  padding:const EdgeInsets.fromLTRB(10.0,30.0,0.0,0.0),
  margin: const EdgeInsets.all(10.0),
  decoration:new BoxDecoration(
    gradient:const LinearGradient(
      colors:[Colors.lightBlue,Colors.greenAccent,Colors.purple]
    ),
    border:Border.all(width:2.0,color:Colors.red)
  ),
),
```

关键代码其实就是：

```dart
border:Border.all(width:2.0,color:Colors.red)
```

这时候就有了边框显示，我就不给大家上图片了，如果有需要视频中查看吧。

这节课就到这里，希望小伙伴们都能动手练习起来。在这里附上全部代码，方便小伙伴们学习。

```dart
import 'package:flutter/material.dart';
void main () => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context ){
      return MaterialApp(
        title:'Text widget',
        home:Scaffold(
          body:Center(
          child:Container(
            child:new Text('Hello JSPang',style: TextStyle(fontSize: 40.0),),
            alignment: Alignment.topLeft,
            width:500.0,
            height:400.0,
            padding:const EdgeInsets.fromLTRB(10.0,30.0,0.0,0.0),
            margin: const EdgeInsets.all(10.0),
            decoration:new BoxDecoration(
              gradient:const LinearGradient(
                colors:[Colors.lightBlue,Colors.greenAccent,Colors.purple]
              ),
              border:Border.all(width:2.0,color:Colors.red)
            ),
          ),
          ),
        ),
      );
  }
}
```



## 第04节：Image图片组件的使用

我们都希望自己作的程序赏心悦目，俗话说的好一图顶千言，所以图片的运用在程序制作里至关重要。这节课我们就来看一下图片组件的使用。

### 加入图片的几种方式

- **Image.asset**:加载资源图片，就是加载项目资源目录中的图片,加入图片后会增大打包的包体体积，用的是相对路径。
- **Image.network**:网络资源图片，意思就是你需要加入一段http://xxxx.xxx的这样的网络路径地址。
- **Image.file**:加载本地图片，就是加载本地文件中的图片，这个是一个绝对路径，跟包体无关。
- **Image.memory**: 加载Uint8List资源图片,这个我目前用的不是很多，所以没什么发言权。

我们现在就以加入网络图片为例子，在Container里加入一张图片，代码如下：



```dart
import 'package:flutter/material.dart';
void main () => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context ){
      return MaterialApp(
        title:'Text widget',
        home:Scaffold(
          body:Center(
          child:Container(
            child:new Image.network(
              'https://img4.mukewang.com/szimg/5d1032ab08719e0906000338-360-202.jpg',
              scale:1.0,
            ),
            width:300.0,
            height:200.0,
            color: Colors.lightBlue,
          ),
          ),
        ),
      );
  }
}
```

这时候就可以看到图片被加入进来了，当然我们还顺便设置了容器的宽和高。



### fit属性的设置

fit属性可以控制图片的拉伸和挤压，这些都是根据图片的父级容器来的，我们先来看看这些属性（建议此部分组好看视频理解）。

- **BoxFit.fill**:全图显示，图片会被拉伸，并充满父容器。
- **BoxFit.contain**:全图显示，显示原比例，可能会有空隙。
- **BoxFit.cover**：显示可能拉伸，可能裁切，充满（图片要充满整个容器，还不变形）。
- **BoxFit.fitWidth**：宽度充满（横向充满），显示可能拉伸，可能裁切。
- **BoxFit.fitHeight**：高度充满（竖向充满）,显示可能拉伸，可能裁切。
- **BoxFit.scaleDown**：效果和contain差不多，但是此属性不允许显示超过源图片大小，可小不可大。

这部分我会在视频中充分演示，每一个效果都会作对应的演示，建议收看视频进行理解。



### 图片的混合模式

<font color=#FF0000>图片混合模式（colorBlendMode）和color属性配合使用，能让图片改变颜色</font>，里边的模式非常的多，产生的效果也是非常丰富的。在这里作几个简单的例子，让大家看一下效果，剩下的留给小伙伴自己探索。

```dart
child:new Image.network(
  'https://img4.mukewang.com/szimg/5d1032ab08719e0906000338-360-202.jpg',
    color: Colors.greenAccent,
    colorBlendMode: BlendMode.darken,
),
```

- color：是要混合的颜色，如果你只设置color是没有意义的。
- colorBlendMode:是混合模式，相当于我们如何混合。



### repeat图片重复

- ImageRepeat.repeat : <font color=#FF0000>横向和纵向都进行重复，直到铺满整个画布。</font>
- ImageRepeat.repeatX: **横向重复，纵向不重复。**
- ImageRepeat.repeatY：**纵向重复，横向不重复。**

来个全部重复的代码。

```dart
child:new Image.network(
  'https://img4.mukewang.com/szimg/5d1032ab08719e0906000338-360-202.jpg',
   repeat: ImageRepeat.repeat,
),
```

图片的使用在程序中我觉的是非常有意思的，也能制作出很多酷炫的效果。



## 第05节：ListView 列表组件简介

列表组件的知识其实是很多的，也是一个经常使用的组件，我们这里先作一个简介，让大家有个直观的感受，先敲开大门，大家就好深入了。这节我们先学习最简单的列表组件如何制作。



### ListView的声明

学习不仅要学，还要不断的练习，这节我们重新熟悉一下一个Flutter页面的基本写法，因为前面已经学过，所以我相信很多小伙伴已经都会了，但是我么年主要是练习，代码如下：

```dart
import 'package:flutter/material.dart';

void main () => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context){
    return MaterialApp(
      title:'JSPang Flutter Demo',
      home:Scaffold(
        appBar:new AppBar(
          title:new Text('ListView Widget')
        ),
        body: new Text('ListView Text')
      ),
    );
  }
}
```



有了最基本的结构后，就可以加入ListView组件，在`body`代码处加入下面的代码：

```dart
body: new ListView(
  children:<Widget>[
    new ListTile(
      leading:new Icon(Icons.access_time),
      title:new Text('access_time')
    )
  ]
),
```

我们使用了ListView，然后在他的内部`children`中，使用了`widget`数组，因为是一个列表，所以它接受一个数组，然后有使用了`listTile`组件（列表瓦片），在组件中放置了图标和文字。

当然我们还可以多加入几行列表查看效果。

```dart
body: new ListView(
    children: <Widget>[
      new ListTile(
        leading: new Icon(Icons.border_right),
        title: new Text('border_right'),
      ),
      new ListTile(
        leading: new Icon(Icons.android),
        title: new Text('android'),
      ),
      new ListTile(
        leading: new Icon(Icons.arrow_back_ios),
        title: new Text('arrow_back_ios'),
      ),
    ],
  ),
```



### 图片列表的使用

上节课学习了Image Widget，在这里我们就在列表中加入图片来试一下。我们插入4幅图片，然后看一下效果，代码如下：

```dart
body: new ListView(
    children: <Widget>[
      new Image.network('https://img1.mukewang.com/szimg/5d5b6dd109a8f14512000676-360-202.png'),
      new Image.network('https://img4.mukewang.com/szimg/59b8a486000107fb05400300-360-202.jpg'),
      new Image.network('https://img2.mukewang.com/szimg/5c18d2d8000141c506000338-360-202.jpg'),
      new Image.network('https://img4.mukewang.com/szimg/5d1032ab08719e0906000338-360-202.jpg'),
      new Image.network('https://img2.mukewang.com/szimg/5d31765d08c90cba06000338-360-202.jpg'),
    ],
  ),
```

我们使用了网络的方式，插入了5张图片，并且这5张图片形成了一个列表。小伙伴们快动手试一试吧。



## 第06节：横向列表的使用

已经对ListView有了清楚的认识，也做出了普通的纵向（竖向列表）。这节课我们看看横向列表如何使用。其实还是使用我们的ListView组件，只是在ListView组件里加一个`ScrollDirection`属性。

### 制作横向列表

这个我们先来看效果，然后再具体讲解使用方法:

```dart
import 'package:flutter/material.dart';
void main () => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context ){
      return MaterialApp(
        title:'Text widget',
        home: Scaffold(
          appBar:  new AppBar(
            title:  new Text('ListVew Widget'),
          ),
          body: Center(
            child: Container(
              height: 200.0,
              child: new ListView(
                scrollDirection: Axis.horizontal,
                children: <Widget>[
                  new Container(
                    width: 180.0,
                    color: Colors.lightBlue,
                  ),
                  new Container(
                    width: 180.0,
                    color: Colors.amber,
                  ),
                  new Container(
                    width: 180.0,
                    color: Colors.deepOrange,
                  ),
                  new Container(
                    width: 180.0,
                    color: Colors.deepPurpleAccent,
                  ),
                ],
              ),
            ),
          ),

        ),
      );
  }
}
```

我们先是加入了Center组件，作用是让我们的横向列表可以居中到屏幕的中间位置，然后在center组件的下面加入了Container容器组件，并设置了容器组件的高是200，在容器组件里我们加入了`ListView`组件，然后设置了组件的scrollDirection属性。然后再ListView的子组件里加入了Container容器组件，然后设置了不同颜色，效果如图。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/flutter_di2ji_5.png)

### scrollDirection属性

ListView组件的`scrollDirection`属性只有两个值，一个是横向滚动，一个是纵向滚动。默认的就是垂直滚动，所以如果是垂直滚动，我们一般都不进行设置。

- Axis.horizontal:横向滚动或者叫水平方向滚动。
- Axis.vertical:纵向滚动或者叫垂直方向滚动。



### 优化代码简介

学到这里，我相信很多小伙伴一定心里很多草泥马在崩腾了，Flutter太反人类了，全是嵌套，让我们如何维护。其实这不能怪Flutter，这是我为了教学简单，所以没有把组件分开定义。

**现在把列表组件独立定义成一个类，然后我们再加入到主组件中。再工作中会把组件分的很细，这样既能很好的复用有便于维护，还有利于分工，我个人是非常喜欢Flutter这种万物皆组件的形式的。**

我们声明一个MyList的类，然后把嵌套的代码放到这个类里,代码如下。

```dart
class MyList extends StatelessWidget{
  @override
  Widget build(BuildContext context){
    return ListView(
        scrollDirection: Axis.horizontal,
        children: <Widget>[
          new Container(
            width:180.0,
            color: Colors.lightBlue,
          ), new Container(
            width:180.0,
            color: Colors.amber,
          ), new Container(
            width:180.0,
            color: Colors.deepOrange,
          ),new Container(
            width:180.0,
            color: Colors.deepPurpleAccent,
          ),
        ],
    );
  }
}
```

然后再MyAPP类里直接使用这个类，这样就减少了嵌套。全部代码如下，详细的讲解看视频吧。

```dart
import 'package:flutter/material.dart';
void main () => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context ){
      return MaterialApp(
        title:'ListView widget',
        home:Scaffold(
          body:Center(
          child:Container(
            height:200.0,
            child:MyList()
            ),
          ),
        ),
      );
  }
}


class MyList extends StatelessWidget{
  @override
  Widget build(BuildContext context){
    return ListView(
        scrollDirection: Axis.horizontal,
        children: <Widget>[
          new Container(
            width:180.0,
            color: Colors.lightBlue,
          ), new Container(
            width:180.0,
            color: Colors.amber,
          ), new Container(
            width:180.0,
            color: Colors.deepOrange,
          ),new Container(
            width:180.0,
            color: Colors.deepPurpleAccent,
          ),
        ],
    );
  }
}
```



## 第07节：动态列表的使用

有经验的程序员都知道，其实我们在实际开发中，这种写死的，或者叫静态的列表使用的非常少。我们常用的是动态列表，比如我们的数据从后台读取过来，然后存入一个变量数组里，然后以数组的内容循环出一个列表。



再直观的讲，比如我们访问淘宝的页面，这时候数据是动态的，这样的列表如何实现,这节课就学习一下。



### List类型的使用

List是Dart的集合类型之一,其实你可以把它简单理解为数组（反正我是这么认为的），其他语言也都有这个类型。它的声明有几种方式：

- `var myList = List()`: 非固定长度的声明。
- `var myList = List(2)`: 固定长度的声明。
- `var myList= List<String>()`:固定类型的声明方式。
- `var myList = [1,2,3]`: 对List直接赋值。

那我们这里使用的是一个List传递，然后直接用List中的`generate`方法进行生产List里的元素。最后的结果是生产了一个带值的List变量。代码如下：



说明:再`main`函数的runApp中调用了MyApp类，再使用类的使用传递了一个`items`参数,并使用generate生成器对`items`进行赋值。

generate方法传递两个参数，第一个参数是生成的个数，第二个是方法。

### 接受参数

我们已经传递了参数，那MyApp这个类是需要接收的。

```dart
final List<String> items;
 MyApp({Key key, @required this.items}):super(key:key);
```



这是一个构造函数，除了Key，我们增加了一个必传参数，这里的`@required`意思就必传。`:super`如果父类没有无名无参数的默认构造函数，则子类必须手动调用一个父类构造函数。

这样我们就可以接收一个传递过来的参数了，当然我们要事先进行声明。

### 动态列表 ListView.builder()

接受了值之后，就可以直接调用动态列表进行生成了。具体代码如下：

```dart
import 'package:flutter/material.dart';
void main () => runApp(MyApp(
  items: new List<String>.generate(1000, (i)=> "Item $i")
));

class MyApp extends StatelessWidget{

  final List<String> items;
  MyApp({Key key, @required this.items}):super(key:key);
  @override
  Widget build(BuildContext context ){
      return MaterialApp(
        title:'ListView widget',
        home:Scaffold(
          body:new ListView.builder(
            itemCount:items.length,
            itemBuilder:(context,index){
              return new ListTile(
                title:new Text('${items[index]}'),
              );
            }
          )
        ),
      );
  }
}
```

现在我们可以启动虚拟机来查看，我们的列表的效果了。这个就是工作中我们常使用的列表的方式，当然我们也可以重新做一个列表类，把组件作的美美的。



## 第08节：GridView网格列表组件

列表组件已经学会了，那还有一种常用的列表，叫做网格列表。网格列表经常用来显示多张图片，比如我们经常使用的手机里的相册功能，大部分形式都是网格列表。

### 简单例子演示

我们先不做一个相册的应用，而是使用文字，作一个最简单的网格列表，目的是先熟悉一下`GridView`的基本语法，代码如下，视频中会进行详细讲解：

```dart
import 'package:flutter/material.dart';
void main () => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context ){
      return MaterialApp(
        title:'ListView widget',
        home:Scaffold(
          body:GridView.count(
            padding:const EdgeInsets.all(20.0),
            crossAxisSpacing: 10.0,
            crossAxisCount: 3,
            children: <Widget>[
              const Text('I am Jspang'),
              const Text('I love Web'),
              const Text('jspang.com'),
              const Text('我喜欢玩游戏'),
              const Text('我喜欢看书'),
              const Text('我喜欢吃火锅')
            ],
          )
        ),
      );
  }
}
```



我们在body属性中加入了网格组件，然后给了一些常用属性:

- padding:表示内边距，这个小伙伴们应该很熟悉。
- crossAxisSpacing:网格间的空当，相当于每个网格之间的间距。
- crossAxisCount:网格的列数，相当于一行放置的网格数量。

### 图片网格列表

加入文字作网格列表总是怪怪的，也不是很直观，我们利用图片来作一些网格列表。当然我们用一种更原生的方法，现在官方已经不鼓励使用这种方法了，但是为了你碰到时，不至于不知道怎么回事，所以我们作一下这种形式，但主要是为了作图片布局。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Flutter/flutter_di2ji_6.png)

代码如下：

```dart
import 'package:flutter/material.dart';
void main () => runApp(MyApp());

class MyApp extends StatelessWidget{
  @override
  Widget build(BuildContext context ){
      return MaterialApp(
        title:'ListView widget',
        home:Scaffold(
          body:GridView(
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 3,
              mainAxisSpacing: 2.0,
              crossAxisSpacing: 2.0,
              childAspectRatio: 0.7
            ),
            children: <Widget>[
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2566618190.jpg',fit: BoxFit.cover,),
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2551393832.jpg',fit: BoxFit.cover,),
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2555084871.jpg',fit: BoxFit.cover,),

              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2547108654.jpg',fit: BoxFit.cover,),
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2555952192.jpg',fit: BoxFit.cover,),
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2551119672.jpg',fit: BoxFit.cover,),

              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2553992741.jpg',fit: BoxFit.cover,),
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2545020183.jpg',fit: BoxFit.cover,),
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2554370800.jpg',fit: BoxFit.cover,),

              new Image.network('https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2564832427.jpg',fit: BoxFit.cover,),
              new Image.network('https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2563766934.jpg',fit: BoxFit.cover,),
              new Image.network('https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2565063457.jpg',fit: BoxFit.cover,),
            ],
          )
        ),
      );
  }
}
```

childAspectRatio:宽高比，这个值的意思是宽是高的多少倍，如果宽是高的2倍，那我们就写2.0，如果高是宽的2倍，我们就写0.5。希望小伙伴们理解一下。



---

[完]