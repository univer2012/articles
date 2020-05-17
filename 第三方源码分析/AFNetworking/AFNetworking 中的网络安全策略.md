参考：
1. [AFNetworking 中的网络安全策略](https://blog.csdn.net/bravegogo/article/details/60765674)


# 1. 前言

这一篇的想讲的，一个就是分析一下`AFSecurityPolicy`文件，看看`AFNetworking`的网络安全策略，尤其指HTTPS（大家可以先简单了解下HTTPS）。再一个就是分析下`AFNetworkReachabilityManager`文件，看看AFNetworking如何解决网络状态的检测。

# 2. AFSecurityPolicy - 网络安全策略

 [之前](http://www.cnblogs.com/polobymulberry/p/5140806.html) 我们在`AFURLSessionManager`中实现`了NSURLSessionDelegate`的代理方法`- (void)URLSession:didReceiveChallenge:completionHandler:`，其中我们提到过，如果iOS客户端需要向服务器发送一个凭证(Credential)来确认认证挑战（authentication challenge），那么先得使用`AFNetworking`的安全策略类-`AFSecurityPolicy`来检查服务器端是否可以信任（在authenticationMethod属性`为NSURLAuthenticationMethodServerTrust`情况下），而检查的方式就是通过`- [AFSecurityPolicy evaluateServerTrust:forDomain:]`这个函数。
### 2.1 - [AFSecurityPolicy evaluateServerTrust:forDomain:]

 根据severTrust和domain来检查服务器端发来的证书是否可信

 其中SecTrustRef是一个CoreFoundation类型，用于对服务器端传来的X.509证书评估的


而我们都知道，数字证书的签发机构CA，在接收到申请者的资料后进行核对并确定信息的真实有效，然后就会制作一份符合X.509标准的文件。证书中的证书内容包含的持有者信息和公钥等都是由申请者提供的，而数字签名则是CA机构对证书内容进行hash加密后得到的，而这个数字签名就是我们验证证书是否是有可信CA签发的数据。

```
- (BOOL)evaluateServerTrust:(SecTrustRef)serverTrust forDomain:(nullable NSString *)domain;
```