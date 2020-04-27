参考：
1. [UDP 单播、广播和多播 ](https://www.cnblogs.com/jingliming/p/4477264.html)


使用UDP协议进行信息的传输之前不需要建议连接。换句话说就是客户端向服务器发送信息，客户端只需要给出服务器的ip地址和端口号，然后将信息封装到一个待发送的报文中并且发送出去。至于服务器端是否存在，或者能否收到该报文，客户端根本不用管。     

==单播用于两个主机之间的端对端通信==，==广播用于一个主机对整个局域网上所有主机上的数据通信==。单播和广播是两个极端，要么对一个主机进行通信，要么对整个局域网上的主机进行通信。实际情况下，经常需要对一组特定的主机进行通信，而不是整个局域网上的所有主机，这就是多播的用途。

　　通常我们讨论的udp的程序都是一对一的单播程序。本章将讨论一对多的服务：广播（broadcast）、多播（multicast）。对于广播，网络中的所有主机都会接收一份数据副本。对于多播，消息只是发送到一个多播地址，网络知识将数据分发给哪些表示想要接收发送到该多播地址的数据的主机。总得来说，只有UDP套接字允许广播或多播。

# 一、UDP广播

广播UDP与单播UDP的区别就是IP地址不同，广播使用广播地址255.255.255.255，将消息发送到在同一广播网络上的每个主机。值得强调的是：==本地广播信息是不会被路由器转发==。当然这是十分容易理解的，因为如果路由器转发了广播信息，那么势必会引起网络瘫痪。这也是为什么IP协议的设计者故意没有定义互联网范围的广播机制。

广播地址通常用于在网络游戏中处于同一本地网络的玩家之间交流状态信息等。

其实广播顾名思义，就是想局域网内所有的人说话，==但是广播还是要指明接收者的端口号的==，因为不可能接受者的所有端口都来收听广播。

UDP服务端代码：
```
#include<iostream>
#include<stdio.h>
#include<sys/socket.h>
#include<unistd.h>
#include<sys/types.h>
#include<netdb.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<string.h>
using namespace std;
int main()
{
    setvbuf(stdout,NULL,_IONBF,0);
    fflush(stdout);
    int sock=-1;
    if((sock=socket(AF_INET,SOCK_DGRAM,0))==-1)
    {
        cout<<"sock error"<<endl;
        return -1;
    }
    const int opt=-1;
    int nb=0;
    nb=setsockopt(sock,SOL_SOCKET,SO_BROADCAST,(char*)&opt,sizeof(opt));//设置套接字类型
    if(nb==-1)
    {
        cout<<"set socket error...\n"<<endl;
        return -1;
    }
    struct sockaddr_in addrto;
    bzero(&addrto,sizeof(struct sockaddr_in));
    addrto.sin_family=AF_INET;
    addrto.sin_addr.s_addr=htonl(INADDR_BROADCAST);//套接字地址为广播地址
    addrto.sin_port=htons(6000);//套接字广播端口号为6000
    int nlen=sizeof(addrto);
    while(1)
    {
        sleep(1);
        char msg[]={"the message broadcast"};
        int ret=sendto(sock,msg,strlen(msg),0,(sockaddr*)&addrto,nlen);//向广播地址发布消息
        if(ret<0)
        {
            cout<<"send error...\n"<<endl;
            return -1;
        }
        else 
        {
            printf("ok\n");
        }
    }
    return 0;
}
```

UDP广播客户端代码：
```
#include<iostream>
#include<stdio.h>
#include<sys/socket.h>
#include<unistd.h>
#include<sys/types.h>
#include<netdb.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<string.h>


using namespace std;
int main()
{
        setvbuf(stdout,NULL,_IONBF,0);
        fflush(stdout);
        struct sockaddr_in addrto;
        bzero(&addrto,sizeof(struct sockaddr_in));
        addrto.sin_family=AF_INET;
        addrto.sin_addr.s_addr=htonl(INADDR_ANY);
        addrto.sin_port=htons(6000);
        socklen_t len=sizeof(addrto);
        int sock=-1;
        if((sock=socket(AF_INET,SOCK_DGRAM,0))==-1)
        {
                cout<<"socket error..."<<endl;
                return -1;
        }
        const int opt=-1;
        int nb=0;
        nb=setsockopt(sock,SOL_SOCKET,SO_BROADCAST,(char*)&opt,sizeof(opt));
        if(nb==-1)
        {
                cout<<"set socket errror..."<<endl;
                return -1;
        }
        if(bind(sock,(struct sockaddr*)&(addrto),len)==-1)
        {
                cout<<"bind error..."<<endl;
                return -1;
        }
        char msg[100]={0};
        while(1)
        {
                int ret=recvfrom(sock,msg,100,0,(struct sockaddr*)&addrto,&len);
                if(ret<=0)
                {
                        cout<<"read error..."<<endl;
                }
                else
                {
                        printf("%s\t",msg);
                }
                sleep(1);
        }
        return 0;
}
```

# 二、UDP多播
## 1、多播（组播）的概念

多播，也称为“组播”，将网络中同一业务类型主机进行了逻辑上的分组，进行数据收发的时候其数据仅仅在同一分组中进行，其他的主机没有加入此分组不能收发对应的数据。

　　在广域网上广播的时候，其中的交换机和路由器只向需要获取数据的主机复制并转发数据。主机可以向路由器请求加入或退出某个组，网络中的路由器和交换机有选择地复制并传输数据，将数据仅仅传输给组内的主机。多播的这种功能，可以一次将数据发送到多个主机，又能保证不影响其他不需要（未加入组）的主机的其他通 信。

相对于传统的一对一的单播，多播具有如下的==优点==：

　　1、具有同种业务的主机加入同一数据流，共享同一通道，节省了带宽和服务器的优点，具有广播的优点而又没有广播所需要的带宽。

　　2、服务器的总带宽不受客户端带宽的限制。由于组播协议由接收者的需求来确定是否进行数据流的转发，所以服务器端的带宽是常量，与客户端的数量无关。

　　3、与单播一样，多播是允许在广域网即Internet上进行传输的，而广播仅仅在同一局域网上才能进行。

==组播的缺点==：

　　1、多播与单播相比没有纠错机制，当发生错误的时候难以弥补，但是可以在应用层来实现此种功能。

　　2、多播的网络支持存在缺陷，需要路由器及网络协议栈的支持。

　　3、多播的应用主要有网上视频、网上会议等。


## 2、广域网的多播

多播的地址是特定的，D类地址用于多播。D类IP地址就是多播IP地址，即224.0.0.0至239.255.255.255之间的IP地址，并被划分为局部连接多播地址、预留多播地址和管理权限多播地址3类：

　　1、局部多播地址：在224.0.0.0～224.0.0.255之间，这是为路由协议和其他用途保留的地址，路由器并不转发属于此范围的IP包。

　　2、预留多播地址：在224.0.1.0～238.255.255.255之间，可用于全球范围（如Internet）或网络协议。

　　3、管理权限多播地址：在239.0.0.0～239.255.255.255之间，可供组织内部使用，类似于私有IP地址，不能用于Internet，可限制多播范围。

　　多播的程序设计使用setsockopt()函数和getsockopt()函数来实现，组播的选项是IP层的，其选项值和含义参见11.5所示。