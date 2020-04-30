面试题：https://blog.csdn.net/david21984/article/details/57451917

## iOS 面试题（一）寻找最近公共 View
> 题目：找出两个 UIView 的最近的公共 View，如果不存在，则输出 nil 。
>
> 分析：这其实是数据结构里面的找最近公共祖先的问题。

 一个UIViewController中的所有view之间的关系其实可以看成一颗树，UIViewController的view变量是这颗树的根节点，其它的view都是根节点的直接或间接子节点。

所以我们可以通过 view 的 superview 属性，一直找到根节点。需要注意的是，在代码中，我们还需要考虑各种非法输入，如果输入了 nil，则也需要处理，避免异常。以下是找到指定 view 到根 view 的路径代码：
```objc
+ (NSArray *)superViews:(UIView *)view {
    if (view == nil) {
        return @[];
    }
    NSMutableArray *result = [NSMutableArray array];
    while (view != nil) {
        [result addObject:view];
        view = view.superview;
    }
    return [result copy];
}
```

 然后对于两个 view A 和 view B，我们可以得到两个路径，而本题中我们要找的是这里面最近的一个公共节点。

一个简单直接的办法：拿第一个路径中的所有节点，去第二个节点中查找。假设路径的平均长度是 N，因为每个节点都要找 N 次，一共有 N 个节点，所以这个办法的时间复杂度是 O（N^2）。
```objc
+ (UIView *)commonView_1:(UIView *)viewA andView:(UIView *)viewB {
    NSArray *arr1 = [self superViews:viewA];
    NSArray *arr2 = [self superViews:viewB];
    for (NSUInteger i = 0; i < arr1.count; ++i) {
        UIView *targetView = arr1[i];
        for (NSUInteger j = 0; j < arr2.count; ++j) {
            if (targetView == arr2[j]) {
                return targetView;
            }
        }
    }
    return nil;
}
```
一个改进的办法：我们将一个路径中的所有点先放进 NSSet 中。因为 NSSet 的内部实现是一个 hash 表，所以查找元素的时间复杂度变成了 O（1），我们一共有 N 个节点，所以总时间复杂度优化到了 O（N）。
```objc
+ (UIView *)commonView_2:(UIView *)viewA andView:(UIView *)viewB {
    NSArray *arr1 = [self superViews:viewA];
    NSArray *arr2 = [self superViews:viewB];
    NSSet *set = [NSSet setWithArray:arr2];
    for (NSUInteger i = 0; i < arr1.count; ++i) {
        UIView *targetView = arr1[i];
        if ([set containsObject:targetView]) {
            return targetView;
        }
    }
    return nil;
}
```

除了使用 NSSet 外，我们还可以使用类似归并排序的思想，用两个「指针」，分别指向两个路径的根节点，然后从根节点开始，找第一个不同的节点，第一个不同节点的上一个公共节点，就是我们的答案。代码如下：
```objc
/* O(N) Solution */
+ (UIView *)commonView_3:(UIView *)viewA andView:(UIView *)viewB {
    NSArray *arr1 = [self superViews:viewA];
    NSArray *arr2 = [self superViews:viewB];
    NSInteger p1 = arr1.count - 1;
    NSInteger p2 = arr2.count - 1;
    UIView *answer = nil;
    while (p1 >= 0 && p2 >= 0) {
        if (arr1[p1] == arr2[p2]) {
            answer = arr1[p1];
        }
        p1--;
        p2--;
    }
    return answer;
}
```
我们还可以使用 UIView 的 `isDescendant` 方法来简化我们的代码，不过这样写的话，时间复杂度应该也是 O(N^2) 的。lexrus 提供了如下的 Swift 版本的代码：
```swift
/// without flatMap
extension UIView {    
func commonSuperview(of view: UIView) -> UIView? {        
    if let s = superview {            
       if view.isDescendant(of: s) {                
                 return s
            } else {                
                 return s.commonSuperview(of: view)
            }
        }        
       return nil
    }
}
```
特别地，如果我们利用 Optinal 的 flatMap 方法，可以将上面的代码简化得更短，基本上算是一行代码搞定。怎么样，你学会了吗？
```swift
extension UIView {
    func commonSuperview(of view: UIView) -> UIView? {
        return superview.flatMap { 
           view.isDescendant(of: $0) ? 
             $0 : $0.commonSuperview(of: view) 
        }
    }
}
```

## iOS 面试题（二）什么时候在 block 中不需要使用 weakSelf
> 问题:我们知道，在使用 block 的时候，为了避免产生循环引用，通常需要使用 weakSelf 与 strongSelf，写下面这样的代码：
```objc
__weak typeof(self) weakSelf = self;
[self doSomeBlockJob:^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (strongSelf) {
        ...
    }
}];
```
> 那么请问：什么时候在 block里面用self，不需要使用weakself?

**当block本身不被self 持有，而被别的对象持有，同时不产生循环引用的时候，就不需要使用weakself了**。最常见的代码就是UIView的动画代码，我们在使用`UIView animateWithDuration:animations:`方法 做动画的时候，并不需要使用weakself，因为引用持有关系是：

> **UIView 的某个负责动画的对象持有block，block 持有了self。因为 self 并不持有 block，所以就没有循环引用产生，所以不需要使用 weak self 了。**
```
[UIView animateWithDuration:0.2 animations:^{
    self.alpha = 1;
}];
```
> 当动画结束时，UIView会结束持有这个 block，如果没有别的对象持有block的话，block 对象就会释放掉，从而 block会释放掉对于 self 的持有。整个内存引用关系被解除。

## iOS 面试题（三）什么时候在 block 中不需要使用 weakSelf

我们知道，在使用 block 的时候，为了避免产生循环引用，通常需要使用 weakSelf 与 strongSelf，写下面这样的代码：
```
__weak typeof(self) weakSelf = self;
[self doSomeBackgroundJob:^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (strongSelf) {
        ...
    }
}];
```
 那么请问：为什么 block 里面还需要写一个 strong self，如果不写会怎么样？

在 block 中先写一个 strong self，**其实是为了避免在 block 的执行过程中，突然出现 self 被释放的尴尬情况。通常情况下，如果不这么做的话，还是很容易出现一些奇怪的逻辑，甚至闪退。**

我们以AFNetworking中的AFNetworkReachabilityManager.m的一段代码举例：
```
__weak __typeof(self)weakSelf = self;
AFNetworkReachabilityStatusBlock callback = ^(AFNetworkReachabilityStatus status) {
    __strong __typeof(weakSelf)strongSelf = weakSelf;
    strongSelf.networkReachabilityStatus = status;
    if (strongSelf.networkReachabilityStatusBlock) {strongSelf.networkReachabilityStatusBlock(status);
    }
};
```
 如果没有strongSelf的那行代码，那么后面的每一行代码执行时，self都可能被释放掉了，这样很可能造成逻辑异常。

特别是当我们正在执行 `strongSelf.networkReachabilityStatusBlock(status);` 这个 block闭包时，如果这个 block 执行到一半时 self 释放，那么多半情况下会 Crash。




    这里有一篇文章详细解释了这个问题：[点击查看文章](https://dhoerl.wordpress.com/2013/04/23/i-finally-figured-out-weakself-and-strongself/)
    昨天的读者中，拓荒者 和 陈祥龙 同学在评论中也正确回答出了本题。
    
    拓荒者:
    1.在block里使用strongSelf是防止在block执行过程中self被释放。 2.可以通过在执行完block代码后手动把block置为nil来打破引用循环，AFNetworking就是这样处理的，避免使用者不了解引用循环造成内存泄露。实际业务中暂时没遇到这种需求，请巧哥指点什么情况下会有这种需求。
    
    陈祥龙:
    strongSelf 一般是在为了避免 block 回调时 weak Self变成了nil ，异步执行一些操作时可能会出现这种情况，不知道我说得对不对。因业务需要不能使用weakSelf 这种情况还真没遇到过
    
    另外，还有读者提了两个有意思的问题，大家可以思考一下：
    Yuen 提问：“数组” 和 “字典” 的 enumeratXXXUsingBlock: 是否要使用 weakSelf 和 strongSelf 呢？
    潇湘雨同学提问：block 里 strong self 后，block 不是也会持有 self 吗？而 self 又持有 block ，那不是又循环引用了？


## iOS 面试题（四）：block 什么时候需要构造循环引用

 问题：有没有这样一个需求场景，block 会产生循环引用，但是业务又需要你不能使用 weak self? 如果有，请举一个例子并且解释这种情况下如何解决循环引用问题。

答案：需要不使用 weak self 的场景是：你需要构造一个循环引用，以便保证引用双方都存在。比如你有一个后台的任务，希望任务执行完后，通知另外一个实例。在我们开源的 `YTKNetwork` 网络库的源码中，就有这样的场景。

在 YTKNetwork 库中，我们的每一个网络请求 API 会持有回调的 block，回调的 block 会持有 self，而如果 self 也持有网络请求 API 的话，我们就构造了一个循环引用。虽然我们构造出了循环引用，但是因为在网络请求结束时，网络请求 API 会主动释放对 block 的持有，因此，整个循环链条被解开，循环引用就被打破了，所以不会有内存泄漏问题。代码其实很简单，如下所示：
```
//  YTKBaseRequest.m
- (void)clearCompletionBlock {
    // nil out to break the retain cycle.
    self.successCompletionBlock = nil;
    self.failureCompletionBlock = nil;
}
```
 总结来说，解决循环引用问题主要有两个办法：

第一个办法是「事前避免」，我们在会产生循环引用的地方使用 weak 弱引用，以避免产生循环引用。

第二个办法是「事后补救」，我们明确知道会存在循环引用，但是我们在合理的位置主动断开环中的一个引用，使得对象得以回收。

## iOS 面试题（五）：weak 的内部实现原理

 问题：weak 变量在引用计数为0时，会被自动设置成 nil，这个特性是如何实现的？
答案：在 [Friday QA](https://mikeash.com/pyblog/friday-qa-2010-07-16-zeroing-weak-references-in-objective-c.html) 上，有一期专门介绍 weak的实现原理。

《Objective-C高级编程》一书中也介绍了相关的内容。
简单来说，**系统有一个全局的 CFMutableDictionary 实例，来保存每个对象的 weak 指针列表，因为每个对象可能有多个 weak 指针，所以这个实例的值是 CFMutableSet 类型。**

剩下我们要做的，就是**在引用计数变成 0 的时候，去这个全局的字典里面，找到所有的 weak 指针，将其值设置成 nil**。如何做到这一点呢？[Friday QA](https://mikeash.com/pyblog/friday-qa-2010-07-16-zeroing-weak-references-in-objective-c.html) 上介绍了一种类似 <u>KVO 实现的方式</u>。**当对象存在 weak 指针时，我们可以将这个实例指向一个新创建的子类，然后修改这个子类的 release 方法，在 release 方法中，去从全局的 CFMutableDictionary 字典中找到所有的 weak 对象，并且设置成 nil。**我摘抄了 Friday QA 上的实现的核心代码，如下：
```objc
    Class subclass = objc_allocateClassPair(class, newNameC, 0);
    Method release = class_getInstanceMethod(class, @selector(release));
    Method dealloc = class_getInstanceMethod(class, @selector(dealloc));
    class_addMethod(subclass, @selector(release), (IMP)CustomSubclassRelease, method_getTypeEncoding(release));
    class_addMethod(subclass, @selector(dealloc), (IMP)CustomSubclassDealloc, method_getTypeEncoding(dealloc));
    objc_registerClassPair(subclass);
```

当然，这并不代表苹果官方是这么实现的，因为苹果的这部分代码并没有开源。《Objective-C高级编程》一书中介绍了 GNUStep 项目中的开源代码，思想也是类似的。所以我认为虽然实现细节会有差异，但是大致的实现思路应该差别不大。

## iOS 面试题（六）：自己写的 view 成员，应该用 weak 还是 strong？
问题：我们知道，从 Storyboard 往编译器拖出来的 UI 控件的属性是 weak 的，如下所示
```
@property (weak, nonatomic) IBOutlet UIButton *myButton;
```

 那么，如果有一些 UI 控件我们要用代码的方式来创建，那么它应该用 weak 还是 strong 呢？为什么？

答案：这是一道有意思的问题，这个问题是我当时和 Lancy 一起写猿题库 App 时产生的一次小争论。简单来说，这道题并没有标准答案，但是答案背后的解释却非常有价值，能够看出一个人对于引用计数，对于 view 的生命周期的理解是否到位。

从昨天的评论上，我们就能看到一些理解非常不到位的解释，例如：
```
@spume 说：Storyboard 拖线使用 weak 是为了规避出现循环引用的问题。
```

这个理解是错误的，Storyboard 拖出来的控件即使是 strong 的，也不会有循环引用问题。
Storyboard 拖出来的**UI 控件默认用 weak，根源还是苹果希望只有这些 UI 控件的父 View 来强引用它们。而 ViewController 只需要强引用 ViewController.view 成员，就可以间接持有所有的 UI 控件**。<font color=#FF0000>这样有一个好处是：在以前，当系统收到 Memory Warning 时，会触发 ViewController 的`viewDidUnload` 方法。这样的弱引用方式，可以让整个 view 整体都得到释放，也更方便重建时整体重新构造。</font>

但是首先 viewDidUnload 方法在 iOS 6 开始就被废弃掉了，苹果用了更简单有效地方式来解决内存警告时的视图资源释放，具体如何做的呢？嗯，这个可以当作某一期的面试题展开介绍。总之就是，**除非你特殊地操作 view 成员，否则ViewController.view 的生命期和 ViewController 是一样的。**

**在这种情况下， UI 控件是不是 weak 其实关系并不大。当 UI 控件是 weak 时，它的引用计数是 1，持有它的是它的 superview。当 UI 控件是 strong 时，它的引用计数是 2，持有它的有两个地方，一个是它的 superview，另一个是这个 strong 指针。所以，不管是手写代码还是 Storyboard，UI 控件是 strong 都不会有循环引用。**

那么回到我们的最初的问题，<font color=#FF0000>自己写的 view 成员，应该用 weak 还是 strong？</font>**我个人觉得应该用 strong，因为用 weak 并没有什么特别的优势**，加上上一篇面试题文章中，我们还看到，**其实 weak 变量会有额外的系统维护开销的。如果你没有使用它的特别理由，那么用 strong 的话应该更好。**

另外有读者也提到，**如果你要做 Lazy 加载，那么你也只能选择用 strong。**

<font color=#038103>当然，如果你非要用 weak，其实也没什么问题。只需要注意在赋值前，先把这个对象用 addSubView 加到父 view 上，否则可能刚刚创建完，它就被释放了。</font>

在我心目中，这才是我喜欢的面试题，没有标准答案，每种方案各有各的特点，面试者能够足够分清楚每种方案的优缺点，结合具体的场景做选择，这才是优秀的面试者。

> 1.懒加载的对象必须用strong的原因在于，如果使用weak，对象并没有被强引用，过了懒加载对象就会被释放掉。

## iOS 面试题（七）：为什么 Objective-C 的方法调用要用方括号?

 问题：为什么 Objective-C 的方法调用要用方括号 [obj foo]，而不是别的语言常常使用的点 obj.foo ?

答案：
首先要说的是，Objective-C 的历史相当久远，如果你查 wiki 的话，你会发现：Objective-C 和 C++ 这两种语言的发行年份都是 1983 年。在设计之初，二者都是作为 C 语言的面向对象的接班人，希望成为事实上的标准。最后结果大家都知道了，C++ 最终胜利了，而 Objective-C 在之后的几十年中，基本上变成了苹果自己家玩的玩具。不过最终，由于 iPhone 的出现，Objective-C 迎来了第二春，在 TOBIE 语言排行榜上，从 20 名开外一路上升，排名曾经超越过 C++，达到了第三名（下图），但是随着 Swift 的出现，Objective-C 的排名则一路下滑。
![](http://upload-images.jianshu.io/upload_images/1448717-f188318c63ad7002.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 >   Objective-C 在设计之初参考了不少 Smalltalk 的设计，而消息发送则是向 Smalltalk 学来的。Objective-C 当时采用了方括号的形式来表示发送消息，为什么没有选择用点呢？我个人觉得是，当时市面上并没有别的面向对象语言的设计参考，而 Objective-C 「发明」了方括号的形式来给对象发消息，而 C++ 则「发明」了用点的方式来 “发消息”。有人可能会争论说 C++ 的「点」并不是真正的发消息，但是其实二者都是表示「调用对象所属的成员函数」。

>    另外，有读者评论说使用方括号的形式是为了向下兼容 C 语言，我并不觉得中括号是唯一选择，C++ 不也兼容了 C 语言么？Swift 不也可以调用 C 函数么？
>      最终，其实是 C++ 的「发明」显得更舒服一些，所以后来的各种语言都借鉴了 C++ 的这种设计，也包括 Objective-C 在内。Objective-C 2.0 版本中，引入了 dot syntax，即：
>      a = obj.foo 等价于 a = [obj foo]
>      obj.foo = 1 则等价于 [obj setFoo:1]

>    Objective-C 其实在设计之中确实是比较特立独行的，除了方括号的函数调用方式外，还包括比较长的，可读性很强的函数命名风格。

 >   我个人并不讨厌 Objective-C 的这种设计，但是从 Swift 语言的设计来看，苹果也开始放弃一些 Objective-C 的特点了，比如就去掉了方括号这种函数调用方式。