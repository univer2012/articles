## 三、将结果转为 JSON 对象

### 1，实现方法

（1）如果服务器返回的数据是 `json` 格式的话，我们可以使用`iOS` 内置的 `JSONSerialization` 将其转成 `JSON` 对象，方便我们使用。

```swift
curl -X GET 
"https://www.douban.com/j/app/radio/channels" -i -v
Success (324ms): Status 200
--- 请求成功！返回的如下数据 ---
["channels": <__NSArrayI 0x7fee9cd14eb0>(
{
    "abbr_en" = My;
    "channel_id" = 0;
    name = "\U79c1\U4eba\U5146\U8d6b";
    "name_en" = "Personal Radio";
    "seq_id" = 0;
},
{
    "abbr_en" = "";
    "channel_id" = 1;
    name = "\U534e\U8bed";
    "name_en" = "";
    "seq_id" = 0;
},
//... ...
```



```swift
//创建URL对象
let urlString = "https://www.douban.com/j/app/radio/channels"
let url = URL(string: urlString)
//创建请求对象
let request = URLRequest(url: url!)

//创建并发起请求
URLSession.shared.rx.data(request: request).subscribe(onNext: { (data) in
    let json = try? JSONSerialization.jsonObject(with: data, options: .allowFragments) as! [String: Any]
    print("--- 请求成功！返回的如下数据 ---")
    print(json!)
}).disposed(by: disposeBag)
```



（2）当然我们在订阅前就进行转换也是可以的：

```swift
//创建URL对象
let urlString = "https://www.douban.com/j/app/radio/channels"
let url = URL(string: urlString)
//创建请求对象
let request = URLRequest(url: url!)

//创建并发起请求
URLSession.shared.rx.data(request: request).map{
    try JSONSerialization.jsonObject(with: $0, options: .allowFragments) as! [String: Any]
}
.subscribe(onNext: { (data) in
    print("--- 请求成功！ 返回的如下数据 ---")
    print(data)
}).disposed(
```



（3）**还有更简单的方法，就是直接使用 `RxSwift` 提供的 `rx.json` 方法去获取数据，它会直接将结果转成 `JSON` 对象。**

```swift
//创建URL对象
let urlString = "https://www.douban.com/j/app/radio/channels"
let url = URL(string: urlString)
//创建请求对象
let request = URLRequest(url: url!)

//创建并发起请求
URLSession.shared.rx.json(request: request).subscribe(onNext: { (data) in
    let json = data as! [String: Any]
    print("--- 请求成功！ 返回的如下数据 ---")
    print(json)
}).disposed(by: disposeBag)
```

### 2，使用样例

（1）效果图

我们将获取到的豆瓣频道列表数据转换成 `JSON` 对象，并绑定到表格上显示。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Rxswift/Rx_44_1.png)

（2）样例代码

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_44ViewController: UIViewController {
    
    var tableView: UITableView!
    
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //创建表格视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView)


        //创建URL对象
        let urlString = "https://www.douban.com/j/app/radio/channels"
        let url = URL(string: urlString)
        //创建请求对象
        let request = URLRequest(url: url!)
        
        //获取列表数据
        let data = URLSession.shared.rx.json(request: request).map{ result -> [[String: Any]] in
            if let data = result as? [String: Any], let channels = data["channels"] as? [[String: Any]] {
                return channels
            } else {
                return []
            }
        }
        
        //将数据绑定到表格
        data.bind(to: tableView.rx.items){ (tableView, row, element) in
            
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
            cell.textLabel?.text = "\(row): \(element["name"]!)"
            return cell
            
        }.disposed(by: disposeBag)
        
    }

}
```

## 四，将结果映射成自定义对象

### 1，准备工作

（1）要实现数据到模型（`model`）的转换，我们首先需要引入一个第三方的数据模型转换框架：`ObjectMapper`。关于它的安装配置，以及相关说明可以参考航ge之前写的文章：

- [Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://link.jianshu.com?t=https%3A%2F%2Fwww.hangge.com%2Fblog%2Fcache%2Fdetail_1673.html)

（2）为了让 `ObjectMapper` 能够更好地与 `RxSwift` 配合使用，我们对 `Observable` 进行扩展（`RxObjectMapper.swift`），增加数据转模型对象、以及数据转模型对象数组这两个方法。

`DoubanObject.swift`:

```swift
import ObjectMapper
import RxSwift
//数据映射错误
public enum RxObjectMapperError: Error {
    case parsingError
}
//扩展Observable：增加模型映射方法
public extension Observable where Element: Any {
    //将JSON数据转成对象
    public func mapObject<T>(type: T.Type) -> Observable<T> where T: Mappable {
        let mapper = Mapper<T>()
        
        return self.map({ (element) -> T in
            guard let parsedElement = mapper.map(JSONObject: element) else {
                throw RxObjectMapperError.parsingError
            }
            return parsedElement
        })
    }
    //将JSON数据转成数组
    public func mapArray<T>(type: T.Type) -> Observable<[T]> where T: Mappable {
        let mapper = Mapper<T>()
        
        return self.map({ (element) -> [T] in
            guard let parsedArray = mapper.mapArray(JSONObject: element) else {
                throw RxObjectMapperError.parsingError
            }
            return parsedArray
        })
    }
}
```

### 2，使用样例

（1）我们还是以前面的豆瓣音乐频道数据为例。首先我定义好相关模型（需要实现 `ObjectMapper` 的 `Mappable` 协议，并设置好成员对象与 `JSON` 属性的相互映射关系。）

```swift
//豆瓣接口模型
class Douban: Mappable {
    var channels: [Channel]?
    init() {
    }
    required init?(map: Map) {
    }
    
    func mapping(map: Map) {
        channels <- map["channels"]
    }
}
//频道模型
class Channel: Mappable {
    var name: String?
    var nameEn: String?
    var channelId: String?
    var seqId: Int?
    var abbrEn: String?
    init() { }
    required init(map:Map) {}
    
    //Mappable
    func mapping(map: Map) {
        name <- map["name"]
        nameEn <- map["name_en"]
        channelId <- map["channel_id"]
        seqId <- map["seq_id"]
        abbrEn <- map["abbr_en"]
    }
}
```

（2）下面样例演示如何获取数据，并转换成对应的模型。

```
curl -X GET 
"https://www.douban.com/j/app/radio/channels" -i -v
Success (432ms): Status 200
--- 共47个频道 ---
华语 (id:1
欧美 (id:2
七零 (id:3
八零 (id:4
九零 (id:5
粤语 (id:6
摇滚 (id:7
民谣 (id:8
轻音乐 (id:9
原声 (id:10
Fly by midnight  (id:267
//... ...
```



```swift
//创建URL对象
let urlString = "https://www.douban.com/j/app/radio/channels"
let url = URL(string: urlString)
//创建请求对象
let request = URLRequest(url: url!)

//创建并发起请求
URLSession.shared.rx.json(request: request).mapObject(type: Douban.self)
    .subscribe(onNext: { (douban: Douban) in
        if let channels = douban.channels {
            print("--- 共\(channels.count)个频道 ---")
            for channel in channels {
                if let name = channel.name, let channelId = channel.channelId {
                    print("\(name) (id:\(channelId)")
                }
            }
        }
}).disposed(by: disposeBag)
```

（3）下面样例演示将数据换成模型，并绑定到表格上显示。

![img](https:////upload-images.jianshu.io/upload_images/3788243-76e0016ddc34da51.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_44ViewController: UIViewController {
    
    var tableView: UITableView!
    
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //创建表格视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView)


        //创建URL对象
        let urlString = "https://www.douban.com/j/app/radio/channels"
        let url = URL(string: urlString)
        //创建请求对象
        let request = URLRequest(url: url!)
        
        //获取列表数据
        let data = URLSession.shared.rx.json(request: request).mapObject(type: Douban.self).map{$0.channels ?? []}
        
        //将数据绑定到表格
        data.bind(to: tableView.rx.items){ (tableView, row, element) in
            
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
            cell.textLabel?.text = "\(row): \(element.name!)"
            return cell
            
        }.disposed(by: disposeBag)
        
    }

}
```