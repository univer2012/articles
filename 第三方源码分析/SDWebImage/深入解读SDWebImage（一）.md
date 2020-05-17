来自：[深入解读SDWebImage（一）](https://www.jianshu.com/p/efe8565c4218)



---



![img](https:////upload-images.jianshu.io/upload_images/2761218-622240b53d85e6ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/976)

解读SDWebImage



SDWebImage具有缓存支持的异步映像下载程序。并添加了像UI元素分类类UIImageView、UIButton、MKAnnotationView，可以直接为这些UI元素添加图片。

# 首先我们看一下SDWebImage中的主要功能：

## 功能

1、对`UIImageView`、`UIButton`、`MKAnnotationView`添加Web图像和高速缓存管理
 2、异步图像下载器
 3、具有自动缓存到期处理的异步内存+磁盘映像缓存
 4、背景图像解压缩
 5、对动画图像的支持
 6、可以自定义和组合的转换，可在下载后立即应用于图像
 7、可以自定义加载器（如照片库）来扩展图像加载功能
 8、加载中的indicator显示
 9、保证不会下载相同的URL
 10、下载过程或者资源保存过程用到了GCD和ARC
 11、提前将获取到的图片放到主线程，保证不会阻塞主线程

## 支持的图片格式

1、UIImage（JPEG，PNG，...）支持的图像格式，包括GIF；
 2、WebP格式，包括动画WebP；
 3、支持其它图片格式，包括APNG，BPG...

## 模块化

装载机

- [SDWebImagePhotosPlugin](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FSDWebImage%2FSDWebImagePhotosPlugin) - 支持从照片加载图像的插件

与第三方库集成

- [SDWebImageFLPlugin](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FSDWebImage%2FSDWebImageFLPlugin) - 支持[FLAnimatedImage](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FFlipboard%2FFLAnimatedImage)作为动画GIF引擎的插件
- [SDWebImageYYPlugin](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FSDWebImage%2FSDWebImageYYPlugin) - 用于集成[YYImage](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fibireme%2FYYImage)和[YYCache](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fibireme%2FYYCache)以进行图像渲染和缓存的插件
- [SDWebImageProgressiveJPEGDemo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FSDWebImage%2FSDWebImageProgressiveJPEGDemo) - 使用`SDWebImage`+ [Concorde库](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fcontentful-labs%2FConcorde)进行逐行JPEG解码的演示项目

# SDWebImage源码分析

![img](https:////upload-images.jianshu.io/upload_images/2761218-d71f645479563705.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/1024)

SDWebImage架构图



![img](https:////upload-images.jianshu.io/upload_images/2761218-10ab2a42100962cc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

时序图

从SDWebImage中提供的架构图中我们可以大概的看出，整个库分为两层，Top Level、Base Module。

Top Level：当`UIImageView`调用加载image方法，会进入`SDWebImage`中的`UIImageView`分类。在分类中调用负责加载UIImage的核心代码块ImageManager，其主要负责调度Image Cache/Image Loader，这两者分别从缓存或者网络端加载图片，并且又进行了细分。Cache中获取图片业务，拆分到了memory/disk两个分类中；Image Loader中又分为从网络端获取或者从系统的Photos中获取。

Base Module：获取到图片的二进制数据处理，二进制解压缩，二进制中格式字节判断出具体的图片类型。

- 核心类
  - `UIImageView+WebCache`：SDWebImage加载图片的入口，随后都会进入`sd_setImageWithURL: placeholderImage:`中。
  - `UIView+WebCache`：由于分类中不能添加成员变量，所以runtime关联了`sd_imageURL`图片的url、`sd_imageProgress`下载进度；`sd_cancelCurrentImageLoad`方法取消当前下载，`sd_imageIndicator`设置当前加载动画，`sd_internalSetImageWithURL:`内部通过`SDWebImageManager`调用加载图片过程并返回调用进度
  - `SDWebImageManager`：内部分别由`SDImageCache`、`SDWebImageDownloader`来处理下载任务。



## 一、UIImageView+WebCache层

面向UIImageView的是`UIImageView+WebCache`这个分类，将图片的URL，占位图片直接给这个类，下面是这个类的公共接口：

```objc
- (void)sd_setImageWithURL:(nullable NSURL *)url NS_REFINED_FOR_SWIFT;

- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder NS_REFINED_FOR_SWIFT;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options NS_REFINED_FOR_SWIFT;

- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                   context:(nullable SDWebImageContext *)context;

- (void)sd_setImageWithURL:(nullable NSURL *)url
                 completed:(nullable SDExternalCompletionBlock)completedBlock;


- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                 completed:(nullable SDExternalCompletionBlock)completedBlock NS_REFINED_FOR_SWIFT;

- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                 completed:(nullable SDExternalCompletionBlock)completedBlock;

- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock;

- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                   context:(nullable SDWebImageContext *)context
                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
```

其中这几个方法最终都会进入

```objc
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                   context:(nullable SDWebImageContext *)context
                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
```



`SDWebImageOptions`，设置的图片加载以及缓存策略，有几种类型

```objc
typedef NS_OPTIONS(NSUInteger, SDWebImageOptions) {
    /**
     默认情况下，当一个URL下载失败的时候，这个URL会被加入黑名单列表，
     下次再有这个url的请求则停止请求。
     如果为true，这个值表示需要再尝试请求。
     */
    SDWebImageRetryFailed = 1 << 0,
    
    /**
     默认情况下，当UI可以交互的时候就开始加载图片。这个标记可以阻止这个时候加载。
     而是当UIScrollView开始减速滑动的时候开始加载。
     */
    SDWebImageLowPriority = 1 << 1,
    
    /**
     这个flag启动渐进式下载图像，类似浏览器加载图像那样逐步显示（从上倒下加载）
     */
    SDWebImageProgressiveLoad = 1 << 2,
    
    /**
     一个图片缓存了，还是会重新请求.并且缓存侧略依据NSURLCache而不是SDImageCache。即在URL没变但是服务器图片发生更新时使用，这时我们需要在NSMutableRequest中设置上（If-Modified-Since-其实是缓存的最后修改时间，有后台返回），这个参数是在上一次网络请求之后，NSResponse中的Last-Modified获取，并保存下载，在下次发送网络请求的时候添加到请求头中；这样当我们下载同一张照片的时候，其实是拿取的上次GET请求的NSURLCache缓存中的图片，并且如果后台发生变化会重新请求图片
     */
    SDWebImageRefreshCached = 1 << 3,
    
    /**
     启动后台下载，实现原理是通过向系统询问后台的额外时间来完成请求的。 如果后台任务到期，则操作将被取消
     */
    SDWebImageContinueInBackground = 1 << 4,
    
    /**
     当设置了NSMutableURLRequest.HTTPShouldHandleCookies = YES时，可以控制存储NSHTTPCookieStorage中的cookie
     */
    SDWebImageHandleCookies = 1 << 5,
    
    /**
     允许不安全的SSL证书，用于测试环境，在正式环境中谨慎使用
     */
    SDWebImageAllowInvalidSSLCertificates = 1 << 6,
    
    /**
     默认情况下,image在加载的时候是按照他们在队列中的顺序装载的(就是先进先出)。这个flag会把他们移动到队列的前端,并且立刻装载,而不是等到当前队列装载的时候再装载。
     */
    SDWebImageHighPriority = 1 << 7,
    
    /**
    默认情况下,占位图会在图片下载的时候显示.这个flag开启会延迟占位图显示的时间,等到图片下载完成之后才会显示占位图.
     */
    SDWebImageDelayPlaceholder = 1 << 8,
    
    /**
     一般不会在动画图片上调用 transformDownloadedImage 代理方法，这样不能够管理动画图片
     这个flag为尝试转换动画图片
     */
    SDWebImageTransformAnimatedImage = 1 << 9,
    
    /**
     图片在下载后被加载到imageView。这个flag避免自动设置图片，来手动设置一下图片（引用一个滤镜或者加入透入动画）
     */
    SDWebImageAvoidAutoSetImage = 1 << 10,
    
    /**
     默认情况下，图像将根据其原始大小进行解码。 在iOS上，此flat会将图片缩小到与设备的受限内存兼容的大小。    但如果设置了SDWebImageAvoidDecodeImage则此flat不起作用。 如果设置了SDWebImageProgressiveLoad它将被忽略
     */
    SDWebImageScaleDownLargeImages = 1 << 11,
    
    /**
     结合SDWebImageQueryMemoryData设置同步查询图像数据（一般不建议这么使用，除非是在同一个runloop里避免单元格复用时发生闪现）
     */
    SDWebImageQueryMemoryData = 1 << 12,
    
    /**
     结合SDWebImageQueryMemoryData设置同步查询图像数据（一般不建议这么使用，除非是在同一个runloop里避免单元格复用时发生闪现）
     */
    SDWebImageQueryMemoryDataSync = 1 << 13,
    
    /**
     如果内存查询没有的时候，强制同步磁盘查询（这三个查询可以组合使用，一般不建议这么使用，除非是在同一个runloop里避免单元格复用时发生闪现）
     */
    SDWebImageQueryDiskDataSync = 1 << 14,
    
    /**
     * 默认情况下，当缓存丢失时，SD将从网络下载图像。 此flat可以防止这样，使其仅从缓存加载。
     */
    SDWebImageFromCacheOnly = 1 << 15,
    
    /**
     * 默认情况下，SD在下载之前先从缓存中查找，此flat可以防止这样，使其仅从网络下载
     */
    SDWebImageFromLoaderOnly = 1 << 16,
    
    /**
     * 默认情况下，SD在图像加载完成后使用SDWebImageTransition进行某些视图转换，此转换仅适用于从网络下载图像。 此flat可以强制为内存和磁盘缓存应用视图转换。
     */
    SDWebImageForceTransition = 1 << 17,
    
    /**
    默认情况下，SD在查询缓存和从网络下载时会在后台解码图像，这有助于提高性能，因为在屏幕上渲染图像时，需要首先对其进行解码。这发生在Core Animation的主队列中。然而此过程也可能会增加内存使用量。
     如果由于过多的内存消耗而遇到问题，可以用此flat禁止解码图像。
     */
    SDWebImageAvoidDecodeImage = 1 << 18,
    
    /**
     * 默认情况下，SD会解码动画图像，该flat强制只解码第一帧并生成静态图。
     */
    SDWebImageDecodeFirstFrameOnly = 1 << 19,
    
    /**
     默认情况下，对于SDAnimatedImage，SD会在渲染过程中解码动画图像帧以减少内存使用量。 但是用户可以指定将所有帧预加载到内存中，以便在大量imageView共享动画图像时降低CPU使用率。这实际上会在后台队列中触发preloadAllAnimatedImageFrames（仅限磁盘缓存和下载）。
     */
    SDWebImagePreloadAllFrames = 1 << 20
};
```

还有一个变量是`SDWebImageContext *context` 。可以看到`SDWebImageContext ` 其实就是以 `SDWebImageContextOption`为key、id(指定类型或者协议)为value 的`NSDictionary` 。

`SDWebImageContextOption`有下面的几种类型：

```objc
#pragma mark - Context option

SDWebImageContextOption const SDWebImageContextSetImageOperationKey = @"setImageOperationKey";
SDWebImageContextOption const SDWebImageContextCustomManager = @"customManager";
SDWebImageContextOption const SDWebImageContextImageCache = @"imageCache";
SDWebImageContextOption const SDWebImageContextImageLoader = @"imageLoader";
SDWebImageContextOption const SDWebImageContextImageCoder = @"imageCoder";
SDWebImageContextOption const SDWebImageContextImageTransformer = @"imageTransformer";
SDWebImageContextOption const SDWebImageContextImageScaleFactor = @"imageScaleFactor";
SDWebImageContextOption const SDWebImageContextImagePreserveAspectRatio = @"imagePreserveAspectRatio";
SDWebImageContextOption const SDWebImageContextImageThumbnailPixelSize = @"imageThumbnailPixelSize";
SDWebImageContextOption const SDWebImageContextStoreCacheType = @"storeCacheType";
SDWebImageContextOption const SDWebImageContextOriginalStoreCacheType = @"originalStoreCacheType";
SDWebImageContextOption const SDWebImageContextAnimatedImageClass = @"animatedImageClass";
SDWebImageContextOption const SDWebImageContextDownloadRequestModifier = @"downloadRequestModifier";
SDWebImageContextOption const SDWebImageContextDownloadResponseModifier = @"downloadResponseModifier";
SDWebImageContextOption const SDWebImageContextDownloadDecryptor = @"downloadDecryptor";
SDWebImageContextOption const SDWebImageContextCacheKeyFilter = @"cacheKeyFilter";
SDWebImageContextOption const SDWebImageContextCacheSerializer = @"cacheSerializer";
```

在`UIImageView+WebCache`内部调用的最终方法`sd_setImageWithURL:`进入到了`UIView+WebCache` 。下面我们看一下`UIView+WebCache`

## 二、UIView+WebCache

首先还是看看API：

```objc
@interface UIView (WebCache)

/**
 当前图片的URL
 * Get the current image URL.
 */
@property (nonatomic, strong, readonly, nullable) NSURL *sd_imageURL;

/**
 * 当前view中图片loading的下载进度
 可以用KVO监听当前图片的下载进度，注意更新UI要回到主线程
**/
@property (nonatomic, strong, null_resettable) NSProgress *sd_imageProgress;


- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                           context:(nullable SDWebImageContext *)context
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDImageLoaderProgressBlock)progressBlock
                         completed:(nullable SDInternalCompletionBlock)completedBlock;

/**
 * 取消当前图片的获取请求
 */
- (void)sd_cancelCurrentImageLoad;


#pragma mark - Image Transition

/**
 图片的过渡属性，内部包含了很多的属性duration、过渡的动画样式
 默认是nil
 */
@property (nonatomic, strong, nullable) SDWebImageTransition *sd_imageTransition;

/**
 图片loading的indicator图标，默认是nil
 */
@property (nonatomic, strong, nullable) id<SDWebImageIndicator> sd_imageIndicator;

@end
```

看一下具体的方法实现：

1. `sd_imageURL`属性，为属性重写setter、getter方法，并用关联对象保存成员变量的值

```objc
- (nullable NSURL *)sd_imageURL {
    return objc_getAssociatedObject(self, @selector(sd_imageURL));
}

- (void)setSd_imageURL:(NSURL * _Nullable)sd_imageURL {
    objc_setAssociatedObject(self, @selector(sd_imageURL), sd_imageURL, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

2. `sd_imageProgress` 进度的方法和上边的类似。

```objc
- (NSProgress *)sd_imageProgress {
    NSProgress *progress = objc_getAssociatedObject(self, @selector(sd_imageProgress));
    if (!progress) {
        progress = [[NSProgress alloc] initWithParent:nil userInfo:nil];
        self.sd_imageProgress = progress;
    }
    return progress;
}

- (void)setSd_imageProgress:(NSProgress *)sd_imageProgress {
    objc_setAssociatedObject(self, @selector(sd_imageProgress), sd_imageProgress, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

3. 接下来是最重要的`sd_internalSetImageWithURL:` 获取图片，并展示到`UIButton`、`UIImageView`上

```objc
- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                           context:(nullable SDWebImageContext *)context
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDImageLoaderProgressBlock)progressBlock
                         completed:(nullable SDInternalCompletionBlock)completedBlock {
    context = [context copy]; // copy to avoid mutable object
    
    NSString *validOperationKey = context[SDWebImageContextSetImageOperationKey];
    if (!validOperationKey) {
        validOperationKey = NSStringFromClass([self class]);
    }
    self.sd_latestOperationKey = validOperationKey;
    //1.判断是否有正在执行的任务
    //有正在进行的任务，那么取消下载任务
    [self sd_cancelImageLoadOperationWithKey:validOperationKey];
    self.sd_imageURL = url;
    
    //2.设置Placeholder
    //SDWebImageDelayPlaceholder
    //!(options & SDWebImageDelayPlaceholder）不需要延时加载
    if (!(options & SDWebImageDelayPlaceholder)) {
        
        //必须保证是在主线程中执行
        dispatch_main_async_safe(^{
            
            //更新placeholder，设置当前的uiimageview
            [self sd_setImage:placeholder imageData:nil basedOnClassOrViaCustomSetImageBlock:setImageBlock cacheType:SDImageCacheTypeNone imageURL:url];
        });
    }
    
    //3.进入SDWebImageManager来管理到是下载/缓存获取图片
    if (url) {
        //获取当前关联的NSProgress，并设置值为0
        NSProgress *imageProgress = objc_getAssociatedObject(self, @selector(sd_imageProgress));
        if (imageProgress) {
            imageProgress.totalUnitCount = 0;
            imageProgress.completedUnitCount = 0;
        }
        
#if SD_UIKIT || SD_MAC
        // check and start image indicator
        
        //开始图片indicator展示
        [self sd_startImageIndicator];
        id<SDWebImageIndicator> imageIndicator = self.sd_imageIndicator;
#endif
        
        SDWebImageManager *manager = context[SDWebImageContextCustomManager];
        if (!manager) {
            manager = [SDWebImageManager sharedManager];
        }
        
        //开始封装下载progerees的block，用于会掉
        SDImageLoaderProgressBlock combinedProgressBlock = ^(NSInteger receivedSize, NSInteger expectedSize, NSURL * _Nullable targetURL) {
            //不断更新当前imageProgress进度
            if (imageProgress) {
                imageProgress.totalUnitCount = expectedSize;
                imageProgress.completedUnitCount = receivedSize;
            }
#if SD_UIKIT || SD_MAC
            //假如imageIndicator存在更新updateIndicatorProgress:方法
            if ([imageIndicator respondsToSelector:@selector(updateIndicatorProgress:)]) {
                //获取进度
                double progress = 0;
                if (expectedSize != 0) {
                    progress = (double)receivedSize / expectedSize;
                }
                progress = MAX(MIN(progress, 1), 0); // 0.0 - 1.0
                //回到主线程更新imageIndicator
                dispatch_async(dispatch_get_main_queue(), ^{
                    [imageIndicator updateIndicatorProgress:progress];
                });
            }
#endif
            if (progressBlock) {
                progressBlock(receivedSize, expectedSize, targetURL);
            }
        };
        //开始下载图片
        @weakify(self);
        id <SDWebImageOperation> operation = [manager loadImageWithURL:url options:options context:context progress:combinedProgressBlock completed:^(UIImage *image, NSData *data, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
            @strongify(self);
            if (!self) { return; }
            // if the progress not been updated, mark it to complete state
            //图片加载进度字段
            if (imageProgress && finished && !error && imageProgress.totalUnitCount == 0 && imageProgress.completedUnitCount == 0) {
                imageProgress.totalUnitCount = SDWebImageProgressUnitCountUnknown;
                imageProgress.completedUnitCount = SDWebImageProgressUnitCountUnknown;
            }
            
#if SD_UIKIT || SD_MAC
            // 加载完成，停止indicator展示
            if (finished) {
                [self sd_stopImageIndicator];
            }
#endif
            
            //图片请求完成之后
            //可以设置SDWebImageAvoidAutoSetImage，是否可以设置图片
            BOOL shouldCallCompletedBlock = finished || (options & SDWebImageAvoidAutoSetImage);
            
            //是否需要自动设置图片/图片不存在并且立刻展示plachholder
            BOOL shouldNotSetImage = ((image && (options & SDWebImageAvoidAutoSetImage)) ||
                                      (!image && !(options & SDWebImageDelayPlaceholder)));
            SDWebImageNoParamsBlock callCompletedBlockClojure = ^{
                if (!self) { return; }
                
                //如果外部不需要设置图片，那么自动调整
                if (!shouldNotSetImage) {
                    [self sd_setNeedsLayout];
                }
                
                //否则加载完成之后会掉给外界
                if (completedBlock && shouldCallCompletedBlock) {
                    completedBlock(image, data, error, cacheType, finished, url);
                }
            };
            
            //1.设置了SDWebImageAvoidAutoSetImage
            //2.没有获得图片但是有SDWebImageDelayPlaceholder
            if (shouldNotSetImage) {
                dispatch_main_async_safe(callCompletedBlockClojure);
                return;
            }
            
            UIImage *targetImage = nil;
            NSData *targetData = nil;
            //设置图片
            //如果图片没有，并且placeholder延迟设置，那么将placeholder放到targetImage处
            if (image) {
                // case 2a: we got an image and the SDWebImageAvoidAutoSetImage is not set
                targetImage = image;
                targetData = data;
            
            } else if (options & SDWebImageDelayPlaceholder) {
                // case 2b: we got no image and the SDWebImageDelayPlaceholder flag is set
                targetImage = placeholder;
                targetData = nil;
            }
            
#if SD_UIKIT || SD_MAC
            // check whether we should use the image transition
            //检查是否需要图片的过渡动画Transition
            //SDWebImageForceTransition 强制进行过渡动画
            SDWebImageTransition *transition = nil;
            if (finished && (options & SDWebImageForceTransition || cacheType == SDImageCacheTypeNone)) {
                transition = self.sd_imageTransition;
            }
#endif
            dispatch_main_async_safe(^{
#if SD_UIKIT || SD_MAC
                //将获取到的图片image显示到对应的UIButton或者UIImageView中
                [self sd_setImage:targetImage imageData:targetData basedOnClassOrViaCustomSetImageBlock:setImageBlock transition:transition cacheType:cacheType imageURL:imageURL];
#else
                [self sd_setImage:targetImage imageData:targetData basedOnClassOrViaCustomSetImageBlock:setImageBlock cacheType:cacheType imageURL:imageURL];
#endif
                callCompletedBlockClojure();
            });
        }];
        //设置当前的下载operation到MapTable中
        [self sd_setImageLoadOperation:operation forKey:validOperationKey];
    } else {
#if SD_UIKIT || SD_MAC
        //假如URL不存在，停止indicator的展示
        [self sd_stopImageIndicator];
#endif
        //返回completedBlock
        dispatch_main_async_safe(^{
            if (completedBlock) {
                NSError *error = [NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorInvalidURL userInfo:@{NSLocalizedDescriptionKey : @"Image url is nil"}];
                completedBlock(nil, nil, error, SDImageCacheTypeNone, YES, url);
            }
        });
    }
}
```

当前方法中有几个点需要注意：

**第一**：取消当前的获取图片任务 `- sd_cancelCurrentImageLoad` 当前view中有一个MapTable列表，用于记录当前所有的下载任务，可内部调用方法`sd_cancelImageLoadOperationWithKey:`在方法内部，实现`[NSURLSessionTask cancel]`结束当前的任务。

```objc
- (void)sd_cancelImageLoadOperationWithKey:(nullable NSString *)key {
    if (key) {
        // Cancel in progress downloader from queue
        //NSMapTable 其实就是字典，key是强引用，value是弱引用
        SDOperationsDictionary *operationDictionary = [self sd_operationDictionary];
        id<SDWebImageOperation> operation;

        //递归锁防止数据冲突
        @synchronized (self) {
            operation = [operationDictionary objectForKey:key];
        }
        if (operation) {
            if ([operation conformsToProtocol:@protocol(SDWebImageOperation)]) {
                //取消任务  
                //内部调用[NSURLSessionTask cancel]结束当前的任务
                [operation cancel];
            }
            @synchronized (self) {
                
                [operationDictionary removeObjectForKey:key];
            }
        }
    }
}
```

**第二**：在该方法内部调用`[SDWebImageManager sharedManager]` ，用来处理接下来缓存数据和下载数据之间的逻辑，在后边我们在讲解。



## 三、sd_imageTransition属性

这个属性是<u>设置的图片的过渡属性</u>，内部包含了很多的属性duration、过渡的动画样式，下面是获取到image之后，通过Transition动画，将图片加载到当前的视图上，这个属性是需要开发者外部传递进来的。

```objc
//当拿到图片之后展示到NSButton、UIButton、UIImageView上的（包括提前设置placeholder也会调用这个方法）
//设置了过度动画
- (void)sd_setImage:(UIImage *)image imageData:(NSData *)imageData basedOnClassOrViaCustomSetImageBlock:(SDSetImageBlock)setImageBlock transition:(SDWebImageTransition *)transition cacheType:(SDImageCacheType)cacheType imageURL:(NSURL *)imageURL {
    UIView *view = self;
    SDSetImageBlock finalSetImageBlock;
    //进行判断是否是需要回掉block，如果是那么将block回掉
    if (setImageBlock) {
        finalSetImageBlock = setImageBlock;
    //否则直接判断是UIImageView或者是UIButton
    } else if ([view isKindOfClass:[UIImageView class]]) {
        UIImageView *imageView = (UIImageView *)view;
        finalSetImageBlock = ^(UIImage *setImage, NSData *setImageData, SDImageCacheType setCacheType, NSURL *setImageURL) {
            imageView.image = setImage;
        };
    }
#if SD_UIKIT
    //在手机端是UIButton
    else if ([view isKindOfClass:[UIButton class]]) {
        UIButton *button = (UIButton *)view;
        finalSetImageBlock = ^(UIImage *setImage, NSData *setImageData, SDImageCacheType setCacheType, NSURL *setImageURL) {
            [button setImage:setImage forState:UIControlStateNormal];
        };
    }
#endif
#if SD_MAC
    //在MAC中是NSButton
    else if ([view isKindOfClass:[NSButton class]]) {
        NSButton *button = (NSButton *)view;
        finalSetImageBlock = ^(UIImage *setImage, NSData *setImageData, SDImageCacheType setCacheType, NSURL *setImageURL) {
            button.image = setImage;
        };
    }
#endif
    
    if (transition) {
#if SD_UIKIT
        //过渡动画
        //UIViewAnimationOptions 中0代表  UIViewAnimationOptionLayoutSubviews作为子视图添加
        //
        [UIView transitionWithView:view duration:0 options:0 animations:^{
            //prepares是过渡动画开始之前的准备工作。所以在0 duration去渲染完之前的placeholder
            if (transition.prepares) {
                transition.prepares(view, image, imageData, cacheType, imageURL);
            }
        } completion:^(BOOL finished) {
            //当当前的过渡动画执行完之后，执行下面的过渡动画，才真正开始进行过渡
            [UIView transitionWithView:view duration:transition.duration options:transition.animationOptions animations:^{
                //finalSetImageBlock 最终设置图片的block，在这里执行图片的过渡效果
                if (finalSetImageBlock && !transition.avoidAutoSetImage) {
                    finalSetImageBlock(image, imageData, cacheType, imageURL);
                }
                //animations 过渡动画中，想要去改改变的某些对象
                if (transition.animations) {
                    transition.animations(view, image);
                }
            } completion:transition.completion];
        }];
#elif SD_MAC
//下边是Mac端
        [NSAnimationContext runAnimationGroup:^(NSAnimationContext * _Nonnull prepareContext) {
            // 0 duration to let AppKit render placeholder and prepares block
            prepareContext.duration = 0;
            if (transition.prepares) {
                transition.prepares(view, image, imageData, cacheType, imageURL);
            }
        } completionHandler:^{
            [NSAnimationContext runAnimationGroup:^(NSAnimationContext * _Nonnull context) {
                context.duration = transition.duration;
                context.timingFunction = transition.timingFunction;
                context.allowsImplicitAnimation = (transition.animationOptions & SDWebImageAnimationOptionAllowsImplicitAnimation);
                if (finalSetImageBlock && !transition.avoidAutoSetImage) {
                    finalSetImageBlock(image, imageData, cacheType, imageURL);
                }
                if (transition.animations) {
                    transition.animations(view, image);
                }
            } completionHandler:^{
                if (transition.completion) {
                    transition.completion(YES);
                }
            }];
        }];
#endif
    } else {
        if (finalSetImageBlock) {
            finalSetImageBlock(image, imageData, cacheType, imageURL);
        }
    }
}
```

其中有这些过渡Transition的类型，可以作为参考

```objc
typedef NS_OPTIONS(NSUInteger, UIViewAnimationOptions) {
    UIViewAnimationOptionLayoutSubviews            = 1 <<  0,
    UIViewAnimationOptionAllowUserInteraction      = 1 <<  1, // turn on user interaction while animating
    UIViewAnimationOptionBeginFromCurrentState     = 1 <<  2, // start all views from current value, not initial value
    UIViewAnimationOptionRepeat                    = 1 <<  3, // repeat animation indefinitely
    UIViewAnimationOptionAutoreverse               = 1 <<  4, // if repeat, run animation back and forth
    UIViewAnimationOptionOverrideInheritedDuration = 1 <<  5, // ignore nested duration
    UIViewAnimationOptionOverrideInheritedCurve    = 1 <<  6, // ignore nested curve
    UIViewAnimationOptionAllowAnimatedContent      = 1 <<  7, // animate contents (applies to transitions only)
    UIViewAnimationOptionShowHideTransitionViews   = 1 <<  8, // flip to/from hidden state instead of adding/removing
    UIViewAnimationOptionOverrideInheritedOptions  = 1 <<  9, // do not inherit any options or animation type
    
    UIViewAnimationOptionCurveEaseInOut            = 0 << 16, // default
    UIViewAnimationOptionCurveEaseIn               = 1 << 16,
    UIViewAnimationOptionCurveEaseOut              = 2 << 16,
    UIViewAnimationOptionCurveLinear               = 3 << 16,
    
    UIViewAnimationOptionTransitionNone            = 0 << 20, // default
    UIViewAnimationOptionTransitionFlipFromLeft    = 1 << 20,
    UIViewAnimationOptionTransitionFlipFromRight   = 2 << 20,
    UIViewAnimationOptionTransitionCurlUp          = 3 << 20,
    UIViewAnimationOptionTransitionCurlDown        = 4 << 20,
    UIViewAnimationOptionTransitionCrossDissolve   = 5 << 20,
    UIViewAnimationOptionTransitionFlipFromTop     = 6 << 20,
    UIViewAnimationOptionTransitionFlipFromBottom  = 7 << 20,

    UIViewAnimationOptionPreferredFramesPerSecondDefault     = 0 << 24,
    UIViewAnimationOptionPreferredFramesPerSecond60          = 3 << 24,
    UIViewAnimationOptionPreferredFramesPerSecond30          = 7 << 24,
    
} NS_ENUM_AVAILABLE_IOS(4_0);
```

具体的用法，比较简单，

```objc
///「注：下面这句代码没出现在源码中，是作者自己写的」

//图片设置fadeTransition类型的过渡效果
self.image.sd_imageTransition = SDWebImageTransition.fadeTransition;
```

`SDWebImageTransition`类是专门来处理过渡动画的，并在分类中设置了对应的过渡效果属性，可以自己看看哪种比较合适就用哪一种

```objc
/// Fade transition.
@property (nonatomic, class, nonnull, readonly) SDWebImageTransition *fadeTransition;
/// Flip from left transition.
@property (nonatomic, class, nonnull, readonly) SDWebImageTransition *flipFromLeftTransition;
/// Flip from right transition.
@property (nonatomic, class, nonnull, readonly) SDWebImageTransition *flipFromRightTransition;
/// Flip from top transition.
@property (nonatomic, class, nonnull, readonly) SDWebImageTransition *flipFromTopTransition;
/// Flip from bottom transition.
@property (nonatomic, class, nonnull, readonly) SDWebImageTransition *flipFromBottomTransition;
/// Curl up transition.
@property (nonatomic, class, nonnull, readonly) SDWebImageTransition *curlUpTransition;
/// Curl down transition.
@property (nonatomic, class, nonnull, readonly) SDWebImageTransition *curlDownTransition;
```

## 四、sd_imageIndicator

图片下载loading的indicator，主要是这两个方法，这个属性也是需要开发者自己去设置的

```objc
//对属性sd_imageIndicator 加载indicator添加关联对象
- (id<SDWebImageIndicator>)sd_imageIndicator {
    return objc_getAssociatedObject(self, @selector(sd_imageIndicator));
}

- (void)setSd_imageIndicator:(id<SDWebImageIndicator>)sd_imageIndicator {
    // 由于是之前添加了一个indicator，所以每次进来的时候
    //都需要移除之前的indicator
    id<SDWebImageIndicator> previousIndicator = self.sd_imageIndicator;
    [previousIndicator.indicatorView removeFromSuperview];
    
    //重新设置关联对象sd_imageIndicator
    objc_setAssociatedObject(self, @selector(sd_imageIndicator), sd_imageIndicator, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    
    //在当前UIImageView/UIButton中添加indicatorView
    //这样就展示出indicatorView来了
    UIView *view = sd_imageIndicator.indicatorView;
    if (CGRectEqualToRect(view.frame, CGRectZero)) {
        view.frame = self.bounds;
    }
    // Center the indicator view
#if SD_MAC
    CGPoint center = CGPointMake(NSMidX(self.bounds), NSMidY(self.bounds));
    NSRect frame = view.frame;
    view.frame = NSMakeRect(center.x - NSMidX(frame), center.y - NSMidY(frame), NSWidth(frame), NSHeight(frame));
#else
    //设置展示在当前的中心center位置
    view.center = CGPointMake(CGRectGetMidX(self.bounds), CGRectGetMidY(self.bounds));
#endif
    view.hidden = NO;
    [self addSubview:view];
}
```



`SDWebImage`中已经为我们封装好了对应的类`SDWebImageActivityIndicator`，以及相应的方法，来实现对应的loading中的indicator效果只需要按照下面代码进行调用，

```objc
///「注：下面这句代码没出现在源码中，是作者自己写的」

//设置indicator方式为grayLargeIndicator，会显示一个大的灰色indicator
self.image.sd_imageIndicator = SDWebImageActivityIndicator.grayLargeIndicator
```

在`SDWebImageActivityIndicator`对应的分类中还有很多类型，下面是所有的类型

```objc
/// These indicator use the fixed color without dark mode support
/// gray-style activity indicator
@property (nonatomic, class, nonnull, readonly) SDWebImageActivityIndicator *grayIndicator;
/// large gray-style activity indicator
@property (nonatomic, class, nonnull, readonly) SDWebImageActivityIndicator *grayLargeIndicator;
/// white-style activity indicator
@property (nonatomic, class, nonnull, readonly) SDWebImageActivityIndicator *whiteIndicator;
/// large white-style activity indicator
@property (nonatomic, class, nonnull, readonly) SDWebImageActivityIndicator *whiteLargeIndicator;
/// These indicator use the system style, supports dark mode if available (iOS 13+/macOS 10.14+)
/// large activity indicator
@property (nonatomic, class, nonnull, readonly) SDWebImageActivityIndicator *largeIndicator;
/// medium activity indicator
@property (nonatomic, class, nonnull, readonly) SDWebImageActivityIndicator *mediumIndicator;
```



总结一下在`UIView+WebCache`中用到的其它类：

1. `UIView+WebCacheOperation` - 内部的MapTable中，key保存了`NSStringFromClass([self class])` 视图类对象字符，value是`SDWebImageCombinedOperation` - 区分加载任务而生成的中间类。

   ```objc
   		NSString *validOperationKey = context[SDWebImageContextSetImageOperationKey];
       if (!validOperationKey) {
           validOperationKey = NSStringFromClass([self class]);
       }
       self.sd_latestOperationKey = validOperationKey;
       //1.判断是否有正在执行的任务
       //有正在进行的任务，那么取消下载任务
       [self sd_cancelImageLoadOperationWithKey:validOperationKey];
   ```

   

2. `SDWebImageManager` - 主要处理下一步具体图片的获取流程

3. `SDWebImageTransition` - 专门处理过渡动画

4. `SDWebImageActivityIndicator` - 处理loading过程中的indicator动画



下一篇：[深入理解SDWebImage（二）](https://www.jianshu.com/p/c49f9642efb4)



---

【完】