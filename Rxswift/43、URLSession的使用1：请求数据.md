`RxSwift`（或者说 `RxCocoa`）除了对系统原生的 `UI` 控件提供了 `rx` 扩展外，对 `URLSession` 也进行了扩展，从而让我们可以很方便地发送 `HTTP` 请求。

## 一、请求网络数据
### 1，通过 `rx.response` 请求数据
（1）下面代码我们通过豆瓣提供的音乐频道列表接口获取数据，并将返回结果输出到控制台中。

```swift
返回的数据是： {"channels":[{"name_en":"Personal Radio","seq_id":0,"abbr_en":"My","name":"私人兆赫","channel_id":0},{"name":"华语","seq_id":0,"abbr_en":"","channel_id":"1","name_en":""},{"name":"欧美","seq_id":1,"abbr_en":"","channel_id":"2","name_en":""},{"name":"七零","seq_id":2,"abbr_en":"","channel_id":"3","name_en":""},{"name":"八零","seq_id":3,"abbr_en":"","channel_id":"4","name_en":""},{"name":"九零","seq_id":4,"abbr_en":"","channel_id":"5","name_en":""},
//... ...
```

```swift
let disposeBag = DisposeBag()

let urlString = "https://www.douban.com/j/app/radio/channels"
let url = URL(string: urlString)
let request = URLRequest(url: url!)

URLSession.shared.rx.response(request: request).subscribe(onNext: { (response: HTTPURLResponse, data: Data) in
    let str = String(data: data, encoding: String.Encoding.utf8)
    print("返回的数据是：",str ?? "")
})
    .disposed(by: disposeBag)
```
（2）从上面样例可以发现，不管请求成功与否都会进入到 `onNext` 这个回调中。如果我们需要根据响应状态进行一些相应操作，比如：


* 状态码在 **200 ~ 300 则正常显示数据**。
* 如果是异常状态码（比如：**404）则弹出告警提示框**。

这个借助 `response` 参数进行判断即可。我们把 `url` 改成一个错误的地址，运行结果如下：

```swift
curl -X GET 
"https://www.douban.com/xxxxxxxx/app/radio/channels" -i -v
Failure (306ms): Status 404
请求失败！
```

```swift
let disposeBag = DisposeBag()

let urlString = "https://www.douban.com/xxxxxxxx/app/radio/channels"
let url = URL(string: urlString)
//创建请求对象
let request = URLRequest(url: url!)

URLSession.shared.rx.response(request: request).subscribe(onNext: { (response, data) in
    //判断响应结果状态码
    if 200 ..< 300 ~= response.statusCode {
        let str = String(data: data, encoding: String.Encoding.utf8)
        print("返回的数据时：",str ?? "")
    } else {
        print("请求失败！")
    }
}).disposed(by: disposeBag)
```

### 2，通过` rx.data` 请求数据
> `rx.data` 与 `rx.response` 的区别：
>
> * 如果不需要获取底层的 `response`，只需知道请求是否成功，以及成功时返回的结果，那么建议使用 `rx.data`。
> * 因为 `rx.data` 会自动对响应状态码进行判断，只有成功的响应（状态码为 200~300）才会进入到 `onNext` 这个回调，否则进入 `onError` 这个回调。

（1）如果不需要考虑请求失败的情况，只对成功返回的结果做处理可以在 `onNext` 回调中进行相关操作。

```
请求成功！返回的数据是： {"channels":[{"name_en":"Personal Radio","seq_id":0,"abbr_en":"My","name":"私人兆赫","channel_id":0},{"name":"华语","seq_id":0,"abbr_en":"","channel_id":"1","name_en":""},{"name":"欧美","seq_id":1,"abbr_en":"","channel_id":"2","name_en":""},{"name":"七零","seq_id":2,"abbr_en":"","channel_id":"3","name_en":""},{"name":"八零","seq_id":3,"abbr_en":"","channel_id":"4","name_en":""}
```
```swift
let urlString = "https://www.douban.com/j/app/radio/channels"
let url = URL(string: urlString)
let request = URLRequest(url: url!)

URLSession.shared.rx.data(request: request).subscribe(onNext: { (data) in
    let str = String(data: data, encoding: String.Encoding.utf8)
    print("请求成功！返回的数据是：",str ?? "")
}).disposed(by: disposeBag)
```

（2）如果还要处理失败的情况，可以在 `onError` 回调中操作。
```swift
curl -X GET 
"https://www.douban.com/xxxxxx/app/radio/channels" -i -v
Failure (1818ms): Status 404
请求失败！错误原因： HTTP request failed with `404`.
```
```swift
let urlString = "https://www.douban.com/xxxxxx/app/radio/channels"
let url = URL(string: urlString)
let request = URLRequest(url: url!)

URLSession.shared.rx.data(request: request).subscribe(onNext: { (data) in
    let str = String(data: data, encoding: String.Encoding.utf8)
    print("请求成功！返回的数据是：",str ?? "")
}, onError: {error in
    print("请求失败！错误原因：",error)
})
    .disposed(by: disposeBag)
```
## 二、手动发起请求、取消请求
在很多情况下，网络请求并不是由程序自动发起的。可能需要我们点击个按钮，或者切换个标签时才去请求数据。除了手动发起请求外，同样的可能还需要手动取消上一次的网络请求（如果未完成）。下面通过样例演示这个如何实现。

### 1，效果图
（1）每次点击“发起请求”按钮则去请求一次数据。

（2）如果请求没返回时，点击“取消请求”则可将其取消（取消后即使返回数据也不处理了）。

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/Rxswift/Rx_43_1.png)

### 2，样例代码

```swift
import UIKit

import RxSwift
import RxCocoa
import RxDataSources



class SHRxswift_43ViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    //"发起请求"按钮
    @IBOutlet weak var startBtn: UIButton!
    //“取消请求”按钮
    @IBOutlet weak var cancelBtn: UIButton!
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        //创建URL对象
        let urlString = "https://www.douban.com/j/app/radio/channels"
        let url = URL(string: urlString)
        //创建请求对象
        let request = URLRequest(url: url!)

        startBtn.rx.tap.asObservable().flatMap{
            URLSession.shared.rx.data(request: request).takeUntil(self.cancelBtn.rx.tap)
        }
        .subscribe(onNext: { (data) in
            let str = String(data: data, encoding: String.Encoding.utf8)
            print("请求成功！，返回的数据时：",str ?? "")
            
        }, onError: { (error) in
            print("请求失败！错误原因：",error)
            
        }).disposed(by: disposeBag)
        
    }
}
```

---
[完]