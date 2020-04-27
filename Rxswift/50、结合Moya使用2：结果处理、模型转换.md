# 50、结合Moya使用2：结果处理、模型转换
## 三、将结果转为 JSON 对象
### 1，实现方法
（1）如果服务器返回的数据是 `json` 格式的话，直接通过 `Moya` 提供的 `mapJSON` 方法即可将其转成 `JSON` 对象。

注意：关于 `DouBanProvider` 里的具体内容，可以参考上文（[点击查看](https://link.jianshu.com/?t=http%3A%2F%2Fwww.hangge.com%2Fblog%2Fcache%2Fdetail_2012.html)）。

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_50ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //获取数据
        DouBanProvider.rx.request(.channels).subscribe(onSuccess: { (response) in
            //数据处理
            let json = try? response.mapJSON() as! [String: Any]
            print("--- 请求成功！返回的如下数据 ---")
            print(json!)
        }) { (error) in
            print("数据请求失败！错误原因：",error)
        }.disposed(by: disposeBag)
    }

}
```
（2）或者使用下面这种写法也是可以的。
```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_50ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //获取数据
        DouBanProvider.rx.request(.channels).mapJSON().subscribe(onSuccess: { (data) in
            //数据处理
            let json = data as! [String: Any]
            print("--- 请求成功！返回的如下数据 ---")
            print(json)
        }) { (error) in
            print("数据请求失败！错误原因：",error)
        }.disposed(by: disposeBag)
    }

}
```
（3）运行结果如下：

```swift
--- 请求成功！返回的如下数据 ---
["channels": <__NSArrayI 0x7f94c3f3e7b0>(
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
{
    "abbr_en" = "";
    "channel_id" = 2;
    name = "\U6b27\U7f8e";
    "name_en" = "";
    "seq_id" = 1;
},
//... ...
```



### 2，使用样例
（1）效果图

* 我们使用 `Moya` 调用豆瓣 `FM` 的 `API` 接口，获取所有的频道列表并显示在表格中。
* 点击任意一个频道，调用另一个接口随机获取该频道下的一首歌曲，并弹出显示。

![](https://upload-images.jianshu.io/upload_images/3788243-94a72111bd557816.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)

![](https://upload-images.jianshu.io/upload_images/3788243-cfd8f997bfdf3b11.png?imageMogr2/auto-orient/strip|imageView2/2/w/320)

（2）样例代码
```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_50ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    //显示频道列表的tableView
    var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView)
        
        //获取列表数据
        let data = DouBanProvider.rx.request(.channels).mapJSON().map { (data) -> [[String: Any]] in
            if let json = data as? [String: Any], let channels = json["channels"] as? [[String: Any]] {
                return channels
            } else {
                return []
            }
        }.asObservable()
        
        //将数据绑定到表格
        data.bind(to: tableView.rx.items){ (tableView, row, element) in
            
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
            cell.textLabel?.text = "\(element["name"]!)"
            cell.accessoryType = .disclosureIndicator
            return cell
            
        }.disposed(by: disposeBag)
        
        //单元格点击
        //接口需要在vpn翻墙才能访问
        tableView.rx.modelSelected([String: Any].self)
            .map{ $0["channel_id"] as! String }
            .flatMap{ DouBanProvider.rx.request(.playlist($0)) }
            .mapJSON()
            .subscribe(onNext: {[weak self] (data) in
                //解析数据，获取歌曲信息
                if let json = data as? [String: Any],
                    let musics = json["song"] as? [[String: Any]] {
                    
                    let artist = musics[0]["artist"]!
                    let title = musics[0]["title"]!
                    let message = "歌手：\(artist)\n歌曲：\(title)"
                    //将歌曲信息弹出显示
                    self?.showAlert(title: "歌曲信息", message: message)
                }
            }).disposed(by: disposeBag)
        
    }
    
    //显示消息
    func showAlert(title:String, message:String) {
        let alertController = UIAlertController(title: title, message: message, preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(cancelAction)
        self.present(alertController, animated: true, completion: nil)
    }

}
```

## 四，将结果映射成自定义对象
### 1，准备工作
（1）要实现数据转模型（`model`），我们这里还要先引入一个第三方的数据模型转换框架：`ObjectMapper`。关于它的安装配置，以及相关说明可以参考航哥之前写的文章：
* [Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://link.jianshu.com/?t=http%3A%2F%2Fwww.hangge.com%2Fblog%2Fcache%2Fdetail_1673.html)

（2）为了让 `ObjectMapper` 能够更好地与 `Moya` 配合使用，我们需要使用 `Moya-ObjectMapper` 这个 `Observable` 扩展库。它的作用是增加数据转模型对象、以及数据转模型对象数组这两个方法。我们现将其下载到本地。

* GitHub 主页：https://github.com/ivanbruel/Moya-ObjectMapper

（3）`Moya-ObjectMapper` 配置很简单只需把 `sourcs` 文件夹中的如下 3 个文件添加到项目中来即可。


* Response+ObjectMapper.swift
* ObservableType+ObjectMapper.swift
* Single+ObjectMapper.swift

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Rxswift/Rx_50_3.png)

### 2，使用样例
（1）我们还是以前面的豆瓣音乐频道数据为例。首先我定义好相关模型（需要实现 `ObjectMapper` 的 `Mappable` 协议，并设置好成员对象与`JSON` 属性的相互映射关系。）

```swift
//MARK: Moya --> Douban 模型
///文章位置：50（结合Moya使用2：结果处理、模型转换）
///豆瓣接口模型
struct Douban: Mappable {
    var channels: [Channel]?
    
    init?(map: Map) { }
    
    mutating func mapping(map: Map) {
        channels <- map["channels"]
    }
}
//频道模型
struct Channel: Mappable {
    var name: String?
    var nameEn: String?
    var channelId: String?
    var seqId: Int?
    var abbrEn: String?
    
    init?(map:Map) { }
    
    //Mappable
    mutating func mapping(map: Map) {
        name <- map["name"]
        nameEn <- map["name_en"]
        channelId <- map["channel_id"]
        seqId <- map["seq_id"]
        abbrEn <- map["abbr_en"]
    }
}
//歌曲列表模型
struct Playlist: Mappable {
    init?(map: Map) {
        
    }
    
    mutating func mapping(map: Map) {
        r <- map["r"]
        isShowQuickStart <- map["is_show_quick_start"]
        song <- map["song"]
    }
    
    var r: Int!
    var isShowQuickStart: Int!
    var song:[Song]!
}
//歌曲模型
struct Song: Mappable {
    var title: String!
    var singers: [Singers]!
    init?(map: Map) { }
    
    // Mappable
    mutating func mapping(map: Map) {
        title <- map["title"]
        singers <- map["singers"]
    }
}
//歌手模型
struct Singers {
    var style: Array<String>!
    var name: String!
    var region: Array<String>!
    var name_usual: String!
    var genre: Array<String>!
    var avatar: String!
    var related_site_id: Int!
    var is_site_artist: Bool!
    var id: String!
    
    init?(map: Map) { }
    
    // Mappable
    mutating func mapping(map: Map) {
        style <- map["style"]
        name <- map["name"]
        region <- map["region"]
        name_usual <- map["name_usual"]
        genre <- map["genre"]
        avatar <- map["avatar"]
        related_site_id <- map["related_site_id"]
        is_site_artist <- map["is_site_artist"]
        id <- map["id"]
    }
}
```
（2）下面样例演示如何获取数据，并转换成对应的模型。
```swift
--- 共47个频道 ---
华语 (id:1)
欧美 (id:2)
七零 (id:3)
八零 (id:4)
九零 (id:5)
粤语 (id:6)
摇滚 (id:7)
民谣 (id:8)
轻音乐 (id:9)
原声 (id:10)
Fly by midnight  (id:267)
//... ...
```

```swift
//获取数据
DouBanProvider.rx.request(.channels)
    .mapObject(Douban.self)
    .subscribe(onSuccess: { (douban) in
        
        if let channels = douban.channels {
            print("--- 共\(channels.count)个频道 ---")
            for channel in channels {
                if let name = channel.name, let channelId = channel.channelId {
                    print("\(name) (id:\(channelId)")
                }
            }
        }
}) { (error) in
    print("数据请求失败！错误原因：",error)
}.disposed(by: disposeBag)
```
（3）下面样例演示将数据换成模型，并绑定到表格上显示。
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Rxswift/Rx_50_2.png)

```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_50ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    //显示频道列表的tableView
    var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView)

        //获取列表数据
        let data = DouBanProvider.rx.request(.channels)
            .mapObject(Douban.self)
            .map{ $0.channels ?? [] }
            .asObservable()

        //将数据绑定到表格
        data.bind(to: tableView.rx.items){ (tableView, row, element) in

            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
            cell.textLabel?.text = "\(element.name!)"
            cell.accessoryType = .disclosureIndicator
            return cell

        }.disposed(by: disposeBag)

        //单元格点击
        //接口需要在vpn翻墙才能访问
        tableView.rx.modelSelected(Channel.self)
            .map{ $0.channelId! }
            .flatMap{ DouBanProvider.rx.request(.playlist($0)) }
            .mapObject(type: Playlist.self)
            .subscribe(onNext: {[weak self] (playlist) in
                //解析数据，获取歌曲信息
                if playlist.song.count > 0 {
                    let artist = playlist.song[0].artist!
                    let title = playlist.song[0].title!
                    let message = "歌手：\(artist)\n歌曲：\(title)"
                    //将歌曲信息弹出显示
                    self?.showAlert(title: "歌曲信息", message: message)
                }
            },onError: {error in
                print("报错，原因是：",error.localizedDescription)
            }).disposed(by: disposeBag)
        
    }
    
    //显示消息
    func showAlert(title:String, message:String) {
        let alertController = UIAlertController(title: title, message: message, preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(cancelAction)
        self.present(alertController, animated: true, completion: nil)
    }

}
```

## 功能改进：将网络请求服务提取出来
（1）上面的样例中我们是在 `VC` 里是直接调用 `Moya` 的 `Provider` 进行数据请求，并进行模型转换。

（2）我们也可以把网络请求和数据转换相关代码提取出来，作为一个专门的 `Service`。比如 `DouBanNetworkService`，内容如下：
```swift
import Foundation

import RxSwift
import RxCocoa
import ObjectMapper

class DouBanNetworkService {
    //获取频道数据
    func loadChannels() -> Observable<[Channel]> {
        return DouBanProvider.rx.request(.channels)
            .mapObject(Douban.self)
            .map { $0.channels ?? [] }
            .asObservable()
    }
    //获取歌曲列表数据
    func loadPlaylist(channelId: String) -> Observable<Playlist> {
        return DouBanProvider.rx.request(.playlist(channelId))
            .mapObject(Playlist.self)
            .asObservable()
    }
    //获取频道下第一首歌曲
    func loadFirstSong(channelId: String) -> Observable<Song> {
        return loadPlaylist(channelId: channelId)
            .filter { $0.song.count > 0 }
            .map { $0.song[0] }
    }
    
}
```

（3）`VC` 这边不再直接调用 `provider`，而是通过这个 `Service` 就获取需要的数据。可以看到代码简洁许多：
```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_50ViewController: UIViewController {
    let disposeBag = DisposeBag()
    
    //显示频道列表的tableView
    var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        self.view.addSubview(self.tableView)

        //豆瓣网络请求服务
        let networkService = DouBanNetworkService()
        
        //获取列表数据
        let data = networkService.loadChannels()

        //将数据绑定到表格
        data.bind(to: tableView.rx.items){ (tableView, row, element) in

            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
            cell.textLabel?.text = "\(element.name!)"
            cell.accessoryType = .disclosureIndicator
            return cell

        }.disposed(by: disposeBag)

        //单元格点击
        //接口需要在vpn翻墙才能访问
        tableView.rx.modelSelected(Channel.self)
            .map{ $0.channelId! }
            .flatMap( networkService.loadFirstSong )
            .subscribe(onNext: {[weak self] (song:Song) in
                //将歌曲信息弹出显示
                let message = "歌手：\(song.artist!)\n歌曲：\(song.title!)"
                self?.showAlert(title: "歌曲信息", message: message)
            
            },onError: {error in
                print("报错，原因是：",error.localizedDescription)
            }).disposed(by: disposeBag)
        
    }
    
    //显示消息
    func showAlert(title:String, message:String) {
        let alertController = UIAlertController(title: title, message: message, preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(cancelAction)
        self.present(alertController, animated: true, completion: nil)
    }

}
```
---
[完]

