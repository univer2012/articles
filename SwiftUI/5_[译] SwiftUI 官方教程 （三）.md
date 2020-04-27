

来自：[[译] SwiftUI 官方教程 （三）](https://juejin.im/post/5cfa719c6fb9a07eb67d8139)



> **由于 API 变动，此文章部分内容已失效，最新完整中文教程及代码请查看 [github.com/WillieWangW…](https://github.com/WillieWangWei/SwiftUI-Tutorials)**

# 构建列表与导航

> 完成了基础的地标详情 view 后，我们需要为用户提供查看完整地标列表，以及查看每个地标详情的方法。
>
> 在本文中，我们将会创建可显示任何地标信息的 view ，并动态生成滚动列表，用户可以点按该列表以查看地标的详细视图。另外，我们还将使用 Xcode 的 `canvas` 来显示不同设备的大小，以此来微调 UI。
>
> 下载项目文件并按照以下步骤操作。
>
> - 预计完成时间：35 分钟
> - 初始项目文件：[下载](https://docs-assets.developer.apple.com/published/93fec057db/BuildingListsAndNavigation.zip)

## 1. 了解样本数据

在 [上一个教程](https://github.com/WillieWangWei/SwiftUI-Tutorials/wiki/Creating-and-Combining-Views) 中，我们把数据硬编码到了所有自定义 view 中。在本文中，我们来学习如何将数据传递到自定义 view 中并显示。

下载初始项目并熟悉一下样本数据。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c169f7d50c?imageView2/0/w/1280/h/960/ignore-error/1)



1.1 在 `Project navigator` 中，选择 `Models` > `Landmark.swift` 。

`Landmark.swift` 声明了一个 `Landmark` 结构体，用来存储 app 需要显示的所有地标数据，并从 `landmarkData.json` 导入一组地标数据。

> Landmark.swift

```swift
import SwiftUI
import CoreLocation

struct Landmark: Hashable, Codable {
    var id: Int
    var name: String
    fileprivate var imageName: String
    fileprivate var coordinates: Coordinates
    var state: String
    var park: String
    var category: Category

    var locationCoordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(
            latitude: coordinates.latitude,
            longitude: coordinates.longitude)
    }

    func image(forSize size: Int) -> Image {
        ImageStore.shared.image(name: imageName, size: size)
    }

    enum Category: String, CaseIterable, Codable, Hashable {
        case featured = "Featured"
        case lakes = "Lakes"
        case rivers = "Rivers"
    }
}

struct Coordinates: Hashable, Codable {
    var latitude: Double
    var longitude: Double
}
```

> Data.swift

```swift
import UIKit
import SwiftUI
import CoreLocation

let landmarkData: [Landmark] = load("landmarkData.json")


func load<T: Decodable>(_ filename: String, as type: T.Type = T.self) -> T {
    let data: Data
    
    guard let file = Bundle.main.url(forResource: filename, withExtension: nil) else {
        fatalError("Couldn't find \(filename) in main bundle.")
    }
    
    do {
        data = try Data(contentsOf: file)
    } catch {
        fatalError("Couldn't load \(filename) form main bundle:\n\(error)")
    }
    
    do {
        let decoder = JSONDecoder()
        return try decoder.decode(T.self, from: data)
    } catch {
        fatalError("Couldn't parse \(filename) as \(T.self):\n\(error)")
    }
}


final class ImageStore {
    typealias _ImageDictionary = [String: CGImage]
    fileprivate var images: _ImageDictionary = [:]
    
    fileprivate static var scale = 2
    
    fileprivate var size = 2
    
    static var shared = ImageStore()
    
    func image(name: String) -> Image {
        let index = _guaranteeImage(name: name)
        return Image(images.values[index], scale: CGFloat(ImageStore.scale), label: Text(verbatim: name))
    }
    
    func image(name: String,size: Int) -> Image {
        let index = _guaranteeImage(name: name)
        return Image(images.values[index], scale: CGFloat(size), label: Text(verbatim: name))
    }
    
    
    
    fileprivate func _guaranteeImage(name: String) -> _ImageDictionary.Index {
        if let index = images.index(forKey: name) { return index }
        
        images[name] = ImageStore.loadImage(name: name)
        return images.index(forKey: name)!
    }
    
    static func loadImage(name: String) -> CGImage {
        guard
            let url = Bundle.main.url(forResource: name, withExtension: "jpg"),
            let imageSource = CGImageSourceCreateWithURL(url as NSURL, nil),
            let image = CGImageSourceCreateImageAtIndex(imageSource, 0, nil)
        else {
            fatalError("Couldn't load image \(name).jpg from main bundle.")
        }
        return image
        
    }
}
```



1.2 在 `Project navigator` 中，选择 `Resources` > `landmarkData.json`。

我们会在本教程的剩余部分以及随后的所有内容中使用此样本数据。

> landmarkData.json

```json
[
    {
        "name": "Turtle Rock",
        "category": "Featured",
        "city": "Twentynine Palms",
        "state": "California",
        "id": 1001,
        "park": "Joshua Tree National Park",
        "coordinates": {
            "longitude": -116.166868,
            "latitude": 34.011286
        },
        "imageName": "turtlerock"
    },
    {
        "name": "Silver Salmon Creek",
        "category": "Lakes",
        "city": "Port Alsworth",
        "state": "Alaska",
        "id": 1002,
        "park": "Lake Clark National Park and Preserve",
        "coordinates": {
            "longitude": -152.665167,
            "latitude": 59.980167
        },
        "imageName": "silversalmoncreek"
    },
    ...
]
```

1.3 需要注意的是， [上一个教程](https://github.com/WillieWangWei/SwiftUI-Tutorials/wiki/Creating-and-Combining-Views) 中的 `ContentView` 类型现在更名为 `LandmarkDetail` 。

接下来我们还会创建多个 view 类型。

> LandmarkDetail.swift

```swift
import SwiftUI

struct LandmarkDetail: View {
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

struct LandmarkDetail_Preview: PreviewProvider {
    static var previews: some View {
        LandmarkDetail()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c169e243cc?imageView2/0/w/1280/h/960/ignore-error/1)



## 2. 创建 Row View

我们在本文中构建的第一个 view 是用于显示每个地标详情的 `row` 。 `row` 将地标数据存储在 `landmark` 属性中，这样一个 `row` 就可以显示任何地标。稍后我们会把多个 `row` 组合成一个地标列表。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c169d2cdf9?imageView2/0/w/1280/h/960/ignore-error/1)



2.1 创建一个新的 `SwiftUI` view，命名为 `LandmarkRow.swift` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c16a05e55d?imageView2/0/w/1280/h/960/ignore-error/1)



2.2 如果预览没有显示，请选择 `Editor` > `Editor and Canvas` , 然后单击 `Get Started` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c169fac97b?imageView2/0/w/1280/h/960/ignore-error/1)



2.3 给 `LandmarkRow` 添加一个存储属性 `landmark` 。

当你添加 `landmark` 属性时，预览会停止工作，因为 `LandmarkRow` 类型在初始化时需要一个 `landmark` 实例。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        Text("Hello World")
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkRow()
    }
}
```

为了恢复预览，我们需要修改 `PreviewProvider` 。

2.4 在 `LandmarkRow_Previews` 的静态属性 `previews` 中，给 `LandmarkRow` 的初始化方法添加 `landmark` 参数，并将 `landmarkData` 数组的第一个元素赋值给 `landmark` 参数。

这时预览就会显示 `Hello World` 的文字。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        Text("Hello World")
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkRow(landmark: landmarkData[0])
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c332a0bf88?imageView2/0/w/1280/h/960/ignore-error/1)



恢复预览后，我们就可以构建 `row` 的布局了。

2.5 把现有的 text view 嵌套到一个 `HStack` 中。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            Text("Hello World")
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkRow(landmark: landmarkData[0])
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c2884b8af7?imageView2/0/w/1280/h/960/ignore-error/1)



2.6 将 text view 的内容修改成 `landmark.name` 。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            Text(landmark.name)
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkRow(landmark: landmarkData[0])
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c2351d5558?imageView2/0/w/1280/h/960/ignore-error/1)



2.7 在 text view 前添加一个图片来完成 `row` 。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            landmark.image.resizable().frame(width: 50, height: 50)
            Text(landmark.name)
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkRow(landmark: landmarkData[0])
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c205d5d3aa?imageView2/0/w/1280/h/960/ignore-error/1)



## 3. 自定义 Row 的预览

Xcode的 `canvas` 会自动识别并显示当前编辑器中符合 `PreviewProvider` 协议的任何类型。 `preview provider` 返回一个或多个 view ，其中包含了用来配置大小和设备的选项。

通过自定义 `preview provider` 的返回值，我们可以让预览来显示需要的内容。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c20ad55592?imageView2/0/w/1280/h/960/ignore-error/1)



3.1 在 `LandmarkRow_Previews` 中，把 `landmark` 的参数改成 `landmarkData` 数组的第二个元素。

预览会立即从第一个元素切换到第二个元素的显示。

> LandmarkRow.swift

```python
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            landmark.image.resizable().frame(width: 50, height: 50)
            Text(landmark.name)
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkRow(landmark: landmarkData[1])
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c3440498ce?imageView2/0/w/1280/h/960/ignore-error/1)



3.2 用 `previewLayout(_:)` 方法设置 `row` 在列表中的大概大小。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            landmark.image.resizable().frame(width: 50, height: 50)
            Text(landmark.name)
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkRow(landmark: landmarkData[1])
            .previewLayout(.fixed(width: 300, height: 70))
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c36be1c421?imageView2/0/w/1280/h/960/ignore-error/1)



我们可以在 `preview provider` 中使用 `Group` 来返回多个预览。

3.3 把返回的 `row` 包装到一个 `Group` 中，并且把第一个 `row` 添加回来。

`Group` 是一个组合 view 的容器。 Xcode 会在 `canvas` 中把 `Group` 的子 view 作为分开的预览渲染出来。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            landmark.image.resizable().frame(width: 50, height: 50)
            Text(landmark.name)
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            LandmarkRow(landmark: landmarkData[0])
                .previewLayout(.fixed(width: 300, height: 70))
            LandmarkRow(landmark: landmarkData[1])
                .previewLayout(.fixed(width: 300, height: 70))
        }
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c3cde29030?imageView2/0/w/1280/h/960/ignore-error/1)



把 `previewLayout(_:)` 的调用移到 `group` 声明的外面来精简代码。

一个 view 的子项会继承 view 的上下文设置，比如这里的预览设置。

> LandmarkRow.swift

```swift
import SwiftUI

struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            landmark.image.resizable().frame(width: 50, height: 50)
            Text(landmark.name)
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



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c3d54ba669?imageView2/0/w/1280/h/960/ignore-error/1)



在 `preview provider` 中编写的代码只会改变 Xcode 在 `canvas` 中的显示。由于 `#if DEBUG` 指令的存在，当 app 发布时，编译器会删除这些代码。

## 4. 创建地标列表

使用 `SwiftUI` 的 `List` 类型可以显示平台特有的列表 view 。列表的元素可以是静态的，就像我们创建的 `stacks` 的子 view 一样；也可以是动态生成的。甚至可以把静态和动态生成的 view 混合在一起。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c3904dabb8?imageView2/0/w/1280/h/960/ignore-error/1)



4.1 创建一个新的 `SwiftUI` view，命名为 `LandmarkList.swift` 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c4199183fb?imageView2/0/w/1280/h/960/ignore-error/1)



4.2 把默认的 `Text` view 换成 `List` ，然后传入两个包含头两个地标数据的 `LandmarkRow` 对象，作为 `List` 的子项。

预览会以适合 iOS 样式的列表来显示这两个地标。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    var body: some View {
        List {
            LandmarkRow(landmark: landmarkData[0])
            LandmarkRow(landmark: landmarkData[1])
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c3d8b208e7?imageView2/0/w/1280/h/960/ignore-error/1)



## 5. 动态化列表

相比于给 `list` 指定单个元素，我们还可以直接从集合中生成 `row` 。

通过传递一个数据集合和一个给每个元素提供 view 的闭包来让 `list` 显示集合的元素。 `list` 通过传递的闭包来把每个集合中的元素转换成子 view 。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c521a93084?imageView2/0/w/1280/h/960/ignore-error/1)



5.1 移除现有的两个静态地标 `row` ，然后给 `List` 的初始化方法传递 `landmarkData` 。

`list` 使用 `identifiable` 的数据，我们可以使用以下两个方法之一来让数据变成 `identifiable` ：调用 `identified(by:)` 方法，使用 `key path` 属性来唯一标识每个元素，或者让数据类型遵循 `Identifiable` 协议。

```swift
LandmarkList.swift

import SwiftUI

struct LandmarkList: View {
    var body: some View {
        List(landmarkData) { landmark in
            
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```

5.2 在闭包中返回 `LandmarkRow` ，我们就完成了自动生成内容的 `list` 。

这会给 `landmarkData` 数组中的每一个元素创建一个 `LandmarkRow` 。

```swift
LandmarkList.swift

import SwiftUI

struct LandmarkList: View {
    var body: some View {
        List(landmarkData) { landmark in
            LandmarkRow(landmark: landmark)
        }
        
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```





![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c55c41197b?imageView2/0/w/1280/h/960/ignore-error/1)



接下来，我们通过给 `Landmark` 类型添加遵循 `Identifiable` 的声明来简化代码。

5.3 切换到 `Landmark.swift` ，声明遵循 `Identifiable` 协议。

当 `Landmark` 类型声明了 `Identifiable` 协议需要的 `id` 属性后，我们就完成了对 `Landmark` 的修改。

> Landmark.swift

```swift
import SwiftUI
import CoreLocation

struct Landmark: Hashable, Codable,Identifiable {
    var id: Int
    var name: String
    fileprivate var imageName: String
    fileprivate var coordinates: Coordinates
    var state: String
    var park: String
    //var category: Category

    var locationCoordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(
            latitude: coordinates.latitude,
            longitude: coordinates.longitude)
    }

    enum Category: String, CaseIterable, Codable, Hashable {
        case featured = "Featured"
        case lakes = "Lakes"
        case rivers = "Rivers"
    }
}

extension Landmark {
    var image: Image {
        ImageStore.shared.image(name: imageName)
    }
    
}

struct Coordinates: Hashable, Codable {
    var latitude: Double
    var longitude: Double
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c44e1da5b9?imageView2/0/w/1280/h/960/ignore-error/1)



5.4 切回 `LandmarkList`，删除 `identified(by:)` 的调用。

从现在开始，我们可以直接使用 `Landmark` 元素的集合。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    var body: some View {
        List(landmarkData) { landmark in
            LandmarkRow(landmark: landmark)
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324d3c13e5102?imageView2/0/w/1280/h/960/ignore-error/1)



## 6. 在列表和详情之间设置导航

虽然列表已经能显示了，但是我们还不能通过点击单个地标来查看地标详情页面。

把 `list` 嵌入一个 `NavigationView` 中，并把每个 `row` 嵌套在一个 `NavigationButton` 中来设置到目标 view 的转场，这样 `list` 就具有了导航功能。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c5e65706ca?imageView2/0/w/1280/h/960/ignore-error/1)



6.1 把自动创建地标的 `list` 嵌入到一个 `NavigationView` 中。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    var body: some View {
        NavigationView {
            List(landmarkData) { landmark in
                LandmarkRow(landmark: landmark)
            }
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c604795537?imageView2/0/w/1280/h/960/ignore-error/1)



调用 `navigationBarTitle(_:)` 方法来设置 `list` 显示时导航栏的标题。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    var body: some View {
        NavigationView {
            List(landmarkData) { landmark in
                LandmarkRow(landmark: landmark)
            }
            .navigationBarTitle(Text("Landmarks"))
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324d3c1fb6d64?imageView2/0/w/1280/h/960/ignore-error/1)



6.3 在 `list` 的闭包中，把返回的 `row` 包装在一个 `NavigationButton` 中，并把 `LandmarkDetail` view 作为目标。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
    var body: some View {
        NavigationView {
            List(landmarkData) { landmark in
                NavigationButton(destination: LandmarkDetail()) {
                    LandmarkRow(landmark: landmark)
                }
            }
            .navigationBarTitle(Text("Landmarks"))
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324d3c20c11d2?imageView2/0/w/1280/h/960/ignore-error/1)



6.4 切换到实时模式后可以直接在预览中尝试导航功能。单击 `Live Preview` 按钮，然后点击地标来访问详情页面。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c62c479908?imageView2/0/w/1280/h/960/ignore-error/1)



## 7. 给子 View 传递数据

`LandmarkDetail` 现在依然使用硬编码的数据来显示地标。像 `LandmarkRow` 一样，`LandmarkDetail` 类型和它组合的其他 view 都需要一个 `landmark` 属性作为它们的数据源。

在开始子 view 的内容时，我们会把 `CircleImage` 、 `MapView` 和 `LandmarkDetail` 的显示从硬编码改为传入的数据。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324c9262b72b0?imageView2/0/w/1280/h/960/ignore-error/1)



7.1 在 `CircleImage.swif` 中，添加存储属性 `image` 。

这是使用 `SwiftUI` 构建 view 时的常见模式。我们的自定义 view 通常会为特定视图包装和封装一些 `modifiers` 。

> CircleImage.swift

```swift
import SwiftUI

struct CircleImage: View {
    var image: Image
    
    var body: some View {
        image
            .clipShape(Circle())
            .overlay(
                Circle().stroke(Color.white, lineWidth: 4))
        .shadow(radius: 10)
        
    }
}

#if DEBUG
struct CircleImage_Previews: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
#endif
```

7.2 更新 `preview provider` ，传递一个 `Turtle Rock` 的图片。

> CircleImage.swift

```swift
import SwiftUI

struct CircleImage: View {
    var image: Image

    var body: some View {
        image
            .clipShape(Circle())
            .overlay(Circle().stroke(Color.white, lineWidth: 4))
            .shadow(radius: 10)
    }
}

struct CircleImage_Preview: PreviewProvider {
    static var previews: some View {
        CircleImage(image: Image("turtlerock"))
    }
}
```

7.3 在 `MapView.swift` 中，给 `MapView` 添加一个 `coordinate` 属性，然后把经纬度的硬编码换成使用这个属性。

> MapView.swift

```swift
import SwiftUI
import MapKit

struct MapView:View, UIViewRepresentable {
    
    typealias UIViewType = MKMapView
    
    var coordinate: CLLocationCoordinate2D
    
    func makeUIView(context: UIViewRepresentableContext<MapView>) -> MapView.UIViewType {
        
        MKMapView(frame: .zero)
        
    }
    
    
    func updateUIView(_ uiView: MapView.UIViewType, context: UIViewRepresentableContext<MapView>) {
        
        let span = MKCoordinateSpan(latitudeDelta: 2.0, longitudeDelta: 2.0)
        let region = MKCoordinateRegion(center: coordinate, span: span)
        uiView.setRegion(region, animated: true)
    }
}
#if DEBUG
struct MapView_Previews: PreviewProvider {
    static var previews: some View {
        MapView()
    }
}
#endif
```



7.4 更新 `preview provider` ，传递数据数组中第一个地标的坐标。

> MapView.swift

```swift
import SwiftUI
import MapKit

struct MapView: UIViewRepresentable {
    var coordinate: CLLocationCoordinate2D

    func makeUIView(context: Context) -> MKMapView {
        MKMapView(frame: .zero)
    }

    func updateUIView(_ view: MKMapView, context: Context) {
        let span = MKCoordinateSpan(latitudeDelta: 0.02, longitudeDelta: 0.02)
        let region = MKCoordinateRegion(center: coordinate, span: span)
        view.setRegion(region, animated: true)
    }
}

struct MapView_Preview: PreviewProvider {
    static var previews: some View {
        MapView(coordinate: landmarkData[0].locationCoordinate)
    }
}
```



7.5 在 `LandmarkDetail.swift` 中，给 `LandmarkDetail` 类型添加 `landmark` 属性。

> LandmarkDetail.swift

```swift
import SwiftUI

struct LandmarkDetail: View {
    
    var landmark: Landmark
    
    var body: some View {
        VStack {
            MapView()
                .edgesIgnoringSafeArea(.top)
                .frame(height: 300)

            CircleImage()
                .offset(x: 0, y: -130)
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

#if DEBUG
struct LandmarkDetail_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkDetail()
    }
}
#endif
```

7.6 更新 `preview provider` ，使用 `landmarkData` 中的第一个地标。

> LandmarkDetail.swift

```swift
import SwiftUI

struct LandmarkDetail: View {
    var landmark: Landmark

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

#if DEBUG
struct LandmarkDetail_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkDetail(landmark: landmarkData[0])
    }
}
#endif
```

7.7 将所需数据传递给我们的自定义类型。

> LandmarkDetail.swift

```swift
import SwiftUI

struct LandmarkDetail: View {
    
    var landmark: Landmark
    
    var body: some View {
        VStack {
            MapView(coordinate: landmark.locationCoordinate)
                .edgesIgnoringSafeArea(.top)
                .frame(height: 300)

            CircleImage(image: landmark.image)
                .offset(x: 0, y: -130)
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

#if DEBUG
struct LandmarkDetail_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkDetail(landmark: landmarkData[0])
    }
}
#endif
```

7.8 最后，调用 `navigationBarTitle(_:displayMode:)` 方法，给导航栏添加显示详情 view 时的标题。

> LandmarkDetail.swift

```swift
import SwiftUI

struct LandmarkDetail: View {
    
    var landmark: Landmark
    
    var body: some View {
        VStack {
            MapView(coordinate: landmark.locationCoordinate)
                .edgesIgnoringSafeArea(.top)
                .frame(height: 300)

            CircleImage(image: landmark.image)
                .offset(x: 0, y: -130)
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
        .navigationBarTitle(Text(landmark.name), displayMode: .inline)
    }
}

#if DEBUG
struct LandmarkDetail_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkDetail(landmark: landmarkData[0])
    }
}
#endif
```

7.9 在 `SceneDelegate.swift` 中，把 app 的 `rootView` 改成 `LandmarkList` 。

当我们不使用预览而是在模拟器中独立运行 app 时，app 会以 `SceneDelegate` 中定义的 `rootView` 开始显示。

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
        //let contentView = ContentView()
        let contentView = LandmarkList()

        // Use a UIHostingController as window root view controller.
        if let windowScene = scene as? UIWindowScene {
            let window = UIWindow(windowScene: windowScene)
            window.rootViewController = UIHostingController(rootView: contentView)
            self.window = window
            window.makeKeyAndVisible()
        }
    }
    /// ... ...
}
```

![](https://user-gold-cdn.xitu.io/2019/6/7/16b324c945c728ac?imageView2/0/w/1280/h/960/ignore-error/1)

7.10 在 `LandmarkList.swift` 中，给目标 `LandmarkDetail` 传递当前的地标。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
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

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
    }
}
```

<img src="https://user-gold-cdn.xitu.io/2019/6/7/16b324c94c5eb9e8?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:50%;" />



7.11 切换到实时预览，可以查看从列表导航到正确的地标详情 view 了。



<img src="https://user-gold-cdn.xitu.io/2019/6/7/16b324c9fa280cda?imageView2/0/w/1280/h/960/ignore-error/1" alt="img" style="zoom:50%;" />



## 8. 动态生成预览

接下来，我们会在 `LandmarkList_Previews` 中添加代码以在不同的设备尺寸上渲染列表。默认情况下，预览会以当前的 `scheme` 中设备的大小进行渲染。我们可以通过调用 `previewDevice(_:)` 方法来改变预览设备。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324ccd3b57883?imageView2/0/w/1280/h/960/ignore-error/1)



8.1 首先，改变当前 `list` 的预览来显示 iPhone SE 的尺寸。

我们可以输入任何 Xcode `scheme` 菜单中显示的设备名称。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
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

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
        .previewDevice(PreviewDevice(rawValue: "iPhone SE"))
    }
}
```

<img src="https://user-gold-cdn.xitu.io/2019/6/7/16b324d3c8bbd358?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:50%;" />

8.2 在 `list` 预览中用设备名称数组作为数据，将 `LandmarkList` 嵌入到 `ForEach` 实例中。

`ForEach` 以与 `list` 相同的方式对集合进行操作，这样我们就可以在任何可以使用子视图的地方使用它，比如 `stacks` ， `lists` ，`groups` 等。当数据元素像这里使用的字符串一样是简单的值类型时，我们可以使用 `\.self` 作为标识符的 `key path` 。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
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

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        
        ForEach(["iPhone SE", "iPhone XS Max"], id:\.self) { deviceName in
            LandmarkList()
            .previewDevice(PreviewDevice(rawValue: deviceName))
        }
        
    }
}
```

<img src="https://user-gold-cdn.xitu.io/2019/6/7/16b324caf9f1fcfb?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:67%;" />

8.3 使用 `previewDisplayName(_:)` 方法把设备名称作为 `labels` 添加到预览中。

> LandmarkList.swift

```swift
import SwiftUI

struct LandmarkList: View {
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

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        
        ForEach(["iPhone SE", "iPhone XS Max"], id:\.self) { deviceName in
            LandmarkList()
            .previewDevice(PreviewDevice(rawValue: deviceName))
            .previewDisplayName(deviceName)
        }
        
    }
}
```

<img src="https://user-gold-cdn.xitu.io/2019/6/7/16b324cd6448478b?imageView2/0/w/1280/h/960/ignore-error/1" style="zoom:67%;" />

8.4 我们可以在 `canvas` 中体验不同的设备，对比它们在渲染 `view` 时的差异。



![img](https://user-gold-cdn.xitu.io/2019/6/7/16b324d3cfd0305b?imageView2/0/w/1280/h/960/ignore-error/1)

---

【完】

