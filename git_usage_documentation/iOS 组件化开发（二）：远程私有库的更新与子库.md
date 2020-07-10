来自：[iOS 组件化开发（二）：远程私有库的更新与子库](https://www.jianshu.com/p/b84f49880919)



---



> 在上一篇【[iOS 组件化开发（一）：远程私有库的基本使用](https://www.jianshu.com/p/a23573c0ed4f)】中我们已经实战了远程私有库的基本操作，但是组件不可能上传一次就完事了，随着业务的增加，我们的组件可能还需要添加更多的东西，或者修复一些问题，这就需要我们对私有库代码进行升级与维护

这里以对基础组件里添加了一个Cache工具为例

![img](https:////upload-images.jianshu.io/upload_images/2144614-b8e905ea4f0d25d8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

添加Cache工具

添加完成后我们需要更新到远程仓库

## 一、更新远程仓库

cd 到本地仓库的位置，执行以下操作

### 1、代码更新

```csharp
$ git add .
$ git commit -m '更新描述'
$ export http_proxy=http://127.0.0.1:1087;export https_proxy=http://127.0.0.1:1087;
$ git push origin master
```

![img](https://upload-images.jianshu.io/upload_images/2144614-c5cd5145005eacc9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1170)

代码升级

### 2、版本更新

**版本更新 这一步非常重要，为更新索引库做准备**

```bash
git tag -a '新版本号' -m '注释'
git push --tags
```

![img](https://upload-images.jianshu.io/upload_images/2144614-355a0de8be1e103e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1012)

版本升级

查看远程仓库，标签数已经有2个了，点进去就可以看到0.2.0，这里我们就不去看了



![img](https://upload-images.jianshu.io/upload_images/2144614-c4c0227164c38207.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

## 二、修改描述文件并更新索引库

### 1、修改Sepc

打开你的`xx.podspec`文件，将原本的版本号改为`0.2.0`，与刚刚的tag保持一致

```bash
s.version = '0.2.0'
```

### 2、验证远程Spec

```cpp
$ pod spec lint --private --allow-warnings
```

![1.png](https://upload-images.jianshu.io/upload_images/843214-b819957969b66c4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

验证远程Spec

### 3、更新索引库

```css
pod repo push 索引库名称 xxx.podspec
```

![2.png](https://upload-images.jianshu.io/upload_images/843214-1df306516967ce83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

//... ...

![3.png](https://upload-images.jianshu.io/upload_images/843214-7f191b1140d0047d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更新索引库



> 如果出现了类似`[!] The `MyLibrary.podspec` specification does not validate.` 的报错，可以使用 `pod repo push DZSpace MyLibrary.podspec --allow-warnings --verbose` 来查看具体问题出来哪里，
>
> 修改还是报错，这个时候我们修改清除pod的缓存，因为使用pod repo push的使用，pod会从缓存中复制工程，然后在编译验证。
>
> 所以修改后总是报错，清除缓存或直接删除重新执行就OK，
>
> 
>
> pod 查看缓存命令：
> 使用 `pod cache list` 查看看缓存
>
> 使用 `pod cache clean --all` 清除缓存
>
>  
>
> 来自：[制作Cocoapods管理自己的类库](https://blog.csdn.net/dlm_211314/article/details/83580838)





## 三、更新使用

```cpp
// --no-repo-update 不更新本地索引库
// 因为刚刚已经自己手动更新过了，所以这里我们选择跳过更新
pod update --no-repo-update
```

![img](https://upload-images.jianshu.io/upload_images/2144614-95ceb8c7d86d3bd3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1134)

更新框架

![img](https://upload-images.jianshu.io/upload_images/2144614-825190c5bd0a5532.png?imageMogr2/auto-orient/strip|imageView2/2/w/642)

更新成功



------



## 四、第三方依赖

当我们的私有库需要依赖其它第三方才可以正常使用时，我们就需要在spec文件中开启依赖，例如下面所示代码，表明当前仓库需要依赖AFN和SDWebImage

```bash
s.dependency 'AFNetworking', '~> 3.2.0'
s.dependency 'SDWebImage', '~> 4.3.3'
```

修改后更新操作同上所述，这里就不再赘述了。





但是这里存在一个问题，如果来了一位新的小伙伴，他所负责的部分只需要LXFBase下的Category，而LXFBase下的Cache才需要依赖SDWebImage，此时他若是pod一整个LXFBase岂不是平白无故安装了第三方依赖库，那应该怎么做呢？

> 方案就是可以通过子库Subspecs来解决因需要一个小小的工具而依赖整个基础组件的问题

## 五、子库Subspecs

什么是Subspecs？这里我们可以搜索一下SDWebImage

```bash
pod search 'SDWebImage'
```

![img](https://upload-images.jianshu.io/upload_images/2144614-41558a53974a19fc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1150)

Subspecs

可以看到，如果我们只需要用到SDWebImage中的GIF功能，那么并不需要将整个SDWebImage都下载下来，在Podfile中将~~`pod 'SDWebImage'`~~ 改为 `pod SDWebImage/GIF`即可单独使用这一功能

那接下来我们就来看看怎么描述一个子库吧

### 子库格式

```ruby
s.subspec '子库名称' do |别名|

end
```

因为这里已经分离出子库了，所以`s.source_files`和`s.dependency`就不能这么使用了，需要我们在子库里分别指定，所以我们直接把原来的`s.source_files`和`s.dependency`都注释掉。写法参考如下

```ruby
# s.source_files = 'LXFBase/Classes/**/*'
# s.dependency 'SDWebImage', '~> 4.3.3'

s.subspec 'Cache' do |c|
  c.source_files = 'LXFBase/Classes/Cache/**/*'
  c.dependency 'SDWebImage', '~> 4.3.3'
end

s.subspec 'Category' do |c|
  c.source_files = 'LXFBase/Classes/Category/**/*'
end

s.subspec 'Tool' do |t|
  t.source_files = 'LXFBase/Classes/Tool/**/*'
end
```

修改后再按之前的步骤更新索引库和组件库就可以了

**ps: 在添加第三方依赖描述后做验证或者上传操作可能会很慢，因为它在克隆第三方库如SDWebImage，有兴趣的可以在命令后面加入`--verbose`来查看详情情况**

```cpp
pod spec lint --private --verbose
```

在成功更新组件库和索引库后我们再来搜索一下试试

```bash
pod search 'LXFBase'
```

![img](https://upload-images.jianshu.io/upload_images/2144614-f4124af7d2d2ad49.png?imageMogr2/auto-orient/strip|imageView2/2/w/980)

subspec添加成功

现在就可以爱装哪个就装哪个了，在Podfile中指定要安装的子库就行了

```bash
pod 'LXFBase/Cache'
```

```bash
pod install
```



![img](https://upload-images.jianshu.io/upload_images/2144614-92735ae4531f4e58.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

安装指定子库与依赖库





---

【完】