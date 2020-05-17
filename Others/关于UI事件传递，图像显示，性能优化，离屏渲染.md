来自：[关于UI事件传递，图像显示，性能优化，离屏渲染](https://juejin.im/post/5e12dd7de51d4541162c9aa2)



---



- **UIView与CALayer**
- **事件传递与视图响应链**
- **图像显示原理**
- **UI卡顿掉帧原因**
- **滑动优化方案**
- **UI绘制原理**
- **离屏渲染**

## 一、UIView与CALayer

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b3444321928?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image

<单一职责原则>
 UIView为CALayer提供内容，以及负责处理触摸等事件，参与响应链
 CALayer负责显示内容contents


## 二、事件传递与视图响应链 :

```objectivec
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;

```



![](https://user-gold-cdn.xitu.io/2020/1/6/16f79b3444ac9447?imageView2/0/w/1280/h/960/format/png/ignore-error/1)





流程图：

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b3444e824f2?imageView2/0/w/1280/h/960/format/png/ignore-error/1)



如果事件一直传递到UIAppliction还是没处理，那就会忽略掉。



## 三、图像显示原理

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b3447251c97?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image

1.CPU:输出位图
 2.GPU :图层渲染，纹理合成
 3.把结果放到帧缓冲区(frame buffer)中
 4.再由视频控制器根据vsync信号在指定时间之前去提取帧缓冲区的屏幕显示内容
 5.显示到屏幕上

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b344a71bb30?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image



CPU工作

 1.Layout: UI布局，文本计算
 2.Display: 绘制
 3.Prepare: 图片解码
 4.Commit：提交位图





GPU渲染管线(OpenGL)


 顶点着色，图元装配，光栅化，片段着色，片段处理



## 四、UI卡顿掉帧原因

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b344b04a3ba?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image

iOS设备的硬件时钟会发出Vsync（垂直同步信号），然后App的CPU会去计算屏幕要显示的内容，之后将计算好的内容提交到GPU去渲染。随后，GPU将渲染结果提交到帧缓冲区，等到下一个VSync到来时将缓冲区的帧显示到屏幕上。也就是说，一帧的显示是由CPU和GPU共同决定的。
 一般来说，页面滑动流畅是60fps，也就是1s有60帧更新，即每隔16.7ms就要产生一帧画面，而如果CPU和GPU加起来的处理时间超过了16.7ms，就会造成掉帧甚至卡顿。



## 五、滑动优化方案
 CPU：把以下操作放在子线程中
 1.对象创建、调整、销毁
 2.预排版（布局计算、文本计算、缓存高度等等）
 3.预渲染（文本等异步绘制，图片解码等）

GPU:
 纹理渲染，视图混合

一般遇到性能问题时，考虑以下问题：
 是否受到CPU或者GPU的限制？
 是否有不必要的CPU渲染？
 是否有太多的离屏渲染操作？
 是否有太多的图层混合操作？
 是否有奇怪的图片格式或者尺寸？
 是否涉及到昂贵的view或者效果？
 view的层次结构是否合理？



## 六、UI绘制原理

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b3468fe9871?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image

![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b3469c6a2af?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

image



异步绘制：

 [self.layer.delegate displayLayer: ]
 代理负责生成对应的bitmap
 设置该bitmap作为该layer.contents属性的值



![img](https://user-gold-cdn.xitu.io/2020/1/6/16f79b346a4c6e4c?imageView2/0/w/1280/h/960/format/png/ignore-error/1)

时序图



## 七、离屏渲染

On-Screen Rendering:当前屏幕渲染，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行
 Off-Screen Rendering:离屏渲染，分为CPU离屏渲染和GPU离屏渲染两种形式。GPU离屏渲染指的是GPU在当前屏幕缓冲区外新开辟一个缓冲区进行渲染操作
 应当尽量避免的则是GPU离屏渲染



GPU离屏渲染何时会触发呢？


 圆角（当和maskToBounds一起使用时）、图层蒙版、阴影，设置



```objectivec
layer.shouldRasterize ＝ YES;
```



**为什么要避免GPU离屏渲染？**


 GPU需要做额外的渲染操作。通常GPU在做渲染的时候是很快的，但是涉及到offscreen-render的时候情况就可能有些不同。因为需要额外开辟一个新的缓冲区进行渲染，然后绘制到当前屏幕的过程需要做onscreen跟offscreen上下文之间的切换。这个过程的消耗会比较昂贵，涉及到OpenGL的pipeline跟barrier，而且offscreen-render在每一帧都会涉及到，因此处理不当肯定会对性能产生一定的影响。另外由于离屏渲染会增加GPU的工作量，可能会导致CPU+GPU的处理时间超出16.7ms，导致掉帧卡顿。所以可以的话应尽量减少offscreen-render的图层

### 

---



【完】