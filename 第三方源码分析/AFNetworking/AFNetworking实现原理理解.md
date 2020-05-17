参考：
1. [AFNetworking实现原理理解](https://www.jianshu.com/p/02b25f6d1e1f)
2. 

## NSURLSession:

### NSURLSession由三个基本模块构成：
1. `NSURLSession`

2. `NSURLSessionConfiguation`

3. `NSURLSessionTask`

`NSURLSession`相对于平时通信中的会话，但本身却不会进行网络数据传输，它会创建多个`NSURLSessionTask`去执行每次的网络请求。

`NSURLSession`的行为取决于三个方面。包括**NSURLSession的类型、NSURLSessionTask的类型和在创建task时APP是否处于前端**

### NSURLSession有三种类型

1. `defaultSession`将cache和creditials储存于本地

2. `Ephemeral Session`对数据更加保密安全，并不会向本地储存任何数据，将cache和creditials储存在内存中，并和Session绑定，当Session销毁时，对应的数据也会被销毁。

3. `backgroundSession`可以时APP处于后台时继续数据传输，其行为与defaultSession类似，但是所有的数据传输均由一个非本APP的进程来管理。也有一些功能上的限制。

在创建Session对象时通过NSURLSessionConfigration来配置，可设置Session的delegate

==Session一但配置完成，就不能修改，除非创建一个新的Session对象。==

NSURLSessionTask包括三种Task类型，分别为：`NSURLSessionDataTask`，`NSURLSessionDownLoadTask`，`NSURLSessionUploadTask`。

所有的Task状态都是暂停的，需要用`[Task resume]`启动Task

##### `NSURLSession`有两种获取数据的方式：

初始化session时指定delegate，在代理方法中返回数据，需要实现NSURLSession的两个代理方法

初始化Session时未指定delegate的，通过block回调返回数据。


##### NSURLSession对象的销毁，有两种销毁模式：

`- (void)invalidateAndCancel` 取消该Session中的所有Task，销毁所有delegate、block和Session自身，调用后Session不能再复用。

`- (void)finishTasksAndInvalidate `会立即返回，但不会取消已启动的task，而是当这些task完成时，调用delegate

这里有个地方需要注意，即：==`NSURLSession`对象对其delegate都是强引用的，只有当Session对象`invalidate`， 才会释放delegate，否则会出现memory leak。==

使用Session加速网络访问速度，使用同一个Session中的task访问数据，不用每次都实现三次握手，复用之前服务器和客户端之间的网络链接，从而加快访问速度。


# AFNetworking：
AFNetworking是封装的`NSURLSession`的网络请求


## AFNetworking由五个模块组成：
分别由`NSURLSession`,`Security`,`Reachability`,`Serialization`,`UIKit`五部分组成

1. NSURLSession：网络通信模块（核心模块） 对应 AFNetworking中的 AFURLSessionManager和对HTTP协议进行特化处理的AFHTTPSessionManager,AFHTTPSessionManager是继承于AFURLSessionmanager的

2. Security：网络通讯安全策略模块  对应 `AFSecurityPolicy`

3. Reachability：网络状态监听模块 对应AFNetworkReachabilityManager

4. Seriaalization：网络通信信息序列化、反序列化模块 对应 AFURLResponseSerialization

5. UIKit：对于iOS UIKit的扩展库

### 网络请求的过程：
创建`NSURLSessionConfig`对象 --> 用创建的config对象配置初始化NSURLSession --> 创建NSURLSessionTask对象并resume执行，用delegate或者block回调返回数据。

AFURLSessionManager封装了上述网络交互功能

`AFURLSessionManager`请求过程

1. 初始化AFURLSessionManager。

2. 获取AFURLSessionManager的Task对象

3. 启动Task

`AFURLSessionManager`会为每一个Task创建一个`AFURLSessionmanagerTaskDelegate`对象，manager会让其处理各个Task的具体事务，从而实现了manager对多个Task的管理

初始化好manager后，获取一个网络请求的Task，生成一个Task对象，并创建了一个`AFURLSessionmanagerTaskDelegate`并将其关联，设置Task的上传和下载delegate，通过KVO监听download进度和upload进度

### NSURLSessionDelegate的响应

因为`AFURLSessionmanager`所管理的AFURLSession的delegate指向其自身，因此所有的NSURLSessiondelegate的回调地址都是`AFURLSessionmanager`，而`AFURLSessionmanager`又会根据是否需要具体处理会将AF  delegate所响应的delegate，传递到对应的AF delegate去

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AFNetworking%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E7%90%86%E8%A7%A3_1.png)


![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AFNetworking%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E7%90%86%E8%A7%A3_2.png)