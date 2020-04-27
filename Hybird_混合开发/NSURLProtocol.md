参考：
1. [iOS - NSURLProtocol详解和应用](https://www.cnblogs.com/junhuawang/p/8465982.html)
2. [ iOS进阶之使用 NSURLProtocol 拦截 HTTP 请求（转载）](https://www.cnblogs.com/sjxjjx/p/7928143.html)
3. 


### 问题：因dns发生域名劫持 需要手动将URL请求的域名重定向到指定的IP地址

　　最近在项目里由于电信那边发生dns发生域名劫持，因此需要手动将URL请求的域名重定向到指定的IP地址，但是由于请求可能是通过`NSURLConnection`,`NSURLSession`或者`AFNetworking`等方式，因此要想统一进行处理，一开始是想通过Method Swizzling去hook cfnetworking底层方法，后来发现其实有个更好的方法--`NSURLProtocol`。



### NSURLProtocol

`NSURLProtocol`能够让你去重新定义苹果的[URL加载系统](https://link.jianshu.com/?t=https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165-BCICJDHA) (URL Loading System)的行为，URL Loading System里有许多类用于处理URL请求，比如NSURL，NSURLRequest，NSURLConnection和NSURLSession等，**当URL Loading System使用NSURLRequest去获取资源的时候，它会创建一个`NSURLProtocol`子类的实例，你不应该直接实例化一个`NSURLProtocol`，==NSURLProtocol看起来像是一个协议，但其实这是一个类，而且必须使用该类的子类，并且需要被注册==。**


## 使用场景

不管你是通过`UIWebView`, `NSURLConnection` 或者第三方库 (AFNetworking, MKNetworkKit等)，他们都是基于`NSURLConnection`或者`NSURLSession`实现的，因此你可以通过NSURLProtocol做自定义的操作。

1. 重定向网络请求
2. 忽略网络请求，使用本地缓存
3. 自定义网络请求的返回结果
4. 拦截图片加载请求，转为从本地文件加载
5. 一些全局的网络请求设置
6. 快速进行测试环境的切换
7. 过滤掉一些非法请求
8. 网络的缓存处理（H5离线包 和 网络图片缓存）
9. 可以拦截`UIWebView`，基于系统的`NSURLConnection`或者`NSURLSession`进行封装的网络请求。==目前`WKWebView`无法被`NSURLProtocol`拦截==。

10. 当有多个自定义`NSURLProtocol`注册到系统中的话，会按照他们注册的反向顺序依次调用URL加载流程。当其中有一个NSURLProtocol拦截到请求的话，后续的NSURLProtocol就无法拦截到该请求。

### 拦截网络请求

#### 1.子类化NSURLProtocol并注册
```
@interface CustomURLProtocol : NSURLProtocol

@end
```
然后在`application:didFinishLaunchingWithOptions:`方法中注册该`CustomURLProtocol`，一旦注册完毕后，它就有机会来处理所有交付给URL Loading system的网络请求。
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //注册protocol
    [NSURLProtocol registerClass:[CustomURLProtocol class]];
    return YES;
}
```

#### 2.实现CustomURLProtocol
```
#import "CustomURLProtocol.h"

static NSString* const URLProtocolHandledKey = @"URLProtocolHandledKey";

@interface CustomURLProtocol ()<NSURLConnectionDelegate>
@property (nonatomic, strong) NSURLConnection *connection;

@end
@implementation CustomURLProtocol

//这里面写重写的方法

@end
```

注册好了之后，现在可以开始实现NSURLProtocol的一些方法：

1. `+canInitWithRequest:`

这个方法主要是说明你是否打算处理对应的request，**如果不打算处理，返回NO，URL Loading System会使用系统默认的行为去处理**；**如果打算处理，返回YES，然后你就需要处理该请求的所有东西，包括获取请求数据并返回给 URL Loading System。**

网络数据可以简单的通过`NSURLConnection`去获取，而且每个NSURLProtocol对象都有一个`NSURLProtocolClient`实例，可以通过该client将获取到的数据返回给URL Loading System。

这里有个需要注意的地方，想象一下，当你去加载一个URL资源的时候，URL Loading System会询问CustomURLProtocol是否能处理该请求，你返回YES，然后URL Loading System会创建一个CustomURLProtocol实例然后调用NSURLConnection去获取数据，然而这也会调用URL Loading System，而你在+canInitWithRequest:中又总是返回YES，这样URL Loading System又会创建一个CustomURLProtocol实例导致无限循环。我们应该==保证每个request只被处理一次，可以通过`+setProperty:forKey:inRequest:`标示那些已经处理过的request，然后在`+canInitWithRequest:`中查询该request是否已经处理过了，如果是则返回NO。== 
```
+ (BOOL)canInitWithRequest:(NSURLRequest *)request
{
  //只处理http和https请求 返回NO默认让系统去处理
    NSString *scheme = [[request URL] scheme];
    if ( ([scheme caseInsensitiveCompare:@"http"] == NSOrderedSame ||
     [scheme caseInsensitiveCompare:@"https"] == NSOrderedSame))
    {
        //看看是否已经处理过了，防止无限循环 根据业务来截取 
        if ([NSURLProtocol propertyForKey:URLProtocolHandledKey inRequest:request]) {
            return NO;
        }
         //还要在这里截取DSN解析请求中的链接 判断拦截域名请求的链接如果是返回NO
         if (判断拦截域名请求的链接) {
            return NO;
        }
        return YES;
    }
    return NO;
}
```

2. `+canonicalRequestForRequest` 如果没有特殊需求，直接返回request就可以了, 可以在开始加载中`startLoading`方法中 修改request，比如添加header，修改host,请求重定向等
```
+ (NSURLRequest *) canonicalRequestForRequest:(NSURLRequest *)request {
　　return request;
 }
```

3. `+requestIsCacheEquivalent:toRequest:`
主要判断两个request是否相同，如果相同的话可以使用缓存数据，通常只需要调用父类的实现。
```
+ (BOOL)requestIsCacheEquivalent:(NSURLRequest *)a toRequest:(NSURLRequest *)b
{
    return [super requestIsCacheEquivalent:a toRequest:b];
}
```

3. `-startLoading`、 `-stopLoading`

这两个方法主要是开始和取消相应的request，而且需要标示那些已经处理过的request。
```
- (void)startLoading
{
    NSMutableURLRequest *mutableReqeust = [[self request] mutableCopy];
    //标示改request已经处理过了，防止无限循环
    [NSURLProtocol setProperty:@YES forKey:URLProtocolHandledKey inRequest:mutableReqeust];
    //重定向
    self.connection = [NSURLConnection connectionWithRequest:[self dealDNS: mutableReqeust] delegate:self];
}

- (void)stopLoading
{
    [self.connection cancel];
 　self.connection = nil ;
}

//解决劫持 重定向到ip地址
- (NSMutableURLRequest *)dealDNS:(NSMutableURLRequest *)request{

    if ([request.URL host].length == 0) {
            return request;
        }
    
    NSString *originUrlString = [request.URL absoluteString];
    NSString *originHostString = [request.URL host];
    NSRange hostRange = [originUrlString rangeOfString:originHostString];
    if (hostRange.location == NSNotFound) {
        return request;
    }

     //根据当前host(如baidu.com)请求获取到IP(如172.128.3.3)  并替换到URL中
    //定向到IP 改IP需要根据 host 去请求获取到
    NSString *ip = 根据 host请求获取; // (如baidu.com)同步请求获取到IP(如172.128.3.3)  注意⚠️这个请求的链接需要在canInitWithRequest里面过滤掉
    
    // 替换域名
    //URL：https://www.baidu.com   替换成了https://172.128.3.3
    NSString *urlString = [originUrlString stringByReplacingCharactersInRange:hostRange withString:ip];
    NSURL *url = [NSURL URLWithString:urlString];
    request.URL = url;

     //给改IP设置Cookie
     [CookieManager setWebViewCookieForDomain:ipUrl.host];

    return request;
}
```

4. NSURLConnectionDataDelegate方法

在处理网络请求的时候会调用到该代理方法，我们需要将收到的消息通过client返回给URL Loading System。
```
- (void) connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
}

- (void) connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    [self.client URLProtocol:self didLoadData:data];
}

- (void) connectionDidFinishLoading:(NSURLConnection *)connection {
    [self.client URLProtocolDidFinishLoading:self];
}

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    [self.client URLProtocol:self didFailWithError:error];
}

//解决发送IP地址的HTTPS请求 证书验证
- (void)connection:(NSURLConnection *)connection willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge{

    if (!challenge) {
        return;
    }
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust])
    {
        //构造一个NSURLCredential发送给发起方
        NSURLCredential *credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];

        [[challenge sender] useCredential:credential forAuthenticationChallenge:challenge];

    } else {

       //对于其他验证方法直接进行处理流程
        [[challenge sender] continueWithoutCredentialForAuthenticationChallenge:challenge];
    }
}
```

注意⚠️：这里面在 **手动将URL请求的域名重定向到指定的IP地址** 去请求的时候需要 对HTTPS请求校验证书