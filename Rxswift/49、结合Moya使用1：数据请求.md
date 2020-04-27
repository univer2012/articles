# 49、结合Moya使用1：数据请求

`Moya` 是一个基于 `Alamofire` 的更高层网络请求封装抽象层。它可以对我们项目中的所有请求进行集中管理，方便开发与维护。同时 `Moya` 自身也提供了对 `RxSwift` 的扩展，通过与 `RxSwift` 的结合，能让 `Moya` 变得更加强大。下面我就通过样例演示如何使用“`RxSwift + Moya`”这个组合进行开发。

## 一 、安装配置
作者之前也写过关于 `Moya` 的文章（[点击查看](https://link.jianshu.com/?t=http%3A%2F%2Fwww.hangge.com%2Fblog%2Fcache%2Fdetail_1797.html)），当时是通过 `CocoaPods` 来进行安装配置。也可改用手动配置。

需要用到的库下载到本地：

* RxSwift：https://github.com/ReactiveX/RxSwift
* Alamofire：https://github.com/Alamofire/Alamofire
* Moya：https://github.com/Moya/Moya
* Result：https://github.com/antitypical/Result


## 二、数据请求样例
### 1，效果图
我们使用 `Moya` 调用豆瓣 FM 的 `API` 接口，获取所有的频道列表并输出到控制台中。

```swift
返回的数据时： {"channels":[{"name_en":"Personal Radio","seq_id":0,"abbr_en":"My","name":"私人兆赫","channel_id":0},{"name":"华语","seq_id":0,"abbr_en":"","channel_id":"1","name_en":""},{"name":"欧美","seq_id":1,"abbr_en":"","channel_id":"2","name_en":""},{"name":"七零","seq_id":2,"abbr_en":"","channel_id":"3","name_en":""},{"name":"八零","seq_id":3,"abbr_en":"","channel_id":"4","name_en":""},{"name":"九零","seq_id":4,"abbr_en":"","channel_id":"5","name_en":""},{"name":"粤语","seq_id":5,"abbr_en":"","channel_id":"6","name_en":""},{"name":"摇滚","seq_id":6,"abbr_en":"","channel_id":"7","name_en":""},{"name":"民谣","seq_id":7,"abbr_en":"","channel_id":"8","name_en":""},{"name":"轻音乐","seq_id":8,"abbr_en":"","channel_id":"9","name_en":""},
//... ...
```



### 2，网络请求层
我们先创建一个 `DouBanAPI.swift` 文件作为网络请求层，里面的内容如下：

1. 首先定义一个 `provider`，即请求发起对象。往后我们如果要发起网络请求就使用这个 `provider`。
2. 接着声明一个 `enum` 来对请求进行明确分类，这里我们定义两个枚举值分别表示获取频道列表、获取歌曲信息。
3. 最后让这个 `enum` 实现 `TargetType` 协议，在这里面定义我们各个请求的 `url`、参数、`header` 等信息。
```swift
import Foundation
import Moya

//初始化豆瓣FM请求的provider
let DouBanProvider = MoyaProvider<DouBanAPI>()

/** 下面定义豆瓣FM请求的endpoints（供provider使用）**/

//请求分类
public enum DouBanAPI {
    case channels   //获取频道列表
    case playlist(String)   //获取歌曲
}
//请求配置
extension DouBanAPI: TargetType {
    //服务器地址
    public var baseURL: URL {
        switch self {
        case .channels:
            return URL(string: "https://www.douban.com")!
        case .playlist(_):
            return URL(string: "https://douban.fm")!
        }
    }
    //各个请求的具体路径
    public var path: String {
        switch self {
        case .channels:
            return "/j/app/radio/channels"
        case .playlist(_):
            return "/j/mine/playlist"
        }
    }
    //请求类型
    public var method: Moya.Method {
        return .get
    }
    //请求任务事件（这里附带上参数）
    public var task: Task {
        switch self {
        case .playlist(let channel):
            var params: [String: Any] = [:]
            params["channel"] = channel
            params["type"] = "n"
            params["from"] = "mainsite"
            return .requestParameters(parameters: params, encoding: URLEncoding.default)
        default:
            return .requestPlain
        }
    }
    //是否执行Alamofire验证
    public var validate: Bool {
        return false
    }
    //这个就是做单元测试模拟的数据，只会在单元测试文件中有作用
    public var sampleData: Data {
        return "{}".data(using: String.Encoding.utf8)!
    }
    //请求头
    public var headers: [String : String]? {
        return nil
    }
}
```
### 3，使用样例
（1）我们在视图控制器中通过上面的定义的 `provider` 即可发起请求，获取数据。具体代码如下：
```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_49ViewController: UIViewController {

    let disposeBag = DisposeBag()
    
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //获取数据
        DouBanProvider.rx.request(.channels).subscribe { (event) in
            switch event {
            case let .success(response):
                //数据处理
                let str = String(data: response.data, encoding: String.Encoding.utf8)
                print("返回的数据时：",str ?? "")
            case let .error(error):
                print("数据请求失败！错误原因：",error)
            }
        }.disposed(by: disposeBag)
        
    }
    
}
```

（2）订阅相关的代码还可以换种方式写：
```swift
import UIKit
import RxSwift
import RxCocoa

class SHRxswift_49ViewController: UIViewController {

    let disposeBag = DisposeBag()
    
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //获取数据
        DouBanProvider.rx.request(.channels).subscribe(onSuccess: { (response) in
            //数据处理
            let str = String(data: response.data, encoding: String.Encoding.utf8)
            print("返回的数据时：",str ?? "")
        }) { (error) in
            print("数据请求失败！错误原因：",error)
        }.disposed(by: disposeBag)
        
    }
    
}
```
---
[完]


