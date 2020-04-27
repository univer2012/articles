来自：[Swift - RxSwift的使用详解62 (订阅UITableViewCell里的按钮点击事件)](https://www.jianshu.com/p/baffcccfdf9b)



们知道通过订阅 `tableView` 的 `itemSelected` 或 `modelSelected` 这两个 `Rx` 扩展方法，可以对单元格的点击事件进行响应，并执行相关的业务代码。

但有时我们并不需要整个 `cell` 都能进行点击响应，可能是点击单元格内的按钮时才触发相关的操作，下面通过样例演示这个功能的实现。

### 1，效果图

（1）点击单元格右侧的按钮后，会弹出显示该单元格的内容以及索引值。

（2）而点击单元格其他位置，不触发任何操作。

![img](https:////upload-images.jianshu.io/upload_images/3788243-9d8062af0c5c71ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)



![img](https:////upload-images.jianshu.io/upload_images/3788243-b24d3ac88fbcf6c8.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)

### 2，样例代码

（1）`MyTableCell.swift`（自定义单元格类）

注意 `prepareForReuse()` 方法里的 `disposeBag = DisposeBag()`
 每次 `prepareForReuse()` 方法执行时都会初始化一个新的 `disposeBag`。这是因为 `cell` 是可以复用的，这样当 `cell` 每次重用的时候，便会自动释放之前的 `disposeBag`，从而保证 `cell` 被重用的时候不会被多次订阅，避免错误发生。



```swift
import UIKit
import RxSwift

//单元格类
class SHRxswift_62MyTableViewCell: UITableViewCell {
    
    var button: UIButton!
    
    var disposeBag = DisposeBag()
    
    //单元格重用时调用
    override func prepareForReuse() {
        super.prepareForReuse()
        disposeBag = DisposeBag()
    }

    //初始化
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        //添加按钮
        button = UIButton(frame: CGRect(x: 0, y: 0, width: 40, height: 25))
        button.setTitle("点击", for: .normal)
        button.backgroundColor = .orange
        button.layer.cornerRadius = 5
        button.titleLabel?.font = UIFont.systemFont(ofSize: 13)
        addSubview(button)
    }
    
    //布局
    override func layoutSubviews() {
        super.layoutSubviews()
        button.center = CGPoint(x: bounds.size.width - 35, y: bounds.midY)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
}
```



（2）`ViewController.swift`（主视图控制器）

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_62ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //创建表格视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        
        //创建一个重用的单元格
        self.tableView.register(SHRxswift_62MyTableViewCell.self, forCellReuseIdentifier: "Cell")
        
        //单元格无法选中
        self.tableView.allowsSelection = false
        self.view.addSubview(self.tableView)
        
        //初始化数据
        let items = Observable.just([
            "文本输入框的用法",
            "开过按钮的用法",
            "进度条的用法",
            "文本标签的用法",
        ])
        
        //设置单元格数据（其实就是对 cellForRowAt 的封装）
        items.bind(to: tableView.rx.items) { (tableView, row, element) in
            //初始化cell
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell") as! SHRxswift_62MyTableViewCell
            cell.textLabel?.text = "\(element)"
            
            //cell中按钮点击事件订阅
            cell.button.rx.tap.asDriver()
                .drive(onNext: { [weak self] in
                    self?.showAlert(title: "\(row)", message: element)
                })
                .disposed(by: cell.disposeBag)
            
            return cell
        }
        .disposed(by: disposeBag)
    }
    
    
    //显示弹出框消息
    func showAlert(title:String, message:String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        
        let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        
        alert.addAction(cancelAction)
        
        self.present(alert, animated: true)
    }

}
```



[RxSwift使用详解系列](https://www.jianshu.com/p/f61a5a988590)
 [原文出自:www.hangge.com转载请保留原文链接](http://www.hangge.com/blog/cache/detail_2047.html)



---

【完】