来自：[iOS APP渲染性能优化](https://www.jianshu.com/p/c9cdf044487b)

---

本文讲述在APP的开发过程中，需要从哪些方面对渲染性能进行优化。

## 1.尽量避免使用半透明

### 1.1原因

在使用半透明时，会发生Color blending现象。在渲染的时候，为了得到某个像素的最终的颜色值，系统将不得不将当前Layer和它下方的Layer进行颜色的混合，这样就增加了计算量。如果非透明，那么系统只需要根据当前Layer，就能得到最终的颜色值。

### 1.2检测方法

在模拟器运行时，可以通过Debug菜单项，勾选'Color Blended Layers'。



![img](https:////upload-images.jianshu.io/upload_images/4989738-d048e50b3575199e.png?imageMogr2/auto-orient/strip|imageView2/2/w/614)

menu_color_blending.png

在设备上运行时，可以选择Instruments中的Core Animation模板，然后在Debug Options中开启'Color Blended Layers'，如图：



![img](https:////upload-images.jianshu.io/upload_images/4989738-92f7b63d61ec66e2.png?imageMogr2/auto-orient/strip|imageView2/2/w/506)

WX20171101-152631@2x.png

检测结果如下图所示，中间的视图处于'blending enabled'状态，被高亮为红色。



![img](https:////upload-images.jianshu.io/upload_images/4989738-f1905e813f378a05.png?imageMogr2/auto-orient/strip|imageView2/2/w/389)

color_blend_result.png

### 1.3解决办法

Color blending可由下面的方式触发：

- UIView的alpha被设置为小于1大于0的数
- UIView的backgroundColor的alpha通道的值为小于1大于0的数
- CALayer的opacity被设置为小于1大于0的数
- UIImageView使用的图片资源中有透明像素

实际开发中，不太可能完全避免上面列出的触发原因，这种情况下，我们要注意避免对需要频繁更新的视图使用半透明。

## 2.UIImageView的大小和资源大小一致；UILabel的高度要为整数

### 2.1原因

- 当UIImageView的大小和图片资源的大小不一致时，会发生'Misaligned Images'问题，导致源像素和目的像素不能对齐。
- 当UILabel的高度为非整数的像素值时，同样会发生'Misaligned Images'问题。

Misaligned Image表示要绘制的点无法直接映射到屏幕上的像素点，此时系统需要对相邻的像素点做anti-aliasing反锯齿计算，增加了图形负担，通常这种问题出在对某些View的Frame重新计算和设置时产生的。

### 2.2检测方法

在模拟器运行时，可以通过Debug菜单项，勾选'Color Misaligned Images'选项。
 在设备上运行时，可以选择Instruments中的CoreAnimation模板，然后在Debug Options中开启'Color Misaligned Images。
 检测结果如下，对于UILabel，高亮为洋红色；对于UIImageView，高亮为黄色。



![img](https:////upload-images.jianshu.io/upload_images/4989738-cbc4ec2d9f0d218b.png?imageMogr2/auto-orient/strip|imageView2/2/w/392)

misaligned_images.png

### 2.3解决办法

Misaligned Images可由下面的原因导致：

- 硬编码的UILabel的高度值，转换为像素值后，不为整数。例如10.3在@2x, @3x的设备上均不为整数。10.5在@2x上为整数，在@3x上不为整数。
- 使用NSString或NSAttributedString的`boundingRectWithSize`函数计算出来的高度值，没有进行向上取整，导致出现非整数值。
- 图片资源的大小，除以相应的设备屏幕的scale，出现了非整数。例如在@2x的设备上，图片的宽为101。
- 图片资源的大小，没有保持严格的scale比例。例如@2x的大小为40x40，然而@3x的大小却不是60x60。
- 图片资源没有提供全部scale(@2x, @3x)版本
- 图片是网络下载的，例如在使用SDWebImage时。

除了最后一个原因，都很好解决。

## 3.不要使用非32bit颜色格式的图片资源

### 3.1原因

苹果的GPU只解析32bit的颜色格式。如果一张图片的颜色格式不是32bit，CPU会先进行颜色格式转换，再让GPU渲染，这样就加重了CPU的负担。
 苹果官方称这种现象为'Copied Images'。

### 3.2检测方法

在模拟器和真机上都可以检测出来，勾选'Color Copied Images'选项即可。
 示例如下，图中的Mario图片是8bit的png图片，被检测出并高亮显示。



![img](https:////upload-images.jianshu.io/upload_images/4989738-4105f2976875a766.png?imageMogr2/auto-orient/strip|imageView2/2/w/314)

copied_images.png

### 3.3解决办法

此问题比较容易解决，对于设计师导出的图片资源，应该是32bit的颜色。
 另外发现即使是一个8 bit的png图，如果通过Assets进行管理，则也不会有问题，只有当图片被以传统的方式加入到项目里时，才会出现此问题。
 这也再次说明了，图片资源应该尽可能的都通过Assets进行管理。

## 4.不要直接使用layer.cornerRadius和layer.masksToBounds设置图片的圆角

**注意：此条仅对iOS9以下系统有效。从iOS9开始，直接使用上面的两个属性设置圆角，不会有任何性能问题。**

### 4.1原因

使用这两个属性设置图片的圆角时，会触发离屏渲染: Offscreen rendering。所谓离屏渲染，就是不能由GPU执行，必须由CPU执行的渲染。在渲染的过程中，一旦检测到需要离屏渲染，那么需要从GPU切换到CPU，在执行完后还要再切换回GPU。而CPU在执行离屏渲染的时候，还需要分配额外的内存。

### 4.2检测方法

在模拟器和Instruments中都有'Color Offscreen-Rendered Yellow'选项，勾选即可进行检测。
 检测结果如下，被离屏渲染的UIImageView被黄色高亮显示。



![img](https:////upload-images.jianshu.io/upload_images/4989738-cea27681b604ac55.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/375)

IMG_5950.PNG

### 4.3解决办法

要实现圆角效果，但又不想触发离屏渲染，可以基于原始图片，得到一个圆角的图片，然后使用。得到圆角图片的过程虽然有一定的开销，但是只需要付出一次代价，而离屏渲染却是每帧都要付出代价。
 使用下面的UIImage的扩展方法，可以方便的得到圆角图片：

```objc
@implementation UIImage (UIImageCornerRadius)

- (UIImage *)imageWithRoundedCornersAndSize:(CGSize)sizeToFit  {
    CGRect rect = (CGRect){0.f, 0.f, sizeToFit};
    
    UIGraphicsBeginImageContextWithOptions(sizeToFit, NO, UIScreen.mainScreen.scale);
    CGContextAddPath(UIGraphicsGetCurrentContext(),
                     [UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:sizeToFit.width].CGPath);
    CGContextClip(UIGraphicsGetCurrentContext());
    
    [self drawInRect:rect];
    UIImage *output = UIGraphicsGetImageFromCurrentImageContext();
    
    UIGraphicsEndImageContext();
    
    return output;
}

@end

```



## 5.设置layer的阴影时，显式的指定shadowPath

### 5.1原因

如果我们用如下代码去设置阴影效果，那么就会触发离屏渲染。

```objc
UIView *view = [[UIView alloc] initWithFrame: CGRectMake(100, 100, 100, 100)];
view.layer.shadowRadius = 30;
view.layer.shadowOpacity = 0.5f;
view.layer.shadowColor = [UIColor blackColor].CGColor;
view.layer.shadowOffset = CGSizeMake(0, 6);
```

苹果是这么讲的：

> “Letting Core Animation determine the shape of a shadow can be expensive and impact your app’s performance. Rather than letting Core Animation determine the shape of the shadow, specify the shadow shape explicitly using the shadowPath property of CALayer. When you specify a path object for this property, Core Animation uses that shape to draw and cache the shadow effect. For layers, whose shape never changes or rarely changes, this greatly improves the performance by reducing the amount of rendering done by Core Animation.”

翻译过来就是说，让Core Animation去决定阴影的形状，是很昂贵的并且会影响应用的性能。开发者应该显式的去指定阴影的形状，而这可以通过shadowPath这个属性来轻易的实现。

### 5.2解决办法

不要使用shadowRadius这个属性，改为使用shadowPath。将上面的代码改为：

```objc
UIView *view = [[UIView alloc] initWithFrame: CGRectMake(100, 100, 100, 100)];
view.layer.shadowOpacity = 0.5f;
view.layer.shadowColor = [UIColor blackColor].CGColor;
view.layer.shadowPath = [UIBezierPath bezierPathWithRect: CGRectOffset(view.bounds, 0, 6)].CGPath;
```

下图显示了在开启'Color Offscreen-Rendered Yellow'选项的情况下，两种写法的对比结果。



![img](https:////upload-images.jianshu.io/upload_images/4989738-f0b375d0cc45a347.png?imageMogr2/auto-orient/strip|imageView2/2/w/708)

shadowPath.png

上方的使用了shadowRadius，被标注为离屏渲染。下方的使用了shadowPath，不会发生离屏渲染。

## 6.其它注意事项

- 一般不要去修改UIView.opaque属性的默认值（YES）。
- 视图要尽可能的少，不要创建不必要的视图。
- 不到迫不得已，不要通过override drawRect:来实现自定义的视图。

## 7.参考

[Mastering UIKit Performance](https://link.jianshu.com?t=https://yalantis.com/blog/mastering-uikit-performance/)
 [WWDC 2014 Video: Advanced Graphics and Animations for iOS Apps](https://link.jianshu.com?t=https://developer.apple.com/videos/play/wwdc2014/419/)





[iOSApp性能优化](https://www.cnblogs.com/StevenHuSir/p/10215114.html)

