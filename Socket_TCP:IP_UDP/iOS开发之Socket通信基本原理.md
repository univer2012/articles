参考：
1. [iOS开发之Socket通信基本原理](https://www.jianshu.com/p/cf30e90c8269)
2. [iOS socket原理及连接过程详解](https://www.cnblogs.com/somethingWithiOS/p/5727834.html)



# 1.socket基本概念

在移动开发中，我们在很频繁地和后台接口进行数据通讯，通常是http请求，http是一种无状态的协议，无状态是指Web浏览器和Web服务器之间不需要建立持久的连接，这意味着当一个客户端向服务器端发出请求，然后Web服务器返回响应(response)，连接就被关闭了，在服务器端不保留连接的有关信息，http遵循请求(Request)/应答(Response)模型。Web浏览器向Web服务器发送请求，Web服务器处理请求并返回适当的应答。所有http连接都被构造成一套请求和应答,  服务器和客户端是如何建立http请求连接过程的呢？答案是通过建立TCP连接，即http是基于TCP三次握手进行连接的。

一次完整的HTTP请求过程从TCP三次握手建立连接成功后开始，客户端按照指定的格式开始向服务端发送HTTP请求，服务端接收请求后，解析HTTP请求，处理完业务逻辑，最后返回一个HTTP的响应给客户端，HTTP的响应内容同样有标准的格式。无论是什么客户端或者是什么服务端，大家只要按照HTTP的协议标准来实现的话，那么它一定是通用的。在此文就不对http请求做过多的介绍了，那什么是socket呢？socket起源于Unix，而Unix/Linux基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式来操作。我的理解就是Socket就是该模式的一个实现，socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭) ,这里着重介绍介绍TCP/UDP的socket通信机制和基本原理。

# 2. 基本原理

客户端和服务端建立长连接经过了以下的步骤：
![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E5%BC%80%E5%8F%91%E4%B9%8BSocket%E9%80%9A%E4%BF%A1%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86_1.jpg)

## （1）连接建立步骤

#### 服务器端的步骤是：
1. 创建一个socket，用函数`socket()；`
2. 设置socket属性，用函数`setsockopt();` * 可选
3. 绑定IP地址、端口等信息到socket上，用函数`bind();`
4. 开启监听，用函数`listen()；`
5. 接收客户端上来的连接，用函数`accept()；`
6. 收发数据，用函数send()和`recv()`，或者`read()`和`write();`
7. 关闭网络连接；
8. 关闭监听；


#### 客户端的步骤为：

1. 创建一个socket，用函数socket()；
2. 设置socket属性，用函数setsockopt();* 可选
3. 绑定IP地址、端口等信息到socket上，用函数bind();* 可选
4. 设置要连接的对方的IP地址和端口等属性；
5. 连接服务器，用函数connect()；
6. 收发数据，用函数send()和recv()，或者read()和write();
7. 关闭网络连接；

作为TCP/IP传输协议的一种，客户端和服务端通过TCP建立socket连接的内部原理远比上图的步骤复杂，客户端和服务端是通过何种方式来建立连接的呢 ？

我们知道客户端要和服务端进行连接需要通过3次握手：
1. 客户端向服务器发送一个`SYN J`
2. 服务器向客户端响应一个`SYN K`，并对SYN J进行确认`ACK J+1`
3. 客户端再像服务器发一个确认`ACK K+1`

只有就完了三次握手，但是这个三次握手发生在socket的那几个函数中呢？请看下图：

![2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E5%BC%80%E5%8F%91%E4%B9%8BSocket%E9%80%9A%E4%BF%A1%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86_2.png)

其大致过程类似于拨号打电话:

A打电话给B，B接收，B说喂，A听说到了喂，然后A也说喂，其中A和B都知道了对方是在给自己打电话，相互发送了一个信号表示我可以听到你的来电，即表示收到了信号，连接是可靠的，即可以开始建立连接，来进行详谈了。

从图中可以看出，
1. 当客户端调用connect时，触发了连接请求，向服务器发送了`SYN J`包，这时connect进入阻塞状态；
2. 服务器监听到连接请求(服务端启动之后就在不断地监听连接请求)，即收到`SYN J`包，调用accept函数接收请求向客户端发送`SYN K ，ACK J+1`，这时accept进入阻塞状态；
3. 客户端收到服务器的`SYN K ，ACK J+1`之后，这时connect返回，并对SYN K进行确认；
4. 服务器收到`ACK K+1`时，accept返回，至此三次握手完毕，连接建立。


## （2）断开连接步骤

断开连接，即tcp的四次挥手步骤：

![3](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E5%BC%80%E5%8F%91%E4%B9%8BSocket%E9%80%9A%E4%BF%A1%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86_3.png)

从上图可以看到，客户端或者服务端首先调用close主动关闭连接，这时TCP发送一个FIN M；

另一端接收到FIN M之后，执行被动关闭，对这个FIN进行确认。它的接收也作为文件结束符传递给应用进程，因为FIN的接收意味着应用进程在相应的连接上再也接收不到额外数据；

一段时间之后，接收到文件结束符的应用进程调用close关闭它的socket。这导致它的TCP也发送一个FIN N；

接收到这个FIN的源发送端TCP对它进行确认。

这样每个方向上都有一个FIN和ACK.

与之对应的UDP的连接步骤要简单许多，分别如下：
　　
#### UDP编程的服务器端一般步骤是：
1. 创建一个socket，用函数socket()；
2. 设置socket属性，用函数setsockopt();* 可选
3. 绑定IP地址、端口等信息到socket上，用函数bind();
4. 循环接收数据，用函数recvfrom();
5. 关闭网络连接；

#### UDP编程的客户端一般步骤是：
1. 创建一个socket，用函数socket()；
2. 设置socket属性，用函数setsockopt();* 可选
3. 绑定IP地址、端口等信息到socket上，用函数bind();* 可选
4. 设置对方的IP地址和端口等属性;
5. 发送数据，用函数sendto();
6. 关闭网络连接；

在iOS开发中的socket连接是基于C函数，本文不讲底层的代码实现过程（本人技术有限🤣），本文 使用了大神写的`CocoaAsyncSocket`和`YYNetwork`来模拟了一下客服端和服务端建立的socket长连接：

![4](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E5%BC%80%E5%8F%91%E4%B9%8BSocket%E9%80%9A%E4%BF%A1%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86_4.png)

客户端的代码如下所示：
```
// MARK: - viewControlelr'view's lifeCircle
- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    [_client disconnect];
    [_timer  invalidate];
    _timer = nil;
    NSLog(@"socket断开了连接！");
}

- (void)readData {
    [_client readDataToLength:1000 withTimeout:-1 tag:123];
}

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    _client = [[GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_main_queue()];
    NSError *error = nil;
    //连接服务端 IP和Port
    [_client  connectToHost:@"192.168.0.192"
                     onPort:8080
                      error:&error];
    if (error) {
        NSLog(@"链接服务端失败!");
    } else{
        
    }
    if (@available(iOS 10.0, *)) {
        __weak typeof(self)weakSelf = self;
        _timer = [NSTimer scheduledTimerWithTimeInterval:5.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
            ClientVC *strongSelf = weakSelf;
            //向服务器发送数据
            
            [strongSelf sendDataToServer];
        }];
    } else {
        _timer = [NSTimer scheduledTimerWithTimeInterval:5.0 target:self selector:@selector(sendDataToServer) userInfo:nil repeats:YES];
    }
    [_timer fire];
    [[NSRunLoop currentRunLoop] addTimer:_timer forMode:NSRunLoopCommonModes];
    
    self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"readData" style:UIBarButtonItemStylePlain target:self action:@selector(readData)];
    //监听接收服务端传过来的数据
    [_client readDataWithTimeout:-1 tag:123];
}

-(void)sendDataToServer {
    [_client writeData:[@"HelloWord!" dataUsingEncoding:NSUTF8StringEncoding] withTimeout:1.0 tag:123];
}
// MARK: -GCDAsyncSocketDelegate

- (void)socket:(GCDAsyncSocket *)sock didConnectToHost:(NSString *)host port:(uint16_t)port {
    NSLog(@"连接上了服务端!");
}

// MARK: - 数据接收
- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag {
    NSString *recieveStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    self.navigationItem.title = recieveStr;
    NSLog(@"接收到服务端的数据:%@",recieveStr);
    //监听服务端发送过来的数据
    [_client readDataWithTimeout:-1 tag:123];
    //本地推送一波
    [LocalPushCenter localPushForDate:[NSDate date]
                               forKey:recieveStr
                            alertBody:recieveStr
                          alertAction:recieveStr
                            soundName:nil
                          launchImage:nil
                             userInfo:@{@"info":recieveStr}
                           badgeCount:1
                       repeatInterval:NSCalendarUnitDay];
    
}

- (void)socket:(GCDAsyncSocket *)sock didReadPartialDataOfLength:(NSUInteger)partialLength tag:(long)tag {
}


// MARK: - 数据发送
- (void)socket:(GCDAsyncSocket *)sock didWriteDataWithTag:(long)tag {
    NSLog(@"向服务端发送数据!");
}

// MARK: - 与服务端断开连接
- (void)socketDidDisconnect:(GCDAsyncSocket *)sock withError:(NSError *)err {
    NSLog(@"与服务端断开了!");
}

// MARK: - memory management
-(void)dealloc {
    [_timer invalidate];
    _timer = nil;
    _client.delegate = nil;
    [_client disconnect];
    _client = nil;
}
```