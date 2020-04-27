来自：[[译] SwiftUI 官方教程 （四）](https://juejin.im/post/5cfa9c1d518825399965a857)



# 处理用户输入

> 在 `Landmarks` app 中，用户可以标记他们喜欢的地点，并在列表中过滤出来。要实现这个功能，我们要先在列表中添加一个开关，这样用户可以只看到他们收藏的内容。另外还会添加一个星形按钮，用户可以点击该按钮来收藏地标。
>
> 下载起始项目文件并按照以下步骤操作，也可以打开已完成的项目自行浏览代码。
>
> - 预计完成时间：20 分钟
> - 初始项目文件：[下载](https://docs-assets.developer.apple.com/published/0c72f72e11/HandlingUserInput.zip)

## 1. 标记用户收藏的地标

首先，通过优化列表来清晰地给用户显示他们的收藏。给每个被收藏地标的 `LandmarkRow` 添加一颗星。



![img](https://user-gold-cdn.xitu.io/2019/6/8/16b32f2282ca7a09?imageView2/0/w/1280/h/960/ignore-error/1)



1.1 打开起始项目，在 `Project navigator` 中选择 `LandmarkRow.swift` 。



![img](https://user-gold-cdn.xitu.io/2019/6/8/16b32f228233d9ae?imageView2/0/w/1280/h/960/ignore-error/1)



1.2 在 `spacer` 的下面添加一个 `if` 语句，在其中添加一个星形图片来测试当前地标是否被收藏。

在 `SwiftUI` block 中，我们使用 `if` 语句来有条件的引入 view 。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark
    var body: some View {
        HStack {
            landmark.image.resizable().frame(width: 50, height: 50)
            Text(landmark.name)
            Spacer()
            
            if landmark.isFavorite {
                Image(systemName: "star.fill")
                    .imageScale(.medium)
            }
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            LandmarkRow(landmark: landmarkData[0])
            LandmarkRow(landmark: landmarkData[1])
            
        }
        .previewLayout(.fixed(width: 300, height: 70))
        
    }
}
```

<img src="https://user-gold-cdn.xitu.io/2019/6/8/16b32f2282c88f2d?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:67%;" />

1.3 由于系统图片是基于矢量的，所以我们可以通过 `foregroundColor(_:)` 方法来修改它们的颜色。

当 `landmark` 的 `isFavorite` 属性为 `true` 时，星星就会显示。稍后我们会在教程中看到如何修改这个属性。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark
    var body: some View {
        HStack {
            landmark.image.resizable().frame(width: 50, height: 50)
            Text(landmark.name)
            Spacer()
            
            if landmark.isFavorite {
                Image(systemName: "star.fill")
                    .imageScale(.medium)
                    .foregroundColor(.yellow)
            }
        }
    }
}
//...
```

![](https://user-gold-cdn.xitu.io/2019/6/8/16b32f22c1d4412f?imageView2/0/w/1280/h/960/ignore-error/1)

## 2. 过滤 List View

我们可以自定义 list view 让它显示所有的地标，也可以只显示用户收藏的。为此，我们需要给 `LandmarkList` 类型添加一点 `state` 。

`state` 是一个值或一组值，它可以随时间变化，并且会影响视图的行为、内容或布局。我们用具有 `@State` 特征的属性将 `state` 添加到 view 中。



![img](https://user-gold-cdn.xitu.io/2019/6/8/16b32f2284343fa4?imageView2/0/w/1280/h/960/ignore-error/1)



2.1 在 `Project navigator` 中选择 `LandmarkList.swift` ，添加一个名叫 `showFavoritesOnly` 的 `@State` 属性，把它的初始值设为 `false` 。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    
    @State var showFavoritesOnly = false
    
    var body: some View {
        NavigationView {
            List(landmarkData) { landmark in
                
                NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
                    LandmarkRow(landmark: landmark)
                }
                
            }
            .navigationBarTitle(Text("Landmarks"))
        }
        
    }
}
//...
```

<img src="https://user-gold-cdn.xitu.io/2019/6/8/16b32f2300c040e1?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:50%;" />

2.2 点击 `Resume` 按钮来刷新 `canvas` 。

当我们对 view 的结构进行更改，比如添加或修改属性时，需要手动刷新 `canvas` 。



![img](https://user-gold-cdn.xitu.io/2019/6/8/16b32f237a7d3acf?imageView2/0/w/1280/h/960/ignore-error/1)



2.3 通过检查 `showFavoritesOnly` 属性和每个 `landmark.isFavorite` 的值来过滤地标列表。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    
    @State var showFavoritesOnly = false
    
    var body: some View {
        NavigationView {
            List(landmarkData) { landmark in
                
                if !self.showFavoritesOnly || landmark.isFavorite {
                    NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
                        LandmarkRow(landmark: landmark)
                    }
                }
            }
            .navigationBarTitle(Text("Landmarks"))
        }
        
    }
}
//...
```

<img src="https://user-gold-cdn.xitu.io/2019/6/8/16b32f23238094eb?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:50%;" />

## 3. 添加控件来切换状态

为了让用户控制列表的过滤，我们需要一个可以修改 `showFavoritesOnly` 值的控件。通过给切换控件传递一个 `binding` 来实现这个需求。

`binding` 是对可变状态的引用。当用户将状态从关闭切换为打开然后再关闭时，控件使用 `binding` 来更新 view 相应的状态



![img](https://user-gold-cdn.xitu.io/2019/6/8/16b32f23563aca1f?imageView2/0/w/1280/h/960/ignore-error/1)



3.1 创建一个嵌套的 `ForEach group` 将 `landmarks` 转换为 `rows` 。

**若要在列表中组合静态和动态 view ，或者将两个或多个不同的动态 view 组合在一起，要使用 `ForEach` 类型，而不是将数据集合传递给 `List` 。**

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    
    @State var showFavoritesOnly = false
    
    var body: some View {
        NavigationView {
            List {
                ForEach(landmarkData) { landmark in
                    
                    if !self.showFavoritesOnly || landmark.isFavorite {
                        NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
                            LandmarkRow(landmark: landmark)
                        }
                    }
                }
            }
            .navigationBarTitle(Text("Landmarks"))
        }
        
    }
}
//...
```

<img src="https://user-gold-cdn.xitu.io/2019/6/8/16b32f23f7a79c39?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:50%;" />

3.2 添加一个 `Toggle` view 作为 `List` view 的第一个子项，然后给 `showFavoritesOnly` 传递一个 `binding`。

我们使用 `$` 前缀来访问一个状态变量或者它的属性的 `binding` 。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    
    @State var showFavoritesOnly = false
    
    var body: some View {
        NavigationView {
            List {
                
                Toggle(isOn: $showFavoritesOnly) {
                    Text("Favorites only")
                }
                
                ForEach(landmarkData) { landmark in
                    
                    if !self.showFavoritesOnly || landmark.isFavorite {
                        NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
                            LandmarkRow(landmark: landmark)
                        }
                    }
                }
            }
            .navigationBarTitle(Text("Landmarks"))
        }
        
    }
}
//...
```

<img src="https://user-gold-cdn.xitu.io/2019/6/8/16b32f343796f35e?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:50%;" />

3.3 使用实时预览并点击切换来尝试这个新功能。



<img src="https://user-gold-cdn.xitu.io/2019/6/8/16b32f52033274e3?imageslim" alt="img" style="zoom:50%;" />



## 4. 使用 Bindable Object 进行存储

为了让用户控制哪些特定地标被收藏，我们先要把地标数据存储在 `bindable object` 中。

`bindable object` 是数据的自定义对象，它可以从 `SwiftUI` 环境中的存储绑定到 view 上。 `SwiftUI` 监视 `bindable object` 中任何可能影响 view 的修改，并在修改后显示正确的 view 版本。



![img](https://user-gold-cdn.xitu.io/2019/6/8/16b32f24fc40fcf9?imageView2/0/w/1280/h/960/ignore-error/1)



4.1 创建一个新 `Swift` 文件，命名为 `UserData.swift` ，然后声明一个模型类型。

> UserData.swift

```swift
import SwiftUI

final class UserData: ObservableObject {
    
}
```

4.2 添加必要属性 `didChange` ，使用 `PassthroughSubject` 作为发布者。

`PassthroughSubject` 是 `Combine` 框架中一个简易的发布者，它把任何值都直接传递给它的订阅者。 `SwiftUI` 通过这个发布者订阅我们的对象，然后当数据改变时更新所有需要更新的 view 。

> UserData.swift

```swift
import SwiftUI
import Combine

final class UserData: ObservableObject {
    let didChange = PassthroughSubject<UserData, Never>()
}
```

4.3 添加存储属性 `showFavoritesOnly` 和 `landmarks` 以及它们的初始值。

> UserData.swift

```swift
import SwiftUI
import Combine

final class UserData: ObservableObject {
    let didChange = PassthroughSubject<UserData, Never>()
    
    var showFavoritesOnly = false
    var landmarks = landmarkData
}
```

当客户端更新模型的数据时，`bindable object` 需要通知它的订阅者。当任何属性更改时， `UserData` 应通过它的 `didChange` 发布者发布更改。

4.4 给通过 `didChange` 发布者发送更新的两个属性创建 `didSet handlers` 。

> UserData.swift

```swift
import SwiftUI
import Combine

final class UserData: ObservableObject {
    let didChange = PassthroughSubject<UserData, Never>()
    
    var showFavoritesOnly = false {
        didSet {
            didChange.send(self)
        }
    }
    var landmarks = landmarkData {
        didSet {
            didChange.send(self)
        }
    }
}
```

## 5. 在 View 中接受模型对象

现在已经创建了 `UserData` 对象，我们需要更新 view 来将 `UserData` 对象用作 app 的数据存储。



![img](https://user-gold-cdn.xitu.io/2019/6/8/16b32f24fc5c2e9d?imageView2/0/w/1280/h/960/ignore-error/1)



5.1 在 `LandmarkList.swift` 中，将 `showFavoritesOnly` 声明换成一个 `@EnvironmentObject` 属性，然后给 `preview` 添加一个 `environmentObject(_:)` 方法。

一旦将 `environmentObject(_:)` 应用于父级， `userData` 属性就会自动获取它的值。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    
    
    @EnvironmentObject var userData: UserData
    
    @State var showFavoritesOnly = false
    
    var body: some View {
        NavigationView {
            List {
                
                Toggle(isOn: $showFavoritesOnly) {
                    Text("Favorites only")
                }
                
                ForEach(landmarkData) { landmark in
                    
                    if !self.showFavoritesOnly || landmark.isFavorite {
                        NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
                            LandmarkRow(landmark: landmark)
                        }
                    }
                }
            }
            .navigationBarTitle(Text("Landmarks"))
        }
        
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
        .environmentObject(UserData())
        
    }
}
```

<img src="https://user-gold-cdn.xitu.io/2019/6/8/16b32f253d375e94?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:50%;" />

5.4 在 `SceneDelegate.swift` 中，给 `LandmarkList` 添加 `environmentObject(_:)` 方法。

如果我们不是使用预览，而是在模拟器或真机上构建或运行 `Landmarks` ，这个更新可以确保 `LandmarkList` 在环境中持有 `UserData` 对象。

> SceneDelegate.swift

```swift
import UIKit
import SwiftUI

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?


    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
        // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
        // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).

        // Create the SwiftUI view that provides the window contents.
        let contentView = LandmarkList()
            .environmentObject(UserData())

        // Use a UIHostingController as window root view controller.
        if let windowScene = scene as? UIWindowScene {
            let window = UIWindow(windowScene: windowScene)
            window.rootViewController = UIHostingController(rootView: contentView)
            self.window = window
            window.makeKeyAndVisible()
        }
    }
    //...
}
```

5.5 更新 `LandmarkDetail` view 来使用环境中的 `UserData` 对象。

我们使用 `landmarkIndex` 访问或更新 `landmark` 的收藏状态，这样就可以始终得到该数据的正确版本。

> LandmarkDetail.swift

```
import SwiftUI
```

