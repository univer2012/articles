来自：[用SwiftUI写一个简单页面](https://juejin.im/post/5dd278d8e51d453fc123bff6)



![img](https://user-gold-cdn.xitu.io/2019/11/21/16e8bb70075a56d1?imageView2/0/w/1280/h/960/ignore-error/1)



> *前言： 之前同事[WYongW](https://www.jianshu.com/u/2f31928b08bf)写了一篇[《用Flutter写一个简单页面》](https://www.jianshu.com/p/3c548b048788)，本篇将和大家一起调研一下苹果今年推出的`SwiftUI`框架。 接下来，让我们一起入门一下`SwiftUI`（尝尝鲜）。*

------

### 一、SwiftUI是什么？

##### 1. 定义：

简单来说，`SwiftUI`是苹果在 **“WWDC-2019”** 推出的一款全新的 **“声明式UI”** 框架。 拆开看，`Swift` + `UI`。（由此可以看出Swift越来越重要）

##### 2. 特点：

- **“简洁迅速”的Swift**：越来越简洁的`Swift`语法，配上`Swift`迅速的优势。
- **“即视”的UI**：降低调试成本，一边写`code`、一边就可查看`UI`。
- **跨平台**：一套代码，即可完成`iOS`、`iPadOS`、`macOS`、`watchOS`的开发与适配。



![img](https://user-gold-cdn.xitu.io/2019/11/21/16e8bb7007496212?imageView2/0/w/1280/h/960/ignore-error/1)



- **“声明式”编程**：

简单来说，对比之前的 **“指令式”编程**，我们通常需要告诉计算机**“怎么做”**？ 而**“声明式”编程**是让我们告诉计算机 **“做什么”**？（至于最底层怎么做，我们无需关心。）

举个例子，对于写UI而言，

- 指令式编程：就是，**怎么画？** 把每个`frame`、`layout`等等统统需要计算到位。
- 声明式编程：就是，**画什么？** 把想要的效果描述出来，其他都交给框架去做。

##### 3. 开发环境：

这么新的技术肯定需要环境的支持。`SwiftUI`所需要的开发环境，如下：

- Xcode：`Xcode 11.1+`。
- MacOS：`MacOS 10.15+`。
- iOS：`iOS 13+`。

> *PS：由于`SwiftUI`只能应用与`iOS 13`系统以上的设备。 因此，这项技术不建议用在需要适配低版本（`iOS 13` 以下）的App上。 不过如果是无需适配低版本的新项目，或者学习者全可以上手“玩一玩”。 毕竟苹果的新技术还是很有意思的嘛~*

### 二、SwiftUI的基本组件（语法）

> 这块知识比较“基础”且“重要”。只有记住了这些基本组件，我们才能用较少的代码开发出精美的App。

下面，我将给大家介绍一些重要的`SwiftUI`组件：

组件介绍：

| 名称       | 含义                                                     |
| ---------- | -------------------------------------------------------- |
| Text       | 用来显示文本的组件，类似UIKit中的`UILabel`。             |
| Image      | 用来展示图片的组件，类似UIKit中的`UIImageView`。         |
| Button     | 用于可点击的按钮组件，类似UIKit中的`UIButton`。          |
| List       | 用来展示列表的组件，类似UIKit中的`UITableView`。         |
| ScrollView | 用来支持滑动的组件，类似UIKit中的`UIScrollView`。        |
| Spacer     | 一个灵活的空间，用来填充空白的组件。                     |
| Divider    | 一条分割线，用来划分区域的组件。                         |
| VStack     | 将子视图按**“竖直方向”**排列布局。（`Vertical stack`）   |
| HStack     | 将子视图按**“水平方向”**排列布局。（`Horizontal stack`） |
| ZStack     | 将子视图按**“两轴方向均对齐”**布局（居中，有重叠效果）   |

基本组件：

- Text：用来显示文本的组件，类似UIKit中的`UILabel`。

```
Text("Hello, we are QiShare!").foregroundColor(.blue).font(.system(size: 32.0))
复制代码
```

- Image：用来展示图片的组件，类似UIKit中的`UIImageView`。

```
Image.init(systemName: "star.fill").foregroundColor(.yellow)
复制代码
```

- Button：用于可点击的按钮组件，类似UIKit中的`UIButton`。

```
Button(action: { self.showingProfile.toggle() }) {
    Image(systemName: "paperplane.fill")
        .imageScale(.large)
        .accessibility(label: Text("Right"))
        .padding()
}
复制代码
```

- List：用来展示列表的组件，类似UIKit中的`UITableView`。

```
List(0..<5){_ in
        NavigationLink.init(destination: VStack(alignment:.center){
            Image.init(systemName: "\(item+1).square.fill").foregroundColor(.green)
            Text("详情界面\(item + 1)").font(.system(size: 16))
    }) {
          //ListRow
       }
复制代码
```

- ScrollView：用来支持滑动的组件，类似UIKit中的`UIScrollView`。
- Spacer：一个灵活的空间，用来填充空白的组件。
- Divider：一条分割线，用来划分区域的组件。

布局组件：

- VStack：将子视图按**“竖直方向”**布局。（Vertical stack）
- HStack：将子视图按**“水平方向”**布局。（Horizontal stack）
- ZStack：将子视图按**“两轴方向均对齐”**布局。

功能组件：

- NavigationView：负责App中导航功能的组件，类似UIKit中的`UINavigationView`。
- NavigationLink：负责App页面跳转的组件，类似于`UINavigationView`中的`push`与`pop`功能。

```
NavigationView {
    List(0..<5){_ in
        NavigationLink.init(destination: VStack(alignment:.center){
            Image.init(systemName: "\(item+1).square.fill").foregroundColor(.green)
            Text("详情界面\(item + 1)").font(.system(size: 16))
    }) {
          //ListRow
       }
}
.navigationBarTitle("导航\(item)",displayMode: .inline)
复制代码
```

- TabView：负责App中的标签页功能的组件，类似UIKit中的`UITabBarController`。

```
TabView {
    Text("The First Tab")
        .tabItem {
            Image(systemName: "1.square.fill")
            Text("First")
        }
    Text("Another Tab")
        .tabItem {
            Image(systemName: "2.square.fill")
            Text("Second")
        }
    Text("The Last Tab")
        .tabItem {
            Image(systemName: "3.square.fill")
            Text("Third")
        }
}
.font(.headline)
复制代码
```

### 三、SwiftUI快速上手实践

下面让我们快速实现一个有TabView、NavigationView、List的Demo。



![img](https://user-gold-cdn.xitu.io/2019/11/21/16e8bb70076801f8?imageView2/0/w/1280/h/960/ignore-error/1)



`SF Symbols` 是从 `iOS 13` 和 `macOS 10.15` 开始内置于系统中的字符图标库，它提供了上千种常见的线条图标，而且我们可以任意地为它们设置尺寸，颜色等属性。Apple 甚至准备了专门的app：[SF Symbols](https://developer.apple.com/design/resources/) 来帮助你查看可用的符号：

接下来就让我们用这些`Symbols`制作个小Demo。

- ContentView：

```
import SwiftUI

struct ContentView: View {
    
    @State var isLeftNav = false
    @State var isRightNav = false
    
    init() {
        //修改导航栏文字颜色
        UINavigationBar.appearance().largeTitleTextAttributes = [.foregroundColor: UIColor.systemBlue]
        UINavigationBar.appearance().titleTextAttributes = [.foregroundColor: UIColor.systemBlue]
        UINavigationBar.appearance().tintColor = .systemBlue
    }
    
    var body: some View {
        TabView {
            
            // Tab1:
            NavigationView {
                List(Symbols, id:\.self) {
                    ListRow(symbol: $0)
                }
                .navigationBarTitle(Text("SF Symbols"))
                .navigationBarItems(leading: leftNavButton, trailing: rightNavButton)
            }.tabItem {
                Image.init(systemName: "star.fill")
                Text("Tab1").font(.subheadline)
            }
            
            // Tab2:
            NavigationView {
                Text("This is the second tab.")
            }.tabItem {
                Image.init(systemName: "star.fill")
                Text("Tab2").font(.subheadline)
            }
        }
    }
    
    var leftNavButton: some View {
        Button(action: { self.isLeftNav.toggle() }) {
            Image(systemName: "person.crop.circle")
                .imageScale(.large)
                .accessibility(label: Text("Left"))
                .padding()
        }
        .sheet(isPresented: $isLeftNav) {
            VStack {
                Text("Hello, we are QiShare!").foregroundColor(.blue).font(.system(size: 32.0))

                HStack {
                    Spacer()
                    Spacer()
                    Text("an iOS Team. ").fontWeight(.black).foregroundColor(.purple)
                    Spacer()
                    Text("We are learning SwiftUI.").foregroundColor(.blue)
                    Spacer()
                }
            }
        }
    }
    
    var rightNavButton: some View {
        Button(action: { self.isRightNav.toggle() }) {
            Image(systemName: "paperplane.fill")
                .imageScale(.large)
                .accessibility(label: Text("Right"))
                .padding()
        }
        .sheet(isPresented: $isRightNav, onDismiss: {
            print("dissmiss RrightNav")
        }) {
            ZStack {
                Text("This is the Right Navi Button.")
            }
        }
    }
}
复制代码
```

- ListRow：List对应的Cell

```
struct ListRow: View {
    var symbol: String
    var body: some View {
        NavigationLink(destination: ListDetail(symbol: symbol)) {
            
            HStack {
                //图片
                Image(systemName: symbol)
                    .resizable()
                    .frame(width: 60, height: 60)
                    .foregroundColor(Colors.randomElement())
                //分割
                Divider()
                Spacer()
                //文字
                Text(symbol)
                Spacer()
            }
        }
    }
}
复制代码
```

- ListDetail：

```
import SwiftUI

struct ListDetail: View {
    
    var symbol: String
    
    var body: some View {
        VStack {
            
            Text("Image:").font(.headline)
            
            Spacer()
            
            Image(systemName: symbol)
                .foregroundColor(Colors.randomElement())
                .imageScale(.large)
                .scaleEffect(3)
                .padding(.bottom, 100)
            
            Divider()
            
            Text("Image Name:").font(.headline)
            Spacer()
            Text(symbol)
                .font(.largeTitle)
            Spacer()
        }
        .navigationBarTitle(symbol)
    }
}
复制代码
```

源码：[本文Demo](https://github.com/QiShare/Qi_SwiftUI_Demo)

---

【完】