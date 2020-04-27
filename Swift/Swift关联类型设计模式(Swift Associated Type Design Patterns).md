# Swift关联类型设计模式(Swift Associated Type Design Patterns)

来自：[Swift Associated Type Design Patterns](https://medium.com/@bobgodwinx/swift-associated-type-design-patterns-6c56c5b0a73a)



Swift是一种多范式编程语言，这意味着你可以做面向对象、切面式(aspect-oriented，面向方面)、过程性、函数性或POP。仅举几例。最后一个“POP”表示面向协议的编程。在2015年全球开发者大会上，一切都改变了，戴夫·亚伯拉罕斯(Dave Abrahams)就这个概念和新的思维方式发表了演讲。他是这样开头的:

### 新的思维方式:

> 接下来的40分钟，我们将把通常的编程思维方式放在一边。我们在这里一起做的事情不一定很容易，但我向你保证，如果你和我在一起，你的时间是值得的。



如果你还没看过这个视频，我郑重建议你现在就看。因为我在这篇文章中要做的就是分解相同的视频。

同一年，[Alexis Gallagher发表了一次演讲](https://www.youtube.com/watch?v=XWoNjiSPqI8)，在演讲中他试图解决在使用swift编程语言处理相关类型时遇到的一些困难。这不是一个容易理解的概念，[Benedikt Terhechte](https://appventure.me/guides/associated_types/associated_types.html)写过关于这个主题的文章，Russ Bishop也写过[关于关联类型的回忆录](http://www.russbishop.net/swift-associated-types)，John Sundell也[对关联类型表示了敬意](https://www.swiftbysundell.com/posts/type-erasure-using-closures-in-swift)。Robert Edwards给出了他[关于类型擦除的分解](https://www.bignerdranch.com/blog/breaking-down-type-erasures-in-swift/)，Lee kah seng写下了他关于如何在使用[协议相关类型](https://medium.com/@kahseng.lee123/accomplishing-dynamic-dispatch-on-pats-protocol-with-associated-types-b29d1242e939)时获得动态分派(dynamic dispatch)的发现。他们都试图理解[参数多态性](https://en.wikipedia.org/wiki/Parametric_polymorphism)(parametric polymorphism)是如何工作的。



### 恼人的错误信息之美:

> protocol can only be used as a generic constraint because it has self or associated type.
>
> 协议只能用作通用约束，因为它具有自类型或关联类型。

在你看到上面提到的错误而开始砸键盘之前。让我们准确地定义什么是关联类型。

### 关联类型的定义：

`associatedtype`是一个未知`具体类型(Concrete Type)`的协议通用占位符，需要在**编译时**进行具体化。

### 明确的编译时间 vs 运行时:

运行时(Runtime)和编译时(Compile time)是指软件应用程序的不同阶段的编程术语。编译时是将代码转换为可执行代码的实例，而运行时是将转换后的可执行代码实际执行的实例。



### 关联类型的起源:

这一概念首次出现在《函数式编程杂志》(*The Journal of Functional Programming*)的一份出版物中，其标题为:[泛型编程语言支持的扩展比较研究](https://pdfs.semanticscholar.org/e5fb/cfb07adfbdd0e035b2f6ac020e51cd562dfe.pdf)( *extended comparative study of language support for generic programming*)。他们强调多类型概念，这是Swift协议关联类型的根源。Swift还从Scala的[特性和抽象类型](https://docs.scala-lang.org/tour/abstract-types.html)、Haskell的[多参数类型类](https://en.wikibooks.org/wiki/Haskell/Advanced_type_classes)和[Rust相关类型](https://doc.rust-lang.org/1.12.0/book/associated-types.html)中获得了一些灵感。然后，它利用[标准库中的多类型概念](https://github.com/apple/swift/blob/2fe4254cb712fa101a220f95b6ade8f99f43dc74/stdlib/public/core/ExistentialCollection.swift.gyb#L45)实现其集合类型。

### 关联类型解决的问题:

* 引入`associatedtype`是为了解决面向对象的子类型所不具备的丰富多类型抽象问题。
* 设计用于解决已知的简单`generic protocol(泛型协议)`实现，特别是当复杂性随着泛型类型引入的增加而急剧增加时。
* 使语言更具表现力的同时，维护静态类型安全。

### 关联类型的优势:

* 它们有助于避免实现细节的泄露，而这通常需要反复指定。
* associatedtype捕获`Types`之间的丰富`Type`关系。
* 它们有助于在协议子`Type`中指定对象的精确类型，而不会影响`Type`定义。
* 它们提供了您无法放入与对象相关的类型层次结构中的关系，尤其是在[Liskov替换原则](https://medium.com/@bobgodwinx/solid-principles-part-3-43aad943b056)(Liskov substitution principle)无法处理`Type`多态关系的情况下。
* 它们执行一个同构集合，该集合使用优化的静态分派代码来增强编译器。

### 关联类型注意事项(Caveats)：

* 很难理解，因为它有很高的学习曲线。
* 它会将您锁定在动态调度(**Dynamic** dispatch)之外。通过强制执行静态调度(**Static** dispatch)。
* 它只能在协议中使用。

### 动态调度与静态调度的区别:

动态分派是选择要在运行时调用多态操作(方法或函数)的哪个实现的过程，而静态调度是完全解析的编译时形式的多态操作(polymorphic operation)。



###用关联类型工作:



声明一个具有关联类型的协议非常简单(Declaring a protocol with associated types is pretty straight forward)，如下面的例子所示:

```swift
protocol TableViewCell {
    ///Unknown `Concrete Type` declared as `T`
    associatedtype T
    /// A function that accepts the unknown `Concrete Type`
    /// as it's parameter
    func configure(_ model: T)
}
```

> Tips：
>
>  [Protocols with Associated Types](http://www.slideshare.net/natashatherobot/practical-protocols-with-associated-types) (PATs)  ：有关联类型的协议

🤗我们可以很容易地实现协议采用(protocol adoption)，更好的说法是，在**Cocoa**术语中遵守协议，如下:

```swift
class Detail {
    /// `Concrete Type` that will replace the `T`
}

class ExtendedDetail {
    /// Potential `Concrete Type` that will replace the `T`
}

class Cell: TableViewCell {
    /// `associatedtype` adoption
    typealias T = Detail
    /// now the compiler knows and reuires
    /// to inject only `Detail` into the `func`
    func configure(_ model: T) {
        /// Configure your cool cell :)
    }
}
```

🍱让我们来明确区分什么是`associatedType`。如前所述，它们是通用占位符，但不是通用类型。您也可以将其称为参数多态性(parametric polymorphism)。

看看👀:

```swift
/// usage
let extended = ExtendedDetail()
let detail = Detail()
let detailCell = Cell()
/// This will error :
detailCell.configure(extended)
/// This will be successful.
detailCell.configure(detail)
```

在上面的实现中，应用程序将在#line 6处捕捉到以下错误:

> Cannot convert value of type 'ExtendedDetail' to expected argument type 'Detail'
> 无法将类型“ExtendedDetail”的值转换为预期的参数类型“Detail”



This is because on adoption or conformation to the protocol we are required to specify the `Concrete Type` and we did that by saying `typealias T = Detail` so therefore our function already knows at **Compile** time the `Concrete Type` to expect and that’s why it raises an exception if we try to use `ExtendedDetail` instead of `Detail` .

这是因为在采用或符合协议时，我们需要指定具体类型(`Concrete Type`)，我们通过输入`typealias T = Detail`来实现，所以我们的函数**在编译时**就已经知道了期望的具体类型(`Concrete Type`)，这就是为什么当我们尝试使用`ExtendedDetail`而不是`Detail`时，它会引发一个异常。

让我们添加`ExtendedCell`的实现，它遵循相同的协议，但是使用不同的具体类型(`Concrete Type`)。

```swift
class Cell: TableViewCell {
    /// `associatedtype` adoption
    typealias T = Detail
    /// now the compiler knows and reuires
    /// to inject only `Detail` into the `func`
    func configure(_ model: T) {
        /// Configure your cool cell :)
    }
}

class ExtendedCell: TableViewCell {
    /// `associatedtype` adoption
    typealias T = ExtendedDetail
    /// now the compiler knows and reuires
    /// to inject only `Detail` into the `func`
    func configure(_ model: T) {
        /// Configure your cool cell :)
    }
}

/// usage
let extended = ExtendedDetail()
let detail = Detail()
let detailCell = Cell()
let extendedDetailCell = ExtendedCell()
///下句会报错：Protocol 'TableViewCell' can only be used as a generic constraint because it has Self or associated type requirements
let cells: [TableViewCell] = [extendedDetailCell, detailCell]
```

🤷🏽‍♂️更进一步说，如果我们天真地决定创建一个TableViewCell集合，如上面的代码所示，那么美丽的错误消息将在`#line 27`🙈被触发:

> Protocol 'TableViewCell' can only be used as a generic constraint because it has Self or associated type requirements
>
> 协议'TableViewCell'只能作为一个通用约束使用，因为它有自己或关联类型的要求。

🧚🏽‍♂️这里唯一的救星,这个错误被称为类型擦除(*type erasure*)，但在我们开始之前我们先来看看这个术语(term)是什么意思?



### 类型擦除的定义:

> Type erasure refers to the compile-time process by which explicit type  annotations are removed from a program, before it is executed at  run-time.
>
> 类型擦除是指程序在运行时执行之前，从程序中删除显式类型注释(*explicit type annotations*)的编译时过程。
> explicit  [ɪkˈsplɪsɪt] adj. 明确的；清楚的；直率的；详述的



我们可以应用三种模式来解决通用约束需求(generic constraints requirement)的问题。

* 约束类型擦除(Constrained Type Erasure)：擦除类型，但在其上保留约束。
* 无约束类型擦除(Unconstrained Type Erasure)：擦除没有约束的类型。
* 阴影类型擦除(Shadow Type Erasure)：通过隐藏类型来擦除类型。
	camouflage  [ˈkæməflɑːʒ] n. 伪装，掩饰		vt. 伪装，掩饰		vi. 伪装起来

### 约束类型擦除(Constrained Type Erasure)

此模式在包装器类(`wrapper class`)上添加初始化器约束，以确保注入的泛型类型(`generic type`)与`associatedtype`匹配

guarantee   [ˌɡærənˈtiː] n. 保证；担保；保证人；保证书；抵押品		vt. 保证；担保

```swift
/// Rows `Interface`
protocol Row {
    /// PAT Placeholder for unknown Concrete Type `Model`
    associatedtype Model
    /// Recieves a parameter of Concrete Type `Model`
    func configure(with model: Model)
}
/// Concrete Type `Product`
struct Product { }
/// Concrete Type `Item`
struct Item { }

//MARK: - Constrained Type Erasure

/// Wrapper `AnyRow`
struct AnyRow<I>: Row {
    private let configureClosure: (I) -> Void
    /// Initialiser guaratees that `Model`
    /// should be a `Type` of `I`
    init<T: Row>(_ row: T) where T.Model == I {
        /// Matches the row `configure` func
        /// to the private the `configureClosure`
        configureClosure = row.configure
    }
    /// Conforming to `Row` protocol
    func configure(with model: I) {
        configureClosure(model)
    }
}
/// `ProductCell`
class ProductCell: Row {
    typealias Model = Product
    let name: String
    
    init(name: String) {
        self.name = name
    }
    /// Conforming to `Row` protocol
    func configure(with model: Model) {
        print("PATs PlaceHolder is now `Product` Concrete Type")
        print("This will now be configured based on \(type(of: self))")
    }
}
/// `ProductDetailsCell`
class ProductDetailsCell: Row {
    typealias Model = Product
    let name: String
    let category: String
    
    init(name: String, category: String) {
        self.name = name
        self.category = category
    }
    /// Conforming to `Row` protocol
    func configure(with model: Model) {
        print("PATs PlaceHolder is now `Product` Concrete Type")
        print("This will now be configured based on \(type(of: self))")
    }
}
/// Usage of PAT for Homogeneous Requirement
let productCell = ProductCell(name: "product-name")
let productDetailsCell = ProductDetailsCell(name: "product-name", category: "ABC-HT")
/// We get only a `Homogeneous` Collection Type
let cells: [AnyRow<Product>] = [AnyRow(productCell), AnyRow(productDetailsCell)]
let product = Product()
cells.forEach { (cell) in
    cell.configure(with: product)
}
```

💪🏾在上面的代码中我们使用`AnyRow`来消除`Type`需求当符合`Row`协议类型时。仔细看看`#line20`，我们发现使用子句:`where T.Model == I`来约束`init`函数。这也将我们锁定在一个同质集合`Type`中，如`#line 64`所示



### 无约束类型擦除(Unconstrained Type Erasure)

如果我们想要一个异构(heterogeneous)的集合类型，那么这个模式就能帮我们解决问题。Swift语言为处理非特定`Types`——`Any` & `AnyObject`，提供了两种特殊的`Types`。

>  heterogeneous  [ˌhetərəˈdʒiːniəs]adj. 由很多种类组成的，混杂的；各种各样的；（化学）不均一的，多相的；（数学）不纯一的，参差的

* `Any`可以表示任何类型的实例，包括函数类型。
* `AnyObject`可以表示任何类类型(class type)的实例。

有了这些信息，让我们实现一个类型擦除，它可以使用`Any`指定的异构收集。

```swift
/// Heterogeneous Requirement and Dynamic dispatch availability
/// Generic Wrapper `AnyCellRow` to match Heterogeneous Types + Dynamic Dispatch 通用包装器‘AnyCellRow’匹配异构类型+动态分派
struct AnyCellRow: Row {
    private let configureClosure: (Any) -> Void
    
    init<T: Row>(_ row: T) {
        configureClosure = { object in
            /// Asserting that `object` received is `type` of `T.Model`
            guard let model = object as? T.Model else { return }
            /// call the `T.configure` function on success
            row.configure(with: model)
        }
    }
    /// Conforming to `Row` protocol
    func configure(with model: Any) {
        configureClosure(model)
    }
}
/// `ItemCell`
class ItemCell: Row {
    typealias Model = Item
    let id: String
    
    init(id: String) {
        self.id = id
    }
    /// Conforming to `Row` protocol
    func configure(with model: Item) {
        print("PATs PlaceHolder is now `Item` Concrete Type)")
        print("This will now be configured based on \(type(of: self))")
    }
}
/// Usage of PAT for Heterogenous Requirement + Dynamic dispatch
let item = Item()
let itemCell = ItemCell(id: "an-itemCell")
/// We get a `Heterogenous`collection Type
let allCells = [AnyCellRow(productCell),
                AnyCellRow(productDetailsCell),
                AnyCellRow(itemCell)]

for (index, cell) in allCells.enumerated() {
    index <= 1 ? cell.configure(with: product) : cell.configure(with: item)
}
```

🎉通过`AnyCellRow`包装器的帮助，我们在遵守`Row`协议时消除了类型(`Type`)要求。init函数没有子句，现在我们的配置中有一个异构集合类型(`Type`)和动态调度(dynamic dispatch)。👍🏿

> 单词：
> conform [kənˈfɔːm] vi. 符合；遵照；适应环境
> vt. 使遵守；使一致；使顺从
> adj. 一致的；顺从的

### 阴影类型擦除(Shadow Type Erasure)

为了实现阴影类型擦除，我们需要添加另一个协议和重构`Row`协议，如下:



> 单词：
>
> refactor  [ri'fæktə]	n. 重构      	 vt. 重构

```swift
//MARK: - `Shadowed` Protocol Based Type Erasure

/// `shadow` protocol
protocol TableRow {
    /// - Recieves a parameter of Concrete Type `Any`
    func configure(with model: Any)
}
/// `Row` To be shadowed.
protocol Row: TableRow {
    associatedtype Model
    /// - Recieves a parameter of Concrete Type `Model`
    func configure(with model: Model)
}
```

The next step is to add a default implementation using swift `extension` for the `Row` protocol functions and properties.

下一步是为`Row`协议函数和属性添加一个使用swift `extension`的默认实现。

```swift
/// Default `extension` to conform to `TableRow`
extension TableRow {
    /// TableRow - conformation
    func configure(with model: Any) {
        /// Just throw a fatalError
        /// because we don't need it.
        fatalError()
    }
}
```

👏🏾有了这个我们就可以躲在幕后了，我们还可以使用`TableRow`作为一等公民，如下所示:

```swift
/// Usage of shadowed protocol styled type erasure
let rows: [TableRow] = [ProductCell(name: "Hello"), ItemCell(id: "123456")]

for row in rows {
    if let cell = row as? ProductCell {
        cell.configure(with: Product())
    }
    
    if let cell = row as? ItemCell {
        cell.configure(with: Item())
    }
}
```

### 总结(**Summary**)：

这些模式相互延迟，并产生不同的结果。

* 第一个是最优化的，因为你的集合将内联(inlined) swift编译器的静态调度。
* 第二种方法使我们可以使用动态调度(dynamic dispatch)和异构收集，但是这里有一个警告(caveat)，因为`AnyCellRow`可以用任何`Type`实例化，即使`Type`与我们正在处理的内容无关。
* 最后一种方法似乎介于两者之间，但是如果您有一个包含大量协议的大型代码库，那么它就会变得冗长和重复(verbose and repetitive)。

> 单词：
> caveat   [ˈkæviæt]  n. 警告；中止诉讼手续的申请；货物出门概不退换；停止支付的广告
> verbose  [vɜːˈbəʊs] adj. 冗长的；啰嗦的
> repetitive [rɪˈpetətɪv] adj. 重复的



### 来自Swift的`Any`和`AnoObject`的警告

只在显式地(explicitly)需要`Any`和`AnyObject`提供的行为和功能时才使用它们。在代码中指定希望使用的类型总是更好的。

> 单词：
> explicitly  [ɪkˈsplɪsɪtli] adv. 明确地；明白地
> specific [spəˈsɪfɪk] adj. 特殊的，特定的；明确的；详细的；[药] 具有特效的
> 							n. 特性；细节；特效药

在这条消息之后，我确实说过应该由您根据您所处的情况来选择最佳的套件(suites)。需要记住的一点是，需要记住的一点是，`Any`在某些情况下可能会比较棘手(tricky)。Any类型表示任何类型的值，包括可选类型。但当查询时，如果对象不是`nil`，它将在下拉类型转换中返回`true`。关于这方面的更多信息，请咨询[Swift Type Casting](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html)。



您可以在我的[GitHub Playground repository](https://github.com/bobgodwinx/Playground)找到本文讨论的代码示例

谢谢你的阅读，我希望我能娱乐(entertain)你。我喜欢用Swift语言写博客，所以如果你有什么特别想让我写的，请随时联系[twitter](https://twitter.com/bobgodwinx)。



### 特别感谢：

![He is a genius](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Swift_Associated_Type_Design_Patterns1.png)

## Other Articles:

- [MVVM ❤️ RxSwift ❤️ Swift 4.x](https://medium.com/@bobgodwinx/mvvm-️-rxswift-️-swift-4-x-d86d6e24419e)
- [Declarative Programming with Swift](https://medium.com/@bobgodwinx/declarative-programming-with-rxswift-37a6d42a7d)(使用Swift进行声明式编程)
- [Solid Principles — Dependency Inversion](https://medium.com/@bobgodwinx/solid-principles-part-5-b1d2047c2d55)(一致原则——依赖倒置)
- [Solid Principles — Interface Segregation](https://medium.com/@bobgodwinx/solid-principles-part-4-13de4d4d7571)(一致原则——界面分离)
- [Solid Principles — Liskov Substitution](https://medium.com/@bobgodwinx/solid-principles-part-3-43aad943b056)(一致原则——Loskov代换)
- [Solid Principles —Open/Closed](https://medium.com/@bobgodwinx/solid-principles-part-2-a22d4c8ed906)(一致原则——打开/关闭)
- [Solid Principles —Single Responsibility](https://medium.com/@bobgodwinx/solid-principles-part-1-f3d11b3159f0)(一致原则——单一职责)
- [Swift Access Levels. fileprivate](https://medium.com/@bobgodwinx/swift-access-levels-fileprivate-606f5c2c165)(Swift快速访问 Levels.fileprivate)

> 单词：
> tricky [ˈtrɪki] adj. 狡猾的；机警的；棘手的
> repository   [rɪˈpɒzətri] n. 贮藏室，仓库；知识库；智囊团
> Segregation  [ˌseɡrɪˈɡeɪʃn] n. 隔离，分离；种族隔离
> Inversion  [ɪnˈvɜːʃn]  n. 倒置；反向；倒转
> Substitution  [ˌsʌbstɪˈtjuːʃn] n. 代替；[数] 置换；代替物
> Responsibility   [rɪˌspɒnsəˈbɪlətɪ] n. 责任，职责；义务