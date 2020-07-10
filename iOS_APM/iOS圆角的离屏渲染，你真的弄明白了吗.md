# iOS圆角的离屏渲染，你真的弄明白了吗

来自：[iOS圆角的离屏渲染，你真的弄明白了吗](https://mp.weixin.qq.com/s/Du2gMRZnBg77BAaYr-wFyA)

---





> 测试环境
>
> Xcode 11.4
>
> iPhone 11 Pro
>
> iOS 13.5

### **1. 如何设置圆角才会触发离屏渲染**



我们经常看到，圆角会触发离屏渲染。但其实这个说法是**不准确的**，因为圆角触发离屏渲染也是**有条件**的！



我们先来看看苹果官方文档对于`cornerRadius`的描述：



> Setting the radius to a value greater than `0.0` causes the layer to begin drawing rounded corners on its background. By default, the corner radius does not apply to the image in the layer’s `contents` property; it applies only to the background color and border of the layer. However, setting the `masksToBounds` property to `true` causes the content to be clipped to the rounded corners.



我们发现设置`cornerRadius`大于0时，只为layer的`backgroundColor`和`border`设置圆角；而不会对layer的`contents`设置圆角，除非同时设置了`layer.masksToBounds`为`true`（对应UIView的`clipsToBounds`属性）。



如果这时，你认为`layer``.``masksToBounds`或者`clipsToBounds`设置为`true`就会触发离屏渲染，这是不完全正确的。



我们先打开模拟器的离屏渲染颜色标记：





![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOQZr5bfTbLAEeHp5OX0xltNJF4cGWibRfib9qHiaxOUV6YxIFNWvTwbf5A/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)





- 不设置`layer.masksToBounds`或者`clipsToBounds`，其默认值为`NO`

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    UIView *view1 = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 200.0, 200.0)];
    // 设置背景色
    view1.backgroundColor = UIColor.redColor;
    // 设置边框宽度和颜色
    view1.layer.borderWidth = 2.0;
    view1.layer.borderColor = UIColor.blackColor.CGColor;
    // 设置圆角
    view1.layer.cornerRadius = 100.0;
    
    view1.center = self.view.center;
    [self.view addSubview:view1];
}
```



![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOAkyVyiaNDnyKwXHH4eInr5ldb8jE4rATF3hWJ2FZpKic0icbBcB3Tk3Kg/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)



我们看到只有背景色、边框以及圆角的时候，确实不会触发离屏渲染。

- 设置`layer.masksToBounds`或者`clipsToBounds`为`YES`

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    UIView *view1 = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 200.0, 200.0)];
    // 设置背景色
    view1.backgroundColor = UIColor.redColor;
    // 设置边框宽度和颜色
    view1.layer.borderWidth = 2.0;
    view1.layer.borderColor = UIColor.blackColor.CGColor;
    // 设置圆角
    view1.layer.cornerRadius = 100.0;
  
    // 设置裁剪
    view1.clipsToBounds = YES;
    
    view1.center = self.view.center;
    [self.view addSubview:view1];
}
```



![img](https://mmbiz.qpic.cn/mmbiz_png/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOMSj3LbK79XmaPsENo3Cyf2eHiaKPuevPo60ODqKicKlRUb0us0g4rrrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当我们开启`layer.masksToBounds`或者`clipsToBounds`时，同样的没有触发离屏渲染。这是因为我们还没有设置图片。



- 设置`layer.masksToBounds`或者`clipsToBounds`为`YES`，同时设置图片

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    UIView *view1 = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 200.0, 200.0)];
    // 设置背景色
    view1.backgroundColor = UIColor.redColor;
    // 设置边框宽度和颜色
    view1.layer.borderWidth = 2.0;
    view1.layer.borderColor = UIColor.blackColor.CGColor;
    
    //设置图片
    view1.layer.contents = (__bridge id)[UIImage imageNamed:@"pkq"].CGImage;
    
    // 设置圆角
    view1.layer.cornerRadius = 100.0;
    // 设置裁剪
    view1.clipsToBounds = YES;
    view1.center = self.view.center;
    [self.view addSubview:view1];
}
```

![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOw8sHTdJlFnVJeDejQMrbVepSTaVJdAhr2Fm1aq8aGorJOlfM7icDgjw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)

当我们开启`layer.masksToBounds`或者`clipsToBounds`时，同时设置图片时，就会触发离屏渲染。



- 其实不光是图片，我们为视图添加一个有颜色、内容或边框等有图像信息的子视图也会触发离屏渲染。

  > **有图像信息**还包括在视图或者layer的draw方法中进行绘制等。

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    UIView *view1 = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 200.0, 200.0)];
    // 设置背景色
    view1.backgroundColor = UIColor.redColor;
    // 设置边框宽度和颜色
    view1.layer.borderWidth = 2.0;
    view1.layer.borderColor = UIColor.blackColor.CGColor;
    // 设置圆角
    view1.layer.cornerRadius = 100.0;
    // 设置裁剪
    view1.clipsToBounds = YES;
    
    // 子视图
    UIView *view2 = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 100.0, 100.0)];
    // 下面3个任何一个属性
    // 设置背景色
    view2.backgroundColor = UIColor.blueColor;
    // 设置内容
    view2.layer.contents = (__bridge id)([UIImage imageNamed:@"pkq"].CGImage);
    // 设置边框
    view2.layer.borderWidth = 2.0; 
    view2.layer.borderColor = UIColor.blackColor.CGColor;
    [view1 addSubview:view2];
    
    view1.center = self.view.center;
    [self.view addSubview:view1];
}
```



![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOFvRaicVwnIB0u9yhaiaWXTH5dB1Licg3Y0XMbXFbgER4YHOHPKuWMFHOA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)

### **2. 圆角触发离屏渲染的真正原因**

图层的叠加绘制大概遵循“画家算法”。



**油画算法**：先绘制场景中的离观察者较远的物体，再绘制较近的物体。



先绘制红色部分，再绘制⻩色部分，最后再绘制灰⾊部分，即可解决隐藏面消除的问题。即将场景按照物理距离和观察者的距离远近排序，由远及近的绘制即可。



![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOoC8xpZNvSNJubUA1tiaQahNgtoRS4aPMVvhnTQyXU5gicUkFk8WRD6iaw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)

当我们设置了`cornerRadius`以及`masksToBounds`进行圆角+裁剪时，`masksToBounds`裁剪属性会应用到所有的图层上。





![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEO5ZnA0vQcjlVMkibgtNl2VIibsiaDyO7H7hXAdmVcHNl3jl5nDvnnI6yEQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)





本来我们从后往前绘制，绘制完一个图层就可以丢弃了。但现在需要依次在 **Offscreen Buffer**中保存，等待圆角+裁剪处理，即引发了 **离屏渲染** 。



- 背景色、边框、背景色+边框，再加上圆角+裁剪，根据文档说明，因为 **contents = nil**没有需要裁剪处理的内容，所以`masksToBounds`设置为`YES`或者`NO`都没有影响。

- 一旦我们 **为contents设置了内容** ，无论是图片、绘制内容、有图像信息的子视图等，再加上圆角+裁剪，就会触发离屏渲染。

  > 不一定是直接为contents赋值！

###  

### **3. iOS9及以后的优化**

关于圆角，iOS 9及之后的系统版本，苹果进行了一些优化。



```
layer.contents `/`imageView.image
```


我们只设置`contents`或者`UIImageView`的`image`，并加上圆角+裁剪，是不会产生离屏渲染的。但如果加上了背景色、边框或其他有图像内容的图层，还是会产生离屏渲染。

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    UIView *view1 = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 200.0, 200.0)];
    //设置图片
    view1.layer.contents = (__bridge id)[UIImage imageNamed:@"qiyu"].CGImage;
    // 设置圆角
    view1.layer.cornerRadius = 100.0;
    // 设置裁剪
    view1.clipsToBounds = YES;
    
    view1.center = self.view.center;
    [self.view addSubview:view1];
}
```

![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOKJBpCewKnJRL4reodIEY3gIm7kXb7ayKMYEeHRUg8IYVlby0t4ZDcQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)

其实这也是可以理解的，因为只有 **单层** 内容需要添加圆角和裁切，所以可以不需要用到离屏渲染技术。但如果加上了背景色、边框或其他有图像内容的图层，就会产生为 **多层** 添加圆角和裁切，所以还是会触发离屏渲染(如1中的第3个例子)。



所以，我们在使用类似于`UIButton`的视图的时候需要注意：



**`UIButton`**

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    // 创建一个button视图
    UIButton *button = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 200.0, 200.0)];
    //设置图片
    [button setImage:[UIImage imageNamed:@"pkq"] forState:UIControlStateNormal];
    button.center = self.view.center;
    [self.view addSubview:button];
}
```

我们为`UIButton`设置一个图片，其实会添加一个`UIImageView`。

![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOy0galII6IQ3t4Zka9Dqv8XicW6ia9EODLvibg9LyVAI9282u9pUodnCkw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)

为`UIButton`添加圆角和裁剪，则会触发离屏渲染。

```objective-c
// 设置圆角
button.layer.cornerRadius = 100.0;
// 设置裁剪
button.clipsToBounds = YES;
```

![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOBGRjUACaAJ4DnOjxd3S31FJha2Ch9FLLn0jN84SdhMUqd6zp6MoGFQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)

如果改为`UIButton`中的`UIImageView`添加圆角和裁剪，则 **不会触发离屏渲染**。

```objective-c
// 设置圆角
button.imageView.layer.cornerRadius = 100.0;
// 设置裁剪
button.imageView.clipsToBounds = YES;
```

![img](https://mmbiz.qpic.cn/mmbiz/nVic2E9CEobz3lZRfNW0OQch4ghh0OHEOTat5EgFU41q3R63vaovltoOM8U29Ihl209YV0JaCmdOQ5jgKyOC4FQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1)

### 扩展阅读

更多渲染问题可以看下面这篇文章。

- [iOS 渲染原理解析](https://mp.weixin.qq.com/s?__biz=MzA5MTM1NTc2Ng==&mid=2458322656&idx=1&sn=ea1585d20cc71f9a42a7aadc860fd9ad&scene=21#wechat_redirect)



---

【完】