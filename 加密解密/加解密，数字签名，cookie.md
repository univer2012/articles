参考：
1. [iOS - 加解密（对称，非对称）(AES DES base64这里都有)，数字签名，cookie](http://www.cocoachina.com/ios/20151231/14835.html)

首先罗列一些知识点：

### 1. 加密算法通常分为==对称性加密算法==和==非对称性加密算法==：

对于对称性加密算法，信息接收双方都需事先知道密匙和加解密算法且其密匙是相同的，之后便是对数据进行加解密了。

非对称算法与之不同，++发送双方A,B事先均生成一对密匙，然后A将自己的公有密匙发送给B，B将自己的公有密匙发送给A++。==**如果A要给B发送消息，则先需要用B的公有密匙进行消息加密，然后发送给B端，此时B端再用自己的私有密匙进行消息解密**==，B向A发送消息时为同样的道理。


### 2. 公钥私钥和数字签名
关于公钥私钥和数字签名， 通过一个发送邮件的故事让大家有一个深刻的理解，非常棒的案例：

https://blog.csdn.net/baidu_36327010/article/details/78659665

看完这个之后， 相信你会明白非对称加密在网络传输中的安全性的体现， 当然就是之前谈到的https。

总而言之，公钥与私钥的作用是：**==用公钥加密的内容只能用私钥解密，用私钥加密的内容只能 用公钥解密==**。**++公钥加密私钥解密++**， 没问题,也可以说是"**++公共密钥加密系统++**"。**++私钥加密公钥解密++**,一般不这么说，应叫"**++私钥签名,公钥验证++**",也可以说是“**++公共密钥签名系统++**”

引用一段总结的话：

公钥加密私钥解密，没问题,也可以说是"公共密钥加密系统"。私钥加密公钥解密,一般不这么说，应叫"私钥签名,公钥验证",也可以说是“公共密钥签名系统”。

再来说一下，"公共密钥签名系统"目的:

(如果晕就多看几遍，这个没搞清，后面的代码就更晕)

A欲传（信息）给B，但又怕B不确信该信息是A发的。
1. A选计算（信息）的HASH值，如用MD5方式计算,得到：[MD5（信息）]
2. 然后用自已的私钥加密HASH值，得到：[私钥（MD5（信息））]
3. 最后将信息与密文一起传给B，传给B：[（信息） + 私钥（MD5（信息））]。

B接到 ：[（信息） + 私钥（MD5（信息））]

1. 先用相同的HASH算法算出（信息）的HASH值，这里也使用MD5方式 得到： [MD5（信息）!]

2. 再用A的公钥解密 [ 私钥（MD5（信息））] [公钥(私钥（MD5（信息））)] = [（MD5（信息）] 如能解开,证明该 [ 私钥（MD5（信息））]是A发送的

3. 再比效[MD5（信息）!]与[（MD5（信息）] 如果相同,表示（信息）在传递过程中没有被他人修改过

OK, 到现在为止， 你已经懂得了公钥， 私钥， 以及数字证书的概念了， 当然你也知道什么是对称加密和非对称加密，有可能你不是很清楚，为了让你更清楚，给你再讲个活生生的例子，这个例子还要从我的恋爱说起， 高中的时候喜欢上一个女生， 那时候青春年少，还喜欢用纸质给她写情书， 每天写一些“透明的”文字很繁琐，于是有一天，我忽然一个念想，把情书改成用汉语拼音写abcd……xyz, 原来字母是a就用z代替，b用y代替，c用x代替，……z用a代替， 这样，一个只有我们俩能看的懂的情书就这样诞生了。其实现在想想， 这不正是一种**对称式加密**么。哈哈。

说完了故事，再来普及下一点简单的知识喽

### 3. 几种对称性加密算法：AES,DES,3DES

==DES是一种分组数据加密技术（先将数据分成固定长度的小数据块，之后进行加密），速度较快，适用于大量数据加密==，而**3DES是一种基于DES的加密算法，使用3个不同密匙对同一个分组数据块进行3次加密，如此以使得密文强度更高**。

相较于DES和3DES算法而言，AES算法有着更高的速度和资源使用效率，安全级别也较之更高了，被称为下一代加密标准。对于具体的算法我们不做深入的了解， 之前有一篇文章写得很好， 由于时间问题， 我就不给大家找了。

### 4.几种非对称性加密算法：RSA,DSA,ECC
RSA和DSA的安全性及其它各方面性能都差不多，而==ECC较之则有着很多的性能优越，包括处理速度，带宽要求，存储空间等等==

### 5.几种线性散列算法（签名算法）：MD5,SHA1,HMAC

这几种算法==只生成一串不可逆的密文，经常用其效验数据传输过程中是否经过修改==。因为相同的生成算法对于同一明文只会生成唯一的密文，若相同算法生成的密文不同，则证明传输数据进行过了修改。

通常在数据传输过程前，**使用MD5和SHA1算法，均需要发送和接收数据双方，在数据传送之前就知道密匙生成算法**。而**HMAC**与之不同的是，**需要生成一个密匙，发送方用此密匙对数据进行摘要处理（生成密文），接收方再利用此密匙对接收到的数据进行摘要处理，再判断生成的密文是否相同。**

### 6.对于各种加密算法的选用
由于对称加密算法的密钥管理是一个复杂的过程，密钥的管理直接决定着他的安全性，因此==当数据量很小时，我们可以考虑采用非对称加密算法。==

在实际的操作过程中，我们通常采用的方式是：==采用非对称加密算法管理对称算法的密钥，然后用对称加密算法加密数据，这样我们就集成了两类加密算法的优点，既实现了加密速度快的优点，又实现了安全方便管理密钥的优点==。

如果在选定了加密算法后，那采用多少位的密钥呢？**一般来说，密钥越长，运行的速度就越慢，应该根据的我们实际需要的安全级别来选择**，一般来说，==RSA建议采用1024位的数字，ECC建议采用160位，AES采用128为即可==。

需要注意的是：

**哈希函数,比如MD5,SHA，这些都不是加密算法**。要注意他们的区别和用途，很多网友都把md5说成是加密算法，这是严重不正确的啊。

==哈希函数：MD5，SHA 是没有密钥的，相当于指纹的概念，因此也是不可逆的==； md5是128位的，SHA有不同的算法，有128，256等位。。。如SHA-256,SHA-384。然后 就是 Base64,这更加不属于加密算法的范围了，它只是将`byte[]`数组进行了转换。

为什么要转换呢？就是因为很多加密后的密文，或者一些特殊的`byte[]`数组需要显示出来，或者需要进行传递（电子邮件），但是直接转换就会导致很多不可显示的字符，会丢失一些信息，因此就转换位Base64编码，这些都是可显示的字符。所以转换后，长度会增加。它是可逆的。 

再就是 3DES,DES 这才是加密算法，因此也是可逆的，加解密需要密钥,也就是你说的key。最后是 RSA ,这是公钥密码，也就是加密和解密密钥不同，也是可逆的。

罗列了这么多知识点， 我想这篇文章你应该有收藏的必要了吧，为了更形象，更好玩， 我从网上找了一些在线工具：
### 在线工具

#### 1.1 Base64

http://tool.oschina.net/encrypt?type=3
![1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%8A%A0%E8%A7%A3%E5%AF%86%EF%BC%8C%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%EF%BC%8Ccookie_1.png)

可以看出来这是可逆的编解码，注意不是加密幺！！！

#### 1.2 MD5
http://tool.chinaz.com/Tools/MD5.aspx?q=32324&md5type=1

![2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%8A%A0%E8%A7%A3%E5%AF%86%EF%BC%8C%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%EF%BC%8Ccookie_2.png)

其实这个也不能叫加密工具幺!!

#### 1.3 SHA-1,SHA-2,SHA-256,SHA-512,SHA-3

好吧，哈希的工具找到了一个更好的工具连接，里面也有MD5.里面还有哈希的一些说明。

http://www.atool88.com/hash.php  这个链接值的收藏一下， 主要是用到哈希的时候可以经常用。

![3](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%8A%A0%E8%A7%A3%E5%AF%86%EF%BC%8C%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%EF%BC%8Ccookie_3.png)

#### 2.1 AES DES
以上我们主要说了哈希算法和数字证书的一些知识， 现在我们看一下对称加密的一些在线工具

DES：http://e-file.arkoo.com/tools/des3.htm

这个链接可谓非常干净好用，形象直观可逆过程，哈哈。

![4](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%8A%A0%E8%A7%A3%E5%AF%86%EF%BC%8C%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%EF%BC%8Ccookie_4.png)

AES：http://www.seacha.com/tools/aes.html

![5](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%8A%A0%E8%A7%A3%E5%AF%86%EF%BC%8C%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%EF%BC%8Ccookie_5.png)


忽然发现这个在线工具也还凑合：http://encode.chahuo.com/

![6](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%8A%A0%E8%A7%A3%E5%AF%86%EF%BC%8C%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%EF%BC%8Ccookie_6.png)

如此般， 对称加密基本上都弄完了，现在我们只简单的了解下**非对称加解密的RSA**,上面的过程我们已经说的非常详细了吧。重点已经用黑色字体标注出来了。


##### 加密数据
```
    RSAEncryptor *rsa = [[RSAEncryptor alloc] init];  
    NSLog(@"encryptor using rsa");  
    NSString *publicKeyPath = [[NSBundle mainBundle] pathForResource:@"public_key" ofType:@"der"];  
    NSLog(@"public key: %@", publicKeyPath);  
    [rsa loadPublicKeyFromFile:publicKeyPath];  
    NSString *securityText = @"hello ~";  
    NSString *encryptedString = [rsa rsaEncryptString:securityText];  
    NSLog(@"encrypted data: %@", encryptedString);
```

##### 解密数据

iOS下解码需要先加载private key, 之后再对数据解码。解码的时候先进行Base64 decode, 之后在用private key decrypt加密数据.
```
    NSLog(@"decryptor using rsa");  
    [rsa loadPrivateKeyFromFile:[[NSBundle mainBundle] pathForResource:@"private_key" ofType:@"p12"] password:@"123456"];  
    NSString *decryptedString = [rsa rsaDecryptString:encryptedString];  
    NSLog(@"decrypted data: %@", decryptedString);
```

具体详细文章可以参考链接：
1. [iOS中使用RSA对数据进行加密解密](https://witcheryne.iteye.com/blog/2171850)
2. [iOS客户端、java服务器的通信用RSA加密](http://www.cocoachina.com/bbs/read.php?tid=166990)

在支付宝支付过程中就使用了RSA加密。

在这里有必要提醒下小编自己， 有时间需要研究下苹果证书的工作机制。

弄到这里， 我主要是找一些代码给大家用，看看我自己先建一个工程吧。
```
#import @interface Helper : NSObject
//MD5
+ (NSString *) md5:(NSString *)str;
//Base64
+ (NSString *)base64StringFromText:(NSString *)text;
+ (NSString *)textFromBase64String:(NSString *)base64;
+ (NSString *)base64EncodedStringFrom:(NSData *)data;
//DES加密
+(NSString *)encryptSting:(NSString *)sText key:(NSString *)key andDesiv:(NSString *)ivDes;
//DES解密
+(NSString *)decryptWithDESString:(NSString *)sText key:(NSString *)key andiV:(NSString *)iv;
//AES加密
+ (NSData *)AES128EncryptWithKey:(NSString *)key iv:(NSString *)iv withNSData:(NSData *)data;
//AES解密
+ (NSData *)AES128DecryptWithKey:(NSString *)key iv:(NSString *)iv withNSData:(NSData *)data;
@end
```

```
#import "Helper.h"
#import #importstatic const char encodingTable[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
#define LocalStr_None  @""
@implementation Helper
//Md5
+ (NSString *) md5:(NSString *)str {
    if (str == nil) {
        return nil;
    }
    const char *cStr = [str UTF8String];
    unsigned char result[16];
    CC_MD5( cStr, strlen(cStr), result );
    return [NSString stringWithFormat:
            @"XXXXXXXXXXXXXXXX",
            result[0], result[1], result[2], result[3],
            result[4], result[5], result[6], result[7],
            result[8], result[9], result[10], result[11],
            result[12], result[13], result[14], result[15]
            ];
}
//转化为Base64
+ (NSString *)base64StringFromText:(NSString *)text
{
    if (text && ![text isEqualToString:LocalStr_None]) {
        //取项目的bundleIdentifier作为KEY  改动了此处
        //NSString *key = [[NSBundle mainBundle] bundleIdentifier];
        NSData *data = [text dataUsingEncoding:NSUTF8StringEncoding];
        //IOS 自带DES加密 Begin  改动了此处
        //data = [self DESEncrypt:data WithKey:key];
        //IOS 自带DES加密 End
        return [self base64EncodedStringFrom:data];
    }
    else {
        return LocalStr_None;
    }
}
//由base64转化
+ (NSString *)textFromBase64String:(NSString *)base64
{
    if (base64 && ![base64 isEqualToString:LocalStr_None]) {
        //取项目的bundleIdentifier作为KEY   改动了此处
        //NSString *key = [[NSBundle mainBundle] bundleIdentifier];
        NSData *data = [self dataWithBase64EncodedString:base64];
        //IOS 自带DES解密 Begin    改动了此处
        //data = [self DESDecrypt:data WithKey:key];
        //IOS 自带DES加密 End
        return [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    }
    else {
        return LocalStr_None;
    }
}
//DES加密
+(NSString *)encryptSting:(NSString *)sText key:(NSString *)key andDesiv:(NSString *)ivDes
{
    if ((sText == nil || sText.length == 0)
        || (sText == nil || sText.length == 0)
        || (ivDes == nil || ivDes.length == 0)) {
        return @"";
    }
    //gb2312 编码
    NSStringEncoding encoding =CFStringConvertEncodingToNSStringEncoding(kCFStringEncodingGB_18030_2000);
    NSData* encryptData = [sText dataUsingEncoding:encoding];
    size_t  dataInLength = [encryptData length];
    const void *   dataIn = (const void *)[encryptData bytes];
    CCCryptorStatus ccStatus = nil;
    uint8_t *dataOut = NULL; //可以理解位type/typedef 的缩写（有效的维护了代码，比如：一个人用int，一个人用long。最好用typedef来定义）
    size_t dataOutMoved = 0;
    size_t    dataOutAvailable = (dataInLength + kCCBlockSizeDES) & ~(kCCBlockSizeDES - 1);  dataOut = malloc( dataOutAvailable * sizeof(uint8_t));
    memset((void *)dataOut, 0x0, dataOutAvailable);//将已开辟内存空间buffer的首 1 个字节的值设为值 0
    const void *iv = (const void *) [ivDes cStringUsingEncoding:NSASCIIStringEncoding];
    //CCCrypt函数 加密/解密
    ccStatus = CCCrypt(kCCEncrypt,//  加密/解密
                       kCCAlgorithmDES,//  加密根据哪个标准（des，3des，aes。。。。）
                       kCCOptionPKCS7Padding,//  选项分组密码算法(des:对每块分组加一次密  3DES：对每块分组加三个不同的密)
                       [key UTF8String],  //密钥    加密和解密的密钥必须一致
                       kCCKeySizeDES,//   DES 密钥的大小（kCCKeySizeDES=8）
                       iv, //  可选的初始矢量
                       dataIn, // 数据的存储单元
                       dataInLength,// 数据的大小
                       (void *)dataOut,// 用于返回数据
                       dataOutAvailable,
                       &dataOutMoved);
    //编码 base64
    NSData *data = [NSData dataWithBytes:(const void *)dataOut length:(NSUInteger)dataOutMoved];
    Byte *bytes = (Byte *)[data bytes];
    //下面是Byte 转换为16进制。
    NSString *hexStr=@"";
    for(int i=0;i<[data length];i++){
        NSString *newHexStr = [NSString stringWithFormat:@"%x",bytes[i]&0xff];///16进制数
        if([newHexStr length]==1)
            hexStr = [NSString stringWithFormat:@"%@0%@",hexStr,newHexStr];
        else
            hexStr = [NSString stringWithFormat:@"%@%@",hexStr,newHexStr];
    }
    free(dataOut);
    return hexStr;
}
//DES解密
+(NSString *)decryptWithDESString:(NSString *)sText key:(NSString *)key andiV:(NSString *)iv
{
    if ((sText == nil || sText.length == 0)
        || (sText == nil || sText.length == 0)
        || (iv == nil || iv.length == 0)) {
        return @"";
    }
    const void *dataIn;
    size_t dataInLength;
    char *myBuffer = (char *)malloc((int)[sText length] / 2 + 1);
    bzero(myBuffer, [sText length] / 2 + 1);
    for (int i = 0; i < [sText length] - 1; i += 2) {
        unsigned int anInt;
        NSString * hexCharStr = [sText substringWithRange:NSMakeRange(i, 2)];
        NSScanner * scanner = [[NSScanner alloc] initWithString:hexCharStr];
        [scanner scanHexInt:&anInt];
        myBuffer[i / 2] = (char)anInt;
    }
    NSData *decryptData =[NSData dataWithBytes:myBuffer length:[sText length] / 2 ];//转成utf-8并decode
    dataInLength = [decryptData length];
    dataIn = [decryptData bytes];
    free(myBuffer);
    CCCryptorStatus ccStatus = nil;
    uint8_t *dataOut = NULL; //可以理解位type/typedef 的缩写（有效的维护了代码，比如：一个人用int，一个人用long。最好用typedef来定义）
    size_t dataOutAvailable = 0; //size_t  是操作符sizeof返回的结果类型
    size_t dataOutMoved = 0;
    dataOutAvailable = (dataInLength + kCCBlockSizeDES) & ~(kCCBlockSizeDES - 1);
    dataOut = malloc( dataOutAvailable * sizeof(uint8_t));
    memset((void *)dataOut, 0x0, dataOutAvailable);//将已开辟内存空间buffer的首 1 个字节的值设为值 0
    const void *ivDes = (const void *) [iv cStringUsingEncoding:NSASCIIStringEncoding];
    //CCCrypt函数 加密/解密
    ccStatus = CCCrypt(kCCDecrypt,//  加密/解密
                       kCCAlgorithmDES,//  加密根据哪个标准（des，3des，aes。。。。）
                       kCCOptionPKCS7Padding,//  选项分组密码算法(des:对每块分组加一次密  3DES：对每块分组加三个不同的密)
                       [key UTF8String],  //密钥    加密和解密的密钥必须一致
                       kCCKeySizeDES,//   DES 密钥的大小（kCCKeySizeDES=8）
                       ivDes, //  可选的初始矢量
                       dataIn, // 数据的存储单元
                       dataInLength,// 数据的大小
                       (void *)dataOut,// 用于返回数据
                       dataOutAvailable,
                       &dataOutMoved);
    NSStringEncoding encoding =CFStringConvertEncodingToNSStringEncoding(kCFStringEncodingGB_18030_2000);
    NSString *result  = [[NSString alloc] initWithData:[NSData dataWithBytes:(const void *)dataOut length:(NSUInteger)dataOutMoved] encoding:encoding];
    free(dataOut);
    return result;
}
//AES加密
+ (NSData *)AES128EncryptWithKey:(NSString *)key iv:(NSString *)iv withNSData:(NSData *)data
{
    char keyPtr[kCCKeySizeAES128+1];
    bzero(keyPtr, sizeof(keyPtr));
    [key getCString:keyPtr maxLength:sizeof(keyPtr) encoding:NSUTF8StringEncoding];
    char ivPtr[kCCKeySizeAES128+1];
    bzero(ivPtr, sizeof(ivPtr));
    [iv getCString:ivPtr maxLength:sizeof(ivPtr) encoding:NSUTF8StringEncoding];
    NSUInteger dataLength = [data length];
    int diff = kCCKeySizeAES128 - (dataLength % kCCKeySizeAES128);
    int newSize = 0;
    if(diff > 0)
    {
        newSize = (int)(dataLength + diff);
    }
    char dataPtr[newSize];
    memcpy(dataPtr, [data bytes], [data length]);
    for(int i = 0; i < diff; i++)
    {
        dataPtr[i + dataLength] = 0x00;
    }
    size_t bufferSize = newSize + kCCBlockSizeAES128;
    void *buffer = malloc(bufferSize);
    size_t numBytesEncrypted = 0;
    CCCryptorStatus cryptStatus = CCCrypt(kCCEncrypt,
                                          kCCAlgorithmAES128,
                                          0x00, //No padding
                                          keyPtr,
                                          kCCKeySizeAES128,
                                          ivPtr,
                                          dataPtr,
                                          sizeof(dataPtr),
                                          buffer,
                                          bufferSize,
                                          &numBytesEncrypted);
    if(cryptStatus == kCCSuccess)
    {
//        NSData *data =[NSData dataWithBytesNoCopy:buffer length:numBytesEncrypted];
//        NSString *str = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        return [NSData dataWithBytesNoCopy:buffer length:numBytesEncrypted];
    }
    return nil;
}
//AES解密
+ (NSData *)AES128DecryptWithKey:(NSString *)key iv:(NSString *)iv withNSData:(NSData *)data
{
    char keyPtr[kCCKeySizeAES128+1];
    bzero(keyPtr, sizeof(keyPtr));
    [key getCString:keyPtr maxLength:sizeof(keyPtr) encoding:NSUTF8StringEncoding];
    char ivPtr[kCCKeySizeAES128+1];
    bzero(ivPtr, sizeof(ivPtr));
    [iv getCString:ivPtr maxLength:sizeof(ivPtr) encoding:NSUTF8StringEncoding];
    NSUInteger dataLength = [data length];
    size_t bufferSize = dataLength + kCCBlockSizeAES128;
    void *buffer = malloc(bufferSize);
    size_t numBytesEncrypted = 0;
    CCCryptorStatus cryptStatus = CCCrypt(kCCDecrypt,
                                          kCCAlgorithmAES128,
                                          0x00, //No padding
                                          keyPtr,
                                          kCCKeySizeAES128,
                                          ivPtr,
                                          [data bytes],
                                          dataLength,
                                          buffer,
                                          bufferSize,
                                          &numBytesEncrypted);
    if(cryptStatus == kCCSuccess)
    {
//        NSData *data =[NSData dataWithBytesNoCopy:buffer length:numBytesEncrypted];
       // NSString *str = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        return [NSData dataWithBytesNoCopy:buffer length:numBytesEncrypted];
    }
    return nil;
}
/******************************************************************************
 函数名称 : + (NSData *)dataWithBase64EncodedString:(NSString *)string
 函数描述 : base64格式字符串转换为文本数据
 输入参数 : (NSString *)string
 输出参数 : N/A
 返回参数 : (NSData *)
 备注信息 :
 ******************************************************************************/
+ (NSData *)dataWithBase64EncodedString:(NSString *)string
{
    if (string == nil)
        [NSException raise:NSInvalidArgumentException format:nil];
    if ([string length] == 0)
        return [NSData data];
    static char *decodingTable = NULL;
    if (decodingTable == NULL)
    {
        decodingTable = malloc(256);
        if (decodingTable == NULL)
            return nil;
        memset(decodingTable, CHAR_MAX, 256);
        NSUInteger i;
        for (i = 0; i < 64; i++)
            decodingTable[(short)encodingTable[i]] = i;
    }
    const char *characters = [string cStringUsingEncoding:NSASCIIStringEncoding];
    if (characters == NULL)     //  Not an ASCII string!
        return nil;
    char *bytes = malloc((([string length] + 3) / 4) * 3);
    if (bytes == NULL)
        return nil;
    NSUInteger length = 0;
    NSUInteger i = 0;
    while (YES)
    {
        char buffer[4];
        short bufferLength;
        for (bufferLength = 0; bufferLength < 4; i++)
        {
            if (characters[i] == '\0')
                break;
            if (isspace(characters[i]) || characters[i] == '=')
                continue;
            buffer[bufferLength] = decodingTable[(short)characters[i]];
            if (buffer[bufferLength++] == CHAR_MAX)      //  Illegal character!
            {
                free(bytes);
                return nil;
            }
        }
        if (bufferLength == 0)
            break;
        if (bufferLength == 1)      //  At least two characters are needed to produce one byte!
        {
            free(bytes);
            return nil;
        }
        //  Decode the characters in the buffer to bytes.
        bytes[length++] = (buffer[0] << 2) | (buffer[1] >> 4);
        if (bufferLength > 2)
            bytes[length++] = (buffer[1] << 4) | (buffer[2] >> 2);
        if (bufferLength > 3)
            bytes[length++] = (buffer[2] << 6) | buffer[3];
    }
    bytes = realloc(bytes, length);
    return [NSData dataWithBytesNoCopy:bytes length:length];
}
/******************************************************************************
 函数名称 : + (NSString *)base64EncodedStringFrom:(NSData *)data
 函数描述 : 文本数据转换为base64格式字符串
 输入参数 : (NSData *)data
 输出参数 : N/A
 返回参数 : (NSString *)
 备注信息 :
 ******************************************************************************/
+ (NSString *)base64EncodedStringFrom:(NSData *)data
{
    if ([data length] == 0)
        return @"";
    char *characters = malloc((([data length] + 2) / 3) * 4);
    if (characters == NULL)
        return nil;
    NSUInteger length = 0;
    NSUInteger i = 0;
    while (i < [data length])
    {
        char buffer[3] = {0,0,0};
        short bufferLength = 0;
        while (bufferLength < 3 && i < [data length])
            buffer[bufferLength++] = ((char *)[data bytes])[i++];
        //  Encode the bytes in the buffer to four characters, including padding "=" characters if necessary.
        characters[length++] = encodingTable[(buffer[0] & 0xFC) >> 2];
        characters[length++] = encodingTable[((buffer[0] & 0x03) << 4) | ((buffer[1] & 0xF0) >> 4)];
        if (bufferLength > 1)
            characters[length++] = encodingTable[((buffer[1] & 0x0F) << 2) | ((buffer[2] & 0xC0) >> 6)];
        else characters[length++] = '=';
        if (bufferLength > 2)
            characters[length++] = encodingTable[buffer[2] & 0x3F];
        else characters[length++] = '=';
    }
    return [[NSString alloc] initWithBytesNoCopy:characters length:length encoding:NSASCIIStringEncoding freeWhenDone:YES];
}
@end
```

以上是接口和实现文件， 现在我们来看看调用
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //MD5
    NSString *md5Str = [Helper md5:@"我爱你"];
    NSLog(@"md5Str is %@",md5Str);//Log is 4F2016C6B934D55BD7120E5D0E62CCE3
    //Base64
    NSString *Base64Str = [Helper base64StringFromText:@"我爱你"];
    NSLog(@"Base64Str is %@",Base64Str);//Log is 5oiR54ix5L2g
    NSString *oriBase64Str = [Helper textFromBase64String:Base64Str];
    NSLog(@"oriBase64Str is %@",oriBase64Str);//Log is  我爱你
    //DES
    NSString *desEnStr = [Helper encryptSting:@"我爱你" key:@"521" andDesiv:@"521"];
    NSLog(@"desEnStr is %@",desEnStr);//Log is  389280aa791ee933
    NSString *desDeStr =[Helper decryptWithDESString:desEnStr key:@"521" andiV:@"521"];
    NSLog(@"desDeStr is %@",desDeStr);//Log is  我爱你
    //AES
    NSData *aesEnData = [Helper AES128EncryptWithKey:@"521" iv:@"521" withNSData:[@"我爱你" dataUsingEncoding:NSUTF8StringEncoding]];
    NSString *aesEnStr = [Helper base64EncodedStringFrom:aesEnData];
    NSLog(@"aesEnStr is %@",aesEnStr);//Log is HZKhnRLlQ8XjMjpelOAwsQ==
    NSData *aesDeData = [Helper AES128DecryptWithKey:@"521" iv:@"521" withNSData:aesEnData];
    NSString *aesDEStr = [Helper base64EncodedStringFrom:aesDeData];
    NSString *result = [Helper textFromBase64String:aesDEStr];
    NSLog(@"aesDEStr is %@ and result is %@",aesDEStr,result);//Log is aesDEStr is 5oiR54ix5L2gAAAAAAAAAA== and result is 我爱你
    return YES;
}
```

写到这里， 产生了一个问题， 就是上面的AES加密最终生成的Base64字符串和工具不一样 ，不知道是什么问题， 还请高手帮忙解答一下。

大家也可以尝试下网上其他人的代码

//AES可以参考一个链接 http://www.tuicool.com/articles/UVRjmyN


好了， 文章的最后，我们来简单学习和了解下cookie在客户端里的应用

首先带大家了解下什么是cookie吧

### 什么是cookie
![7](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/%E5%8A%A0%E8%A7%A3%E5%AF%86%EF%BC%8C%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%EF%BC%8Ccookie_7.png)

这是维基百科里一段对cookie的描述，可见cookie是服务器生成的发给客户端，具体应用例如：

我们打开淘宝的某一个页面，登陆了账号和密码， 当我们再跳转到其他淘宝界面的时候，我们不必每一次都重新登陆界面， 这就是cookie的作用。 其实cookie还能记录用户选的订单， 至于深一层的了解， 我还不是很清楚， 因为没有学过前端的开发，感兴趣的读者可以自行了解。

其实cookie也可以在客户端使用的。`NSHTTPCookieStorage`在iOS上是一个单例。那么首先我们通过代码的方式看看怎么添加cookie和删除cookie，
```
///增加cookies
+ (void)addCookiesToRequest:(NSMutableDictionary *)cookieDic
{
    NSEnumerator * enumeratorKey = [cookieDic keyEnumerator];
    for (NSObject * key in enumeratorKey) {
        NSHTTPCookie *userInfoCookie = [NSHTTPCookie cookieWithProperties:
                                        [NSDictionary dictionaryWithObjectsAndKeys:
                                         @".baidu.com", NSHTTPCookieDomain,
                                         @"/", NSHTTPCookiePath,
                                         [NSString stringWithFormat:@"%@",key],  NSHTTPCookieName,
                                         [NSDate dateWithTimeIntervalSinceNow:30*24*3600], NSHTTPCookieExpires,
                                         [NSString stringWithFormat:@"%@",[cookieDic objectForKey:key]], NSHTTPCookieValue,
                                         nil]];
        [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:userInfoCookie];
        NSHTTPCookie *txdaiCookie = [NSHTTPCookie cookieWithProperties:
                                        [NSDictionary dictionaryWithObjectsAndKeys:
                                         @".jingdong.com", NSHTTPCookieDomain,
                                         @"/", NSHTTPCookiePath,
                                         [NSString stringWithFormat:@"%@",key],  NSHTTPCookieName,
                                         [NSDate dateWithTimeIntervalSinceNow:30*24*3600], NSHTTPCookieExpires,
                                         [NSString stringWithFormat:@"%@",[cookieDic objectForKey:key]], NSHTTPCookieValue,
                                         nil]];
        [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:txdaiCookie];
    }
}
```

同样删除cookie也非常简单

```
///删除基本cookies
+(void)deleteBaseCookie{
    NSHTTPCookie *passportCookie = [NSHTTPCookie cookieWithProperties:
                                    [NSDictionary dictionaryWithObjectsAndKeys:
                                     @".baidu.com", NSHTTPCookieDomain,
                                     @"/", NSHTTPCookiePath,
                                     @"sfut",  NSHTTPCookieName,
                                     @"", NSHTTPCookieValue,
                                     nil]];
    [[NSHTTPCookieStorage sharedHTTPCookieStorage] deleteCookie:passportCookie];
    NSHTTPCookie *txdaiCookie = [NSHTTPCookie cookieWithProperties:
                                 [NSDictionary dictionaryWithObjectsAndKeys:
                                  @".jingdong.com", NSHTTPCookieDomain,
                                  @"/", NSHTTPCookiePath,
                                  @"sfut",  NSHTTPCookieName,
                                  @"", NSHTTPCookieValue,
                                  nil]];
    [[NSHTTPCookieStorage sharedHTTPCookieStorage] deleteCookie:txdaiCookie];
}
```
那么最后一个问题，就是我们在客户端什么情况下才添加或者删除cookie呢？

当我们的应用在加载一个wap页面的时候，可能wap需要知道客户端的一些信息， 比如你是登陆状态还是什么状态，因为这时候我们就可以设置cookie，wap可以拿到请求的cookie

当然需要注意的是在wap将要销毁的时候，要把cookie信息给移除掉。