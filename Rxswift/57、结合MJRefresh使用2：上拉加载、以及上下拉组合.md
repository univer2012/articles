## 三、上拉加载的样例

### 1，效果图

（1）页面打开后会自动加载 **15** 条数据，并显示在表格中。

（2）而每次上拉表格又会随机生成 **15** 条新的数据，并拼接到原数据下方显示。

![img](https:////upload-images.jianshu.io/upload_images/3788243-5b92de987610132b.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-e3dae666261255d7.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-7d2c77f6d1cb2380.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)

### 2，样例代码

（1）`ViewModel.swift`

由于“**加载更多**”功能需要把新数据添加到老数据尾部，这里我使用 `BehaviorRelay` 作为表格数据序列，因为它可以获取到之前的数据。

```swift
import RxSwift
import RxCocoa

class Rxswift57ViewModel {
    
    //表格数据序列
    let tableData = BehaviorRelay<[String]>(value: [])
    
    //停止刷新状态序列
    let endFooterRefreshing: Driver<Bool>
    
    //Rxswift57ViewModel初始化（根据输入实现对应的输出）
    init(footerRefresh: Driver<Void>,
         dependency: (
            disposeBag:DisposeBag,
        networkService: Network56Service)) {
        
        //上拉结果序列
        let footerRefreshData = footerRefresh
            .startWith(())
            .flatMapLatest { return dependency.networkService.getRandomResult() }
        
        //生成停止上拉加载刷新状态序列
        self.endFooterRefreshing = footerRefreshData.map{ _ in true }
        
        //上拉加载时，将查询到的结果拼接到元数据底部
        footerRefreshData.drive(onNext: { (items) in
            self.tableData.accept(self.tableData.value + items)
        }).disposed(by: dependency.disposeBag)
    }
}
```

（2）`ViewController.swift`

`ViewModel` 初始化后，将表格数据序列绑定到 `tableView`上显示数据。同时将停止刷新序列绑定到 `tableView` 的 `mj_footer` 上让其自动停止刷新。

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_57ViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    //表格
    var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //创建表格视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView!)
        
        //设置尾部刷新控件
        self.tableView.mj_footer = MJRefreshBackNormalFooter()
        
        //初始化 Rxswift57ViewModel
        let viewModel = Rxswift57ViewModel(
            footerRefresh: self.tableView.mj_footer.rx.refreshing.asDriver(),
            dependency: (disposeBag: self.disposeBag,
                         networkService: Network56Service()))
        
        //单元格数据的绑定
        viewModel.tableData.asDriver()
            .drive(tableView.rx.items) { (tableView, row, element) in
                let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
                cell.textLabel?.text = "\(row + 1)、\(element)"
                return cell
        }.disposed(by: disposeBag)
        
        //上拉刷新状态结束的绑定
        viewModel.endFooterRefreshing
        .drive(self.tableView.mj_footer.rx.endRefreshing)
        .disposed(by: disposeBag)
    }

}
```



## 四、下拉刷新 + 上拉加载的样例

### 1，效果图

（1）页面打开后会自动加载 **15** 条数据，并显示在表格中。

（2）当下拉表格时会随机生成 **15** 条新的数据，并替换表格里的原数据。

（3）而当上拉表格又会随机生成 **15** 条新的数据，并拼接到原数据下方显示。

![img](https:////upload-images.jianshu.io/upload_images/3788243-978c9294a60689d6.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-da9163f43e3ac2d3.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)



![img](https:////upload-images.jianshu.io/upload_images/3788243-2667e22d3e7a12e2.png?imageMogr2/auto-orient/strip|imageView2/2/w/243)

### 2，样例代码

（1）`ViewModel.swift`

这里同样使用 `BehaviorRelay` 作为表格数据序列，同时还定义了两个停止刷新序列（分别表示停止下拉刷新、停止上拉刷新）。

```swift
import RxSwift
import RxCocoa


//MARK:有上下拉刷新
class Rxswift57ViewModel {
    
    //表格数据序列
    let tableData = BehaviorRelay<[String]>(value: [])
    
    //停止头部刷新状态
    let endHeaderRefreshing: Driver<Bool>
    
    //停止尾部刷新状态
    let endFooterRefreshing: Driver<Bool>
    
    //Rxswift57ViewModel初始化（根据输入实现对应的输出）
    init(input: (
        headerRefresh: Driver<Void>,
        footerRefresh: Driver<Void>
        ),
         dependency: (
        disposeBag:DisposeBag,
        networkService: Network56Service)) {
        
        //下拉结果序列
        let headerRefreshData = input.headerRefresh
        .startWith(()) //初始化是会先自动加载一次数据
            .flatMapLatest{ return dependency.networkService.getRandomResult() }
        
        //上拉结果序列
        let footerRefreshData = input.footerRefresh
            .flatMapLatest{ return dependency.networkService.getRandomResult() }
        
        //生成停止头部刷新状态z序列
        self.endHeaderRefreshing = headerRefreshData.map{ _ in true }
        
        //生成停止尾部加载刷新状态序列
        self.endFooterRefreshing = footerRefreshData.map{ _ in true }
        
        //下拉刷新时，直接将查询到的结果替换原数据
        headerRefreshData.drive(onNext: { (items) in
            self.tableData.accept(items)
        }).disposed(by: dependency.disposeBag)
        
        //上拉加载时，将查询到的结果拼接到元数据底部
        footerRefreshData.drive(onNext: { (items) in
            self.tableData.accept(self.tableData.value + items)
        }).disposed(by: dependency.disposeBag)
    }
}
```

（2）`ViewController.swift`

`ViewModel` 初始化后，将表格数据序列绑定到 `tableView`上显示数据。同时将两个停止刷新序列分别绑定到 `tableView` 的 `mj_header` 和 `mj_footer` 上让其自动停止刷新。

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_57ViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    //表格
    var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //创建表格视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView!)
        
        //设置头部刷新控件
        self.tableView.mj_header = MJRefreshNormalHeader()
        //设置尾部刷新控件
        self.tableView.mj_footer = MJRefreshBackNormalFooter()
        
        //初始化 Rxswift57ViewModel
        let viewModel = Rxswift57ViewModel(
            input: (
                headerRefresh: self.tableView.mj_header.rx.refreshing.asDriver(),
                footerRefresh: self.tableView.mj_footer.rx.refreshing.asDriver()
            ),
            dependency: (disposeBag: self.disposeBag,
                         networkService: Network56Service()))
        
        //单元格数据的绑定
        viewModel.tableData.asDriver()
            .drive(tableView.rx.items) { (tableView, row, element) in
                let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
                cell.textLabel?.text = "\(row + 1)、\(element)"
                return cell
            }.disposed(by: disposeBag)
        
        //下拉刷新状态结束的绑定
        viewModel.endHeaderRefreshing
            .drive(self.tableView.mj_header.rx.endRefreshing)
            .disposed(by: disposeBag)
        
        //上拉刷新状态结束的绑定
        viewModel.endFooterRefreshing
            .drive(self.tableView.mj_footer.rx.endRefreshing)
            .disposed(by: disposeBag)
    }
    
}
```





## 附：“下拉刷新 + 上拉加载”的功能改进

### 1，功能说明

在前面的样例中下拉刷新和上拉加载这两个行为是独立的，互不影响。也就是说当我们下拉刷新后，在数据返回前又进行了次上拉操作，那么之后表格便会连续刷新两次，影响体验。这里对功能做个改进：

- 当下拉刷新时，如果数据还未返回。这时进行上拉加载会取消前面的下拉刷新操作（包括下拉刷新的数据），只进行上拉数据的加载。
- 同样的，当上拉加载时，如果数据还未放回。这时进行下拉刷新会取消上拉加载操作（包括上拉加载的数据），只进行下拉数据的加载。

同时这次我们不使用 `Driver` 这个特征序列，而是用普通的 `Observable` 序列。

### 2，样例代码

（1）`NetworkService.swift`

```swift
import RxSwift
import RxCocoa

class Network57Service {
    
    //获取随机数据
    func getRandomResult() -> Observable<[String]> {
        print("正在请求数据......")
        let items = (0 ..< 15).map { _ in
            "随机数据\(Int(arc4random()))"
        }
        let observable = Observable.just(items)
        return observable.delay(2, scheduler: MainScheduler.instance)
    }
}
```



（2）`ViewModel.swift`

```swift
import RxSwift
import RxCocoa


//MARK:上下拉刷新的改进 - 用到是Observable
class Rxswift57ViewModel {
    
    //表格数据序列
    let tableData = BehaviorRelay<[String]>(value: [])
    
    //停止头部刷新状态
    let endHeaderRefreshing: Observable<Bool>
    
    //停止尾部刷新状态
    let endFooterRefreshing: Observable<Bool>
    
    //Rxswift57ViewModel初始化（根据输入实现对应的输出）
    init(input: (
        headerRefresh: Observable<Void>,
        footerRefresh: Observable<Void>
        ),
         dependency: (
        disposeBag:DisposeBag,
        networkService: Network57Service)) {
        
        //下拉结果序列
        let headerRefreshData = input.headerRefresh
            .startWith(()) //初始化是会先自动加载一次数据
            .flatMapLatest{ _ in
                dependency.networkService.getRandomResult()
                .takeUntil(input.footerRefresh)
            }.share(replay: 1)//让HTTP请求是被共享的
        
        
        //上拉结果序列
        let footerRefreshData = input.footerRefresh
            .flatMapLatest{ _ in
                dependency.networkService.getRandomResult()
                .takeUntil(input.headerRefresh)
        }.share(replay: 1)//让HTTP请求是被共享的
        
        //生成停止头部刷新状态z序列
        self.endHeaderRefreshing = Observable.merge(
            headerRefreshData.map{ _ in true },
            input.footerRefresh.map{ _ in true }
        )
        
        //生成停止尾部加载刷新状态序列
        self.endFooterRefreshing = Observable.merge(
            footerRefreshData.map{ _ in true },
            input.headerRefresh.map{ _ in true }
        )
        
        //下拉刷新时，直接将查询到的结果替换原数据
        headerRefreshData.subscribe(onNext: { (items) in
            self.tableData.accept(items)
        }).disposed(by: dependency.disposeBag)
        
        //上拉加载时，将查询到的结果拼接到元数据底部
        footerRefreshData.subscribe(onNext: { (items) in
            self.tableData.accept(self.tableData.value + items)
        }).disposed(by: dependency.disposeBag)
    }
}
```



（3）`ViewController.swift`

```swift
import UIKit
import RxSwift
import RxCocoa

//MARK:上下拉刷新的改进 - 用到是Observable
class SHRxswift_57ViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    //表格
    var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //创建表格视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView!)
        
        //设置头部刷新控件
        self.tableView.mj_header = MJRefreshNormalHeader()
        //设置尾部刷新控件
        self.tableView.mj_footer = MJRefreshBackNormalFooter()
        
        //初始化 Rxswift57ViewModel
        let viewModel = Rxswift57ViewModel(
            input: (
                headerRefresh: self.tableView.mj_header.rx.refreshing.asObservable(),
                footerRefresh: self.tableView.mj_footer.rx.refreshing.asObservable()
            ),
            dependency: (disposeBag: self.disposeBag,
                         networkService: Network57Service()))
        
        //单元格数据的绑定
        viewModel.tableData.asDriver()
            .drive(tableView.rx.items) { (tableView, row, element) in
                let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
                cell.textLabel?.text = "\(row + 1)、\(element)"
                return cell
            }.disposed(by: disposeBag)
        
        //下拉刷新状态结束的绑定
        viewModel.endHeaderRefreshing
            .bind(to: self.tableView.mj_header.rx.endRefreshing)
            .disposed(by: disposeBag)
        
        //上拉刷新状态结束的绑定
        viewModel.endFooterRefreshing
            .bind(to: self.tableView.mj_footer.rx.endRefreshing)
            .disposed(by: disposeBag)
    }
    
}
```

---

【完】