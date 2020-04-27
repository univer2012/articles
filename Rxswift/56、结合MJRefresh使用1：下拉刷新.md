`MJRefresh` 是一个使用 `Objective-C` 编写的刷新库，使用简单，功能强大。它既可以实现下拉刷新，也能实现上拉加载。本文通过样例演示如何让 `RxSwift` 与 `MJRefresh` 结合使用。

## 一、准备工作

### 1，配置 MJRefresh

关于 `MJRefresh` 的安装配置和相关介绍，可以参考这篇文章：

- [Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://link.jianshu.com?t=http%3A%2F%2Fwww.hangge.com%2Fblog%2Fcache%2Fdetail_1406.html)

### 2，对 MJRefresh 进行扩展

为了让 `MJRefresh` 可以更好地与 `RxSwift` 配合使用，我这里对它进行扩展（`MJRefresh+Rx.swift`），内容如下：

- 将下拉、上拉的刷新事件转为 `ControlEvent` 类型的可观察序列。
- 增加一个用于停止刷新的绑定属性。

```swift
import RxSwift
import RxCocoa

//对MJRefreshComponent 增加rx扩展
extension Reactive where Base: MJRefreshComponent {
    
    //正在刷新事件
    var refreshing: ControlEvent<Void> {
        let source: Observable<Void> = Observable.create { [weak control = self.base] (observer) -> Disposable in
            
            if let control = control {
                control.refreshingBlock = {
                    observer.on(.next( () ))
                }
            }
            return Disposables.create()
        }
        return ControlEvent(events: source)
    }
    
    //停止刷新
    var endRefreshing: Binder<Bool> {
        return Binder(base) {refresh, isEnd in
            if isEnd {
                refresh.endRefreshing()
            }
        }
    }
}

```



### 3，网络请求服务

这里专门封装了一个网络请求服务层（`NetworkService.swift`），作用是当表格发生上拉、或下拉时，通过它获取数据。为了方便演示，这里没有真正去发起网络请求，而是随机生成 **15** 条数据，并延迟 **1** 秒返回。

```swift
import RxSwift
import RxCocoa

class Network56Service {
    
    //获取随机数据
    func getRandomResult() -> Driver<[String]> {
        print("正在请求数据......")
        let items = (0 ..< 15).map { _ in
            "随机数据\(Int(arc4random()))"
        }
        let observable = Observable.just(items)
        return observable.delay(1, scheduler: MainScheduler.instance).asDriver(onErrorDriveWith: Driver.empty())
    }
}
```

## 二、下拉刷新的样例

### 1，效果图

（1）页面打开后自动会加载 **15** 条数据，并显示在表格中。

（2）而每次下拉表格又会随机生成 **15** 条新的数据，并替换表格里的原数据。

![img](https:////upload-images.jianshu.io/upload_images/3788243-1328c24e5f847b7c.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-33df3683ac936145.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-5cc48aea14fa9156.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)

### 2，样例代码

（1）`ViewModel.swift`

主要是根据下拉刷新事件序列去查询数据，同时数据返回后除了生成表格数据序列外，还有个停止刷新状态的序列。

```swift
import RxSwift
import RxCocoa

class Rxswift56ViewModel {
    
    //表格数据序列
    let tableData: Driver<[String]>
    
    //停止刷新状态序列
    let endHeaderRefreshing: Driver<Bool>
    
    //Rxswift56ViewModel初始化（根据输入实现对应的输出）
    init(headerRefresh: Driver<Void>) {
        
        //网络请求服务
        let networkService = Network56Service()
        
        //生成查询结果l序列
        self.tableData = headerRefresh
        .startWith(())
        .flatMapLatest{_ in networkService.getRandomResult() }
        
        //生成停止刷新状态序列
        self.endHeaderRefreshing = self.tableData.map{ _ in true}
    }
}
```

（2）`ViewController.swift`
 `ViewModel` 初始化后，将表格数据序列绑定到 `tableView` 上显示数据，同时将停止刷新序列绑定到 `tableView` 的 `mj_header` 上让其自动停止刷新。

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_56ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    //表格
    var tableView: UITableView!
        
    override func viewDidLoad() {
        super.viewDidLoad()

        //创建表格视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView)
        
        //设置头部刷新控件
        self.tableView.mj_header = MJRefreshNormalHeader()
        
        //初始化Rxswift56ViewModel
        let viewModel = Rxswift56ViewModel(headerRefresh: self.tableView.mj_header.rx.refreshing.asDriver())
        
        //单元格数据的绑定
        viewModel.tableData.asDriver()
            .drive(tableView.rx.items) {tableView, row, element in
                
                let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
                cell.textLabel?.text = "\(row+1)、\(element)"
                return cell
        }.disposed(by: disposeBag)
        
        //下拉刷新状态结束的绑定
        viewModel.endHeaderRefreshing
            .drive(self.tableView.mj_header.rx.endRefreshing)
            .disposed(by: disposeBag)
    }
    
}
```

---

【完】