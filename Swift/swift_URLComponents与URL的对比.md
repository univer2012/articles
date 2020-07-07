

来自：1.[URLComponents](https://blog.csdn.net/u014084081/article/details/105806025/)

2.[iOS基础之URLComponents](https://www.jianshu.com/p/db16cdd1ce4f)



为什么要使用`URLComponents`，例如要创建一个URL，我们可能会采用如下的拼接方式

```swift
let url = URL(string: "https://test.com/api/test/?search=\(searchTerm)&format=\(format)")
```

但是这种方式使用起来比较麻烦，也很容易出错

使用`URLComponents`可以避免这些问题

```swift
import Foundation

let searchTerm = "obi wan kenobi"
let format = "wookiee"

var urlComponents = URLComponents()
urlComponents.scheme = "https"
urlComponents.host = "swapi.co"
urlComponents.path = "/api/people"
urlComponents.queryItems = [
   URLQueryItem(name: "search", value: searchTerm),
   URLQueryItem(name: "format", value: format)
]

print(urlComponents.url?.absoluteString) 
// https://swapi.co/api/people?search=obi%20wan%20kenobi&format=wookie
12345678910111213141516
```

将`Dictionary`转为`URLQueryItem`数组

```swift
import Foundation

extension URLComponents {
    
    mutating func setQueryItems(with parameters: [String: String]) {
        self.queryItems = parameters.map { URLQueryItem(name: $0.key, value: $0.value) }
    }
}

let queryParams: [String: String] = [
    "search": "obi wan kenobi",
    "format": "wookie"
]

var urlComponents = URLComponents()
urlComponents.scheme = "https"
urlComponents.host = "swapi.co"
urlComponents.path = "/api/people"
urlComponents.setQueryItems(with: queryParams)
print(urlComponents.url?.absoluteString)
// https://swapi.co/api/people?search=obi%20wan%20kenobi&format=wookie
```



