参考：
1. [关于AFNetworking3.0+的使用](https://www.jianshu.com/p/5e187c9d389b)


# 一、声明
1. 为了让刚接触AFNetworking的开发者能够顺序上手，本文专门对AFNetworking的使用方法进行了归纳总结，同时方便自己以后的学习。

# 二、背景
- AFNetworking 1.0建立在NSURLConnection的基础API之上 ，
- AFNetworking 2.0开始使用NSURLConnection的基础API ，以及较新基于NSURLSession的API的选项。
- AFNetworking 3.0现已完全基于NSURLSession的API，这降低了维护的负担，同时支持苹果增强关于NSURLSession提供的任何额外功能。

由于Xcode 7中，`NSURLConnection`的API已经正式被苹果弃用。虽然该API将继续运行，但将没有新功能将被添加，并且苹果已经通知所有基于网络的功能，以充分使`NSURLSession`向前发展。

# 三、NSURLConnection、NSURLSession的介绍
## 3.1、 NSURLConnection的介绍

关于NSURLConnection的使用，本文不做详细的介绍，具体参考简书某位大神的介绍http://www.jianshu.com/p/f291ee58c012

## 3.2、 NSURLSession的介绍
### NSURLSession的优点：
1. 后台上传和下载。当你的程序退出了也能进行网络操作，这对用户和APP来说都是个好消息，不用运行APP就可以下载和上传，这样更节约手机电量。
2. 能够暂停和恢复网络操作。不需要使用NSOperation就可以实现暂停、继续、重启等操作。
3. 可配置的容器。
4. 可以子类化并且可以设置私有存储方式。可以修改数据的存储方式和存储位置。
5. 改进了授权处理机制。
6. 代理更强大。
7. 通过文件系统上传和下载。

#### 创建NSURLSession对象的方法
```
//1.使用静态的sharedSession方法，该类使用共享的会话，该会话使用全局的Cache，Cookie和证书。
+ (NSURLSession *)sharedSession;  

//2.通过sessionWithConfiguration:方法创建对象，也就是创建对应配置的会话，与NSURLSessionConfiguration合作使用。
(NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration;  

//3.通过sessionWithConfiguration:delegate:delegateQueue方法创建对象，二三两种方式可以创建一个新会话并定制其会话类型。该方式中指定了session的委托和委托所处的队列。当不再需要连接时，可以调用Session的invalidateAndCancel直接关闭，或者调用finishTasksAndInvalidate等待当前Task结束后关闭。这时Delegate会收到URLSession:didBecomeInvalidWithError:这个事件。Delegate收到这个事件之后会被解引用。
+ (NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration delegate:(id <NSURLSessionDelegate>)delegate delegateQueue:(NSOperationQueue *)queue;  
```
#### NSURLSessionTask支持的三种任务
加载数据/下载/上传

#### NSURLSessionTask类
NSURLSessionTask是一个抽象类，它有三个子类
```
1.NSURLSessionDataTask
2.NSURLSessionUploadTask
3.NSURLSessionDownloadTask
```
这三个类封装了应用程序的三个基本网络任务：获取数据，比如JSON或XML，以及上传和下载文件。

继承关系如下：
![NSURLSessionTask的继承关系](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%85%B3%E4%BA%8EAFNetworking3.0%2B%E7%9A%84%E4%BD%BF%E7%94%A8_1.png)

#### NSURLSessionTask相关方法
```
//suspend可以让当前的任务暂停
- (void)suspend;
//resume方法不仅可以启动任务,还可以唤醒suspend状态的任务
- (void)resume;
//cancel方法可以取消当前的任务,你也可以向处于suspend状态的任务发送cancel消息,任务如果被取消便不能再恢复到之前的状态.
- (void)cancel;
```
#### NSURLSessionConfiguration配置信息
NSURLSessionConfiguration用于配置会话的属性，可以通过该类配置会话的工作模式：
```
NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
// 超时时间
config.timeoutIntervalForRequest = 10;
// 是否允许使用蜂窝网络(后台传输不适用)
config.allowsCellularAccess = YES;

// 还有很多可以设置的属性
//关于配置信息的实例化方法大概有三种
/*
//默认会话模式（default）：工作模式类似于原来的NSURLConnection，使用的是基于磁盘缓存的持久化策略，使用用户keychain中保存的证书进行认证授权。
+ (NSURLSessionConfiguration *)defaultSessionConfiguration;

//瞬时会话模式（ephemeral）：该模式不使用磁盘保存任何数据。所有和会话相关的caches，证书，cookies等都被保存在RAM中，因此当程序使会话无效，这些缓存的数据就会被自动清空。
+ (NSURLSessionConfiguration *)ephemeralSessionConfiguration;

//后台会话模式（background）：该模式在后台完成上传和下载，在创建Configuration对象的时候需要提供一个NSString类型的ID用于标识完成工作的后台会话。
+ (NSURLSessionConfiguration *)backgroundSessionConfigurationWithIdentifier:(NSString *)identifier
*/
```
### 3.2.1 NSURLSessionDataTask简单GET请求
如果请求的数据比较简单,也不需要对返回的数据做一些复杂的操作.那么我们可以使用带block：
```
    // 快捷方式获得session对象
    NSURLSession *session = [NSURLSession sharedSession];
    /*GET请求将参数拼接在 url 后面
     网络接口 和 参数 以 ? 分隔. 参数和参数之间以 & 符号分隔.注意删除最后一个 & 符号.
     如：http://127.0.0.1/login.php?username=zhangsan&password=zhang
     */
    NSURL *url = [NSURL URLWithString:@"http://www.daka.com/login?username=daka&pwd=123"];
    // GET请求直接根据url实例化网络任务
    /*
     第一个参数：请求路径:内部会自动将路径包装成请求对象
     第二个参数：completionHandler回调（请求完成【成功|失败】的回调）
     data：响应体信息（期望的数据）
     response：响应头信息，主要是对服务器端的描述
     error：错误信息，如果请求失败，则error有值
     */
    NSURLSessionTask *task = [session dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        // 默认是子线程.
        NSLog(@"%@",[[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    // 开启任务
    [task resume];
```

### 3.2.2 NSURLSessionDataTask简单POST请求
POST和GET的区别就在于request,所以使用session的POST请求和GET过程是一样的,区别就在于对request的处理。
```
    NSURL *url = [NSURL URLWithString:@"http://www.daka.com/login"];
    //创建可变请求对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    //POST请求将参数添加在请求体中
    //设置请求方法
    request.HTTPMethod = @"POST";
    //设置请求体
    request.HTTPBody = [@"username=daka&pwd=123" dataUsingEncoding:NSUTF8StringEncoding];
    NSURLSession *session = [NSURLSession sharedSession];
    /*
     第一个参数：请求对象
     第二个参数：completionHandler回调（请求完成【成功|失败】的回调）
     data：响应体信息（期望的数据）
     response：响应头信息，主要是对服务器端的描述
     error：错误信息，如果请求失败，则error有值
     */
    // 由于要先对request先行处理,我们通过request初始化task
    NSURLSessionTask *task = [session dataTaskWithRequest:request
                                        completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
    {
        // 默认是子线程.
        NSLog(@"%@",[[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    [task resume];
```

### 3.2.3 `NSURLSessionDataTask`的`NSURLSessionDataDelegate`代理方法

`NSURLSession`提供了block方式处理返回数据的简便方式,但如果想要在接收数据过程中做进一步的处理,仍然可以调用相关的协议方法。`NSURLSession`的代理方法和`NSURLConnection`有些类似,都是分为接收响应、接收数据、请求完成几个阶段。
```
@interface SGHAFNetworkingViewController ()<NSURLSessionDelegate>
@end
@implementation SGHAFNetworkingViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    // 使用代理方法需要设置代理,但是session的delegate属性是只读的,要想设置代理只能通过这种方式创建session
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[[NSOperationQueue alloc] init]];
    // 创建任务(因为要使用代理方法,就不需要block方式的初始化了)
    NSURLSessionDataTask *task = [session dataTaskWithRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.daka.com/login?userName=daka&pwd=123"]]];
    // 启动任务
    [task resume];
}
//对应的代理方法如下:
// 1.接收到服务器的响应
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler {
    // 允许处理服务器的响应，才会继续接收服务器返回的数据
    completionHandler(NSURLSessionResponseAllow);
}
// 2.接收到服务器的数据（可能调用多次）
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
    // 处理每次接收的数据
}
// 3.请求成功或者失败（如果失败，error有值）
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    // 请求完成,成功或者失败的处理
}
@end
```


> 提示：
> 关键点在代码注释里面都有提及,重要的地方再强调一下:
>
> 如果要使用代理方法,需要设置代理,但从NSURLSession的头文件发现session的delegate属性是只读的。因此设置代理要通过session的初始化方法赋值:`sessionWithConfiguration:delegate:delegateQueue:`
>
> 其中:
> configuration参数：(文章开始提到的)需要传递一个配置,我们暂且使用默认的配置`[NSURLSessionConfiguration defaultSessionConfiguration]`就好(后面会说下这个配置是干嘛用的);
>
> delegateQueue参数：表示协议方法将会在哪个队列(NSOperationQueue)里面执行。
>
> NSURLSession在接收到响应的时候要先对响应做允许处理:`completionHandler(NSURLSessionResponseAllow);`，才会继续接收服务器返回的数据,进入后面的代理方法。
>
> 值得一提的是，如果在接收响应的时候需要对返回的参数进行处理(如获取响应头信息等)，那么这些处理应该放在前面允许操作的前面。

### 3.2.4、NSURLSessionDownloadTask简单下载
`NSURLSessionDownloadTask`同样提供了通过`NSURL`和`NSURLRequest`两种方式来初始化并通过block进行回调的方法.下面以`NSURL`初始化为例:
```
    NSURLSession *session = [NSURLSession sharedSession];
    NSURL *url = [NSURL URLWithString:@"http://www.daka.com/resources/image/icon.png"] ;
    NSURLSessionDownloadTask *task = [session downloadTaskWithURL:url completionHandler:^(NSURL *location, NSURLResponse *response, NSError *error) {
        // location是沙盒中tmp文件夹下的一个临时url,文件下载后会存到这个位置,由于tmp中的文件随时可能被删除,所以我们需要自己需要把下载的文件挪到需要的地方
        NSString *path = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:response.suggestedFilename];
        [[NSFileManager defaultManager] moveItemAtURL:location toURL:[NSURL fileURLWithPath:path] error:nil];
    }];
    // 启动任务
    [task resume];
```


> Tips:
> 需要注意的就是需要将下载到tmp文件夹的文件转移到需要的目录.原因在代码中已经贴出。
>
> response.suggestedFilename是从响应中取出文件在服务器上存储路径的最后部分,如数据在服务器的url为http://www.daka.com/resources/image/icon.png, 那么其suggestedFilename就是icon.png。

### 3.2.5 `NSURLSessionDownloadTask`的`NSURLSessionDownloadDelegate`代理方法
```
// 每次写入调用(会调用多次)
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
      // 可在这里通过已写入的长度和总长度算出下载进度
      CGFloat progress = 1.0 * totalBytesWritten / totalBytesExpectedToWrite; NSLog(@"%f",progress);
}
// 下载完成调用
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
{
      // location还是一个临时路径,需要自己挪到需要的路径(caches下面) NSString *filePath = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:downloadTask.response.suggestedFilename]; [[NSFileManager defaultManager] moveItemAtURL:location toURL:[NSURL fileURLWithPath:filePath] error:nil];
}
// 任务完成调用
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
}
```
### 3.2.6、NSURLSessionDownloadTask断点下载
```
// 使用这种方式取消下载可以得到将来用来恢复的数据,保存起来
[self.task cancelByProducingResumeData:^(NSData *resumeData) {
    self.resumeData = resumeData;
}];
// 由于下载失败导致的下载中断会进入此协议方法,也可以得到用来恢复的数据
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    // 保存恢复数据
    self.resumeData = error.userInfo[NSURLSessionDownloadTaskResumeData];
}
// 恢复下载时接过保存的恢复数据
self.task = [self.session downloadTaskWithResumeData:self.resumeData];
// 启动任务
[self.task resume];
```
//程序强制退出就无法断点下载了，具体方法见http://blog.csdn.net/qianlima210210/article/details/49303703

### 3.2.7 NSURLSessionUploadTask
在`NSURLSession`中，文件上传方式主要有以下两种：
```
    //第一种方式
    NSURLSessionUploadTask *task =
    [[NSURLSession sharedSession] uploadTaskWithRequest:request fromFile:fileName
                                      completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    }];
    //第二种方式
    [self.session uploadTaskWithRequest:request fromData:body
                      completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
            NSLog(@"-------%@", [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }];
    /*
     处于安全性考虑,通常我们会使用POST方式进行文件上传,所以较多使用第二种方式。
     但是,NSURLSession并没有为我们提供比NSURLConnection更方便的文件上传方式。方法中body处的参数需要填写request的请求体(http协议规定格式的大长串).
     */
```
关于`NSURLSessionConfiguration`的使用可以参考http://www.jianshu.com/p/fafc67475c73
以及http://www.tuicool.com/articles/VBv2qe

# 四、AFNetworking3.0+使用
由于苹果已经弃用`NSURLConnection`，所以在此暂时只介绍AFNetworking3.0及3.0+以上的使用方法。

## 4.1、AFNetworking的导入
本人习惯使用cocoaPods 进行管理第三方类库，导入过程不做详细介绍，不会的可以上网查看教程

## 4.2、简单GET请求
```
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
[manager GET:URL parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {
 } success:^(NSURLSessionDataTask * _Nonnull task, id _Nullable responseObject) {
             NSLog(@"这里打印请求成功要做的事");
 }failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
             NSLog(@"%@",error); //这里打印错误信息
}];
```
## 4.3、简单POST请求
```
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
NSMutableDictionary *parameters = @{@"":@"",@"":@""};
[manager POST:URL parameters:parameters progress:^(NSProgress * _Nonnull uploadProgress) {
} success:^(NSURLSessionDataTask * _Nonnull task, id _Nullable responseObject) {
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
}];
```
## 4.4、下载
```
- (void)downLoad{
    //1.创建管理者对象
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    //2.确定请求的URL地址
    NSURL *url = [NSURL URLWithString:@""];
    //3.创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    //4.下载任务
    NSURLSessionDownloadTask *task = [manager downloadTaskWithRequest:request progress:^(NSProgress * _Nonnull downloadProgress) {
        //打印下下载进度
        NSLog(@"%lf",1.0 * downloadProgress.completedUnitCount / downloadProgress.totalUnitCount);
    } destination:^NSURL * _Nonnull(NSURL * _Nonnull targetPath, NSURLResponse * _Nonnull response) {
        //下载地址
        NSLog(@"默认下载地址:%@",targetPath);
        //设置下载路径，通过沙盒获取缓存地址，最后返回NSURL对象
        NSString *filePath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)lastObject];
        return [NSURL URLWithString:filePath];
    } completionHandler:^(NSURLResponse * _Nonnull response, NSURL * _Nullable filePath, NSError * _Nullable error) {
        //下载完成调用的方法 WKNSLog(@"下载完成：");
        NSLog(@"%@--%@",response,filePath);
    }];
    //开始启动任务
    [task resume];
}
```
## 4.5、上传
```
//第一种方法是通过工程中的文件进行上传
- (void)upLoad1{
    //1。创建管理者对象
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    //2.上传文件
    NSDictionary *dict = @{@"username":@"1234"};
    NSString *urlString = @"22222";
    [manager POST:urlString parameters:dict constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
        //上传文件参数
        UIImage *iamge = [UIImage imageNamed:@"123.png"];
        NSData *data = UIImagePNGRepresentation(iamge);
        //这个就是参数
        [formData appendPartWithFileData:data name:@"file" fileName:@"123.png" mimeType:@"image/png"];
    } progress:^(NSProgress * _Nonnull uploadProgress) {
        //打印下上传进度
        NSLog(@"%lf",1.0 *uploadProgress.completedUnitCount / uploadProgress.totalUnitCount);
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        //请求成功
        NSLog(@"请求成功：%@",responseObject);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        //请求失败
        NSLog(@"请求失败：%@",error);
    }];
}
//第二种是通过URL来获取路径，进入沙盒或者系统相册等等
- (void)upLoda2{
    //1.创建管理者对象
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    //2.上传文件
    NSDictionary *dict = @{@"username":@"1234"};
    NSString *urlString = @"22222";
    [manager POST:urlString parameters:dict constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
        [formData appendPartWithFileURL:[NSURL fileURLWithPath:@"文件地址"] name:@"file" fileName:@"1234.png" mimeType:@"application/octet-stream" error:nil];
    } progress:^(NSProgress * _Nonnull uploadProgress) {
        //打印下上传进度
        NSLog(@"%lf",1.0 *uploadProgress.completedUnitCount / uploadProgress.totalUnitCount);
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        //请求成功
        NSLog(@"请求成功：%@",responseObject);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        //请求失败
        NSLog(@"请求失败：%@",error);
    }];
}
```

## 4.6、 网络监听
```
- (void)AFNetworkStatus{
    //1.创建网络监测者
    AFNetworkReachabilityManager *manager = [AFNetworkReachabilityManager sharedManager];
    /*枚举里面四个状态 分别对应 未知 无网络 数据 WiFi
     typedef NS_ENUM(NSInteger, AFNetworkReachabilityStatus) {
     AFNetworkReachabilityStatusUnknown = -1, 未知
     AFNetworkReachabilityStatusNotReachable = 0, 无网络
     AFNetworkReachabilityStatusReachableViaWWAN = 1, 蜂窝数据网络
     AFNetworkReachabilityStatusReachableViaWiFi = 2, WiFi
     };   */
    [manager setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        //这里是监测到网络改变的block 可以写成switch方便
        //在里面可以随便写事件
        switch (status)
        {
            case AFNetworkReachabilityStatusUnknown:
                NSLog(@"未知网络状态");
                break;
            case AFNetworkReachabilityStatusNotReachable:
                NSLog(@"无网络");
                break;
            case AFNetworkReachabilityStatusReachableViaWWAN:
                NSLog(@"蜂窝数据网");
                break;
            case AFNetworkReachabilityStatusReachableViaWiFi:
                NSLog(@"WiFi网络");
                break;
            default:
                break;
        }
    }] ;
}
```

## 4.7、 关于请求、返回格式设置
所有的网络请求,均有manager发起

### 4.7.1、请求格式（requestSerializer）
需要注意的是,默认提交请求的数据是二进制的,返回格式是JSON
如果提交数据是JSON的,需要将请求格式设置为`AFJSONRequestSerializer`
```
    //AFHTTPRequestSerializer            二进制格式(默认请求格式)
     manager.requestSerializer = [AFHTTPRequestSerializer serializer];
    //AFJSONRequestSerializer         JSON
     manager.requestSerializer = [AFJSONRequestSerializer serializer];
    //AFPropertyListRequestSerializer    PList(是一种特殊的XML,解析起来相对容易)
     manager.requestSerializer = [AFPropertyListRequestSerializer serializer];
```
### 4.7.2、 返回格式
```
    //AFHTTPResponseSerializer          二进制格式
     manager.responseSerializer = [AFHTTPResponseSerializer serializer];
   //AFJSONResponseSerializer          JSON（默认情况下返回json,所以有时后返回的不是json，就要重新设置返回格式）
     manager.responseSerializer = [AFJSONResponseSerializer serializer];
   //AFXMLParserResponseSerializer     XML,只能返回XMLParser,还需要自己通过代理方法解析（下面将介绍用NSXMLParser解析xml）
     manager.responseSerializer = [AFXMLParserResponseSerializer serializer];
   //AFXMLDocumentResponseSerializer (Mac OS X)
   //AFPropertyListResponseSerializer   PList
   //AFImageResponseSerializer        Image
   //AFCompoundResponseSerializer      组合
   //通过acceptableContentTypes可以添加接收的类型，如果没有设置，出错情况下会提示,
   //具体参考http://www.jianshu.com/p/212a128c9a33，
   //可以在`AFURLResponseSerialization.m`源代码中添加接收的类型
   /*
   - (instancetype)init {
        self = [super init];
        if (!self) {
        return nil;
    }
        self.acceptableContentTypes = [NSSet setWithObjects:@"application/json", @"text/json", @"text/javascript", nil];
        return self;
   }
  */
   manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"application/xml"];
```
关于该部分的理解，可以参考http://www.jianshu.com/p/3aa19c12000a

### 例
```
//请求解析
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.responseSerializer = [AFXMLParserResponseSerializer serializer];
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"application/xml"];
    manager.requestSerializer = [AFJSONRequestSerializer serializer];
   //afn默认发起的是异步请求
   [manager GET:@"http://localhost/sources/videos.xml" parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"%@",[NSThread currentThread]);//打印结果是主线程，也就是说，异步请求之后，自动返回主线程
//        NSLog(@"%@",responseObject);
        if ([responseObject isKindOfClass:[NSXMLParser class]]) {
            NSXMLParser *parser = responseObject;
            parser.delegate = self;
            dispatch_async(dispatch_get_global_queue(0, 0), ^{
                [parser parse];
            });
        ｝
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"%@",error.localizedDescription);
    }];
//MARK: 代理部分
-(void)parserDidStartDocument:(NSXMLParser *)parser{
    NSLog(@"打开文档，准备开始解析");
}
-(void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary<NSString *,NSString *> *)attributeDict{
    NSLog(@"startElement=%@ attributeDict=%@",elementName,attributeDict);
    if ([elementName isEqualToString:@"video"]) {
        self.currentVideo = [[XFSVideo alloc]init];
        self.currentVideo.videoId = attributeDict[@"videoId"];
    }
}
-(void)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string{
    NSLog(@"foundCharacters=%@",string);
    [self.elementString appendString:string];
}
-(void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName{
    NSLog(@"endelement=%@",elementName);
    NSLog(@"%@",[NSThread currentThread]);
    if ([elementName isEqualToString:@"video"]) {
        [self.videos addObject:self.currentVideo];
        self.currentVideo = nil;
    }else if (![elementName isEqualToString:@"videos"]){
        [self.currentVideo setValue:self.elementString forKeyPath:elementName];
        self.elementString = nil;
    }
}
-(void)parserDidEndDocument:(NSXMLParser *)parser{
    NSLog(@"%@",self.videos);
    NSLog(@"结束文档");
    dispatch_async(dispatch_get_main_queue(), ^{
    });
}
```

代码可参考：https://github.com/univer2012/personal-document/tree/master/ObjectiveCDemo-16-04-07/ObjectiveCDemo-16-04-07/example/16-05-07/modules/%E6%96%AD%E7%82%B9%E7%BB%AD%E4%BC%A0%E4%B8%8B%E8%BD%BD_%E6%99%AE%E9%80%9A%E4%B8%8B%E8%BD%BD_%E5%90%8E%E5%8F%B0%E4%B8%8B%E8%BD%BD_%E5%8F%96%E6%B6%88%E4%B8%8B%E8%BD%BD/ctrl