# 转：让closure在function外继续使用`@escaping`

来自：[讓 closure 在 function 外繼續使用的 @escaping](https://medium.com/%E5%BD%BC%E5%BE%97%E6%BD%98%E7%9A%84-swift-ios-app-%E9%96%8B%E7%99%BC%E5%95%8F%E9%A1%8C%E8%A7%A3%E7%AD%94%E9%9B%86/%E8%AE%93-closure-%E5%9C%A8-function-%E5%A4%96%E7%B9%BC%E7%BA%8C%E4%BD%BF%E7%94%A8%E7%9A%84-escaping-40d50b17f75b)

---
`@escaping`是个让closure在function外继续使用的特别语法。它有点难懂，但你却不能忽略，因为iOS DK里不少function的参数都加了`@escaping`。在了解它的神奇功能前，先让我们看看一下例子，认识它帮我们解决的问题。
```
class Baby {
    var name = "peter"
    var favoriteActivity: (() -> ())!
    func outsideActivity(activity: () -> ()) {
        activity()
        favoriteActivity = activity
    }
}
```
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E8%AE%A9closure%E5%9C%A8function%E5%A4%96%E7%BB%A7%E7%BB%AD%E4%BD%BF%E7%94%A8%60%40escaping%60_1.png)

如图所示，将`activity`储存到`favoriteActivity`会产生红色错误，这是因为Swift认为传入function的function型别参数，比如例子里打印出【打桌球】的closure，只是为了在function里执行，function执行完后就没有利用价值，应该甩了它，不该在function执行完后继续使用，所以它不让我们将`activity`储存到`favoriteActivity`。若能储存，即代表刚刚李自力的`cuteBaby`可以调用`favoriteActivity`，执行当初closure的程式，打印出【打桌球】。

想让Swift知道我们已经对传入function的closure动了真感情，想在function执行完后继续使用，很简单，只要在参数声明前加上`@escaping`即可。英文`escape`的意思是逃脱，所以就像它名字表达的，`@escaping`可让closure从function逃脱，function执行完后仍可继续使用。
```swift
class Baby {
    var name = "peter"
    var favoriteActivity: (() -> ())!
    func outsideActivity(activity: @escaping () -> ()) {
        activity()
        favoriteActivity = activity
    }
}

var cuteBaby = Baby()
cuteBaby.outsideActivity {
    print("打桌球")
}

cuteBaby.favoriteActivity()
```
打印结果：
```
打桌球
打桌球
```

`@escaping`帮我们解决问题，但也带来一些副作用，就像止痛药一样。例如以下例子，当你在closure的`{}`里存取物件自己的属性时，必须额外加上`self`，否则会产生红色错误。(ps：加`self`跟ARC有关，因为加了`@escaping`，closure的程式之后还会执行，所以要加`self`提醒ARC增加`self`的reference count，如此才不会再closure程式执行时`self`已经死掉，无法存取`self`的property或method。)
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E8%AE%A9closure%E5%9C%A8function%E5%A4%96%E7%BB%A7%E7%BB%AD%E4%BD%BF%E7%94%A8%60%40escaping%60_2.png)
```swift
class Baby {
    var name = "peter"
    var favoriteActivity: (() -> ())!
    func outsideActivity(activity: @escaping () -> ()) {
        activity()
        favoriteActivity = activity
    }
}

class Mother {
    var name = "wendy"
    var child = Baby()
    func play() {
        child.outsideActivity {
            print("\(self.name)和h小孩\(self.child.name)打桌球")
        }
    }
}
```

相反的，当参数没有`@escaping`时，传入的closure在存取物件自己的属性时，你爱加不加`self`都可以。
```swift
class Baby {
    var name = "peter"
    var favoriteActivity: (() -> ())!
    func outsideActivity(activity: () -> ()) {
        activity()
    }
}

class Mother {
    var name = "wendy"
    var child = Baby()
    func play() {
        child.outsideActivity {
            print("\(self.name)和h小孩\(self.child.name)打桌球")
        }
    }
}
```

以上应该就是`@escaping`的故事吧。等等，当参数是`optional`时，故事又有了转折。当参数是`optional`时，Swift将认定此参数时`@escaping`，所以在closure的`{}`里存取物件自己的属性时，一样要加上`self`。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E8%AE%A9closure%E5%9C%A8function%E5%A4%96%E7%BB%A7%E7%BB%AD%E4%BD%BF%E7%94%A8%60%40escaping%60_3.png)
```
class Baby {
    var name = "peter"
    func outsideActivity(activity: (() -> ())?) {
        if let activity = activity {
            activity()
        }
    }
}

class Mother {
    var name = "wendy"
    var child = Baby()
    func play() {
        child.outsideActivity {
            print("\(self.name)和小孩\(self.child.name)打桌球")
        }
    }
}
```
---
[完]
