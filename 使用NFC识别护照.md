# 使用NFC识别护照

### 概述
NFC的全称是Near Field Communication，中文意思是“近场通信”。苹果在iPhone 6/6 Plus 及以后的iPhone机型添加了NFC芯片，并从iOS 11引入CoreNFC，但iOS11支持的NFC，只开启了读权限，且只支持一种格式为`NFC Data Exchange Format`（简称`NDEF`）的信息，对应的session对象为`NFCNDEFReaderSession`。仅限于Apple Pay使用，不对第三方开发，使用场景很小。

iOS 13开放了NFC的功能，使得开发者可以做更多的事情。iOS13`NFCNDEFReaderSession`增加了写权限，具备读、写权限。添加了新的session对象`NFCTagReaderSession`。可以读取护照、身份证、公交卡，门禁卡等支持NFC的使用场景。


#### iOS 13支持的iPhone机型如下：
iPhone XS
iPhone XS Max
iPhone XR
iPhone X
iPhone 8
iPhone 8 Plus
iPhone 7
iPhone 7 Plus
iPhone 6s
iPhone 6s Plus
iPhone SE
iPod touch (第七代)

#### 问题1：支持NFC的iPhone机型
答：6、6 plus及以上
#### 问题2：SE支持NFC吗
答：支持。答案链接：https://www.jb51.net/shouji/443092.html

从上面罗列的iOS13和NFC支持设备可以知道，iOS13的NFC功能，最低支持iPhone 6s/6s Plus。


## 创建一个app



