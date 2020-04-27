# 在Swift中从JSON解码money

来自：[Decoding money from JSON in Swift](https://medium.com/wultra-blog/decoding-money-from-json-in-swift-d61a3fcf6404)

尽管看起来简单而琐碎，但在Swift中处理money——尤其是amount部分——可能是一件非常麻烦的事情。

假设我们在Swift中为我们正在开发的货币汇率app分配了这个简单的任务:

**将事务量从JSON反序列化为Swift结构**
![This is JSON that comes from your API](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Decoding%20money%20from%20JSON%20in%20Swift_1.png)

## Easy!
这应该是2分钟的代码。让我们打开Xcode并编写概念证明。
```
import UIKit
import PlaygroundSupport

struct Money: Codable {
    let amount: Double
    let currency: String
}
let json = "{\"amount\": 128.8,\"currency\": \"EUR\"}"
let jsonData = json.data(using: .utf8)!

let decoder = JSONDecoder()

let money = try! decoder.decode(Money.self, from: jsonData)

money.amount
money.currency
```
![This is JSON that comes from your API](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Decoding%20money%20from%20JSON%20in%20Swift_2.png)

## 那么问题来了
每个QA工程师都知道，输入字段是每个app中最常见的低挂起bug的来源。

让我们试着把欧元兑换成[巴林第纳尔](https://en.wikipedia.org/wiki/Bahraini_dinar)，他说。
```
import UIKit
import PlaygroundSupport

struct Money: Codable {
    let amount: Double
    let currency: String
}
let json = "{\"amount\": 9159795.995,\"currency\": \"BHD\"}"
let jsonData = json.data(using: .utf8)!

let decoder = JSONDecoder()

let money = try! decoder.decode(Money.self, from: jsonData)

money.amount
money.currency
```
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Decoding%20money%20from%20JSON%20in%20Swift_3.png)

Damn you, 浮点运算！

使用浮点类型可能不是最好的解决方案，试试`Decimal`。也许一开始就用`Double`是个错误。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Decoding%20money%20from%20JSON%20in%20Swift_4.png)

为什么？为什么有`Codable`的`Decimal`不能工作？它应该（工作的）！

## Try “everything”

1. 使用`NSDecimalNumber`——不行，没有`Codable`.
2. 使用`Double`然后转换为`NSDecimalNumber`——不行，返回`9159795.994999995`.
3. 尝试以字符串的形式获取`amount`——不行，不能反序列化(期望解码字符串，然而发现的是一个数字)
4. 也许有一种类似于`DateDecodingStrategy`的数字解码策略!——没有。
5. 搜索另一种数字数据类型——不，没有可以用的了。
6. 一个小时的谷歌session——不行，没有对这个问题的解决方案(和没有太多的信息)
7. 质疑我们对Swift和iOS本身的编程技能和知识——没有，还好。

## 发生了什么？
当我们绝望的时候，试着自己解析这个数字。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Decoding%20money%20from%20JSON%20in%20Swift_5.png)

当我们向`Decimal`初始化器提供有问题数字的字符串值时，一切正常。那么问题在哪里呢?当“amount”数据类型为`Decimal`时，解析器为什么返回不准确的数字?

## 总是Double
这个issue的问题是`JSONDecoder`本身。它使用`JSONSerialization`类进行反序列化，这个类具有解析数字的简单逻辑。如果这个数不是整数，那么它就是`Double`。就是这样。

由于我们使用的是`JSONDecoder`，我们无法告诉它我们不想先转换成`double`。当数据类型为`Decimal`时，`JSONDecoder`将把该数字解析为`Double`，然后将其转换为`Decimal`。This is bullshit.

[Pull request on GitHub](https://github.com/apple/swift-corelibs-foundation/pull/1657)解决了这个issue！万岁！删除必要的东西需要一些时间，但是不费什么力气，我们最终会得到我们自己的“forked”的`BetterJSONDecoder`。

正如您在这个[pull request](https://github.com/apple/swift-corelibs-foundation/pull/1657/files#diff-e0f6f6cf02b967eaa6fef534a0b6eae6R951)中所看到的，`Decimal`的精度要好得多，但是巨大的数字(在指数方面)仍然表示为Double。在处理加密货币中非常小的部分，或由恶性通货膨胀([hyperinflation](https://en.wikipedia.org/wiki/Hyperinflation))引起的大额amounts时，这可能会造成一些困难。

你也可以在[swift bugs](https://bugs.swift.org/browse/SR-7054)上深入挖掘这个问题。

## 解决方案
最后，您可以使用4种方法来解决这个问题(issue)。

### 四舍五入
您可以将解析后的数字四舍五入到小数点后两位(或者三位在某些货币中)。这是一个低成本且快速的解决方案，这个方案在大多数情况下都有效。

### 自己反序列化
正如上面所看到的，您可以编写自己的`JSONDeserializer`(或者像上面所看到的那样进行修改，或者使用第三方库——但是要小心，例如，流行的[`ObjectMapper`](https://github.com/Hearst-DD/ObjectMapper)也存在与使用`JSONSerialization`类相同的问题)。这可能需要一些时间来开发，并可能在为了带来更多的问题。但如果你需要做一些计算，现在还没有“原生方法”。请注意，对于庞大的数字仍然存在问题。

### 使用String
如果您拥有整个app堆栈(包括后端)，或者可以让开发后端程序的开发人员更改它，使用字符串。Java和其他语言可以比Swift更好地处理大的数字。很遗憾，但这是最简单和最好的解决方案(对其他平台的影响很小)。这也是我最后得到的解决方案。


图像6：我们现在同时使用数字和格式化字符串，以便在前端更准确、更容易地表示
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Decoding%20money%20from%20JSON%20in%20Swift_6.png)

### 使用有效位数和指数
另一个选项是使用更多的“科学符号”，该选项需要在后端更改，[Reddit上](https://www.reddit.com/r/swift/comments/9duphc/decoding_money_from_json_in_swift/e5l5lsr)的一位评论者指出。
您可以使用两个整数来消除小数点的需要。使用这种方法，`Decimal`已经提供了可以接受此值的初始化器。
```
var a = Decimal(integerLiteral: 1089687685)
a._exponent = -7
print("a = ",a)
```
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Decoding%20money%20from%20JSON%20in%20Swift_7.png)

### The Apple way
正如我的前同事曾经告诉我的那样:苹果在WWCC期间在舞台上做的事情看起来简单又漂亮，并且将满足95%的案例的需求。但总有一种情况是你需要的，但你做不到，最终你会得到你自己的答案。

这对苹果来说是很常见的情况。我们不必走得太远。试试上面提到的`DateDecodingStrategy`。对于`.iso8601`，文档中说:[将`Date`解码为iso -8601格式的字符串(在RFC 3339中的格式)](https://developer.apple.com/documentation/foundation/jsondecoder/datedecodingstrategy/iso8601)。我鼓励您反序列化这个有效的案例:`2002-10-02T15:00:00.05Z`。

---
[完]


