# typealias与associatedtype

## 文章1：Swift 5.0-typealias与associatedtype
来自：[Swift 5.0-typealias与associatedtype](https://juejin.im/entry/5cbd4b0e6fb9a03246585112)

Swift 中关键字typealias重命名与associatedtype关联对象，在实际开发中比较常见。

# typealias

typealias 是用来为已经存在的类型重新定义名字的，通过命名，可以使代码变得更加清晰。使用的语法也很简单，使用 typealias 关键字像使用普通的赋值语句一样，可以将某个已经存在的类型赋值为新的名字。

typealias类似于Objective-C中的typedef，简单的将block重命名如下:

```swift
typealias success = (_ data: String) -> Void
typealias fail = (_ error: String) -> Void

func fetchData(_ url: String, success: success, fail: fail) {
}
```

# associatedtype

associatedtypen表示位置的数据类型，只是先定义一个名字，具体表示的类型要在最终使用的时候进行赋值。在定义协议时，可以用associatedtype声明一个或多个类型作为协议定义的一部分。

```swift
protocol NetworkRequest {
    associatedtype DataType
    func didReceiveData(_ data: DataType)
}

class ViewModel: NetworkRequest {
    
    typealias DataType = String
    
    func didReceiveData(_ data: DataType) {
        
    }
    
}
```

## 文章2：associatedtype
来自：[associatedtype](https://www.jianshu.com/p/2e17af7858e0)

> 从字面上来理解，就是相关类型。意思也就是被associatedtype关键字修饰的变量，相当于一个占位符，而不能表示具体的类型。具体的类型需要让实现的类来指定。

```swift
protocol Food { }

protocol Animal {
    func eat(_ food: Food)
}

struct Meat: Food { }

struct Lion: Animal {
    func eat(_ food: Food) {
        if let meat = food as? Meat {	//转换，要在运行时确定
            print("eat \(meat)")
        } else {
            fatalError("Tiger can only eat meat!")
        }
    }
}

let meat = Meat()
Lion().eat(meat)	//打印结果为：eat Meat()
```

在实现中的转换很多时候并没有太多的意义，而且将责任扔给了运行时。**好的做法是让编译的时候就确定`Food`的类型，这种情况就可以用`associatedtype`。**

```swift
protocol Food { }

protocol Animal {
    associatedtype F: Food  //
    func eat(_ food: F)
}

struct Meat: Food { }

struct Lion: Animal {
    func eat(_ food: Meat) { //编译时确定类型，不需要判断类型
        print("eat \(food)")
    }
}

let meat = Meat()
Lion().eat(meat)
```

不过在添加 `associatedtype` 后，`Animal` 协议就不能被当作独立的类型使用了。这时候就需要使用泛型。

```swift
///报错：Protocol 'Animal' can only be used as a generic constraint because it has Self or associated type requirements
///意思是：协议“Animal”只能用作通用约束，因为它有自己或关联类型的要求
func isDangerous(animal: Animal) -> Bool {

}
```

使用泛型如下：

```swift
func isDangerous<T: Animal>(animal: T) -> Bool {
    if animal is Lion {
        return true
    } else {
        return false
    }
}
```

## 文章3：Swift - 关键字（typealias、associatedtype）

来自：[Swift - 关键字（typealias、associatedtype）](https://blog.csdn.net/longshihua/article/details/74347889)

#### Typealias
**`typealias` 是用来为已经存在的类型重新定义名字的**，通过命名，可以使代码变得更加清晰。使用的语法也很简单，**使用typealias 关键字像使用普通的赋值语句一样，可以将某个已经存在的类型赋值为新的名字**。



比如在计算二维平面上的距离和位置的时候，我们一般使用Double来表示距离，用CGPoint来表示位置:

```swift
func distance(_ point: CGPoint, _ anotherPoint: CGPoint) -> Double {
    let dx = Double(anotherPoint.x - point.x)
    let dy = Double(anotherPoint.y - point.y)
    return sqrt(dx * dx + dy * dy)
}
 
let origin = CGPoint(x: 3, y: 0)
let point = CGPoint(x: 4, y: 0)
 
let d = distance(origin, point)
```

虽然在数学上和最后的程序运行上都没什么问题，但是阅读和维护的时候总是觉得有哪里不对。 因为我们没有将数学抽象和实际问题结合起来，使得在阅读代码时我们还需要在大脑中进行一次额外的转换:CGPoint代表一个点，而这个点就是我们在定义的坐标系里的位置; Double是一个 数字，它代表两个点之间的距离。

如果我们使用 `typealias`，就可以将这种转换直接写在代码里，从而减轻阅读和维护的负担:

```swift
typealias Distance = Double
typealias Location = CGPoint

func distance(_ point: CGPoint, _ anotherPoint: CGPoint) -> Double {
    let dx = Distance(anotherPoint.x - point.x)
    let dy = Distance(anotherPoint.y - point.y)
    return sqrt(dx * dx + dy * dy)
}
 
let origin = Location(x: 3, y: 0)
let point = Location(x: 4, y: 0)
 
let d = distance(origin, point)
```

同样的代码，在 typealias 的帮助下，读起来就轻松多了。可能单单这个简单例子不会有特别多的 体会，但是当你遇到更复杂的实际问题时，你就可以不再关心并去思考自己代码里那些成堆的Int或者String之类的基本类型到底代表的是什么东西了，这样你应该能省下不少脑细胞。

注意：开发过程中，当使用的闭包的使用，会经常使用`typealias`

```swift
typealias Success = (_ data: String) -> Void
typealias Failure = (_ error: String) -> Void
 
func request(_ url: String, success: Success, failure: Failure) {
    // do request here ....
}
```

#### typealias与泛型

typealias 是单一的，也就是说**你必须指定将某个特定的类型通过`typealias`赋值为新名字，而不能将整个泛型类型进行重命名**。下面这样的命名都是无法通过编译的:

```swift
class Person<T> {}
typealias Woker = Person
typealias Worker = Person<T>    //报错：Use of undeclared type 'T'
```

1、不过如果我们在别名中也引入泛型，是可以进行对应的:

```swift
class Person<T> {}
typealias Woker = Person
typealias Worker<T> = Person<T>    //可以编译通过
```

2、 _当泛型类型的确定性得到保证后，显然别名也是可以使用的_:

```swift
class Person<T> {}
typealias WokerId = String
typealias Worker = Person<WokerId>    //可以编译通过
```

3、另一个使用场景是**某个类型同时实现多个协议的组合时。我们可以使用`&`符号连接几个协议，然后给它们一个新的更符合上下文的名字，来增强代码可读性**:

```swift
protocol Cat {
    func miaoMiao()
}
protocol Dog {
    func wangWang()
}
typealias Pat = Cat & Dog

//遵循了 Pat协议后，需要同时实现Cat 和 Dog协议里面的内容
class UnknownAnimal: Pat {
    func miaoMiao() {
        print("miao miao...")
    }
    
    func wangWang() {
        print("wang wang...")
    }
}
```

#### associatedtype关联类型

定义一个协议时，有的时候声明一个或多个关联类型作为协议定义的一部分将会非常有用。关联类型为协议中的某个类型提供了一个占位名（或者说别名），其代表的实际类型在协议被采纳时才会被指定。你可以通过 `associatedtype` 关键字来指定关联类型。比如使用协议声明更新cell的方法：

```swift
//模型
struct Model {
    let age: Int
}

//协议，使用关联类型
protocol TableViewCell {
    associatedtype T
    func updateCell(_ data: T)
}

//遵守TableViewCell,需要实现 `typealias T = ...` 和 `updateCell`方法
class MyTableViewCell: UITableViewCell, TableViewCell {
    typealias T = Model
    func updateCell(_ data: Model) {
        // do something ...
    }
    
}
```

