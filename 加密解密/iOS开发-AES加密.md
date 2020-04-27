来自：[iOS开发-AES加密](https://www.jianshu.com/p/7a28b6ae7481)

# AES加密简介

AES全称`Advanced Encryption Standard`，中文名称叫**高级加密标准**，在密码学中被叫做**Rijndael[ˈraɪndel]加密法**，这个标准已经替代原来的DES，成为美国政府采用的一种区块加密标准。微信小程序中的加密传输就是使用的AES加密算法。

![AES流程图](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

备注：

    明文P：没有经过加密的数据。
    
    密钥K：用来加密明文的密码，在对称加密中，加密和解密的密钥是相同的，所以密钥要保证安全，如果一旦密钥泄漏了，那么数据就基本上不存在安全性了。
    
    AES加密函数：设AES加密函数为E，则
    C = E(K,P)，
    其中K为密钥，C为密文。所以通过加密函数E，可以把明文+密钥生成密文。
    
    密文C：经过加密处理后的数据。
    
    AES解密函数：如加密函数一样，设AES解密函数为D，则P = D（C,P），也就是说，可以通过解密函数D，将密文+密钥生成明文。

# AES的基本结构

**AES为分组密码，分组密码就是把明文分成一组一组的，每组的长度相等，每次加密一组数据，一直到整个明文被加密完毕。**

**在AES标准规范中，分组长度只能是128位，也就是说每个分组为16个字节（每个字节8位）。密钥的长度可以使用128位、192位或256位。**密钥的长度不同，推荐加密的轮数也不同。一般加密的轮数如下：

| AES     | 密钥长度（32位比特字） | 分组长度（32位比特字） | 加密轮数 |
| ------- | ---------------------- | ---------------------- | -------- |
| AES-128 | 4                      | 4                      | 10       |
| AES-192 | 6                      | 4                      | 12       |
| AES-256 | 8                      | 4                      | 14       |

#### 这个加密轮数是什么意思呢？

就比如说上边的加密公式C = E（K,P），如果加密10轮，就是需要执行10次这个函数，这个轮函数的前9次都是相同的，只有第10次时有所不同。

AES处理的单位是字节，128位的输入明文分组P和输入密钥K都被分为16个字节，分别记为P = P0 P1 ... P15 和 K = K0 K1 ... K15，如明文分组P = abcdefghijklmnop，其中的字符a对应P0，p对应P15。

一般地，明文分组用字节为单位的正方形矩阵  描述，称为**状态矩阵**。在算法的每一轮中，状态矩阵的内容不断发生变化，最后的结果作为密文输出。该矩阵的排列顺序为从上到下、从左至右依次排列，如图所示：

![AES流程图2](https://github.com/univer2012/personal-document/blob/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE2.png)

现在假设明文分组P = abcdefghijklmnop，则对应的矩阵图为：

![AES流程图3](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE3.png)

图中使用十六进制表示，即使用0x61表示a。可以看到，经过AES加密之后，得到的结果已经看不出原来的样子。

同样的，128位密钥也是用字节为单位的矩阵表示，矩阵的每一列被称为**1个32位比特字**。通过`密钥编排函数`，该密钥矩阵被扩展成一个44个字组成的序列W[0]，w[1]，...，w[43]，该序列的前4个元素W[0],W[1],W[2],W[3]是原始密钥，用于加密运算中的初始密钥，后边40个字分为10组，每组4个字（128比特）分别用于10轮加密运算中的**轮密钥加**，如下图：

![AES流程图4](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE4.png)

上图中，设K = “abcdefghijklmnop”，则K0 = a, K15 = p, W[0] = K0 K1 K2 K3 = “abcd”。

AES的整体结构如下图所示，其中的W[0,3]是指W[0]、W[1]、W[2]和W[3]串联组成的128位密钥。加密的第1轮到第9轮的轮函数一样，包括4个操作：字节代换、行位移、列混合和轮密钥加。最后一轮迭代不执行列混合。另外，在第一轮迭代之前，先将明文和原始密钥进行一次异或加密操作。

![AES流程图5](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE5.png)

上图为AES加密过程和解密过程，解密过程也为10轮，每一轮操作都是之前加密操作的逆操作。由于AES的4个轮操作都是可逆的，因此，解密的操作的第一轮就是顺序执行逆位移位，逆字节代换，轮密钥加和逆列混合。

# AES*算法流程

AES加密算法主要有4个操作：

    字节替代（SubBytes）
    行位移（ShiftRows）
    列混淆（MixColumns）
    轮密钥加（AddRoundKey）

通过之前的介绍，我们也可以了解到，加密和解密是一个相互可逆的过程。而且所有的顺序刚好是相反的。

### 1、字节替代

AES的字节替代实际上就是一个简单的查表操作，AES定义了一个**S盒**和一个**逆S盒**。

#### 1.1 正向字节替代

S盒
![iOS开发_AES加密_S盒](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E5%BC%80%E5%8F%91_AES%E5%8A%A0%E5%AF%86_S%E7%9B%92.png)

状态矩阵中的元素按照下面的方式映射为一个新的字节：把该字节的高4位作为行值，低4位作为列值，取出S盒或者逆S盒中对应的行的元素作为输出。

例如，加密时，输出的字节S1为0x12,则查S盒的第0x01行和0x02列，得到值0xc9,然后替换S1原有的0x12为0xc9。状态矩阵经字节代换后的图如下：

![AES流程图6](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE6.png)

#### 1.2 逆向字节替代
逆向字节替代也就是查询逆S盒来变换，**逆S盒**如下：

![iOS开发_AES加密_逆S盒](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/iOS%E5%BC%80%E5%8F%91_AES%E5%8A%A0%E5%AF%86_%E9%80%86S%E7%9B%92.png)

### 2、行位移

#### 2.1 正向行位移
**行移位是一个简单的左循环移位操作。**当密钥长度为128比特时，状态矩阵的第0行左移0字节，第1行左移1字节，第2行左移2字节，第3行左移3字节，如下图所示：

![AES流程图7](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE7.png)

#### 2.2 逆向行位移
**行移位的逆变换是将状态矩阵中的每一行执行相反的移位操作**，例如AES-128中，状态矩阵的第0行右移0字节，第1行右移1字节，第2行右移2字节，第3行右移3字节。


### 3、列混淆
#### 3.1 正向列混淆

列混合变换是通过矩阵相乘来实现的，经行移位后的状态矩阵与固定的矩阵相乘，得到混淆后的状态矩阵，如下图的公式所示：

![AES流程图8](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE8.png)

状态矩阵中的第j列(0 ≤j≤3)的列混合可以表示为下图所示：
![AES流程图9](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/AES%E6%B5%81%E7%A8%8B%E5%9B%BE9.png)

其中，矩阵元素的乘法和加法都是定义在基于GF(2^8)上的二元运算,并不是通常意义上的乘法和加法。这里涉及到一些信息安全上的数学知识，不过不懂这些知识也行。其实这种二元运算的加法等价于两个字节的异或，乘法则复杂一点。对于一个8位的二进制数来说，使用域上的乘法乘以(00000010)等价于左移1位(低位补0)后，再根据情况同(00011011)进行异或运算，设S1 = (a7 a6 a5 a4 a3 a2 a1 a0)，刚0x02 * S1如下图所示：

![AES流程图10]()

也就是说，如果a7为1，则进行异或运算，否则不进行。

类似地，乘以(00000100)可以拆分成两次乘以(00000010)的运算：

![AES流程图11]()

乘以(0000 0011)可以拆分成先分别乘以(0000 0001)和(0000 0010)，再将两个乘积异或：

![AES流程图12]()

### 3.2 逆向列混淆
逆向列混合变换可由下图的矩阵乘法定义：

![AES流程图13]()

可以验证，逆变换矩阵同正变换矩阵的乘积恰好为单位矩阵。

## 4、轮密钥加

**轮密钥加是将128位轮密钥Ki同状态矩阵中的数据进行逐位异或操作**。

如下图所示。其中，密钥Ki中每个字W[4i],W[4i+1],W[4i+2],W[4i+3]为32位比特字，包含4个字节，他们的生成算法下面在下面介绍。轮密钥加过程可以看成是字逐位异或的结果，也可以看成字节级别或者位级别的操作。也就是说，可以看成S0 S1 S2 S3 组成的32位字与W[4i]的异或运算。

![AES流程图14]()

轮密钥加的逆运算同正向的轮密钥加运算完全一致，这是因为异或的逆操作是其自身。轮密钥加非常简单，但却能够影响S数组中的每一位。

# iOS中的*AES

在iOS中使用AES进行加密解密的一个关键的函数就是：
```
CCCryptorStatus CCCrypt(

 CCOperation op,  /* kCCEncrypt, etc. */

 CCAlgorithm alg, /* kCCAlgorithmAES128, etc. */

 CCOptions options, /* kCCOptionPKCS7Padding, etc. */

 const void *key,

 size_t keyLength,

 const void *iv,  /* optional initialization vector */

 const void *dataIn,  /* optional per op and alg */

 size_t dataInLength,

 void *dataOut, /* data RETURNED here */

 size_t dataOutAvailable,

 size_t *dataOutMoved)

 __OSX_AVAILABLE_STARTING(__MAC_10_4, __IPHONE_2_0);
```

我们首先来介绍一下这么一大堆参数都是什么用：

- `CCOperation op`：用来代表加密或者解密，kCCEncrypt = 加密，kCCDecrypt = 解密；
- `CCAlgorithm alg`：用来代表加密算法，有kCCAlgorithmAES128...具体的可以cmd+ctl点击进去看；
- `CCOptions options`：填充模式，iOS中只提供了kCCOptionPKCS7Padding和kCCOptionECBMode两种，这个在于后台和安卓交互时要注意一点。
- `const void *key`：密钥长度，一般使用char keyPtr[kCCKeySizeAES256+1];，
- `const void *dataIn`：加密信息的比特数
- `size_t dataInLength`：加密信息的长度
- `void *dataOut`：用来输出加密结果
- `size_t dataOutAvailable`：输出的大小

了解了这个函数大概的工作方式之后，我们就可以开始撸起袖子写代码了。

这里我们主要对NSData和NSString进行加密解密。其实实际上NSString也是转换为了NSData，然后再进行加密解密的。


### 1、引入框架
第一步，重中之重的一步就是引入框架了
```
#import <CommonCrypto/CommonCrypto.h>
#import <CommonCrypto/CommonDigest.h>
```