来自：[用SwiftUI给视图添加动画](https://juejin.im/post/5dd66d8bf265da47ba2aa9f5)



> 前言：最近 [奇舞647](https://www.jianshu.com/u/6663b66c3df3) 写了[《用SwiftUI 写了一个简单页面》](https://www.jianshu.com/p/a89d09f86cce)，笔者也参考官方SwiftUI教程及网络博客，写了一个给视图添加改变位置、透明度、旋转、前景色、背景色及转场动画的Demo。Demo效果如下：



![SwiftUIAnimation.gif](https://user-gold-cdn.xitu.io/2019/11/21/16e8d9424dbb5e93?imageslim)



简单说一下Demo，首先在设置应用的rootViewController部分，笔者设置了UIHostingController，传入指定的QiRootView，其中QiRootView是用SwiftUI布局的。

```swift
window.rootViewController = UIHostingController(rootView: QiRootView())
```

简单示例如下：在QiRootView 的body里边写上我们要显示的视图。

```swift
import SwiftUI

struct QiRootView: View {
    var body: some View {
        VStack {
            HStack {
                Button(action: {
                	withAnimation (.easeInOut(duration: 2.0)){
                		self.showDetail.toggle()
                	}
                }) {
                	Text("常用动画")
                }
            }
        }
    }
}
```

Demo 里边，笔者在body里边，上边的HStack默认情况显示了2个按钮，其中左边的按钮给按钮添加了普通的位置、透明度、旋转、圆角、前景色、背景色的动画。右边的按钮会触发state属性showDetail，转场显示出[奇舞647](https://www.jianshu.com/u/6663b66c3df3)之前写的简单页面。 下边的HStack显示了5个按钮，第一个按钮显示还没绘制完的徽章。后4个按钮分别以不同的转场动画显示[奇舞647](https://www.jianshu.com/u/6663b66c3df3)写的简单页面。

下边笔者分2部分介绍给视图添加动画的内容：

1. 给视图添加常用动画
2. 给视图添加转场动画

使用SwiftUI给视图添加动画的方式比较简单。分2种方式可以给视图添加动画。

1. 直接在 View 上使用 `.animation` 类型添加动画
2. 在按钮的`action`里边用`withAnimation { }` 来控制某个 State 属性，进而触发动画。

给视图添加常用动画使用的是直接在 View 上使用 .animation 类型的方式给视图添加动画。

给视图添加转场动画部分使用的是在按钮的action里边用withAnimation { } 来控制某个 State 属性，进而触发的转场动画。

#### 给视图添加常用动画

下边笔者介绍下上述Demo 动画中用到的相应API。

- 改变位置

```swift
.offset(x: changeAnimation ? offsetX2 : offsetX1, y: changeAnimation ? offsetY2 : offsetY1)
```

- 改变尺寸的话，也可以改变后边的width height的值。

```swift
.frame(width: 100.0, height: 100.0, alignment: .center)
```

- 改变视图圆角

```swift
.cornerRadius(changeAnimation ? cornerR2 : cornerR1)
```

- 改变背景色

```swift
.background(changeAnimation ? backColor2 : backColor1)
```

- 改变前景色

```swift
.foregroundColor(changeAnimation ? foreColor2 : foreColor1)
```

- 改变透明度

```swift
.opacity(changeAnimation ? 0.5 : 1.0)
```

- 改变scale

```swift
.scaleEffect(changeAnimation ? 1.5 : 1.0)
```

- 旋转视图

```swift
.rotationEffect(.degrees(changeAnimation ? 90 : 0))
```

呈现动画的方式可以使用如下API 控制动画类型

```swift
.animation(.easeInOut(duration: 2.0))
```
```swift
.animation(.spring())
```

综上用到如下代码：

```swift
import SwiftUI

struct QiRootView: View {
    
    @State private var changeAnimation = false
    private var offsetX1: CGFloat = 0.0
    private var offsetY1: CGFloat = 0.0
    private var offsetX2: CGFloat = 20.0
    private var offsetY2: CGFloat = 20.0
    
    private var cornerR1: CGFloat = 0.0
    private var cornerR2: CGFloat = 50.0
    
    private var foreColor1 = Color.blue
    private var foreColor2 = Color.white
    
    private var backColor1 = Color.gray
    private var backColor2 = Color.black
    
    var body: some View {
        VStack {
           
            HStack {
                
                Button() {
                                Text("位置")
                                    .offset(x: changeAnimation ? offsetX2 : offsetX1, y: changeAnimation ? offsetY2 : offsetY1)
                                    .frame(width: 100.0, height: 100.0, alignment: .center)
                                    .foregroundColor(changeAnimation ? foreColor2 : foreColor1)
                                    .background(changeAnimation ? backColor2 : backColor1)
                                    .opacity(changeAnimation ? 0.5 : 1.0)
                                    .cornerRadius(changeAnimation ? cornerR2 : cornerR1)
                                    .padding()
                                    .rotationEffect(.degrees(changeAnimation ? 90 : 0))
                                    .animation(.easeInOut(duration: 2.0))
                              }
            }
        }
    }
}
```

#### 给视图添加转场动画

转场动画部分，笔者参照官方教程写了四种转场动画。 分别从左边、右边、上边、下边呈现出来[奇舞647](https://www.jianshu.com/u/6663b66c3df3)写的简单页面。 代码层面的传入的转场类型为给AnyTransition添加了extention的转场类型。

```swift
import SwiftUI

extension AnyTransition {
    static var moveAndFade: AnyTransition {
        AnyTransition.move(edge: .trailing)
    }
    static var moveAndFadeLeading: AnyTransition {
        AnyTransition.move(edge: .leading)
    }
    static var moveAndFadeUp: AnyTransition {
        AnyTransition.move(edge: .top)
    }
    static var moveAndFadeBottom: AnyTransition {
        AnyTransition.move(edge: .bottom)
    }
}

struct QiRootView: View {
    
	@State private var showDetail = false
    
    var body: some View {
        VStack {
           
            HStack {
                Button(action: {
                               withAnimation (.easeInOut(duration: 2.0)){
                                   self.showDetail.toggle()
                               }
                }) {
                             Image(systemName: "chevron.right.circle")
                                .imageScale(.large)
                                .rotationEffect(.degrees(showDetail ? 90 : 0))
                                .opacity(showDetail ? 0.5 : 1.0)
                                .scaleEffect(showDetail3 ? 1.5 : 1.0)
                                .padding()
                          }
            }
            
            if showDetail {
                ContentView()
                	.transition(.moveAndFade)
	                // .transition(.moveAndFadeBottom)
					// .transition(.moveAndFadeLeading)
	                // .transition(.moveAndFadeUp)
            }
        }
    }
}
```

#### Demo

Demo 下载地址：[QiSwiftUIAnimation](https://github.com/QiShare/QiSwiftUIAnimation)

#### 参考学习网址

- [Animating Views and Transitions](https://developer.apple.com/tutorials/swiftui/animating-views-and-transitions)
- [SwiftUI 的一些初步探索 (二)](https://onevcat.com/2019/06/swift-ui-firstlook-2/)
- [SwiftUI(四) 基础动画](https://www.jianshu.com/p/5a22747811e8)

---

【完】

