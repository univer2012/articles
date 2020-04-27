面试题：

### iOS 只有一些符号堆栈的情况下，你怎么去定位到相应的崩溃的地方和原因。


参考：
1. [iOS崩溃堆栈符号化，定位问题分分钟搞定](https://blog.csdn.net/xuduzhoud/article/details/50440519)

### 1. 符号表是什么？
 符号表就是指在Xcode项目编译后，在编译生成的二进制文件.app的同级目录下生成的同名的.dSYM文件。

.dSYM文件其实是一个目录，在子目录中包含了一个16进制的保存函数地址映射信息的中转文件，所有Debug的symbols都在这个文件中(包括文件名、函数名、行号等)，所以也称之为**调试符号信息文件**。

==一般地，Xcode项目每次编译后，都会生成一个新的.dSYM文件。因此，App的每一个发布版本，都需要备份一个对应的.dSYM文件，以便后续调试定位问题。==

> 注意：
> 项目每一次编译后，.app和.dSYM成对出现，并且二者有相同的UUID值，以标识是同一次编译的产物。
> UUID值可以使用dwarfdump —uuid来检查：
> ```
> $ dwarfdump --uuid XX.app.dSYM
> $ dwarfdump --uuid XX.app/XX
> ```


那么，问题就来了！
### 2. 符号表有什么用？

 在Xcode开发调试App时，一旦遇到崩溃问题，开发者可以直接使用Xcode的调试器定位分析。

==但如果App发布上线，开发者不可能进行调试，只能通过分析系统记录的崩溃日志来定位问题，在这份崩溃日志文件中，会指出App出错的函数内存地址，而这些函数地址是可以在.dSYM文件中找到具体的文件名、函数名和行号信息的，这正是符号表的重要作用所在。==

> 实际上，使用Xcode的Organizer查看崩溃日志时，也自动根据本地存储的.dSYM文件进行了符号化的操作。
>
> 并且，崩溃日志也有UUID信息，这个UUID和对应的.dSYM文件是一致的，即只有当三者的UUID一致时，才可以正确的把函数地址符号化。

### 3. 符号表怎么生成？

 一般地，Xcode项目默认的配置是会在编译后生成`.dSYM`，开发者无需额外修改配置。

项目的Build Settings的相关配置如下：
```
    Generate Debug Symbols = Yes
    Debug Information Format = DWARF with dSYM File
```

 **采用不同的编译打包方式，产生的.dSYM文件的路径也不相同。**

下面是==几种常用的编译打包方式==：

##### 1、使用xcodebuild编译打包

在Xcode中编译项目后，会在工程目录下的`build/ConfigurationName-iphoneos`目录下生成.app和.app.dSYM文件。

如果使用xcodebuild命令进行编译打包，则可以指定编译结果的存储路径，同样会有.app和.app.dSYM生成。

一般地，我们推荐打包发布时，使用xcodebuild编译打包，方便.app和`.app.dSYM`的匹配存储，避免`.app.dSYM`文件丢失的情况。

##### 2、使用Xcode的Archive导出

如果开发者使用Xcode的Archive导出功能打包，可以切换到Organizer的Projects视图，查看对应项目的Derived Data路径，在其中可以找到当前导出过程产生的.app和.app.dSYM文件

##### 3、使用make编译打包

如果开发团队不使用Xcode编译打包，而是使用make编译生成.o文件，然后打包发布。此时，编译过程不会有.dSYM文件生成。开发者可以使用dsymutil工具从.o文件中提取符号信息。


### 4. 符号表怎么用？
 在前面的内容可以知道，符号表的作用是把崩溃中的函数地址解析为函数名等信息。

如果开发者能够获取到崩溃的函数地址信息，就可以利用符号表分析出具体的出错位置。

Xcode提供了几个工具来帮助开发者执行函数地址符号化的操作。

例如，崩溃问题的函数地址堆栈如下：

错误地址堆栈
```
    3  CoreFoundation           0x254b5949 0x253aa000 + 1096008
    4  CoreFoundation           0x253e6b68 _CF_forwarding_prep_0 + 24
    5  SuperSDKTest             0x0010143b 0x000ef000 + 74808
```
符号化堆栈
```
    3   CoreFoundation          0x254b5949 <redacted> + 712
    4   CoreFoundation          0x253e6b68 _CF_forwarding_prep_0 + 24
    5   SuperSDKTest            0x0010143b -[ViewController didTriggerClick:] + 58
```

> 说明：
>
> 大部分情况下，开发者能获取到的都是错误地址堆栈，需要利用符号表进一步符号化才能分析定位问题。
>
> 部分情况下，开发者也可以利用backtrace看到符号化堆栈，可以大概定位出错的函数、但却不知道具体的位置。通过利用符号表信息，也是可以进一步得到具体的出错位置的。
>
> 目前，许多崩溃监控服务都显示backtrace符号化堆栈，增加了可读性，但分析定位问题时，仍然要进一步符号化处理。

崩溃信息的UUID
```
0xef000 - 0x17efff SuperSDKTest armv7  <38d66f9734ca3843a2bf628bb9015a8b> /var/mobile/.../SuperSDKTest.app/SuperSDKTest
```

 下面，利用两个工具来进行一下符号化的尝试：

#### 1、symbolicatecrash
symbolicatecrash是一个将堆栈地址符号化的脚本，输入参数是苹果官方格式的崩溃日志及本地的.dSYM文件，执行方式如下：
```
$ symbolicatecrash XX.crash [XX.app.dSYM] > xx.sym.crash# 如果输入.dSYM参数，将只解析系统库对应的符号
```

> 使用symbolicatecrash工具的限制就在于只能分析官方格式的崩溃日志，需要从具体的设备中导出，获取和操作都不是很方便，而且，符号化的结果也是没有具体的行号信息的，也经常会出现符号化失败的情况。
>
> 实际上Xcode的Organizer内置了symbolicatecrash工具，所以开发者才可以直接看到符号化的错误日志。

 #### 2、atos

更普遍的情况是，开发者能获取到错误堆栈信息，而使用atos工具就是把地址对应的具体符号信息找到。

atos实际是一个可以把地址转换为函数名（包括行号）的工具，它的执行方式如下：
```
$ xcrun atos -o executable -arch architecture -l loadAddress
  address ...
```



> 说明：
>
> loadAddress 表示函数的动态加载地址，对应崩溃地址堆栈中 + 号前面的地址，即0x000ef000
>
> address 表示运行时地址、对应崩溃地址堆栈中第一个地址，即0x0010143b
>
> 实际上，崩溃地址堆栈中+号前后的地址相加即是运行时地址，即0x000ef000 + 74808 = 0x0010143b

执行命令查询地址的符号，可以看到如下结果：

```
$ xcrun atos -o SuperSDKTest.app.dSYM/Contents/Resources/DWARF/SuperSDKTest -arch armv7 -l 0x000ef000
0x0010143b
-[ViewController didTriggerClick:] (in SuperSDKTest) (ViewController.m:35)
```

开发者在具体的运用中，是可以通过编写一个脚本来实现符号化错误地址堆栈的