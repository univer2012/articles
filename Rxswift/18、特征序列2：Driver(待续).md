来自：[Swift - RxSwift的使用详解18（特征序列2：Driver）](https://www.hangge.com/blog/cache/detail_1942.html)

---

在上文中，我介绍了 **RxSwift** 提供的一些特征序列（**Traits**）：**Single**、**Completable**、**Maybe**。接下来的文章我会接着介绍另外两个特征序列：**Driver**、**ControlEvent**。更准确说，这两个应该算是 **RxCocoa traits**，因为它们是专门服务于 **RxCocoa** 工程的。 

## 四、Driver

### 1，基本介绍

（1）**Driver** 可以说是最复杂的 **trait**，它的目标是提供一种简便的方式在 **UI** 层编写响应式代码。

（2）如果我们的序列满足如下特征，就可以使用它：

- 不会产生 **error** 事件
- 一定在主线程监听（**MainScheduler**）
- 共享状态变化（**shareReplayLatestWhileConnected**）



### 2，为什么要使用 Driver?

（1）**Driver** 最常使用的场景应该就是需要用序列来驱动应用程序的情况了，比如：

- 通过 **CoreData** 模型驱动 **UI**
- 使用一个 **UI** 元素值（绑定）来驱动另一个 **UI** 元素值

（2）与普通的操作系统驱动程序一样，如果出现序列错误，应用程序将停止响应用户输入。

（3）在主线程上观察到这些元素也是极其重要的，因为 **UI** 元素和应用程序逻辑通常不是线程安全的。

（4）此外，使用构建 **Driver** 的可观察的序列，它是共享状态变化。



### 3，使用样例

> 这个是官方提供的样例，大致的意思是根据一个输入框的关键字，来请求数据，然后将获取到的结果绑定到另一个 **Label** 和 **TableView** 中。


 （1）初学者使用 **Observable** 序列加 **bindTo** 绑定来实现这个功能的话可能会这么写：

```

```

