来自：[https://github.com/AndyQ/NFCPassportReader](https://github.com/AndyQ/NFCPassportReader)

## NFCPassportReader

这个package使用ios13 CoreNFC APIs，来处理可用NFC读取的护照

支持功能:

* 基本访问控制(BAC)
* 安全的消息
* 以JPEG和JPEG2000格式读取DG1 (MRZ数据)和DG2(图像)
* 被动身份验证(目前仅在示例应用程序中)



这仍然是非常早期的阶段——代码绝不是完美的，仍然有一些粗糙的边缘-肯定有很多bug，我肯定我没有把事情做得很完美。

它可以读取和验证我的护照(以及我能够测试的其他护照)，但是您的mileage(里程)可能会有所不同。



## 使用

要使用它，您首先需要创建Passport MRZ密钥，它包含护照号、出生日期和有效期(包括校验和)。日期是YYMMDD格式

例如:

```
<passport number><passport number checksum><date of birth><date of birth checksum><expiry date><expiry date checksum>

e.g. for Passport nr 12345678, Date of birth 27-Jan-1998, Expiry 30-Aug-2025 the MRZ Key would be:

Passport number - 12345678
Passport number checksum - 8
Date Of birth - 980127
Date of birth checksum - 7
Expiry date - 250831
Expiry date checksum - 5

mrzKey = "12345678898012772508315"
```



然后，在PassportReader的实例上，调用`readPassport`方法，传入`mrzKey`、要读取的数据组和一个完成块。

如:

```
passportReader.readPassport(mrzKey: mrzKey, tags: [.COM, .DG1, .DG2], completed: { (error) in
   ...
}
```

目前支持的数据组有:COM、DG1、DG2、SOD

然后，它将处理passport和image的读取，如果出现某种错误，它将调用带有TagError错误的完成块，如果成功，则调用nil。

如果成功，则passportReader对象将包含passportMRZ和passportImage字段的有效数据。



## Logging

通过在创建时传入一个日志级别，可以在PassportReader上启用额外的日志记录(非常详细):例如。

```
let reader = PassportReader(logLevel: .debug)
```

注意——目前这只是打印到控制台—我希望以后实现更好的日志记录——可能使用`SwiftyBeaver`



## 样例应用程序

repo中包含一个示例应用程序，演示了该功能。

现在，它包含了一个如何进行被动身份验证的示例，以确保电子护照有效且没有被篡改。



然而这需要使用OpenSSL库(包括作为一个Pod文件从Marcin Krzyżanowski OpenSSL-Universal Pod (https://github.com/krzyzanowskim/OpenSSL)。

我想把这个转移到它自己的Swift包，但目前SPM不支持混合语言，所以很遗憾我还不能。而且，我还不想参与整个过程，所以如果有人想做一些聪明的事情……

它需要一套来自总清单(要么来自公布总清单的国家，要么来自ICAO PKD知识库)的PEM格式的CSCA证书。有关如何获取和创建此文件的详细信息，请参阅scripts文件夹。

示例应用程序中包含的**masterList.pem**文件纯粹是为了确保没有编译器警告，并且只包含一个自生成的pem文件，而且不能实现任何内容!



## 故障排除

如果在进行初始的互认证挑战时，您得到一个错误的SW1代码0x63, SW2代码0x00，原因:没有给出信息，那么这通常是因为您的MRZ密钥不正确，可能是因为您的护照号码不太正确。如果你的护照号码在MRZ中包含一个“<”，那么你需要把它包含在MRZKey中——校验和也应该是正确的。详情请参阅ICAO 9303 Part 11文件中的App-D2 (https://www.icao.int/publications/Documents/9303_p11_cons_en.pdf)



## To do

有一些我想实现的事情，没有特定的顺序:

* 能够转储护照流并将其读入
* 实现PACE验证