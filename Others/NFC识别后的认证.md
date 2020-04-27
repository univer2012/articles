





####Passive Authentication（被动认证）



Passive Authentication is now part of the main library and can be used to ensure that an E-Passport is valid and hasn't been tampered with.

被动身份验证现在是主库的一部分，可以用来确保电子护照有效且没有被篡改。



It requires a set of CSCA certificates in PEM format from a master  list (either from a country that publishes their master list, or the  ICAO PKD repository). See the scripts folder for details on how to get  and create this file.

它需要一套来自总清单(要么来自公布总清单的国家，要么来自ICAO PKD知识库)的PEM格式的CSCA证书。有关如何获取和创建此文件的详细信息，请参阅scripts文件夹。



**The masterList.pem file included in the Sample app is purely  there to ensure no compiler warnings and contains only a single PEM file that was self-generated and won't be able to veryify anything!**

示例应用程序中包含的`masterList.pem`文件纯粹是为了确保没有编译器警告，并且只包含一个自生成的PEM文件，不能对任何内容进行验证!





### Sample app

repo中包含一个示例应用程序，演示了该功能。

### 故障排除

If when doing the initial Mutual Authenticate challenge, you get an  error with and SW1 code 0x63, SW2 code 0x00, reason: No information  given, then this is usualy because your MRZ key is incorrect, and  possibly because your passport number is not quite right.  If your  passport number in the MRZ contains a '<' then you need to include  this in the MRZKey - the checksum should work out correct too.  For more details, check out App-D2 in the ICAO 9303 Part 11 document (https://www.icao.int/publications/Documents/9303_p11_cons_en.pdf)

如果在进行初始的互认证挑战时，您得到一个错误的SW1代码0x63, SW2代码0x00，原因:没有给出信息，那么这通常是因为您的MRZ密钥不正确，可能是因为您的护照号码不太正确。如果你的护照号码在MRZ中包含一个“<”，那么你需要把它包含在MRZKey中——校验和也应该是正确的。详情请参阅ICAO 9303 Part 11文件中的App-D2 (https://www.icao.int/publications/Documents/9303_p11_cons_en.pdf)







没有在ICAO PKD 的国家：

* Israel 以色列

- Portugal 葡萄牙
- 





https://www.icao.int/Security/FAL/PKD/Pages/ICAO-PKDParticipants.aspx



----



来自：[scripts](https://github.com/AndyQ/NFCPassportReader/tree/master/scripts)

# 主列表产生器

### 它是什么？

该脚本将创建一个文件，其中包含一组来自国家主列表或ICAO PKD存储库的PEM格式的唯一CSCA(国家签署证书颁发机构)证书。



这些是用来验证电子护照和其他电子身份证包含的文件签名证书和安全对象DS签名(SOD)。



它可以接受LDIF格式的ICOA PDK主列表文件(它是来自经过验证的国家的主列表的集合)，也可以接受密码消息语法(CMS)格式的单个国家主列表文件。



### 我从哪里可以得到主目录?

国际民航组织将其总名单“免费提供给任何希望下载该名单的个人或国家。然而，下载过程是手动的，不能自动进行。”(https://www.icao.int/Security/FAL/PKD/BVRT/Pages/Access.aspx)



主列表可以从以下地址下载:https://pkddownloadsg.icao.int



它们是LDIF (LDAP数据交换格式)格式



此外，有些国家还提供了他们的总清单(如德国、法国和意大利)。搜索masterlist或者JMRTD项目有一个list(但是有些链接不再工作)  \- https://jmrtd.org/certificates.shtml Yobi Wifi



这些文件通常会被压缩，因此提取这些文件并以.ml结尾。

显然，你应该只使用来自你信任的来源的总清单!



### 需求

该脚本使用Python 3.7 (Python 3的其他版本可能可以工作—没有尝试过)。

它还需要一个支持CMS标志的OpenSSL版本。

macOS的版本(包括Catalina)不支持这个功能，所以你需要从其他地方获得这个功能(比如：Homebrew)。



### 使用

要运行脚本，只需运行:`python extract.py [Country master list]`。

e.g.

```python
python extract.py icaopkd-002-ml-000119.ldif
```



它将遍历文件中包含的masterlist。您将得到一个新的`masterList.pem`文件，它是所有唯一证书的连接。

然后可以将其导入NFCPassportReader应用程序，并使用被动身份验证来验证电子护照。

### 信用

这个脚本基本上是基于 http://wiki.yobi.be/wiki/EPassport 的详细信息，还有一些用于删除重复证书的附加位。





> 专业名词：
>
> CSCA   Country Signing Certificate Authority 国家签署证书机关
>
> ICAO    International Civil Aviation Organization 国际民用航空组织
>
> PKD     Public Key Directory	公钥目录
>
> SOD    Security Object DS Signatures    安全对象DS签名
>
> CMS    Cryptographic Message Syntax   密码消息语法
>
> LDIF	 LDAP Data Interchange Format      LDAP数据交换格式
>
> 关于ICAO PKD的介绍请查阅：[ICAO PKD](https://www.icao.int/Security/FAL/PKD/Pages/default.aspx)

