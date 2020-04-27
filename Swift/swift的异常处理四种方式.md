来自：[Swift错误处理](https://www.jianshu.com/p/b328c0dc2251?utm_campaign=haruki&utm_content=note&utm_medium=reader_share&utm_source=weixin)

#### 定义

错误处理是响应错误以及从错误中回复的过程，那这个过程包括抛出、捕获、传递和操作可恢复错误的支持。

#### 讨论

有些操作可能无法在所有的状态中都能返回一个有意义的值，当然可选类型可以帮忙表示失败的结果，但是错误处理可以让你更好的理解错误原因。
 swift关于错误处理的是协议Error，这个是一个控协议，可以通过**枚举**和**结构体**遵循这个协议来实现错误处理。

swift的异常处理有四种方式

- 通过函数的方式传递下去
- do-try-catch方式捕获
- try？将错误作为可选性处理，错误时返回nil。
- try！ 断言错误不会发生，但是如果发生了会有运行时错误。

#### 例子

举个自动贩卖机的例子。当我们去自动贩卖机买东西的时候，可能遇到以下问题

```swift
enum VendingMachineError: Error {
    case invalidSelection                   //选择无效
    case insufficientFunds(coinsNeeded: Int)//金额不足
    case outOfStock                         //缺货
}
```

接下来我们创建一个自动贩卖机类型

```swift
//商品结构体
struct Item {
    var price: Int
    var count: Int
}

//自动贩卖机
class VendingMachine {
    var inventory = [
        "Candy Bar": Item(price: 12, count: 7),
        "Chips": Item(price: 10, count: 4),
        "Pretzels": Item(price: 7, count: 4)
    ]
    
    var coinsDesposited = 2
    func vend(itemNamed name: String) throws {
        guard let item = inventory[name] else {
            throw VendingMachineError.invalidSelection
        }
        
        guard item.count > 0 else {
            throw VendingMachineError.outOfStock
        }
        
        guard item.price <= coinsDesposited else {
            throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDesposited)
        }
        
        var newItem = item
        newItem.count -= 1
        inventory[name] = newItem
        
        print("dispensing\(name)")
    }
}
```

由上可知，我们分别检验商品是否存在，库存是否充足以及投币是否足够。如果不满足就抛出异常。下面我们分别看一下上面说过的四种错误处理方式。
 可以看出在函数体后，返回值之前添加throws关键字表明该方法可能抛出异常。



#### 1、通过函数的方式传递下去

比如添加了一个购买者姓名的属性，通过`try + 方法`调用的方式可以将异常传递下去。

```swift
func vend(itemNamed name: String, buyerName:String) throws {
    self.buyerName = buyerName
    try vend(itemNamed: name)
}
```

#### 2、do-try-catch方式捕获

```swift
var test = VendingMachine()
do {
    try test.vend(itemNamed: "Chips")
} catch VendingMachineError.invalidSelection {
    print("invalidSelection")
} catch VendingMachineError.outOfStock {
    print("outOfStock")
} catch VendingMachineError.insufficientFunds(let coinsNeeded) {
    print("insufficientFunds \(coinsNeeded)")
}
```



通过do可以捕获捕获方法作用于内抛出的异常，catch可以以枚举的方式处理。

#### 3、try？将错误作为可选性处理，错误时返回nil。

```swift
let x = try? test.vend(itemNamed: "Chips")
print(x)
///打印：
///nil
```

通过try?的方式，当排出异常的时候x值为nil，正常可以返回一个可选型返回值。

#### 4、try！ 断言错误不会发生，但是如果发生了会有运行时错误。

```swift
let x = try! test.vend(itemNamed: "Chips")
print(x)
///崩溃，信息如下：
///Fatal error: 'try!' expression unexpectedly raised an error: __lldb_expr_5.VendingMachineError.insufficientFunds(coinsNeeded: 8): file Swift_do_try_catch_try_throw(s).playground, line 69
```

当我们断言方法不会抛出异常的时候可以用try！抛出异常的时候会有运行时错误，比较危险。





#### 结构体表示Error

有些时候我们要描述的错误比较复杂，我们也可以用结构体遵循Error协议去实现异常处理。
 官方给出的例子,xml解析错误

```swift
struct XMLParsingError: Error {
    enum ErrorKind {
        case invalidCharacter
        case mismatchedTag
        case internalError
    }
    
    let line: Int
    let column: Int
    let kind: ErrorKind
}
```

捕获错误

```swift
do {
    let xmlDoc = try parse(myXMLData)
} catch let e as XMLParsingError {
    print("Parsing error: \(e.kind) [\(e.line):\(e.column)]")
} catch {
    print("Other error: \(error)")
}
```

综上swift中的错误处理是非常灵活的，可以用多种数据结构描述错误。



1. `throws`：是用在可能会抛出异常的方法的后面，在`-> 返回参数`的前面。

2. `throw`：用于抛出异常时，结构为：

   ```swift
   throw 错误枚举值;
   ```

   例如：

   ```swift
   func vend(itemNamed name: String) throws {
       guard let item = inventory[name] else {
           throw VendingMachineError.invalidSelection
       }
       //...
   }
   ```

3. `try`、`try?`、`try!`：用于调用会可能会抛出异常的方法（方法后有`throws`修饰）时，在方法前加这3个关键字中的一个。结构如下：

   ```swift
   try 调用的方法
   ```

   例1：

   ```swift
   class VendingMachine {
       //... ...
       func vend(itemNamed name: String) throws {
       //... ...
       }
   }
   
   func vend(itemNamed name: String, buyerName:String) throws {
       self.buyerName = buyerName
       try vend(itemNamed: name)		//调用后面有`throws`修饰的`vend(itemNamed:)`方法
   }
   ```

   例2：

   ```swift
   let x = try? test.vend(itemNamed: "Chips")
   print(x)
   ///打印：
   ///nil
   ```

   例3：

   ```swift
   let x = try! test.vend(itemNamed: "Chips")
   print(x)
   ///崩溃，信息如下：
   ///Fatal error: 'try!' expression unexpectedly raised an error: __lldb_expr_5.VendingMachineError.insufficientFunds(coinsNeeded: 8): file Swift_do_try_catch_try_throw(s).playground, line 69
   ```

   

   另外，`try`可以和`do - try - catch`一起使用。

