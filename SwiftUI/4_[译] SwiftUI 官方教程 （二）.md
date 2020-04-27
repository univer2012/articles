来自：[[译] SwiftUI 官方教程 （二）](https://juejin.im/post/5cfa706f6fb9a07ee1691a46)

> **由于 API 变动，此文章部分内容已失效，最新完整中文教程及代码请查看 [github.com/WillieWangW…](https://github.com/WillieWangWei/SwiftUI-Tutorials)**



![](https://user-gold-cdn.xitu.io/2019/6/7/16b32477d5a7e49c?imageView2/0/w/1280/h/960/ignore-error/1)

# 创建和组合 View

> 此部分将指引你构建一个发现和分享您喜爱地方的 iOS app —— `Landmarks` 。首先我们来构建显示地标详细信息的 view。
>
> `Landmarks` 使用 `stacks` 将 `image`、`text` 等组件进行组合和分层，以此来给 view 布局。如果想给视图添加地图，我们需要引入标准 `MapKit` 组件。在我们调整设计时，Xcode 可以作出实时反馈，以便我们看到这些调整是如何转换为代码的。
>
> 下载项目文件并按照以下步骤操作。
>
> - 预计完成时间：40 分钟
> - 初始项目文件：[下载](https://docs-assets.developer.apple.com/published/71844d6561/CreatingAndCombiningViews.zip)

## 1. 创建一个新项目并且浏览 Canvas

用 `SwiftUI` 的 app 模板来创建一个新的 Xcode 项目，并且浏览一下这个 canvas。

![](https://user-gold-cdn.xitu.io/2019/6/7/16b3249d55584df0?imageslim)

1.1 打开 Xcode ，在 Xcode 的启动窗口中单击 `Create a new Xcode project` ，或选择 `File` > `New` > `Project` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32477d65e5a0c?imageView2/0/w/1280/h/960/ignore-error/1)



1.2 选择 `iOS` 平台， `Single View App` 模板，然后单击 `Next` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32477d64276dd?imageView2/0/w/1280/h/960/ignore-error/1)



1.3 输入 `Landmarks` 作为 `Product Name` ，勾选 `Use SwiftUI` 复选框，然后单击 `Next` 。选择一个位置保存此项目。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32477d634ff85?imageView2/0/w/1280/h/960/ignore-error/1)



1.4 在 `Project navigator` 中，选中 `ContentView.swift` 。

默认情况下， `SwiftUI` view 文件声明了两个结构体。第一个结构体遵循 `View` 协议，描述 view 的内容和布局。第二个结构体声明该 view 的预览。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```


![](https://user-gold-cdn.xitu.io/2019/6/7/16b32478060d65ed?imageView2/0/w/1280/h/960/ignore-error/1)

1.5 在 `canvas` 中，单击 `Resume` 来显示预览。

> Tip：如果没有 `canvas` ，选择 `Editor` > `Editor and Canvas` 来显示。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3249d556f4d1c?imageView2/0/w/1280/h/960/ignore-error/1)



1.6 在 `body` 属性中，将 `Hello World` 更改为自己的问候语。更改代码时，预览便会实时更新。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello SwiftUI!")
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32478eefb5d64?imageView2/0/w/1280/h/960/ignore-error/1)



## 2. 自定义 Text View

为了自定义 view 的显示，我们可以自己更改代码，或者使用 `inspector` 来帮助我们编写代码。

在构建 `Landmarks` 的过程中，我们可以使用任何编辑器来工作：编写源码、修改 `canvas`、或者通过 `inspectors` ，无论使用哪种工具，代码都会保持更新。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324793cdf8856?imageslim)



接下来，我们使用 `inspector` 来自定义 `text view` 。

2.1 在预览中，按住 `Command` 并单击问候语来显示编辑窗口，然后选择 `Inspect` 。

编辑窗口显示了可以修改的不同属性，具体取决于其 view 类型。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32479cbf9c8fb?imageView2/0/w/1280/h/960/ignore-error/1)



2.2 用 `inspector` 将文本改为 `Turtle Rock` ，这是在 app 中显示的第一个地标的名字。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3247ab8d57059?imageView2/0/w/1280/h/960/ignore-error/1)



2.3 将 `Font` 修改为 `Title` 。

这个修改会让文本使用系统字体，之后它就能正确显示用户的偏好字体大小和设置。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3247a3fa492ac?imageView2/0/w/1280/h/960/ignore-error/1)



Edit the code by hand to add the .color(.green) modifier; this changes the text’s color to green.

To customize a SwiftUI view, you call methods called modifiers. Modifiers wrap a view to change its display or other properties. Each modifier returns a new view, so it’s common to chain multiple modifiers, stacked vertically.

2.4 在代码中添加 `.color(.green)` ，将文本的颜色更改为绿色。

如果想自定义 `SwiftUI` 的 view，我们可以调用一类叫做 `modifiers` 的方法。这类方法通过包装一个 view 来改变它的显示或者其他属性。每个 `modifiers` 方法会返回一个新的 view，因此我们可以链式调用多个 `modifiers` 方法。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Turtle Rock")
            .font(.title)
            .color(.green)
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3247b7b073829?imageView2/0/w/1280/h/960/ignore-error/1)



view 的真实来源是其实是代码，当我们使用 `inspector` 修改或删除 `modifiers` 时，Xcode 会立即更新我们的代码。

2.5 这次我们在代码编辑区按住 `Command` ，单击 `Text` 的声明来打开 `inspector` ，然后选择 `Inspect` 。单击颜色菜单并且选择 `Inherited` ，这样文字又变回了黑色。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3247d0d191c83?imageView2/0/w/1280/h/960/ignore-error/1)



2.6 注意，Xcode 会自动针对修改来更新代码，例如删除了 `.color(.green)` 。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Turtle Rock")
            .font(.title)

    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32491f3eb88cf?imageView2/0/w/1280/h/960/ignore-error/1)



## 3. 用 Stacks 组合 View

在上一节创建标题 view 后，我们来添加 text view，它用来显示地标的详细信息，比如公园的名称和所在的州。

在创建 `SwiftUI` view 时，我们可以在 view 的 `body` 属性中描述其内容、布局和行为。由于 `body` 属性仅返回单个 view，所以我们可以使用 `Stacks` 来组合和嵌入多个 view，让它们以水平、垂直或从后到前的顺序组合在一起。

在本节中，我们使用水平的 `stack` 来显示公园的详细信息，再用垂直的 `stack` 将标题放在详细信息的上面。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3247be5618cc8?imageView2/0/w/1280/h/960/ignore-error/1)



我们可以使用 Xcode 的编辑功能将 view 嵌入到一个容器里，也可以使用 `inspector` 或者 `help` 找到更多帮助。

3.1 按住 `Command` 并单击 text view 的初始化方法，在编辑窗口中选择 `Embed in VStack` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3247d60b1b1cb?imageView2/0/w/1280/h/960/ignore-error/1)



接下来，我们从 `Library` 中拖一个 `Text view` 添加到 `stack` 中。

3.2 单击 Xcode 右上角的加号按钮 `(+)` 打开 `Library` ，然后拖一个 `Text view` ，放在代码中 `Turtle Rock` 的后面。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32480a9b31d2c?imageView2/0/w/1280/h/960/ignore-error/1)



3.3 将 `Placeholder` 改成 `Joshua Tree National Park` 。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Text("Turtle Rock")
                .font(.title)
            Text("Joshua Tree National Park")
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32484733ab073?imageView2/0/w/1280/h/960/ignore-error/1)



调整地点 view 以满足布局需求。

3.4 将地点 view 的 `font` 设置成 `.subheadline` 。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Text("Turtle Rock")
                .font(.title)
            Text("Joshua Tree National Park")
                .font(.subheadline)
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324840c4a3fa6?imageView2/0/w/1280/h/960/ignore-error/1)



3.5 编辑 `VStack` 的初始化方法，将 view 以 `leading` 方式对齐。

默认情况下， `stacks` 会将内容沿其轴居中，并设置适合上下文的间距。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Text("Turtle Rock")
                .font(.title)
            Text("Joshua Tree National Park")
                .font(.subheadline)3
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248492c4ef1f?imageView2/0/w/1280/h/960/ignore-error/1)



接下来，我们在地点的右侧添加另一个 text view 来显示公园所在的州。

3.6 在 `canvas` 中按住 `Command` ，单击 `Joshua Tree National Park` ，然后选择 `Embed in HStack` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248132fab9b3?imageView2/0/w/1280/h/960/ignore-error/1)



3.7 在地点后新加一个 text view，将 `Placeholder` 修改成 `California` ，然后将 `font` 设置成 `.subheadline` 。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Text("Turtle Rock")
                .font(.title)
            HStack {
                Text("Joshua Tree National Park")
                    .font(.subheadline)
                Text("California")
                    .font(.subheadline)
            }
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324829c87834e?imageView2/0/w/1280/h/960/ignore-error/1)



3.8 在水平 `stack` 中添加一个 `Spacer` 来分割及固定 `Joshua Tree National Park` 和 `California` ，这样它们就会共享整个屏幕宽度。

`spacer` 能展开它包含的 view ，使它们共用其父 view 的所有空间，而不是仅通过其内容定义其大小。

> ContentView.swift

```
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Text("Turtle Rock")
                .font(.title)
            HStack {
                Text("Joshua Tree National Park")
                    .font(.subheadline)
                Spacer()
                Text("California")
                    .font(.subheadline)
            }
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32485700b5b67?imageView2/0/w/1280/h/960/ignore-error/1)



3.9 最后，用 `.padding()` 这个修饰方法给地标的名称和信息留出一些空间。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Text("Turtle Rock")
                .font(.title)
            HStack {
                Text("Joshua Tree National Park")
                    .font(.subheadline)
                Spacer()
                Text("California")
                    .font(.subheadline)
            }
        }
        .padding()
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32484d62a9715?imageView2/0/w/1280/h/960/ignore-error/1)



## 4. 自定义 Image View

搞定名称和位置 view 后，我们来给地标添加图片。

这不需要添加很多代码，只需要创建一个自定义 view，然后给图片加上遮罩、边框和阴影即可。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324857391ba5b?imageslim)



首先将图片添加到项目的 `asset catalog` 中。

4.1 在项目的 `Resources` 文件夹中找到 `turtlerock.png` ，将它拖到 `asset catalog` 的编辑器中。 Xcode 会给图片创建一个 `image set` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32485702ccc73?imageView2/0/w/1280/h/960/ignore-error/1)



接下来，创建一个新的 `SwiftUI` view 来自定义 image view。

4.2 选择 `File` > `New` > `File` 打开模板选择器。在 `User Interface` 中，选中 `SwiftUI View` ，然后单击 `Next` 。将文件命名为 `CircleImage.swift` ，然后单击 `Create` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32486633ae456?imageView2/0/w/1280/h/960/ignore-error/1)



现在准备工作已完成。

4.3 使用 `Image(_:)` 初始化方法将 text view 替换为 `Turtle Rock` 的图片。

> CircleImage.swift

```swift
import SwiftUI

struct CircleImage: View {
    var body: some View {
        Image("turtlerock")
    }
}

struct CircleImage_Preview: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324867b2360bf?imageView2/0/w/1280/h/960/ignore-error/1)



4.4 调用 `.clipShape(Circle())` ，将图像裁剪成圆形。

`Circle` 可以当做一个蒙版的形状，也可以通过 `stroke` 或 `fill` 形成 view。

> CircleImage.swift

```swift
import SwiftUI

struct CircleImage: View {
    var body: some View {
        Image("turtlerock")
            .clipShape(Circle())
    }
}

struct CircleImage_Preview: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324a9e7f48c09?imageView2/0/w/1280/h/960/ignore-error/1)



4.5 创建另一个 `gray stroke` 的 `circle` ，然后将其作为 `overlay` 添加到图片上，形成图片的边框。

> CircleImage.swift

```swift
import SwiftUI

struct CircleImage: View {
    var body: some View {
        Image("turtlerock")
            .clipShape(Circle())
            .overlay(
                Circle().stroke(Color.gray, lineWidth: 4))
    }
}

struct CircleImage_Preview: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32488225b2aa1?imageView2/0/w/1280/h/960/ignore-error/1)



4.6 接来下，添加一个半径为 10 point 的阴影。

> CircleImage.swift

```swift
import SwiftUI

struct CircleImage: View {
    var body: some View {
        Image("turtlerock")
            .clipShape(Circle())
            .overlay(
                Circle().stroke(Color.gray, lineWidth: 4))
            .shadow(radius: 10)
    }
}

struct CircleImage_Preview: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324879839c42e?imageView2/0/w/1280/h/960/ignore-error/1)



4.7 将边框的颜色改为 `white` ，完成 image view。

> CircleImage.swift

```swift
import SwiftUI

struct CircleImage: View {
    var body: some View {
        Image("turtlerock")
            .clipShape(Circle())
            .overlay(
                Circle().stroke(Color.white, lineWidth: 4))
            .shadow(radius: 10)
    }
}

struct CircleImage_Preview: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324886de44c92?imageView2/0/w/1280/h/960/ignore-error/1)



## 5. 同时使用 UIKit 和 SwiftUI

至此，我们已准备好创建 map view 了，接下来使用 `MapKit` 中的 `MKMapView` 类来渲染地图。

在 `SwiftUI` 中使用 `UIView` 子类，需要将其他 view 包装在遵循 `UIViewRepresentable` 协议的 `SwiftUI` view 中。 `SwiftUI` 包含了和 `WatchKit` 、 `AppKit` view 类似的协议。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324892f02539d?imageView2/0/w/1280/h/960/ignore-error/1)



首先，我们创建一个可以呈现 `MKMapView` 的自定义 view。

5.1 选择 `File` > `New` > `File` ，选择 `iOS` 平台，选择 `SwiftUI View` 模板，然后单击 `Next` 。将新文件命名为 `MapView.swift` ，然后单击 `Create` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b32489addc30f3?imageView2/0/w/1280/h/960/ignore-error/1)



5.2 给 `MapKit` 添加 `import` 语句，声明 `MapView` 类型遵循 `UIViewRepresentable` 。

可以忽略 Xcode 的错误，接下来的几步会解决这些问题。

> MapView.swift

```swift
import SwiftUI
import MapKit

struct MapView: UIViewRepresentable {
    var body: some View {
        Text("Hello World")
    }
}

struct MapView_Preview: PreviewProvider {
    static var previews: some View {
        MapView()
    }
}
```

`UIViewRepresentable` 协议需要实现两个方法： `makeUIView(context:)` 用来创建一个 `MKMapView`， `updateUIView(_:context:)` 用来配置 view 并响应修改。

5.3 用 `makeUIView(context:)` 方法替换 `body` 属性，该方法创建并返回一个空的 `MKMapView`。

> MapView.swift

```swift
import SwiftUI
import MapKit

struct MapView: UIViewRepresentable {
	
	typealias UIViewType = MKMapView

    func makeUIView(context: UIViewRepresentableContext<MapView>) -> MKMapView {
        return MKMapView(frame: .zero)
    }
}

struct MapView_Preview: PreviewProvider {
    static var previews: some View {
        MapView()
    }
}
```

5.4 实现 `updateUIView(_:context:)` 方法，给 map view 设置坐标，使其在 `Turtle Rock` 上居中。

> MapView.swift

```swift
import SwiftUI
import MapKit

struct MapView : UIViewRepresentable {
    
    typealias UIViewType = MKMapView
    
    func makeUIView(context: UIViewRepresentableContext<MapView>) -> MKMapView {
        return MKMapView(frame: .zero)
    }
    
    func updateUIView(_ uiView: MKMapView, context: UIViewRepresentableContext<MapView>) {
        let coordinate = CLLocationCoordinate2D(
            latitude: 34.011286, longitude: -116.166868)
        let span = MKCoordinateSpan(latitudeDelta: 2.0, longitudeDelta: 2.0)
        let region = MKCoordinateRegion(center: coordinate, span: span)
        uiView.setRegion(region, animated: true)
    }
}

struct MapView_Preview: PreviewProvider {
    static var previews: some View {
        MapView()
    }
}
```

当预览处于 `static mode` 时仅显示 `SwiftUI` view 。因为 `MKMapView` 是一个 `UIView` 的子类，所以需要切换到实时模式才能看到地图。

5.5 单击 `Live Preview` 可将预览切换为实时模式，有时也会用到 `Try Again` 或 `Resume` 按钮。

片刻之后，你会看到 `Joshua Tree National Park` 的地图，这是 `Turtle Rock` 的故乡。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248c735622a9?imageView2/0/w/1280/h/960/ignore-error/1)



## 6. 编写详情 View

现在我们完成了所需的所有组件：名称、地点、圆形图片和地图。

继续使用目前的工具，将这些组件组合起来变成符合最终设计的详情 view。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248ce2ebefe4?imageView2/0/w/1280/h/960/ignore-error/1)



6.1 在项目导航中，选中 `ContentView.swift` 文件。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Text("Turtle Rock")
                .font(.title)
            HStack {
                Text("Joshua Tree National Park")
                    .font(.subheadline)
                Spacer()
                Text("California")
                    .font(.subheadline)
            }
        }
        .padding()
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

6.2 把之前的的 `VStack` 嵌入到另一个新 的 `VStack` 中。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            VStack(alignment: .leading) {
                Text("Turtle Rock")
                    .font(.title)
                HStack(alignment: .top) {
                    Text("Joshua Tree National Park")
                        .font(.subheadline)
                    Spacer()
                    Text("California")
                        .font(.subheadline)
                }
            }
            .padding()
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248d5a9c134f?imageView2/0/w/1280/h/960/ignore-error/1)



6.3 将自定义的 `MapView` 添加到 `stack` 顶部，使用 `frame(width:height:)` 方法来设置 `MapView` 的大小。

如果仅指定了 `height` 参数，view 会自动调整其内容的宽度。此节中， `MapView` 会展开并填充所有可用空间。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            MapView()
                .frame(height: 300)

            VStack(alignment: .leading) {
                Text("Turtle Rock")
                    .font(.title)
                HStack(alignment: .top) {
                    Text("Joshua Tree National Park")
                        .font(.subheadline)
                    Spacer()
                    Text("California")
                        .font(.subheadline)
                }
            }
            .padding()
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248da9d8b5ca?imageView2/0/w/1280/h/960/ignore-error/1)



6.4 单击 `Live Preview` 按钮，在组合 view 中查看渲染的地图。

在此过程中，我们可以继续编辑 view。

6.5 将 `CircleImage` 添加到 `stack` 中。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            MapView()
                .frame(height: 300)

            CircleImage()

            VStack(alignment: .leading) {
                Text("Turtle Rock")
                    .font(.title)
                HStack(alignment: .top) {
                    Text("Joshua Tree National Park")
                        .font(.subheadline)
                    Spacer()
                    Text("California")
                        .font(.subheadline)
                }
            }
            .padding()
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248ed4099725?imageView2/0/w/1280/h/960/ignore-error/1)



6.6 为了将 `image view` 布局在 `map view` 的顶部，我们需要给图片设置 `-130 points` 的偏移量，并从底部填充 `-130 points` 。

图片向上移动后，就为文本腾出了空间。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            MapView()
                .frame(height: 300)

            CircleImage()
                .offset(y: -130)
                .padding(.bottom, -130)

            VStack(alignment: .leading) {
                Text("Turtle Rock")
                    .font(.title)
                HStack(alignment: .top) {
                    Text("Joshua Tree National Park")
                        .font(.subheadline)
                    Spacer()
                    Text("California")
                        .font(.subheadline)
                }
            }
            .padding()
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248fb7283f4b?imageView2/0/w/1280/h/960/ignore-error/1)



6.7 在外部 `VStack` 的底部添加一个 `spacer` ，将内容推到屏幕顶端。

> ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            MapView()
                .frame(height: 300)

            CircleImage()
                .offset(y: -130)
                .padding(.bottom, -130)

            VStack(alignment: .leading) {
                Text("Turtle Rock")
                    .font(.title)
                HStack(alignment: .top) {
                    Text("Joshua Tree National Park")
                        .font(.subheadline)
                    Spacer()
                    Text("California")
                        .font(.subheadline)
                }
            }
            .padding()

            Spacer()
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3248fe48222f6?imageView2/0/w/1280/h/960/ignore-error/1)



6.8 最后，为了将地图内容扩展到屏幕的上边缘，需要将 `edgesIgnoringSafeArea(.top)` 添加到 map view 中。

> ContentView.swift

```
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            MapView()
                .edgesIgnoringSafeArea(.top)
                .frame(height: 300)

            CircleImage()
                .offset(y: -130)
                .padding(.bottom, -130)

            VStack(alignment: .leading) {
                Text("Turtle Rock")
                    .font(.title)
                HStack(alignment: .top) {
                    Text("Joshua Tree National Park")
                        .font(.subheadline)
                    Spacer()
                    Text("California")
                        .font(.subheadline)
                }
            }
            .padding()

            Spacer()
        }
    }
}

struct ContentView_Preview: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b3249140c5fa03?imageView2/0/w/1280/h/960/ignore-error/1)

---

【完】