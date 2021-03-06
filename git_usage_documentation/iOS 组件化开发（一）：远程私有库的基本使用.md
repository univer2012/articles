来自：[iOS 组件化开发（一）：远程私有库的基本使用](https://www.jianshu.com/p/a23573c0ed4f)



---



> 随着项目功能的不断增加，越来越多的开发人员加入，业务主线也随之越来越多，造成耦合越来越严重，编译越来越慢，测试不独立等一系列问题。为了解决此类情况，我们可以考虑到使用组件化开发

- 概念
   组件化就是将一个单一工程的项目, 分解成为各个独立的组件， 然后按照某种方式, 任意组织成一个拥有完整业务逻辑的工程。
- 优势
  - 独立：独立编写、编译、运行、测试
  - 重用：功能代码的重复使用。比如不同项目使用同一功能模块
  - 高效：任意增删模块，实现高效迭代
  - 组件化还可以配合二进制化, 提高项目编译速度
- 组件分类
   大体上分三类：基础组件、功能组件和业务组件
  - 基础组件：也称为公共组件，存放平时定义的宏、常量、协议、分类、对必要的第三方的封装类，以及各种处理工具类，如：时间、日期、设备信息、文件处理、沙盒管理等
  - 功能组件： 自定义视图控件、一些特定功能的封装（如录音、播放音频封装）
  - 业务组件：各种业务线



------



> 本篇先来介绍下远程私有库的基本使用，建议按顺序看完之后，回来再看一遍步骤归纳，加深了解，如有不足之处，欢迎指出，感谢 : )

## 步骤归纳

1. 创建远程索引库和私有库
2. 将远程索引库添加到本地 `pod repo add 索引库名称 索引库地址`
3. 在本地创建一个pod模板库 `pod lib create 组件名称`
    将框架的核心代码添加到Classes目录下
    本地安装测试核心代码是否可用 `pod install`
    修改Spec描述文件
    将修改好的模板库上传至远程私有库
4. 上传代码和打标签
    `git init`
    `git add .`
    `git commit -m "提交描述"`
    `git remote add origin 远程私有库地址`
    `git push origin master`
    `git tag '0.1.0'`
    `git push --tags`
5. 提交spec至私有索引库
    `pod lib lint --private`
    `pod spec lint --private`
    `pod repo push 索引库的本地名称 xx.podspec`
6. 使用
    `source 官方索引库url`
    `source 私有索引库url`
    `pod '组件名称'`
    `pod install`

**接下来我们就来实战如何创建和使用私有库**

## 一、创建私有索引库

这里以码云为例，创建一个LXFSpecs的私有索引库，这玩意的作用如其名，就是用来索引的

![img](https:////upload-images.jianshu.io/upload_images/2144614-2eef2f1ef1231637.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

私有索引库

![img](https:////upload-images.jianshu.io/upload_images/2144614-610cd12a723e7c55.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

LXFSpecs

## 二、本地添加私有索引库

### 1、查看本地索引库

```shell
pod repo
```



![img](https://upload-images.jianshu.io/upload_images/2144614-50435b5218228ebf.png?imageMogr2/auto-orient/strip|imageView2/2/w/990)

查看本地索引库

如图，目前本地仅有github上的那个公有索引库

### 2、添加私有索引库

将我们刚刚新建的私有索引库LXFSpecs添加到本地

```csharp
// pod repo add 索引库名称 索引库地址
pod repo add LXFSpecs https://gitee.com/LinXunFeng/LXFSpecs.git
```

![img](https://upload-images.jianshu.io/upload_images/2144614-bbbefcb5dbd9c322.png?imageMogr2/auto-orient/strip|imageView2/2/w/1138)

现在本地就有两个索引库，好，索引库的事情就先放一边去了～

## 三、创建组件库

码云上的创建操作同上，这里以LXFBase为例，创建基础组件库



![img](https://upload-images.jianshu.io/upload_images/2144614-95e73342ec562ea1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

LXFBase

### 1、快速创建模版库

到合适的位置创建一个与组件名相同的文件夹，cd进去后，使用如下命令

```cpp
// pod lib create 组件名
pod lib create LXFBase
```

![img](https://upload-images.jianshu.io/upload_images/2144614-250f57a5ae90c710.png?imageMogr2/auto-orient/strip|imageView2/2/w/1170)

这里会让你配置一些信息，根据自己的情况自行配置即可。

![img](https://upload-images.jianshu.io/upload_images/2144614-457b42c1494606fa.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162)

配置

### 2、添加组件内容

创建完成后会自动帮我们打开相应的Example项目，LXFBase目录中会出现如图这些文件，我们把基础组件相关的东西丢到Classes文件夹中，并且把`ReplaceMe.m`文件删除

![img](https:////upload-images.jianshu.io/upload_images/2144614-14093ebc54a141b1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

目录结构

默认Classes文件夹中存放的文件就是pod install时要下载下来的文件，当然可以通过修改spec文件的配置来更改位置

### 3、安装与测试本地库

在Example项目的Podfile文件中可以看到

```shell
pod 'LXFBase', :path => '../'
```

模板库已经默认帮我们在Podfile中指定了LXFBase.podspec的位置，使组件LXFBase可以正常安装使用和方便测试

```
pod install
```



![img](https://upload-images.jianshu.io/upload_images/2144614-59edada376a704c6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1168)

pod install

![img](https://upload-images.jianshu.io/upload_images/2144614-dca0605926088593.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

pod

可以看到我们已经将本地的组件添加进Example中了，现在可以尽情地做你想做的测试，确保组件的可用。

测试组件没有问题后，我们接下来就要将podspec文件上传至私有索引库，不过在此之前，需要对spec进行修改。

### 4、 修改Spec

具体的配置说明可以参考[Cocoapods 创建第三方框架](https://www.jianshu.com/p/d022d9adee4e)

![img](https://upload-images.jianshu.io/upload_images/2144614-89861b5b1242f6cc.png?imageMogr2/auto-orient/strip|imageView2/2/w/860)

podspec

主要的修改内容

```python
  s.name             = 'LXFBase'
  s.version          = '0.1.0'
  s.summary          = 'LXFBase.'
  s.description      = <<-DESC
LXFBase是基础组件库，包括分类和常用工具
                       DESC
  s.homepage         = 'https://gitee.com/LinXunFeng/LXFBase'
  s.source           = { :git => 'https://gitee.com/LinXunFeng/LXFBase.git', :tag => s.version.to_s }
  s.source_files = 'LXFBase/Classes/**/*'
```

## 四、上传组件代码

### 1、将代码提交到组件仓库

```shell
git add .
git commit -m 'firstCommit'
git remote add origin https://gitee.com/LinXunFeng/LXFBase.git
// 第一次push如果报错的话可以加上-f
// git push -f origin master
git push origin master
```

### 2、打标签

标签`0.1.0`与spec中的`s.version`保持一致

```bash
git tag '0.1.0'
git push --tags
```

![img](https://upload-images.jianshu.io/upload_images/2144614-ec4b53c54b5f045c.png?imageMogr2/auto-orient/strip|imageView2/2/w/712)

tag

![img](https://upload-images.jianshu.io/upload_images/2144614-0f190b60e773dd35.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

标签上传成功

## 五、提交podspec到私有索引库

在上传spec文件前我们可以做一个验证来节省时间，不然每次都推送很久结果还是验证失败，会气死人的～

### 1、本地验证Spec的必填字段

```shell
// 本地验证不会验证 s.source 中的tag
$ pod lib lint --allow-warnings
```



![img](https://upload-images.jianshu.io/upload_images/2144614-e30082a165d1ea9b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1164)

pod lib lint



### 2、远程验证

```shell
// 远程验证会验证 s.source 中的tag，如果此时没有打上相应的标签则会报错
$ pod spec lint --allow-warnings
```

如果你刚才没有打标签并上传至远程私有库就来进行远程验证，肯定是会报错的

![img](https://upload-images.jianshu.io/upload_images/2144614-8e78901cd75f5fb0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140)

tag Error



在打完并上传tag后再进行远程验证，就会验证成功了，验证成功后我们就可以进行下一步操作：提交podspec文件到索引库

![img](https://upload-images.jianshu.io/upload_images/2144614-adce8ca2b80383a8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162)

pod spec lint

- 验证私有库提示

如果验证的是私有库，则在后面加上`--private` ，否则会有警告。你可以选择`--allow-warnings`来忽略该警告

```shell
pod lib lint --private
pod spec lint --private
```

### 3、提交podspec

```python
// pod repo push 私有索引库名称 spec名称.podspec 
pod repo push LXFSpecs LXFBase.podspec 
```

这里的操作过程：先将我们的代码直接push到本地索引库LXFSpecs，推送后会自动帮我们同步到远程索引库

![img](https:////upload-images.jianshu.io/upload_images/2144614-d9ae91141d7d2e62.png?imageMogr2/auto-orient/strip|imageView2/2/w/1156)

提交成功

再来看看码云上的私有索引库LXFSpecs

![img](https:////upload-images.jianshu.io/upload_images/2144614-1a3ee5ec657a9aef.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

LXFBase.podspec

来测试下搜索我们的组件

```bash
pod search 'LXFBase'
```

![img](https://upload-images.jianshu.io/upload_images/2144614-1f89b336bc74dfcc.png?imageMogr2/auto-orient/strip|imageView2/2/w/822)

搜索成功

## 六、使用私有库

这时我们可以来试试通过pod形式来添加组件LXFBase，创建一个新的项目

### 1、添加Podfile文件

```kotlin
pod init
```

### 2、在Podfile的最顶部添加如下描述

```rust
#第二行是为了保证公有库的正常使用
source 'https://gitee.com/virgilsun2012/LXFBase.git'
source 'https://github.com/CocoaPods/Specs.git'
```

### 3、添加使用组件LXFBase

```bash
pod 'LXFBase', :git => 'git@gitee.com:virgilsun2012/LXFBase.git'
```



![QQ20200601-215701.png](https://upload-images.jianshu.io/upload_images/843214-42f6eff5dc6c0649.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 4、安装组件

```undefined
pod install --verbose  //本人的实践
```

![QQ20200601-215251.png](https://upload-images.jianshu.io/upload_images/843214-2cfb9c71c5104012.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

//... ...

![QQ20200601-215309.png](https://upload-images.jianshu.io/upload_images/843214-2aba20da6d24ceed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



如果报了如下错误，那就多执行几遍：

![QQ20200601-220839.png](https://upload-images.jianshu.io/upload_images/843214-f019ea371380589e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



安装成功



![QQ20200601-215508.png](https://upload-images.jianshu.io/upload_images/843214-79ebd9f11812c18e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功添加组件内容



---

【完】

