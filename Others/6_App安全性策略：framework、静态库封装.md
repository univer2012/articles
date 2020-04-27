

内容来自：【腾讯课堂- iOS高级开发/面试/性能优化/经典框架-- 八点钟学院

6.App安全性策略：framework、静态库封装】

### 创建一个`.a`的静态文件

进行如下操作：

1. `File` --> `New` --> `Project...`

2. 选中`iOS` ，选择`Framework & Library` 中的`Cocoa Touch Static Library`，命名为`EOCLib`

3. 在`EOCLib`中实现如下代码：

`EOCLib.h`:

```objective-c
//
#import <Foundation/Foundation.h>

@interface EOCLib : NSObject
+ (void)eocMethod;
@end
```

`EOCLib.m`:

```objc
#import "EOCLib.h"

@implementation EOCLib

+ (void)eocMethod {
    NSLog(@"eocMethod");
}
    
@end
```

分别选中模拟器和`Generic iOS Device`，点击箭头运行一遍，然后对`Products`--> `libEOCLib.a`右键，选择`Show in Finder`，会发现目录如下：

![图1](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging1.png)



如果要把路径中的`\EOCLib\`去掉，则需要在`BUild Phases` --> `Copy Files(1 item)`的 `Subpath`由原来的`include/$(PRODUCT_NAME)`改为`include`，删除`Debug-iphoneos` 和`Debug-iphonesimulator`，且按`command` + `shift`+`J`清理工程。再分别选中模拟器和`Generic iOS Device`，点击箭头运行一遍，然后对`Products`--> `libEOCLib.a`右键，选择`Show in Finder`，就会得到如图：

![图2](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging2.png)



如果要在这个文件中添加其他文件，则添加后，需要在`BUild Phases` --> `Copy Files(1 item)`中点`+`把对应的`.h`文件添加进去，编译后才会在`include`文件中查看到。

![图3](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging3.png)



我们用终端`cd`到`Debug-iphoneos`和`Debug-iphonesimulator`，然后执行

```
lipo -info libSGHLib.a
```
出现的结果分别是：
```
Non-fat file: libEOCLib.a is architecture: x86_64
```
```
Non-fat file: libEOCLib.a is architecture: arm64
```

代表的意思是支持模拟器，支持iPhone5及以后的机型。



如果我们把`Debug-iphoneos`中的文件导入到应用项目中，则会出现如下的错误：

![图4](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging4.png)

所以，`Debug-iphonesimulator`中的文件只支持模拟器，`Debug-iphoneos`中的文件支持真机。



如果我们要同时支持真机和模拟器，则需要做如下操作：

1. 把`Debug-iphonesimulator` 和`Debug-iphoneos`的`libEOCLib.a`放在同一目录中，名字要改为不一样的名字，比如`libEOCLibR.a`和`libEOCLib.a`
2. 然后用终端`cd`到该目录下，执行如下命令
```
$ lipo -create libEOCLibR.a libEOCLib.a -output test.a
```
得到合并后的`test.a`文件，这个文件就是同时支持模拟器和真机的`.a`文件。执行命令`$ lipo -info test.a`可以得到如下结果：

```
Architectures in the fat file: test.a are: x86_64 arm64
```

说明是同时支持模拟器和真机了。使用时只需要把这个`test.a`文件和原来的`include`文件夹拖进去，即可使用了。



静态库相关的资源，bundle文件，bundle文件是不会被编译的。





### 动态库

进行如下操作：

1. `File` --> `New` --> `Project...`

2. 选中`iOS` ，选择`Framework & Library` 中的`Cocoa Touch Framework`，命名为`EOCFrame`，

3. 创建一个`EOCObject`的`NSObject`类，代码如下：

   `EOCObject.h`:

   ```objc
   #import <Foundation/Foundation.h>
   
   NS_ASSUME_NONNULL_BEGIN
   
   @interface EOCObject : NSObject
   
   + (void)testMethod;
   
   @end
   
   NS_ASSUME_NONNULL_END
   ```

   `EOCObject.m`:

   ```objc
   #import "EOCObject.h"
   
   @implementation EOCObject
   + (void)testMethod{
       NSLog(@"八点钟学院");
   }
   @end
   ```

   4. 把`Build Phases` --> `Headers`的`Project`中的`EOCObject.h`移到`Public`中，才能导入到框架中。

   5. 分别选中模拟器和 真机，点击向右箭头运行，右键`Projects`-->`EOCFrame.framework` ，选中`Show in Finder`，在`EOCFrame.framework`中的`EOCFrame`才是真正的执行文件。如图：
   
   ![图5](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging5.png)



使用时，直接把`EOCFrame.framework`拖进工程，即可使用。如图：

![图6](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging6.png)

运行起来会发现报错：
![图7](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging7.png)

报错没有被映射。动态库使它失去了动态性，这是由于苹果的审核机制导致的。所以要执行：`TARGETS` --> `EOCClass` --> `Build Phases` --> 点击`+`号，选中`New Copy Files Phase`,在出现的`Copy Files`中，`Destination`选择`Frameworks`，然后点`+`号，把`EOCFrame.framework`添加进来。这样就没问题了。

我们会发现，在`EOCFrame.framework`去拿`EOCFrame`进行合并成一个真机和模拟器都可以使用的可执行文件会比较麻烦，这时我们有一个脚本：
1. 在`TARGET` --> `EOCFrame` --> `Build Phases`

2. 点击左上角的`+`号，选择`New Run Script Phase`，在出现的`Run Script`的`Shell`下方的区域输入脚本：

   ```shell
   if [ "${ACTION}" = "build" ]
   then
   INSTALL_DIR=${SRCROOT}/products/${PROJECT_NAME}.framework
   
   DEVICE_DIR=${BUILD_ROOT}/${CONFIGURATION}-iphoneos/${PROJECT_NAME}.framework
   
   SIMULATOR_DIR=${BUILD_ROOT}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}.framework
   
   if [ -d "${INSTALL_DIR}" ]
   then
   rm -rf "${INSTALL_DIR}"
   fi
   
   mkdir -p "${INSTALL_DIR}"
   
   cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}"
   #ditto "${DEVICE_DIR}/Headers" "${INSTALL_DIR}/Headers"
   
   lipo -create "${DEVICE_DIR}/${PROJECT_NAME}" "${SIMULATOR_DIR}/${PROJECT_NAME}" -output "${INSTALL_DIR}/${PROJECT_NAME}"
   
   #open "${DEVICE_DIR}"
   open "${SRCROOT}/Products"
   fi
   ```

   然后再点击向右箭头运行，发现运行出来的`EOCFrame.framework`是空文件，是因为没有生成真机文件夹`Debug-iphoneos`。



我们先把选中`EOCFrame`-->`Generic iOS Device`，运行，生成真机文件夹`Debug-iphoneos`。然后再选中模拟器，运行上面的脚本，发现打开的`Product`文件夹下的`EOCFrame.framework`不再是空文件。如图：

![图8](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging8.png)



把`EOCFrame.framework`中的`EOCFrame`复制出来，然后终端`cd`到该目录，执行：

```
$ lipo -info EOCFrame
```

结果为：

```
Architectures in the fat file: EOCFrame are: x86_64 arm64 
```





实际上是不支持动态库的，其本质是还是静态的。我们可以把动态库变为静态库，方法为：

1. `TARGETS`--> `EOCFrame` -->`Build Settings`
2. 选中`All`和`Combined`，输入`Mach`，找到`Mach-O Type`项，把值改为`Static Library`，那么编译出来就是静态库，而不是动态库了。

编译成`.framework`的静态库后，使用和动态库一样，直接把`EOCFrame.framework`拖进工程，引入文件名即可使用，而不需要像`.a`静态库那样导入`include`文件夹，因为`EOCFrame.framework`的目录下的`Headers`文件夹下已经有`.h`文件了。而且不需要像`.a`文件那样要用脚本合成真机和模拟器一起使用的文件，而直接使用脚本即可生成。





### 静态库`.a`的另一个知识点

在`EOCLib`工程中添加一个`NSObject`的扩展：`NSObject+EOC`，代码如下：

`NSObject+EOC.h`:

```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface NSObject (EOC)
+ (void)eocMethod;
@end

NS_ASSUME_NONNULL_END
```

`NSObject+EOC.m`:

```objc
#import "NSObject+EOC.h"

@implementation NSObject (EOC)
+ (void)eocMethod {
    NSLog(@"eocMethod NSObject");
}
@end
```

并执行：`TARGETS`-->`EOCLib`-->`Build Phases`-->`Copy Files`中把`NSObject+EOC.h`加入其中，点击向右箭头运行，并把运行出来的`libEOCLib.a`和`include`文件夹拖入到应用工程中，然后调用`[NSObject eocMethod]; `，结果如图：

![图9](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging9.png)

这是什么原因导致的呢？这个跟静态库有关系，静态库做分类，因为分类不是新的链接符号，并不会给这个方法生成一个链接符号到执行文件中，所以执行时找不到这个方法。



那么我们应该怎么做呢？



我们需要在应用工程`EOCClass`中配置下：`TARGETS`-->`EOCClass`-->`Build Settings`-->输入`Other Linker`，找到`Other Linker Flags`，点击`+`号输入`-ObjC`，如图：

![图10](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/6_App_security_strategy%3Dframework%2Cstatic%20library%20packaging10.png)



这个就是链接我们静态库的所有OC文件。添加之后再运行，发现可以正常调用了。



在`Other Linker Flags`上添加的字符及其意义：

* `-ObjC`  ： 链接静态库中的所有OC文件；

* `-all_load`  ：链接静态库中的所有文件；

* `-force_load`

  `"$(SRCROOT)/EOCClass/libEOCLib.a"`  ：链接指定文件。方法：先添加`-force_load`，然后再点`+`号，把要链接的指定`.a`文件拖入其中就会出现该文件的路径了。



【完】