## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第60节：购物车-商品选中功能制作) 第60节：购物车_商品选中功能制作

在购物车里是有选择和取消选择，还有全选的功能按钮的。当我们选择时，价格和数量都是跟着自动计算的，列表也是跟着刷新的。这节课主要完成单选和全选按钮的交互效果。

###  制作商品单选按钮的交效果

这些业务逻辑代码，当然需要写到`Provide`中，打开`lib/provide/cart.dart`文件。新建一个`changeCheckState`方法：

```dart
  changeCheckState(CartInfoMode cartItem) async{
     SharedPreferences prefs = await SharedPreferences.getInstance();
     cartString=prefs.getString('cartInfo');  //得到持久化的字符串
     List<Map> tempList= (json.decode(cartString.toString()) as List).cast(); //声明临时List，用于循环，找到修改项的索引
     int tempIndex =0;  //循环使用索引
     int changeIndex=0; //需要修改的索引
     tempList.forEach((item){
         
         if(item['goodsId']==cartItem.goodsId){
          //找到索引进行复制
          changeIndex=tempIndex;
         }
         tempIndex++;
     });
     tempList[changeIndex]=cartItem.toJson(); //把对象变成Map值
     cartString= json.encode(tempList).toString(); //变成字符串
     prefs.setString('cartInfo', cartString);//进行持久化
     await getCartInfo();  //重新读取列表
    
  }
```

业务逻辑写完后到到UI层进行修改，打开`lib/pages/cart_page/cart_item.dart`文件，修改多选按钮的`onTap`方法。

```diff
   //多选按钮
   Widget _cartCheckBt(context, item) {
     return Container(
       child: Checkbox(
         value: item.isCheck,
         activeColor: Colors.pink,
         onChanged: (bool val){ 
-
+          item.isCheck = val;
+          Provide.value<CartProvide>(context).changeCheckState(item);
         },
       ),
     );
   }
```

修改完成后，可以点击测试一下效果，如果一切正常，就可以进行选中和取消的交互了。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#全选按钮交互效果制作) 全选按钮交互效果制作

声明一个状态变量`isAllCheck`,然后在读取购物车商品数据时进行更改。

```diff
  double allPrice = 0;    //总价格
  int allGoodsCount = 0;  //商品总数量

+  bool isAllCheck = true; //是否全选
+
  //添加商品到购物车
  //... ...
```

修改`getCartInfo`方法，就是获取购物车列表的方法.

```diff
    //得到购物车中的商品
   getCartInfo() async {
     SharedPreferences prefs = await SharedPreferences.getInstance();
     //获得购物车中的商品,这时候是一个字符串
     cartString = prefs.getString('cartInfo');
     //把cartList进行初始化，防止数据混乱 
     cartList = [];
     //判断得到的字符串是否有值，如果不判断会报错
     if (cartString == null) {
       cartList = [];
     } else {
       List<Map> tempList = (json.decode(cartString.toString()) as List).cast();
 
       allPrice = 0;
       allGoodsCount = 0;
+      isAllCheck = true;
 
       tempList.forEach((item){
         if (item['isCheck']) {
           allPrice += (item['count'] * item['price']);
           allGoodsCount += item['count'];
+        } else {
+          isAllCheck = false;
         }
-        
+
         cartList.add(new CartInfoMode.fromJson(item));
       });
     }
     notifyListeners();
   }
```

全选按钮的方法和当个商品很类似，也是在`Provide`中，新建一个`changeAllCheckBtnState`方法，写入下面的代码.

```dart
  //点击全选按钮操作
  changeAllCheckBtnState(bool isCheck) async{
    SharedPreferences prefs = await SharedPreferences.getInstance();
    cartString=prefs.getString('cartInfo'); 
    List<Map> tempList= (json.decode(cartString.toString()) as List).cast(); 
    List<Map> newList=[]; //新建一个List，用于组成新的持久化数据。
    for(var item in tempList ){
      var newItem = item; //复制新的变量，因为Dart不让循环时修改原值
      newItem['isCheck']=isCheck; //改变选中状态
      newList.add(newItem);
    } 
   
     cartString= json.encode(newList).toString();//形成字符串
     prefs.setString('cartInfo', cartString);//进行持久化
     await getCartInfo();

  }
```

完成后，到UI界面加入交互效果,打开`lib/pages/cart_page/cart_bottom.dart`文件,修改`selectAllBtn(context)`方法。

```diff
   //全选按钮
-  Widget selectAllBtn() {
+  Widget selectAllBtn(context) {
+
+    bool isAllCheck = Provide.value<CartProvide>(context).isAllCheck;
+
     return Container(
       child: Row(
         children: <Widget>[
           Checkbox(
-            value: true,
+            value: isAllCheck,
             activeColor: Colors.pink,
             onChanged: (bool val) {
-
+              Provide.value<CartProvide>(context).changeAllCheckBtnState(val);
             },
           ),
           Text('全选')
         ],
       ),
     );
   }
```

做完这步，就可以测试一下交互效果了。这的代码比较零散，所以修改的时候要特别注意，防止犯错。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第61节：购物车-商品数量的加减操作) 第61节：购物车_商品数量的加减操作

现在基本购物车页面只差一个商品数量的加减操作了，通过几节课的学习，应该大部分小伙i版已经掌握了编写业务逻辑和持久化的方法。你可以先自己试着能不能做出这个效果。

###  编写业务逻辑方法

直接在`lib/provide/cart.dart`文件中，新建立一个方法`addOrReduceAction()`方法。方法接收两个参数.

- cartItem:要修改的项.
- todo: 是加还是减。

代码如下：

```dart
  addOrReduceAction(var cartItem, String todo )async{
    SharedPreferences prefs = await SharedPreferences.getInstance();
    cartString=prefs.getString('cartInfo'); 
    List<Map> tempList= (json.decode(cartString.toString()) as List).cast();
    int tempIndex =0;
    int changeIndex=0;
    tempList.forEach((item){
         if(item['goodsId']==cartItem.goodsId){
          changeIndex=tempIndex; 
         }
         tempIndex++;
     });
     if(todo=='add'){
       cartItem.count++;
     }else if(cartItem.count>1){
       cartItem.count--;
     }
     tempList[changeIndex]=cartItem.toJson();
     cartString= json.encode(tempList).toString();
     prefs.setString('cartInfo', cartString);//
     await getCartInfo();

  }
```

方法写完后，就可以修改UI部分了，让其有交互效果.

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#ui交互效果的修改) UI交互效果的修改

现在页面中引入`Provide`相关的文件.

```dart
import 'package:provide/provide.dart';
import '../../provide/cart.dart';
```

然后设置接收参数，接收item就可以了

```dart
  var item;
  CartCount(this.item);
```

然后把组件的内部方法都加入参数`context`,这里直接给出所有代码，方便你学习。

```diff
 import 'package:flutter/material.dart';
 import 'package:flutter_screenutil/flutter_screenutil.dart';
+import 'package:provide/provide.dart';
+import '../../provide/cart.dart';
 
 class CartCount extends StatelessWidget {
+  var item;
+  CartCount(this.item);
+
   @override
   Widget build(BuildContext context) {
     return Container(
       width:ScreenUtil().setWidth(165),
       margin: EdgeInsets.only(top: 5.0),
       decoration: BoxDecoration(
         border: Border.all(width: 1,color: Colors.black12) 
       ),
       child: Row(
         children: <Widget>[
-          _reduceBtn(),
+          _reduceBtn(context),
           _countArea(),
-          _addBtn(),
+          _addBtn(context),
         ],
       ),
     );
   }
 
   //减少按钮
-  Widget _reduceBtn() {
+  Widget _reduceBtn(context) {
     return InkWell(
       onTap: () {
-
+        Provide.value<CartProvide>(context).addOrReduceAction(item, 'reduce');
       },
       child: Container(
         width: ScreenUtil().setWidth(45),
         height: ScreenUtil().setHeight(45),
         alignment: Alignment.center,
 
         decoration: BoxDecoration(
-          color: Colors.white,
+          color: item.count ? Colors.white : Colors.black12,
           border: Border(
             right: BorderSide(width: 1,color: Colors.black12)
           )
         ),
-        child: Text('-'),
+        child: item.count > 1 ? Text('-') : Text(' '),
       ),
     );
   }
 
   //添加按钮
-  Widget _addBtn() {
+  Widget _addBtn(context) {
     return InkWell(
       onTap: (){
-
+        Provide.value<CartProvide>(context).addOrReduceAction(item, 'add');
       },
       child: Container(
         width: ScreenUtil().setWidth(45),
         height: ScreenUtil().setHeight(45),
         alignment: Alignment.center,
 
         decoration: BoxDecoration(
           color: Colors.white,
           border: Border(
             left: BorderSide(width: 1,color: Colors.black12)
           )
         ),
         child: Text("+"),
       ),
     );
   }
 
   //中间数列显示区域
   Widget _countArea() {
     return Container(
       width: ScreenUtil().setWidth(70),
       height: ScreenUtil().setHeight(45),
       alignment: Alignment.center,
       color: Colors.white,
-      child: Text('1'),
+      child: Text('${item.count}'),
     );
   }
 
 }
```

全部改完后，还需要到`cart_item.dart`里的`_cartGoodsName`里的调用组件的方法。

```diff
     //商品名称
   Widget _cartGoodsName(item) {
     return Container(
       width: ScreenUtil().setWidth(300),
       padding: EdgeInsets.all(10),
       alignment: Alignment.topLeft,
       child: Column(
         children: <Widget>[
           Text(item.goodsName),
-          CartCount()
+          CartCount(item),
         ],
       ),
     );
   }
```

这步完成后，就应该可以实现商品数量的加减交互了。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第62节：购物车-首页provide化-让跳转随心所欲) 第62节：购物车_首页Provide化 让跳转随心所欲

在开始学习教程时，由于为了教学效果，所以底部导航跳转并没有使用Provide，而是使用了简单的变量，这样作的结果就是其它页面没办法控制首页底部导航的跳转，让项目的跳转非常笨拙，缺乏灵活性。这节课就通过我们小小的改造，把首页`index_page.dart`，加入Provide控制。



###  编写Provide文件

先在`lib/provide`文件夹下面，新建一个`currentIndex.dart`文件,然后声明一个索引变量，这个变量就是控制底部导航和页面跳转的。也就是说我们只要把这个索引进行状态管理，那所以的页面可以轻松的控制首页的跳转了。代码如下：

```dart
import 'package:flutter/material.dart';

class CurrentIndexProvide with ChangeNotifier{
  int currentIndex=0;
  
  changeIndex(int newIndex){
    currentIndex=newIndex;
    notifyListeners();
  }

}
```



然后在`main.dart`实例化：

```diff
 import 'package:flutter/material.dart';
 import './pages/index_page.dart';
 /// ======= provide 类
 import 'package:provide/provide.dart';
 import './provide/child_category.dart';
 import './provide/category_goods_list.dart';
 import './provide/details_info.dart';
 import './provide/cart.dart';
-
+import './provide/currentIndex.dart';
 
 import 'package:fluro/fluro.dart';
 import './routers/routes.dart';
 import './routers/application.dart';
 
 
 
 
 import './provide/counter.dart';
 
 void main(){
   var childCategory= ChildCategory();
   var categoryGoodsListProvide= CategoryGoodsListProvide();
   var detailsInfoProvide= DetailsInfoProvide();
   var cartProvide = CartProvide();
+  var currentIndexProvide = CurrentIndexProvide();
 
   var counter = Counter();
   var providers  = Providers();
  
   providers
     ..provide(Provider<ChildCategory>.value(childCategory))
     ..provide(Provider<CategoryGoodsListProvide>.value(categoryGoodsListProvide))
     ..provide(Provider<DetailsInfoProvide>.value(detailsInfoProvide))
     ..provide(Provider<CartProvide>.value(cartProvide))
+    ..provide(Provider<CurrentIndexProvide>.value(currentIndexProvide))
     ..provide(Provider<Counter>.value(counter));
 
   runApp(ProviderNode(child:MyApp(),providers:providers));
 }
 //... ...
```





### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#重新编写首页) 重新编写首页

现在就要改造首页了，这次改动的地方比较多，所以干脆先注释掉所有代码，然后重新进行编写。

```diff
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'home_page.dart';
import 'category_page.dart';
import 'cart_page.dart';
import 'member_page.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';
import 'package:provide/provide.dart';
import '../provide/currentIndex.dart';


+ class IndexPage extends StatelessWidget {
  final List<BottomNavigationBarItem> bottomTabs = [
    BottomNavigationBarItem(
      icon:Icon(CupertinoIcons.home),
      title:Text('首页')
    ),
    BottomNavigationBarItem(
      icon:Icon(CupertinoIcons.search),
      title:Text('分类')
    ),
    BottomNavigationBarItem(
      icon:Icon(CupertinoIcons.shopping_cart),
      title:Text('购物车')
    ),
     BottomNavigationBarItem(
      icon:Icon(CupertinoIcons.profile_circled),
      title:Text('会员中心')
    ),
  ];

   final List<Widget> tabBodies = [
      HomePage(),
      CategoryPage(),
      CartPage(),
      MemberPage()
   ];
 
-  int currentIndex = 0;
-  var currentPage;
-  @override
-  void initState() {
-    currentPage = tabBodies[currentIndex];
-    _pageController = new PageController()
-    ..addListener((){
-      if (currentPage != _pageController.page.round()) {
-        setState(() {
-          currentPage = _pageController.page.round();
-        });
-      }
-    });
-    super.initState();
-  }
+

   @override
   Widget build(BuildContext context) {
     ScreenUtil.instance = ScreenUtil(width: 750, height: 1334)..init(context);
-    return Scaffold(
-      backgroundColor: Color.fromRGBO(244, 245, 245, 1.0),
-      bottomNavigationBar: BottomNavigationBar(
-        type: BottomNavigationBarType.fixed,
-        currentIndex: currentIndex,
-        items: bottomTabs,
-        onTap: (index){
-          setState(() {
-            currentIndex = index;
-            currentPage = tabBodies[currentIndex];
-          });
-        },
-      ),
-      body: IndexedStack(
-        index: currentIndex,
-        children: tabBodies,
-      ),
+
+    return Provide<CurrentIndexProvide>(
+      builder: (context, child, val){
+        int currentIndex = Provide.value<CurrentIndexProvide>(context).currentIndex;
+
+        return Scaffold(
+          backgroundColor: Color.fromRGBO(244, 245, 245, 1.0),
+          bottomNavigationBar: BottomNavigationBar(
+            type: BottomNavigationBarType.fixed,
+            currentIndex: currentIndex,
+            items: bottomTabs,
+            onTap: (index){
+              Provide.value<CurrentIndexProvide>(context).changeIndex(index);
+            },
+          ),
+          body: IndexedStack(
+            index: currentIndex,
+            children: tabBodies,
+          ),
+        ); 
+
+      },
     );
   }
}
```

修改思路是这样的，把原来的`statfulWidget`换成静态的`statelessWeidget`然后进行主要修改`build`方法里。加入`Provide Widget`，然后再每次变化时得到索引，点击下边导航时改变索引.

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#修改商品详细页，实现跳转) 修改商品详细页，实现跳转

打开`/lib/pages/details_page/details_bottom.dart`文件，先引入`curretnIndex.dart`文件.

```text
import '../../provide/currentIndex.dart';
```

然后修改`build`方法里的购物车图标区域.在图标的`onTap`方法里,加入下面的代码.

```diff
          InkWell( //左边的
            onTap: () {
+              Provide.value<CurrentIndexProvide>(context).changeIndex(2);
+              Navigator.pop(context);
            },
            child: Container(
              width: ScreenUtil().setWidth(110),
              alignment: Alignment.center,
              child: Icon(
                Icons.shopping_cart,
                size: 35,
                color: Colors.red,
              ),
            ),
          ),

```

这步做完，可以试着测试一下了，看看是不是可以从详细页直接跳转到购物车页面了。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第63节：购物车-详细页显示购物车商品数量) 第63节：购物车_详细页显示购物车商品数量

现在购物车的基本功能都已经做完了，但是商品详细页面还有一个小功能没有完成，就是在商品详细页添加商品到购物车时，购物车的图标要动态显示出此时购物车的数量。这节课就利用点时间完成这个功能。

###  修改文件结构

打开`/lib/pages/details_page/details_bottom.dart`文件，修改图片区域，增加层叠组件`Stack Widget`,然后在右上角加入购物车现有商品数量。

```dart
          children: <Widget>[
          //关键代码--------------------start--------------
           Stack(
             children: <Widget>[
               InkWell(
                  onTap: (){
                      Provide.value<CurrentIndexProvide>(context).changeIndex(2);
                      Navigator.pop(context);
                  },
                  child: Container(
                      width: ScreenUtil().setWidth(110) ,
                      alignment: Alignment.center,
                      child:Icon(
                            Icons.shopping_cart,
                            size: 35,
                            color: Colors.red,
                          ), 
                    ) ,
                ),
                Provide<CartProvide>(
                  builder: (context,child,val){
                    int  goodsCount = Provide.value<CartProvide>(context).allGoodsCount;
                    return  Positioned(
                        top:0,
                        right: 10,
                        child: Container(
                          padding:EdgeInsets.fromLTRB(6, 3, 6, 3),
                          decoration: BoxDecoration(
                            color:Colors.pink,
                            border:Border.all(width: 2,color: Colors.white),
                            borderRadius: BorderRadius.circular(12.0)
                          ),
                          child: Text(
                            '${goodsCount}',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: ScreenUtil().setSp(22)
                            ),
                          ),
                        ),
                      ) ;
                  },
                )
              
             ],
           ),
           
           //关键代码--------------------end----------------
```

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#修改provide-cart-dart文件) 修改`provide/cart.dart`文件

因为我们要实现动态展示，所以在添加购物车商品时，应该也有数量的变化，所以需要修改`cart.dart`文件里的`save()`方法。

```diff
     //添加商品到购物车
   save(goodsId, goodsName, count, price, images) async {
     //初始化SharedPreferences
     SharedPreferences prefs = await SharedPreferences.getInstance();
     cartString = prefs.getString('cartInfo'); //获取持久化存储的值
     //判断cartString是否为空，为空说明是第一次添加，或者被key被清除了。
     // 如果有值进行decode操作
     var temp = cartString == null ? [] : json.decode(cartString.toString());
     //把获得值转变成List
     List<Map> tempList = (temp as List).cast();
 
     //声明变量，用于判断购物车中是否已经存在此商品ID
     var isHave = false; //默认为没有
     int ival = 0;       //用于进行循环的索引使用
+
+    allPrice = 0;
+    allGoodsCount = 0;  //把视频总数量设置为0
+
+
     tempList.forEach((item){    //进行循环，找出是否已经存在该商品
       //如果存在，数量进行+1操作
       if (item['goodsId'] == goodsId) {
         tempList[ival]['count'] = item['count'] + 1;
         isHave = true;
       }
+
+      if (item['isCheck']) {
+        allPrice += (cartList[ival].price * cartList[ival].count);
+        allGoodsCount += cartList[ival].count;
+      }
+
       ival ++;
     });
 
     //  如果没有，进行增加
     if (!isHave) {
       Map<String, dynamic> newGoods = {
         'goodsId': goodsId,
         'goodsName': goodsName,
         'count': count,
         'price': price,
         'images': images,
         'isCheck': true   ///是否已经选择
       };
       tempList.add(newGoods);
       cartList.add(new CartInfoMode.fromJson(newGoods));
+
+      allPrice += (count * price);
+      allGoodsCount += count;
+
     }
     //把字符串进行encode操作，
     cartString = json.encode(tempList).toString();
-    print(cartString);
-    print(cartList.toString());
-    prefs.setString('cartInfo', cartString);
+    //print(cartString);
+    //print(cartList.toString());
+    prefs.setString('cartInfo', cartString);  //进行持久化
+
+    notifyListeners();
   }
```

完成后，就可以实现商品详细页购物车中商品数量的动态展示了。也算我们购物车区域所有功能都已经完成了。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第64节：会员中心-首页头部布局) 第64节：会员中心_首页头部布局

这节课开始布局会员中心的UI，如果你前边的课程都认真听了，并且也跟着作了，那这部分的内容对你来说就比较简单了。你可以作为一个练习来作。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#页面大体架构的编写) 页面大体架构的编写

打开以前建立的`/lib/pages/member_page.dart`文件，先删除里边的代码，然后引入我们需要的`package`代码。

```text
import 'package:flutter/material.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';
```

引入package后，就可以编写一个`StatelessWidget`，代码如下：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';


class MemberPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
   
  }
}
```

然后返回一个`Scaffold`，在`body`区域里加入一个ListView。

```dart
import 'package:flutter/material.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';


class MemberPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
     appBar: AppBar(
       title: Text('会员中心'),
     ),
     body:ListView(
       children: <Widget>[
       ],
     ) ,
   );
  }
}
```

这样大体结构就已经编写完成了，编写完成后我们把`ListView`的进行分离出来，编写成不同的方法。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#顶部头像区域编写) 顶部头像区域编写

头像区域我们外边套一层`Container`，然后里边放入`Column`，圆形头像这个部分，我们使用`ClipOval Widget`。代码我直接放在下面了。

```dart
  Widget _topHeader(){

    return Container(
      width: ScreenUtil().setWidth(750),
      padding: EdgeInsets.all(20),
      color: Colors.pinkAccent,
      child: Column(
        children: <Widget>[
          Container(
            margin: EdgeInsets.only(top: 30), 
            child: ClipOval(
              
              child:Image.network('http://blogimages.jspang.com/blogtouxiang1.jpg')
            ),
          ),
          Container(
            margin: EdgeInsets.only(top: 10),
            child: Text(
              '技术胖',
              style: TextStyle(
                fontSize: ScreenUtil().setSp(36),
                color:Colors.white,

              ),
            ),
          )
        ],
      ),
    );

  }
```

写完后把这个组件加入到build的ListView里就可以了。然后就可以进行一个预览了。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第65节：会员中心-订单区域ui编写) 第65节：会员中心_订单区域UI编写

头部区域编写好后，我们就可以编写订单区域了，这部分我们简单分成两个方法来进行编写。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#订单标题区域) 订单标题区域

直接上代码了。

```dart
//我的订单顶部
  Widget _orderTitle(){

    return Container(
      margin: EdgeInsets.only(top:10),
      decoration: BoxDecoration(
        color: Colors.white,
        border: Border(
          bottom:BorderSide(width: 1,color:Colors.black12)
        )
      ),
      child: ListTile(
        leading: Icon(Icons.list),
        title:Text('我的订单'),
        trailing: Icon(Icons.arrow_right),
      ),
    );

  }
```

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#订单列表区域) 订单列表区域

直接上代码

```dart
  Widget _orderType(){

    return Container(
      margin: EdgeInsets.only(top:5),
      width: ScreenUtil().setWidth(750),
      height: ScreenUtil().setHeight(150),
      padding: EdgeInsets.only(top:20),
      color: Colors.white,
      child: Row(
        children: <Widget>[
          Container(
            width: ScreenUtil().setWidth(187),
            child: Column(
              children: <Widget>[
                Icon(
                  Icons.party_mode,
                  size: 30,
                ),
                Text('待付款'),
              ],
            ),
          ),
          //-----------------
          Container(
            width: ScreenUtil().setWidth(187),
            child: Column(
              children: <Widget>[
                Icon(
                  Icons.query_builder,
                  size: 30,
                ),
                Text('待发货'),
              ],
            ),
          ),
           //-----------------
          Container(
            width: ScreenUtil().setWidth(187),
            child: Column(
              children: <Widget>[
                Icon(
                  Icons.directions_car,
                   size: 30,
                ),
                Text('待收货'),
              ],
            ),
          ),
          Container(
            width: ScreenUtil().setWidth(187),
            child: Column(
              children: <Widget>[
                Icon(
                  Icons.content_paste,
                   size: 30,
                ),
                Text('待评价'),
              ],
            ),
          ),
        ],
      ),
    );

  }
```

这两个方法写完后，直接加到`Build`里就可以了。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第66节-会员中心-编写listtile的通用方法) 第66节:会员中心_编写ListTile的通用方法

这节课我们就把会员中心的剩下UI做完，可以看到，订单下面就全部都是类似List的形式了。那我们可以编写一个通用的方法，然后传递不同的值，来快速布局出下面的部分。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#listtile通用方法) ListTile通用方法

我们利用方法传递参数的形式，创建一个可以通用的方法，只要传递不同的参数，就可以形成不同的组件。代码如下

```dart
 Widget _myListTile(String title){

    return Container(
       decoration: BoxDecoration(
        color: Colors.white,
        border: Border(
          bottom:BorderSide(width: 1,color:Colors.black12)
        )
      ),
      child: ListTile(
        leading: Icon(Icons.blur_circular),
        title: Text(title),
        trailing: Icon(Icons.arrow_right),
      ),
    );
  }
```

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#组合list布局) 组合List布局

有了通用的方法后，我们就可以进行组合List布局，代码如下：

```dart
  Widget _actionList(){
    return Container(
      margin: EdgeInsets.only(top: 10),
      child: Column(
        children: <Widget>[
            _myListTile('领取优惠券'),
            _myListTile('已领取优惠券'),
            _myListTile('地址管理'),
            _myListTile('客服电话'),
            _myListTile('关于我们'),
        ],
      ),
    );
  }
```

这个组件编写完成后，可以组合到Build方法里面。这步完成后，就形成了一个完成的会员中心页面。

总结:这节课结束后，我原计划的所有知识点就已经讲完了。但是课程并没有结束，我后边还会不断的更新课程，我管这个叫做加餐。

- 优化现有程序：我会不断优化现有程序和存在的Bug，有重大优化时，就会更新课程。
- 对小伙伴期望的知识点作补充讲解：这个要10人以上提出的共性知识点作补充讲解。
- 后续功能升级：如果后期后台API有重点变化，影响学习，我会录课补充修改。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第67课：加餐-高德地图插件的使用) 第67课：加餐_高德地图插件的使用

这是一个加餐课，很多小伙伴都给我留言说，需要这个功能，经过两天的摸索，总算是可以使用了，当然这个插件的坑也是巨多的。使用的插件叫`amap_base_flutter`,也是国内用的最多的地图一个插件。此节课收到了很多小伙伴的帮助，**特别感谢"鲁隽彧(网名)"**。



### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#_1-注册和建立高德api应用) 1.注册和建立高德API应用

这个需要到高德的网站进行，网站地址为：https://lbs.amap.com/

[ ](https://lbs.amap.com/)

。

> 你需要先注册一个账号，这个过程我就不演示了。这个你自己再弄不明白，那么接下来我就不带你去找小姐姐了。

有了账号之后到控制台-应用管理-创建应用（这个我就再视频中演示了）

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#_2-获得sha1) 2.获得SHA1

在创建应用的时候，需要填入SHA1，这个必须需要在`Android Studio`里进行，`VS Code`里还没有摸清如何获得，如果你知道如何获得，可以文章下方给技术胖留言。（获得方式，在视频中进行演示）

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#_3-获得packagename) 3.获得PackageName

这个的获得比较简单，打开`/android/app/build.gradle`文件，然后找到`applicationId`，这个就是`packageName`，比如我的项目的`packageName`就是`com.example.amap_test`。

把这两项填写好后，我们就可以开心的编写程序了。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#_4-配置andoridmanifest-xml文件) 4.配置`AndoridManifest.xml`文件

这个文件在`/android/app/src/main/AndroidManifest.xml`，然后在``标签里，加入下面的代码:

```text
<meta-data
  android:name="com.amap.api.v2.apikey"
  android:value="自己的key" />
```

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#_5-编写代码) 5.编写代码

需要先进入根目录的`pubspec.yaml`文件，进行依赖注册，这个`package`下载还是需要挺长时间的，我反正用了将近15分钟。

```text
amap_base: ^0.3.5
```

写完后点击右上角的`packages get`，剩下的就是耐心等待。

进入`lib/main.dart`文件，写入下面代码。

进的要用`import`引入`amap_base.dart`文件。

```text
import 'package:flutter/material.dart';
import 'package:amap_base/amap_base.dart';



void main()async{
  runApp(MyApp());

}

class MyApp extends StatelessWidget {
  // This widget is the root of your application.


  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(

        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: '高德地图测试'),
    );
  }
}




class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {

  AMapController _controller;

  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(

        title: Text(widget.title),
      ),
      body:AMapView(
              onAMapViewCreated: (controller) {
                _controller = controller;
              },
              amapOptions: AMapOptions(
                compassEnabled: false,
                zoomControlsEnabled: true,
                logoPosition: LOGO_POSITION_BOTTOM_CENTER,
                camera: CameraPosition(
                  target: LatLng(41.851827, 112.801637),
                  zoom: 4,
                ),
              ),
          
     );
  }

}
```

写完代码后，你要记得不要使用虚拟机进行测试，我在学习的时候，就是使用虚拟机测试，一直是黑屏，后来采用了真机测试，才能出现效果。

这就是我在集成高德地图插件时遇到的几个坑，希望小伙伴们都能别走弯路。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第68节：加餐-极光推送插件使用-1) 第68节：加餐_极光推送插件使用-1

现在每个app都需要有推送功能，这也是一个app的价值所在，和你的顾客产生联系。极光推送是中国很出色的推送服务提供商，有着很好的口碑和稳定性，送达率也是国内领先的。Flutter1.0版本发布后，极光也很及时的退出了Flutter插件。这节课就带着小伙伴了解一下极光推送的使用。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#申请极光账号和建立应用) 申请极光账号和建立应用

极光推送的官方网址为：`https://www.jiguang.cn/`

注册的过程这里我依然省略了，有劳小伙伴们自己辛苦一下。

注册好后，进入'服务中心',然后再进入'开发者平台'，点击创建应用。这时候会出现新页面，让你填写“应用名称”和上传“应用图标”。 创建完成，极光平台就会给我们两个key。

- appKey : 移动客户端使用的key
- Master Secret ： 服务端使用的key

我们这里只做移动端不做服务端，所以只需要`appKey`。得到这个Key也算是极光平台操作完了。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#加入dependencies依赖) 加入dependencies依赖

> github网址:https://github.com/jpush/jpush-flutter-plugin

要使用极光推送插件必须先下载包，要下载包就需要先添加依赖，直接把下面的代码加入`pubspec.yaml`文件中。

```text
jpush_flutter: 0.0.11
```

需要注意的是，使用最新版本，这里使用的只是我录课时的最新版本。

写完代码后，选择`Android Studio`右上角的`Packages get`进行下载，下载完成后进行操作。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#build-gradle添加可以和cpu型号代码) build.gradle添加可以和cpu型号代码

打开`android/app/src/build.gradle`文件，加入如下代码：

```javascript
    defaultConfig {
       ...


        ndk {
            //选择要添加的对应 cpu 类型的 .so 库。
            abiFilters 'armeabi', 'armeabi-v7a', 'x86', 'x86_64', 'mips', 'mips64'// 'arm64-v8a',
            // 还可以添加
        }

        manifestPlaceholders = [
                JPUSH_PKGNAME: applicationId,
                JPUSH_APPKEY : "这里写入你自己申请的Key哦", // NOTE: JPush 上注册的包名对应的 Appkey.
                JPUSH_CHANNEL: "developer-default", //暂时填写默认值即可.
        ]


    }
```

到这里你的第一步工作算是完成了，你已经可以开发推送功能了。这部分如果对于移动开发者来说，可能很容易。所以单独拿出一课来，这样有移动开发经验的可以跳过这节。

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#第69节：加餐-极光推送插件使用-2) 第69节：加餐_极光推送插件使用-2

这节课继续讲解一下极光推送的使用，由于技术胖也是作前端的，PHP也有3年没有碰过了，所以这里讲一下极光推送的本地推送，服务器端代码就不在编写了。工作中应该也不用你编写，这是后端的事情。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#先引入主要文件) 先引入主要文件

打开代码`lib/main.dart`文件，先引入需要使用的主要文件

```text
import 'package:flutter/material.dart';
import 'dart:async';

import 'package:flutter/services.dart';
import 'package:jpush_flutter/jpush_flutter.dart';
```

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#主要方法编写) 主要方法编写

```dart
void main() => runApp(new MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => new _MyAppState();
}

class _MyAppState extends State<MyApp> {
 
  @override
  void initState() {
    super.initState();
  }
// 编写视图
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: new Scaffold(
        appBar: new AppBar(
          title: const Text('极光推送'),
        ),
        body: new Center(
          child:Text('临时的.........') 
        ),
      ),
    );
  }
}
```

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#编写initplatformstate方法) 编写initPlatformState方法

在使用极光推送之前，我们需要初始化一下，初始化时的主要任务就是写一下监听响应方法。在写主要方法之前，需要声明两个变量。

```text
 String debugLable = 'Unknown';   //错误信息
  final JPush jpush = new JPush();  //初始化极光插件
```

然后编写initPlatformState方法

```text
  Future<void> initPlatformState() async {
    String platformVersion;

    try {
      //监听响应方法的编写
      jpush.addEventHandler(
        onReceiveNotification: (Map<String, dynamic> message) async {
          print(">>>>>>>>>>>>>>>>>flutter 接收到推送: $message");
          setState(() {
            debugLable = "接收到推送: $message";
          });
        }
      );

    } on PlatformException {
      platformVersion = '平台版本获取失败，请检查！';
    }


    if (!mounted) return;

    setState(() {
      debugLable = platformVersion;
    });
  }
```

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#编写build的视图) 编写build的视图

```text
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: new Scaffold(
        appBar: new AppBar(
          title: const Text('极光推送'),
        ),
        body: new Center(
            child: new Column(
                children:[
                  new Text('结果: $debugLable\n'),
                  new FlatButton(
                      child: new Text('发送推送消息\n'),
                      onPressed: () {
                        // 三秒后出发本地推送
                        var fireDate = DateTime.fromMillisecondsSinceEpoch(DateTime.now().millisecondsSinceEpoch + 3000);
                        var localNotification = LocalNotification(
                            id: 234,
                            title: '技术胖的飞鸽传说',
                            buildId: 1,
                            content: '看到了说明已经成功了',
                            fireTime: fireDate,
                            subtitle: '一个测试',
                        );
                        jpush.sendLocalNotification(localNotification).then((res) {
                          setState(() {
                            debugLable = res;
                          });
                        });

                      }),

                ]
            )

        ),
      ),
    );
```

这里的详细意思，在视频中解释吧，写注释还是挺累的。为了你能达到很好的学习效果，这里给出全部代码。

```dart
import 'package:flutter/material.dart';
import 'dart:async';

import 'package:flutter/services.dart';
import 'package:jpush_flutter/jpush_flutter.dart';

void main() => runApp(new MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => new _MyAppState();
}

class _MyAppState extends State<MyApp> {
  String debugLable = 'Unknown';   //错误信息
  final JPush jpush = new JPush();  //初始化极光插件
  @override
  void initState() {
    super.initState();
    initPlatformState();  //极光插件平台初始化
  }


  Future<void> initPlatformState() async {
    String platformVersion;

    try {
      //监听响应方法的编写
      jpush.addEventHandler(
        onReceiveNotification: (Map<String, dynamic> message) async {
          print(">>>>>>>>>>>>>>>>>flutter 接收到推送: $message");
          setState(() {
            debugLable = "接收到推送: $message";
          });
        }
      );

    } on PlatformException {
      platformVersion = '平台版本获取失败，请检查！';
    }


    if (!mounted) return;

    setState(() {
      debugLable = platformVersion;
    });
  }



// 编写视图
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: new Scaffold(
        appBar: new AppBar(
          title: const Text('极光推送'),
        ),
        body: new Center(
            child: new Column(
                children:[
                  new Text('结果: $debugLable\n'),
                  new FlatButton(
                      child: new Text('发送推送消息\n'),
                      onPressed: () {
                        // 三秒后出发本地推送
                        var fireDate = DateTime.fromMillisecondsSinceEpoch(DateTime.now().millisecondsSinceEpoch + 3000);
                        var localNotification = LocalNotification(
                            id: 234,
                            title: '技术胖的飞鸽传说',
                            buildId: 1,
                            content: '看到了说明已经成功了',
                            fireTime: fireDate,
                            subtitle: '一个测试',
                        );
                        jpush.sendLocalNotification(localNotification).then((res) {
                          setState(() {
                            debugLable = res;
                          });
                        });

                      }),

                ]
            )

        ),
      ),
    );
  }
}
```

这里就完成了，现在可以打开虚拟机来测试一下效果了，看看推送是不是可以成功实现。

后期更多免费Flutter视频，到https://jspang.com进行观看。

------

## [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#后端接口api文档) 后端接口API文档

URL地址是不断变化的，所以不会提供准确的地址给你们。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#商城首页基本信息) 商城首页基本信息

说明：调用此接口，可获取首页所有的基本信息，包括导航，推荐商品，楼层商品。

参数：lon，lat 接口地址：`wxmini/homePageContent`

返回参数：

- advertesPicture:首页中部广告条。
- category：首页UI分类信息
- floor1:楼层1的商品信息和图片
- floor2:楼层2的商品详细和图片
- floor3:楼层3的商品详细和图片
- recommend:商品推荐的信息
- slides:滑动图片和对应的商品编号
- shopInfo：根据定位获得的门店图片和店长电话

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#火爆专区商品列表) 火爆专区商品列表

参数：page

接口地址：`wxmini/homePageBelowConten`

返回参数：

- image ：商品图片地址，可以直接使用。
- name: 商品名称
- mallPrice：商品商城价格
- price: 商品价格，指市场价格

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#商品类别信息) 商品类别信息

接口地址：`wxmini/getCategory`

返回参数：

- mallCategoryId ： 类别ID，用于控制子类别和商品列表。
- mallCategoryName : 类别名称，例如“白酒”
- bxMallSubDto：二级类别，是个数组
- comments：类别描述，目前全是null
- image：类别图片，可能是以后扩展使用的。

### [#](https://jspang.com/posts/2019/03/01/flutter-shop.html#商品分类页中的商品列表) 商品分类页中的商品列表

接口地址：`wxmini/getMallGoods`

参数：

- categoryId:大类ID，字符串类型
- categorySubId : 子类ID，字符串类型，如果没有可以填写空字符串，例如''
- page: 分页的页数，int类型

返回参数 - goodsId:商品的Id，用于进入商品页时，查询商品详情。 - goodsName: 商品名称 - image： 商品的图片 - oriPrice： 市场价格（贵的价格） - presentPrice：商城价格(便宜的价格)



