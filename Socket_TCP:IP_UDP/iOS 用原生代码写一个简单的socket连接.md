来自：[iOS 用原生代码写一个简单的socket连接](https://www.jianshu.com/p/a019b582204a)



### socket简介(摘取自百度百科)

##### 描述

网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。
 建立网络通信连接至少要一对端口号(socket)。

**socket本质**是编程接口(API)，对TCP/IP的封装，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口；HTTP是轿车，提供了封装或者显示数据的具体形式；Socket是发动机，提供了网络通信的能力。

在连接成功时，应用程序两端都会产生一个Socket实例，操作这个实例，完成所需的会话。对于一个网络连接来说，套接字是平等的，并没有差别，不因为在服务器端或在客户端而产生不同级别。不管是Socket还是ServerSocket它们的工作都是通过SocketImpl类及其子类完成的。



![img](https:////upload-images.jianshu.io/upload_images/2395731-f62d174f01503cd6.png?imageMogr2/auto-orient/strip|imageView2/2/w/771)

##### 连接过程

根据连接启动的方式以及本地[套接字](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%A5%97%E6%8E%A5%E5%AD%97)要连接的目标，套接字之间的连接过程可以分为三个步骤：[服务器](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E6%9C%8D%E5%8A%A1%E5%99%A8)监听，客户端请求，连接确认。

（1）服务器监听：是服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态。

（2）客户端请求：是指由客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和[端口号](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E7%AB%AF%E5%8F%A3%E5%8F%B7)，然后就向服务器端套接字提出连接请求。

（3）连接确认：是指当服务器端套接字监听到或者说接收到客户端套接字的连接请求，它就响应客户端套接字的请求，建立一个新的线程，把[服务器](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E6%9C%8D%E5%8A%A1%E5%99%A8)端套接字的描述发给客户端，一旦客户端确认了此描述，连接就建立好了。而服务器端[套接字](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%A5%97%E6%8E%A5%E5%AD%97)继续处于[监听状态](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E7%9B%91%E5%90%AC%E7%8A%B6%E6%80%81)，继续接收其他客户端套接字的连接请求。

![img](https:////upload-images.jianshu.io/upload_images/2395731-047d42fdde9547f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/663)



### iOS写一个原生socket

##### 1. 导入头文件以及宏定义

```objc
#import <sys/socket.h>
#import <netinet/in.h>
#import <arpa/inet.h>

//htons : 将一个无符号短整型的主机数值转换为网络字节顺序，不同cpu 是不同的顺序 (big-endian大尾顺序 , little-endian小尾顺序)
#define SocketPort htons(8040) //端口
//inet_addr是一个计算机函数，功能是将一个点分十进制的IP转换成一个长整数型数
#define SocketIP   inet_addr("127.0.0.1") // ip

```

##### 2. 创建socket

函数原型：

```
int socket(int domain, int type, int protocol);
```

函数使用：

```objc
//属性，用于接收socket创建成功后的返回值
@property (nonatomic, assign) int clinenId;

_clinenId = socket(AF_INET, SOCK_STREAM, 0);
    
if (_clinenId == -1) {
    NSLog(@"创建socket 失败");
    return;
}
```

参数说明：
 **domain**：协议域，又称协议族（family）。常用的协议族有*AF_INET(ipv4)、AF_INET6(ipv6)、*AF_LOCAL（或称AF_UNIX，Unix域Socket）、AF_ROUTE等。协议族决定了socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用ipv4地址（32位的）与端口号（16位的）的组合、AF_UNIX决定了要用一个绝对路径名作为地址。

**type**：指定Socket类型。常用的socket类型有SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等。流式Socket（SOCK_STREAM）是一种面向连接的Socket，针对于面向连接的TCP服务应用。数据报式Socket（SOCK_DGRAM）是一种无连接的Socket，对应于无连接的[UDP](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FUDP)服务应用。

**protocol**：指定协议。常用协议有IPPROTO_TCP、IPPROTO_UDP、IPPROTO_STCP、IPPROTO_TIPC等，分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议。

**注意：**type和protocol不可以随意组合，如SOCK_STREAM不可以跟IPPROTO_UDP组合。当第三个参数为0时，会自动选择第二个参数类型对应的默认协议。

**返回值：**如果调用成功就返回新创建的[套接字](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%A5%97%E6%8E%A5%E5%AD%97)的描述符，如果失败就返回INVALID_SOCKET（Linux下失败返回-1）。套接字描述符是一个整数类型的值。

##### 3. 创建连接

函数原型：

```
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

函数使用：
 3.1 创建socketAddr

```objc
/**
     __uint8_t    sin_len;          假如没有这个成员，其所占的一个字节被并入到sin_family成员中
     sa_family_t    sin_family;     一般来说AF_INET（地址族）PF_INET（协议族）
     in_port_t    sin_port;         // 端口
     struct    in_addr sin_addr;    // ip
     char        sin_zero[8];       没有实际意义,只是为了　跟SOCKADDR结构在内存中对齐
     */
    struct sockaddr_in socketAddr;
    socketAddr.sin_family   = AF_INET;//当前这个是ipv4
    socketAddr.sin_port     = SocketPort; //这里定义了一个宏
    
    struct in_addr  socketIn_addr;
    socketIn_addr.s_addr    = SocketIP; // 也是宏

    socketAddr.sin_addr     = socketIn_addr;
```

3.2 连接

```objc
int result = connect(_clinenId, (const struct sockaddr *)&socketAddr, sizeof(socketAddr));
    if (result != 0) {
        NSLog(@"连接socket 失败");
        return;
    }
    NSLog(@"连接成功");
```

参数说明：
 **sockfd**：标识一个已连接[套接口](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%A5%97%E6%8E%A5%E5%8F%A3)的描述字，就是我们刚刚创建的那个_clinenId。
 **addr**：[指针](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E6%8C%87%E9%92%88)，指向目的套接字的地址。
 **addrlen**：接收返回地址的缓冲区长度。
 **返回值：**成功则返回0，失败返回非0，错误码GetLastError()。

##### 4. 发送消息

函数原型：

```
ssize_t send(int sockfd, const void *buff, size_t nbytes, int flags);
```

函数使用：

```objc
	  const char *msg = @"消息内容".UTF8String;
    //send() 等同于 write() 多提供了一个参数来控制读写操作
    ssize_t sendLen = send(self.clinenId, msg, strlen(msg), 0);
    NSLog(@"发送了:%ld字节",sendLen);
```

参数说明：
 **sockfd：**一个用于标识已连接套接口的描述字。
 **buff：**包含待发送数据的缓冲区。
 **nbytes：**缓冲区中数据的长度。
 **flags：**调用执行方式。
 **返回值：**如果成功，则返回发送的字节数，失败则返回SOCKET_ERROR，一个中文UTF8 编码对应 3 个字节。所以上面发送了3*4字节。

##### 5. 接收数据

函数原型：

```
ssize_t recv(int sockfd, void *buff, size_t nbytes, int flags);
```

函数使用：

```objc
    uint8_t buffer[1024];
    //recv() 等同于 read() 多提供了一个参数来控制读写操作
    ssize_t recvLen = recv(self.clinenId, buffer, sizeof(buffer), 0);
    NSLog(@"接收到了:%ld字节",recvLen);
    if (recvLen==0) {
        NSLog(@"此次传输长度为0 如果下次还为0 请检查连接");
    }
    // 接收到的数据转换
    NSData *recvData  = [NSData dataWithBytes:buffer length:recvLen];
    NSString *recvStr = [[NSString alloc] initWithData:recvData encoding:NSUTF8StringEncoding];
    NSLog(@"%@",recvStr);
```

参数说明:
 **sockfd：**一个用于标识已连接套接口的描述字。
 **buff：**包含待发送数据的缓冲区。
 **nbytes：**缓冲区中数据的长度。
 **flags：**调用执行方式。
 **返回值：**如果成功，则返回读入的字节数，失败则返回SOCKET_ERROR。

### 完整代码

```objc
#import "SGHNativeSocketViewController.h"

#import <sys/socket.h>
#import <netinet/in.h>
#import <arpa/inet.h>

//htons : 将一个无符号短整型的主机数值转换为网络字节顺序，不同cpu 是不同的顺序 (big-endian大尾顺序 , little-endian小尾顺序)
#define SocketPort htons(8040) //端口
//inet_addr是一个计算机函数，功能是将一个点分十进制的IP转换成一个长整数型数
#define SocketIP   inet_addr("127.0.0.1") // ip

@interface SGHNativeSocketViewController ()

//属性，用于接收socket创建成功后的返回值
@property (nonatomic, assign) int clinenId;
@property (weak, nonatomic) IBOutlet UITextField *sendMsgContent_tf;

@end

@implementation SGHNativeSocketViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
}

#pragma mark - 创建socket建立连接
- (IBAction)socketConnetAction:(UIButton *)sender {
    
    // 1: 创建socket
    int socketID = socket(AF_INET, SOCK_STREAM, 0);
    self.clinenId= socketID;
    if (socketID == -1) {
        NSLog(@"创建socket 失败");
        return;
    }
    
    // 2: 连接socket
    struct sockaddr_in socketAddr;
    socketAddr.sin_family = AF_INET;
    socketAddr.sin_port   = SocketPort;
    struct in_addr socketIn_addr;
    socketIn_addr.s_addr  = SocketIP;
    socketAddr.sin_addr   = socketIn_addr;
    
    int result = connect(socketID, (const struct sockaddr *)&socketAddr, sizeof(socketAddr));

    if (result != 0) {
        NSLog(@"连接失败");
        return;
    }
    NSLog(@"连接成功");
    
    // 调用开始接受信息的方法
    // while 如果主线程会造成堵塞
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self recvMsg];
    });
    
}


#pragma mark - 发送消息

- (IBAction)sendMsgAction:(id)sender {
    //3: 发送消息
    if (self.sendMsgContent_tf.text.length == 0) {
        return;
    }
    const char *msg = self.sendMsgContent_tf.text.UTF8String;
    ssize_t sendLen = send(self.clinenId, msg, strlen(msg), 0);
    NSLog(@"发送 %ld 字节",sendLen);
}

#pragma mark - 接受数据
- (void)recvMsg{
    // 4. 接收数据
    while (1) {
        uint8_t buffer[1024];
        ssize_t recvLen = recv(self.clinenId, buffer, sizeof(buffer), 0);
        NSLog(@"接收到了:%ld字节",recvLen);
        if (recvLen == 0) {
            continue;
        }
        // buffer -> data -> string
        NSData *data = [NSData dataWithBytes:buffer length:recvLen];
        NSString *str= [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"%@---%@",[NSThread currentThread],str);
    }
}

@end
```



### 调试

首先 运行效果



![img](https:////upload-images.jianshu.io/upload_images/2395731-f9d911c4959b5e26.png?imageMogr2/auto-orient/strip|imageView2/2/w/365)



##### 方式一，简单快捷

1. 先打开命令行工具，输入 [`nc -lk 8040`](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fqq_29499107%2Farticle%2Fdetails%2F82384393)

2. 点击「连接socket」按钮

   ![img](https:////upload-images.jianshu.io/upload_images/2395731-a258ade26588ea67.png?imageMogr2/auto-orient/strip|imageView2/2/w/380)

   

3. 命令行工具随便输入字符 回车

   ![img](https:////upload-images.jianshu.io/upload_images/2395731-ccd721c86199af90.png?imageMogr2/auto-orient/strip|imageView2/2/w/646)

   

   4.模拟器随便输入字符，在点击「发送」：

   ![img](https:////upload-images.jianshu.io/upload_images/2395731-f1c56533a681d1d7.png?imageMogr2/auto-orient/strip|imageView2/2/w/563)

   

##### 方式二 自己写一个本地socket服务端

听起来好想很牛逼，其实跟上面写客户端差不多。
 多了bind()，listen()，accept()三步。

头文件、宏、属性

```objc
#import <sys/socket.h>
#import <netinet/in.h>
#import <arpa/inet.h>

#define SocketPort htons(8040)
#define SocketIP   inet_addr("127.0.0.1")

@property (nonatomic, assign) int serverId;
@property (nonatomic, assign) int client_socket;

```

1. socket(）

```objc
	  self.serverId = socket(AF_INET, SOCK_STREAM, 0);
    if (self.serverId == -1) {
        NSLog(@"创建socket 失败");
        return;
    }
    NSLog(@"创建socket 成功");
```

2. bind()

```objc
    struct sockaddr_in socketAddr;
    socketAddr.sin_family   = AF_INET;
    socketAddr.sin_port     = SocketPort;
    struct in_addr  socketIn_addr;
    socketIn_addr.s_addr    = SocketIP;
    socketAddr.sin_addr     = socketIn_addr;
    bzero(&(socketAddr.sin_zero), 8);
    
    // 2: 绑定socket
    int bind_result = bind(self.serverId, (const struct sockaddr *)&socketAddr, sizeof(socketAddr));
    if (bind_result == -1) {
        NSLog(@"绑定socket 失败");
        return;
    }

    NSLog(@"绑定socket成功");
```

3. listen()

```objc
    // 3: 监听socket
    int listen_result = listen(self.serverId, kMaxConnectCount);
    if (listen_result == -1) {
        NSLog(@"监听失败");
        return;
    }
    
    NSLog(@"监听成功");
```

4. accept()

```objc
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        struct sockaddr_in client_address;
        socklen_t address_len;
        // accept函数
        int client_socket = accept(self.serverId, (struct sockaddr *)&client_address, &address_len);
        self.client_socket = client_socket;
        
        if (client_socket == -1) {
            NSLog(@"接受 %u 客户端错误",address_len);           
        }else{
            NSString *acceptInfo = [NSString stringWithFormat:@"客户端 in,socket:%d",client_socket];
            NSLog(@"%@",acceptInfo);
           //开始接受消息
            [self receiveMsgWithClietnSocket:client_socket];
        }
    });
```

5. recv()

```objc
while (1) {
        // 5: 接受客户端传来的数据
        char buf[1024] = {0};
        long iReturn = recv(clientSocket, buf, 1024, 0);
        if (iReturn>0) {
            NSLog(@"客户端来消息了");
            // 接收到的数据转换
            NSData *recvData  = [NSData dataWithBytes:buf length:iReturn];
            NSString *recvStr = [[NSString alloc] initWithData:recvData encoding:NSUTF8StringEncoding];
            NSLog(@"%@",recvStr);
            
        }else if (iReturn == -1){
            NSLog(@"读取消息失败");
            break;
        }else if (iReturn == 0){
            NSLog(@"客户端走了");
            
            close(clientSocket);
            
            break;
        }
    }
```

6. send()

```objc
    const char *msg = @"给客户端发消息".UTF8String;
    ssize_t sendLen = send(self.client_socket, msg, strlen(msg), 0);
    NSLog(@"发送了:%ld字节",sendLen);
```

7. close()

```objc
    int close_result = close(self.client_socket);
    
    if (close_result == -1) {
        NSLog(@"socket 关闭失败");
        return;
    }else{
        NSLog(@"socket 关闭成功");
    }
```

那么这篇文章就到这里，写这篇文章的主要目的是为了让后面学习[GCDAsyncSocket](https://www.jianshu.com/p/539876f5983c)时，加深印象、深入理解其实现原理。



---

【完】

