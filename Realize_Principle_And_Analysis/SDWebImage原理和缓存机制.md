来自：[SDWebImage原理和缓存机制](http://www.jianshu.com/p/7dea5b081d24)

`SDweSDWebImage`提供一个`UIImageView`的类别以支持加载来自互联网的远程图片。具有缓存管理、异步下载，同一个URL下载次数控制和优化等特征。

## 1 独立的异步图像下载

可能会用到单独的异步图片下载，则一定要用 `-downloadImageWithURL: options: progress: completed:` 来建立一个`SDWebImageDownLoader` 的实例。这样就可以有下载进度的回调和下载完成的回调，可以在回调完成进度条相关的操作和显示图片相关的操作。

## 2 独立的异步图像缓存

`SDImageCache`类提供一个管理缓存的单例类。

```objc
SDImageCache *imageCache = [SDImageCache sharedImageCache];
```



**查找和缓存图片时以URL作为key。(先查找内存，如果内存不存在该图片，再查找硬盘；查找硬盘时，以URL的MD5值作为key)。**

查找图片：

```objc
UIImage *cacheImage = [imageCache mageFromKey:myCacheKey];
```



缓存图片：

```objc
[ imageCache storeImage:myImage forKey:myCacheKey];
```



**默认情况下，图片是被存储到内存缓存和磁盘缓存中的**。如果仅仅是想缓存到内存中，可以用下面方法：
`storeImage:forKey:toDisk:` 第三个参数传NO即可。

## 3 主要用到的对象：

1. `UIImageView(WebCache)`，入口封装，实现读取图片完成后的回调。
2. `SDWebImagemanager`,对图片进行管理的中转站，记录那些图片正在读取。调用`SDImageCache` 向下层读取Cache，或者调用`SDWebImageDownloader` 向网络读取对象。实现`SDImageCache`和`SDWebImageDownLoader`的回调。
3. `SDImageCache`,根据URL作为key，对图片进行存储和读取（存在内存（以URL作为key）和存在硬盘两种（以URL的MD5值作为key））。实现图片和内存清理工作。

# 2 SDWebImage加载图片的流程

1. 入口 `setImageWithURL:placeholderImage:options:`会先把 `placeholderImage`显示，然后 `SDWebImageManager`根据 URL 开始处理图片。
2. 进入`SDWebImageManager` 类中调用`downloadWithURL:delegate:options:userInfo:`，交给
   `SDImageCache`，调用`queryDiskCacheForKey:delegate:userInfo:`从缓存查找图片是否已经下载。
3. 先从内存查找是否有图片，如果内存中已经有图片，`SDImageCacheDelegate`回调 `imageCache:didFindImage:forKey:userInfo:`到`SDWebImageManager`。
4. `SDWebImageManagerDelegate` 回调
   `webImageManager:didFinishWithImage:` 到 `UIImageView+WebCache`类扩展中，等前端展示图片。
5. 如果内存中没有图片，生成 `NSOperation`添加到队列，开始从硬盘查找是否已经缓存图片。
6. 根据 URL的MD5值作为Key，在硬盘缓存目录下尝试读取图片文件。这一步是在 `NSOperation` 进行的操作，所以回主线程进行结果回调 `notifyDelegate:`。
7. 如果从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小， 会先清空内存缓存）。`SDImageCacheDelegate`回调 `imageCache:didFindImage:forKey:userInfo:`，进而回调展示图片。
8. 如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片， 回调 `imageCache:didNotFindImageForKey:userInfo:`。
9. 共享或重新生成一个下载器 `SDWebImageDownloader`开始下载图片。
10. 图片下载由 `NSURLSession`来做，实现相关 delegate
    来判断图片下载中、下载完成和下载失败。

11.`-URLSession: dataTask: didReceiveData:` 中利用 ImageIO做了按图片下载进度加载效果。

1. `-URLSession: task: didCompleteWithError:` 数据下载完成后交给 `SDWebImageCodersManager`做图片解码处理。

13.图片解码处理在一个 `NSOperationQueue`完成，不会拖慢主线程 UI.如果有需要 对下载的图片进行二次处理，最好也在这里完成，效率会好很多。

14.在主线程 `notifyDelegateOnMainThreadWithInfo:`
宣告解码完成 `imageDecoder:didFinishDecodingImage:userInfo:` 回调给 SDWebImageDownloader`。