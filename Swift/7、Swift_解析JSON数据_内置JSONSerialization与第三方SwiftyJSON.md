来自：

1. [Swift - 解析JSON数据（内置JSONSerialization与第三方JSONKit）](https://www.hangge.com/blog/cache/detail_647.html)
2. [Swift - SwiftyJSON的使用详解（附样例，用于JSON数据处理）](https://www.hangge.com/blog/cache/detail_968.html)



## 一，使用自带的JSONSerialization

苹果从IOS5.0后推出了SDK自带的JSON解决方案`NSJSONSerialization`。而自Swift3起，这个又改名成`JSONSerialization`。这是一个非常好用的JSON生成和解析工具，效率也比其他第三方开源项目高。



`JSONSerialization`能将JSON转换成Foundation对象，也能将Foundation对象转换成JSON，但转换成JSON的对象必须具有如下属性：

1，顶层对象必须是`Array`或者`Dictionary`

2，所有的对象必须是`String`、`Number`、`Array`、`Dictionary`、`Null`的实例

3，所有`Dictionary`的key必须是`String`类型

4，数字对象不能是非数值或无穷



> 注意：尽量使用`JSONSerialization.isValidJSONObject`先判断能否转换成功。



**样例1：将对象转成json字符串，再转回来**

```swift
import UIKit

class SHJSONSerializationViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        let label = UILabel(frame: CGRect(x: 100, y: 100, width: 300, height: 100))
        label.text = "输出结果在控制台"
        self.view.addSubview(label)
        
        testJson()
    }
    func testJson() {
        //Swift对象
        let user: [String: Any] = [
            "uname":"张三",
            "tel":["mobile":"138", "home":"010"]
        ]
        
        //首先判断能不能转换
        if !JSONSerialization.isValidJSONObject(user) {
            print("is not a valid json object")
            return
        }
        
        //利用自带的json库转换成Data
        //如果设置options为JSONSerialization.WritingOptions.prettyPrinted，则打印格式更好阅读
        let data = try? JSONSerialization.data(withJSONObject: user, options: [])
        //Data转换成String打印输出
        let str = String(data: data!, encoding: String.Encoding.utf8)
        //输出json字符串
        print("Json Str:")
        print(str ?? "")
        
        //MARK: 把Data对象转换回JSON对象
        let json = try? JSONSerialization.jsonObject(with: data!, options: .allowFragments) as! [String: Any]
        print("Json Object:",json ?? [:])
        //验证JSON对象可用性
        let uname = json?["uname"]
        let mobile = (json?["tel"] as! [String: Any]) ["mobile"]
        print("get Json Object:", "uname:\(String(describing: uname)), mobile: \(String(describing: mobile))")
        
    }

}
```

输出结果如下：

```
Json Str:
{"uname":"张三","tel":{"mobile":"138","home":"010"}}
Json Object: ["tel": {
    home = 010;
    mobile = 138;
}, "uname": 张三]
get Json Object: uname:Optional(张三), mobile: Optional(138)
```



**样例2：解析json字符串** 
 （由于是字符串内容是json数组，则转成NSArray。如果字符串是json对象，则转成NSDictionary。）

```swift
func testJson2() {
        let string = "[{\"ID\":1,\"Name\":\"元台禅寺\",\"LineID\":1},{\"ID\":2,\"Name\":\"田坞里山塘\",\"LineID\":1},{\"ID\":3,\"Name\":\"滴水石\",\"LineID\":1}]"
        let data = string.data(using: String.Encoding.utf8)
        
        let jsonArr = try! JSONSerialization.jsonObject(with: data!, options: JSONSerialization.ReadingOptions.mutableContainers) as! [[String: Any]]
        
        print("记录数： \(jsonArr.count)")
        for json in jsonArr {
            print("ID: ",json["ID"]!, "      Name: ",json["Name"]!)
        }
    }
```

打印如下：

```
记录数： 3
ID:  1       Name:  元台禅寺
ID:  2       Name:  田坞里山塘
ID:  3       Name:  滴水石
```





## 二，使用第三方库 - SwiftyJSON 

#### 1，SwiftyJSON介绍与配置

 SwiftyJSON是个使用Swift语言编写的开源库，可以让我们很方便地处理JSON数据（解析数据、生成数据）。 

**GitHub地址：**https://github.com/SwiftyJSON/SwiftyJSON
 **使用配置：**直接将 **SwiftyJSON.swift** 添加到项目中即可。 



 ####2，SwiftyJSON的优点

同 `JSONSerializationSwiftyJSON`相比，在获取多层次结构的JSON数据时。`SwiftyJSON`不需要一直判断这个节点是否存在，是不是我们想要的`class`，下一个节点是否存在，是不是我们想要的class…。同时，`SwiftyJSON`内部会自动对`optional`（可选类型）进行拆包（Wrapping ），大大简化了代码。 



下面通过几个样例作为演示。 

**（1）比如我们有一个如下的JSON数据，表示联系人集合：**

```json
[
    {
        "name": "hangge",
        "age": 100,
        "phones": [
            {
                "name": "公司",
                "number": "123456"
            },
            {
                "name": "家庭",
                "number": "001"
            }
        ]
    },
    {
        "name": "big boss",
        "age": 1,
        "phones": [
            {
                "name": "公司",
                "number": "111111"
            }
        ]
    }
]
```

为便于测试比较，我们先将JSON格式的字符串转为Data： 

```swift
let jsonStr = "[{\"name\": \"hangge\", \"age\": 100, \"phones\": [{\"name\": \"公司\",\"number\": \"123456\"}, {\"name\": \"家庭\",\"number\": \"001\"}]}, {\"name\": \"big boss\",\"age\": 1,\"phones\": [{ \"name\": \"公司\",\"number\": \"111111\"}]}]"

if let jsonData = jsonStr.data(using: String.Encoding.utf8, allowLossyConversion: false) {
    //... ...
}
```

**（2）使用JSONSerializationSwiftyJSON解析**
 比如我们要取第一条联系人的第一个电话号码，每个级别都判断就很麻烦，代码如下： 

```swift
if let userArray = try? JSONSerialization.jsonObject(with: jsonData, options: .allowFragments) as? [[String: AnyObject]],
    let phones = userArray?[0]["phones"] as? [[String: AnyObject]],
    let number = phones[0]["number"] as? String {
    
    //找到电话号码
    print("第一个联系人的第一个电话号码：",number)
}
```

打印：

```
第一个联系人的第一个电话号码： 123456
```



即使使用optional来简化一下，代码也不少： 

```swift
if let userArray = try? JSONSerialization.jsonObject(with: jsonData, options: .allowFragments) as? [[String: AnyObject]],
    let number = (userArray?[0]["phones"] as? [[String: AnyObject]])?[0]["number"] as? String {
    
    //找到电话号码
    print("第一个联系人的第一个电话号码：",number)
}
```



**（3）使用SwiftyJSON解析：**
 不用担心数组越界，不用判断节点，拆包什么的，代码如下： 

```swift
let json = try! JSON(data: jsonData)
if let number = json[0]["phones"][0]["number"].string {
    //找到电话号码
    print("第一个联系人的第一个电话号码：",number)
}
```

打印：

```
第一个联系人的第一个电话号码： 123456
```



如果没取到值，还可以走到错误处理来了，打印一下看看错在哪： 

```swift
let json = try! JSON(data: jsonData)
if let number = json[0]["phones"][0]["number"].string {
    //找到电话号码
    print("第一个联系人的第一个电话号码：",number)
} else {
    //打印错误信息
    print(json[0]["phones"][0]["number"])
}
```



 **3，获取网络数据，并使用SwiftyJSON解析**
 除了解析本地的JSON数据，我们其实更常通过url地址获取远程数据并解析。
 **（1）与URLSession结合** 

```swift
func networkRequest() {
    //创建URL对象
    //原地址无效：http://www.hangge.com/getJsonData.php
    let url = URL(string: "https://www.douban.com/j/app/radio/channels")
    //创建请求对象
    let request = URLRequest(url: url!)
    
    let dataTask = URLSession.shared.dataTask(with: request) { (data, response, error) in
        if error != nil {
            print(error)
        } else {
            
            if let tempData = data,
                let json = try? JSON(data: tempData),
                let name  = json["channels"][0]["name"].string {
                //找到频道的名字
                print("第一个频道的名字：",name)
            }
        }
    } as URLSessionTask
    
    //使用resume方法启动任务
    dataTask.resume()
}
```

打印：

```
第一个频道的名字： 私人兆赫
```



**（2）与Alamofire结合** 

```swift
func requestAlamofire() {
    //创建URL对象
    //原地址无效：http://www.hangge.com/getJsonData.php
    let url = URL(string: "https://www.douban.com/j/app/radio/channels")!
    
    Alamofire.request(url).validate().responseJSON { (response) in
        switch response.result.isSuccess {
        case true:
            if let value = response.result.value {
                let json = JSON(value)
                if let name  = json["channels"][0]["name"].string {
                    //找到频道的名字
                    print("第一个频道的名字：",name)
                }
            }
        case false:
            print(response.result.error)
        }
    }
}
```



**4，获取值** 

**（1）可选值获取（Optional getter）**

通过`.number`、`.string`、`.bool`、`.int`、`.uInt`、`.float`、`.double`、`.array`、`.dictionary`、`int8`、`Uint8`、`int16`、`Uint16`、`int32`、`Uint32`、`int64`、`Uint64`等方法获取到的是可选择值，我们需要自行判断是否存在，同时不存在的话可以获取具体的错误信息。


```swift
//int
if let seq_id = json["channels"][0]["seq_id"].int {
    print(seq_id)
} else {
    print(json["channels"][0]["seq_id"])
}

//String
if let name_en = json["channels"][0]["name_en"].string {
    print(name_en)
} else {
    print(json["channels"][0]["name_en"])
}
```



**（2）不可选值获取（Non-optional getter）**
 **使用 `xxxValue` 这样的属性获取值，如果没获取到的话会返回一个默认值**。省得我们再判断拆包了。

```swift
//If not a Number or nil, return 0
let seq_id = json["channels"][0]["seq_id"].intValue

//If not a String or nil, return ""
let name_en = json["channels"][0]["name_en"].stringValue

//If not a Array or nil, return []
let list: Array<JSON> = json["channels"].arrayValue

//If not a Dictionary or nil, return [:]
let firstDict: Dictionary<String, JSON> = json["channels"][0].dictionaryValue

print("seq_id=",seq_id,"name_en=",name_en,"list=",list,"firstDict=",firstDict)
```



**（3）获取原始数据（Raw object）**

```swift
let jsonObject = json.object as AnyObject
 
let jsonObject = json.rawValue  as AnyObject
 
//JSON转化为Data
let data = json.rawData()
 
//JSON转化为String字符串
if let string = json.rawString() {
    //Do something you want
}
 
//JSON转化为Dictionary字典（[String: AnyObject]?）
if let dic = json.dictionaryObject {
    //Do something you want
}
 
//JSON转化为Array数组（[AnyObject]?）
if let arr = json.arrayObject {
    //Do something you want
}
```

**5，设置值**

```swift
json[0]["age"].int =  101
json[0]["name"].string =  "hangge.com"
json[0]["phones"].arrayObject = [["name":"固话", "number":110],["name":"手机", "number":120]]
json[0]["phones"][0].dictionaryObject = ["name":"固话", "number":100]
```



 **6，下标访问（Subscript）**
 可以通过数字、字符串、数组类型的下标访问JSON数据的层级与属性。比如下面三种方式的结果都是一样的：

```swift
//方式1
let number = json[0]["phones"][0]["number"].stringValue
     
//方式2
let number = json[0,"phones",0,"number"].stringValue
     
//方式3
let keys:[JSONSubscriptType] = [0,"phones",0,"number"]
let number = json[keys].stringValue
```



 **7，循环遍历JSON对象中的所有数据**
 **（1）如果JSON数据是数组类型（Array）**

```swift
for (index, subJson): (String,JSON) in json["channels"] {
    print("\(index): \(subJson)")
}
```

打印：

```
0: {
  "abbr_en" : "My",
  "channel_id" : 0,
  "name_en" : "Personal Radio",
  "name" : "私人兆赫",
  "seq_id" : 0
}
1: {
  "abbr_en" : "",
  "channel_id" : "1",
  "name_en" : "",
  "name" : "华语",
  "seq_id" : 0
}
//... ...
```



**（2）如果JSON数据是字典类型（Dictionary）** 

```swift
for (key,subJson):(String,JSON) in json["channels"][0] {
    print("\(key): \(subJson)")
}
```

打印：

```
seq_id: 0
abbr_en: My
channel_id: 0
name: 私人兆赫
name_en: Personal Radio
```



 **8，构造创建JSON对象数据**
 **（1）空的JSON对象** 

```swift
let json: JSON =  nil
```

**（2）使用简单的数据类型创建JSON对象** 

```swift
func jsonDemo() {
    let json: JSON? =  nil
    
    //StringLiteralConvertible
    let json1: JSON = "I'm a son"
     
    //IntegerLiteralConvertible
    let json2: JSON =  12345
     
    //BooleanLiteralConvertible
    let json3: JSON =  true
     
    //FloatLiteralConvertible
    let json4: JSON =  2.8765
    
    print(json)
    print(json1)
    print(json2)
    print(json3)
    print(json4)
}
```

打印：

```
nil
I'm a son
12345
true
2.8765
```



**（3）使用数组或字典数据创建JSON对象** 

```swift
func jsonDemo2() {
    //DictionaryLiteralConvertible
    let json: JSON =  ["I":"am", "a":"son"]
     
    //ArrayLiteralConvertible
    let json1: JSON =  ["I", "am", "a", "son"]
     
    //Array & Dictionary
    var json2: JSON =  ["name": "Jack", "age": 25, "list": ["a", "b", "c", ["what": "this"]]]
    json2["list"][3]["what"] = "that"
    json2["list",3,"what"] = "that"
    let path:[JSONSubscriptType] = ["list",3,"what"]
    json2[path] = "that"
    
    print("json=",json)
    print("json1=",json1)
    print("json2=",json2)
}
```

打印：

```
json= {
  "I" : "am",
  "a" : "son"
}
json1= [
  "I",
  "am",
  "a",
  "son"
]
json2= {
  "name" : "Jack",
  "list" : [
    "a",
    "b",
    "c",
    {
      "what" : "that"
    }
  ],
  "age" : 25
}
```



---

【完】

