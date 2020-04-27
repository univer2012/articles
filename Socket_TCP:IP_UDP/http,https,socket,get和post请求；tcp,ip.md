参考：
1. [浅谈HTTP和HTTPS的区别](https://www.cnblogs.com/zhangbLearn/p/9534002.html)
2. [iOS TCP为什么要三次握手，TCP为什么可靠, TCP原理](https://blog.csdn.net/shihuboke/article/details/79224774)


## 1、HTTP和HTTPS的区别


HTTPS和HTTP的区别主要如下：

　　1、https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。

　　2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。

　　3、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

　　4、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。


## 2、TCP/IP的 连接：三次握手，终止：四次挥手

#### 一、为什么不能两次握手：

答:（防止已失效的连接请求又传送到服务器端，因而产生错误）

假设:改为两次握手，client端发送的一个连接请求在服务器滞留了，这个连接请求是无效的，client已经是closed的状态了，而服务器认为client想要建立一个新的连接，于是向client发送确认报文段，而client端是closed状态，无论收到什么报文都会丢弃。而如果是两次握手的话，此时就已经建立连接了。

服务器此时会一直等到client端发来数据，此时因为client没有发起建立连接请求，所以client处于CLOSED状态，接受到任何包都会丢弃，这样就浪费掉很多server端的资源。

#### 二、为什么三次握手

 1.三次握手的最主要目的是保证连接是双工的，可靠更多的是通过重传机制来保证的。

 

 2.TCP可靠传输的实现：

  （1）TCP 连接的每一端都必须设有两个窗口——一个发送窗口和一个接收窗口。TCP 的可靠传输机制用字节的序号进行控制。TCP 所有的确认都是基于序号而不是基于报文段。

  （2）发送过的数据未收到确认之前必须保留，以便超时重传时使用。发送窗口没收到确认不动，和收到新的确认后前移。

  （3）发送缓存用来暂时存放： 发送应用程序传送给发送方 TCP 准备发送的数据；TCP 已发送出但尚未收到确认的数据。

  （4）接收缓存用来暂时存放：按序到达的、但尚未被接收应用程序读取的数据； 不按序到达的数据。


### 四次挥手
![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%9B%9B%E6%AC%A1%E6%8C%A5%E6%89%8B%E7%A4%BA%E6%84%8F%E5%9B%BE.png)


（1）第一次挥手：**Client发送一个FIN，用来关闭Client到Server的数据传送**，Client进入FIN_WAIT_1状态。

 （2）第二次挥手：**Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1**（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。

 （3）第三次挥手：**Server发送一个FIN，用来关闭Server到Client的数据传送**，Server进入LAST_ACK状态。

 （4）第四次挥手：**Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手**。



## get请求和post请求的区别

参考：
1. [GET和POST本质上有什么区别，这才是标准答案](https://baijiahao.baidu.com/s?id=1620934682611653374&wfr=spider&for=pc)
2. [IOS网络篇:GET和POST的区别](https://blog.csdn.net/hzdg360/article/details/50512726)

- 最直接的区别，GET请求的参数是放在URL里的，POST请求参数是放在请求body里的；
- GET请求的URL传参有长度限制，而POST请求没有长度限制；
- GET请求的参数只能是ASCII码，所以中文需要URL编码，而POST请求传参没有这个限制；

HTTP请求，最初设定了八种方法。这八种方法本质上没有任何区别。只是让请求，更加有语义而已。

- OPTIONS 返回服务器所支持的请求方法
- GET 向服务器获取指定资源
- HEAD 与GET一致，只不过响应体不返回，只返回响应头
- POST 向服务器提交数据，数据放在请求体里
- PUT 与POST相似，只是具有幂等特性，一般用于更新
- DELETE 删除服务器指定资源
- TRACE 回显服务器端收到的请求，测试的时候会用到这个
- CONNECT 预留，暂无使用

答案2：
1. GET 所有的参数都拼接在URL后面 (安全性比POST要差,所有GET登陆请求都会生成日志并且保存到手机里面！)

GET使用场景为:查找数据

2. POST 参数不拼接到URL后面,所有参数都存放在请求体中。

POST使用场景为:更改数据

3. GET 和 POST 主要区别表现在数据的传递上

A.GET在请求URL后面已"?"的形式跟上发送给服务器请求的参数,多个参数之间已"&"符号隔开

B.由于浏览器和服务器对URL的长度有限制,因此一条URL一般不能超过1KB

C.POST发送给服务器的参数全部都放在请求体中。

D.理论上,POST传递的数据量没有限制(具体取决于服务器性能)。

4.选择使用 GET 和 POST 的建议

 A.如果要传递大量数据,如上传文件,只能POST请求

 B.GET的安全性比POST要差一些,如包含密码\其他敏感信息,建议使用POST

 C.如果仅仅是索取数据(数据查找),建议使用GET

 D 如果是增加、修改、删除、等操作，建议使用POST