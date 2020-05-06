# 一、Xcode清理垃圾文件

### 1. ~/Library/Developer/Xcode/DerivedData/

这个文件夹中保存的是Xcode的缓存文件，曾经在Xcode跑过的所有项目的索引、build的信息等都会保存在这里。删除后在下次打开项目编译的时候将会重新生成。由于这里包含了大量已经没用的项目的信息又懒得去筛选，于是把整个文件夹删了。

### 2. ~/Library/Developer/Xcode/iOS DeviceSupport/

每次把一个设备接入电脑进行真机调试之前，电脑会对设备建立索引，也在此文件夹下生成对该设备系统的支持文件。于是这里存在了一堆对旧版本iOS设备支持的文件。而我最近基本只对iOS9.3的设备进行真机调试。于是删除了所有低于9.3的文件夹。

### 3. ~/Library/Developer/Xcode/Archives/

每次打包App的dSYM等数据就保存在这里，把一些没用的版本删了。如果是上线了的版本还是保留吧。

### 4. ~/Library/Developer/Xcode/Products/

同上，把没用的删了。

### 5. ~/Library/Developer/CoreSimulator/Devices/

一堆模拟器的数据。每个文件夹里包含的就是一个特定系统版本的设备的数据。每个文件夹对应哪个设备可以在其下device.plist中查看。亲测删除之后的效果跟在模拟器里重置相同。省得一个个去重置了，删吧。

### 6.~/Library/Developer/XCPGDevices/

这里保存了playground的项目缓存。全删了。

# 二、删除系统自带的邮件客户端没用的邮件文件

找到系统自带的邮件客户端将邮件文件储存位置：~/资源库/Mail/

然后文件夹里的全部删除。