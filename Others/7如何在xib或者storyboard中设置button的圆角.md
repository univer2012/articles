在xib或者storyboard中，可以给button设置圆角。具体操作步骤如下：

1、切换到`xib`或者`storyboard`，选中`UIButton`对象，打开右侧栏`Hide or show the Utilities`的`Show the Identity inspector`，在`User Defined Runtime Attributtes`中添加2个`Key Path`:

- `layer.cornerRadius`，`Type`是`Number`；
- 和`layer.masksToBounds`，`Type`是`Boolean`；

如图：
[![img](https://univer2012.github.io/2017/04/29/7如何在xib或者storyboard中设置button的圆角/pic1.png)](https://univer2012.github.io/2017/04/29/7如何在xib或者storyboard中设置button的圆角/pic1.png)

运行后效果如下：
[![img](https://univer2012.github.io/2017/04/29/7如何在xib或者storyboard中设置button的圆角/pic2.png)](https://univer2012.github.io/2017/04/29/7如何在xib或者storyboard中设置button的圆角/pic2.png)