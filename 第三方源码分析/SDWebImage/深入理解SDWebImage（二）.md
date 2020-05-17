来自：[深入理解SDWebImage（二）](https://www.jianshu.com/p/c49f9642efb4)



---



![img](https:////upload-images.jianshu.io/upload_images/2761218-622240b53d85e6ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/976)

解读SDWebImage


 按照SDWebImage中给到的主流程，先进入`UIImageView+WebCache`，再进入`UIView+WebCache` 。这两个步骤已经在[深入理解SDWebImage（一）](https://www.jianshu.com/p/efe8565c4218)分析了，接下来主要分析一下SD中的核心 - `SDWebImageManager`。



## 核心方法



```objc
/**
 * 通过URL加载图片，如果cache中存在就从cache中获取，否则开始下载
 *
 * @param url            传入的image的url
 * @param options        获取图片的方式
 * @param context        获取
 * @param progressBlock  获得图片的进度（注意是在子队列中）
 * @param completedBlock  完成获取之后的回掉block
 * @return  返回一个SDWebImageCombinedOperation对象，用于表示当前的图片获取任务，在这个对象中可以取消获取图片任务
 */
- (nullable SDWebImageCombinedOperation *)loadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageOptions)options
                                                   context:(nullable SDWebImageContext *)context
                                                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                                 completed:(nonnull SDInternalCompletionBlock)completedBlock;
```

其中,`SDWebImageContext *`是在5.0中新引入的参数,可以灵活的自定义高级功能。



`SDWebImageContext` / `SDWebImageMutableContext` 是以 `SDWebImageContextOption` 为key、`id`(指定类型或者协议)为value 的`NSDictionary` / `NSMutableDictionary` 。

```objc
typedef void(^SDWebImageNoParamsBlock)(void);
typedef NSString * SDWebImageContextOption NS_EXTENSIBLE_STRING_ENUM;
typedef NSDictionary<SDWebImageContextOption, id> SDWebImageContext;
typedef NSMutableDictionary<SDWebImageContextOption, id> SDWebImageMutableContext;
```

并且其中的 `SDWebImageContextOption` 是一个可扩展的String枚举类型，确实是很好的代码，之前都没这么用过。我们看一下这个枚举中都有什么类型，从网上查询到的一个表格，[感觉总结的很全面,](https://www.jianshu.com/p/cfde8db5c051)

| Key                                      | Value                                                        |                             说明                             |
| :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------: |
| SDWebImageContextSetImageOperationKey    | NSString                                                     | 作为view类别的 operation key使用，用来存储图像下载的operation，用于支持不同图像加载过程的视图实例。如果为nil，则使用类名作为操作键。 |
| SDWebImageContextCustomManager           | SDWebImageManager                                            | 可以传入一个自定义的SDWebImageManager，默认使用[SDWebImageManager sharedManager] |
| SDWebImageContextImageTransformer        | id<SDImageTransformer>                                       | 可以传入一个SDImageTransformer类型，用于转换处理加载出来的图片，并将变换后的图像存储到缓存中。如果设置了，则会忽略manager中的transformer |
| SDWebImageContextImageScaleFactor        | CGFloat                                                      |   CGFloat原始值，为用于指定图像比例且这个数值应大于等于1.0   |
| SDWebImageContextStoreCacheType          | SDImageCacheType                                             | SDImageCacheType原始值，用于刚刚下载图像时指定缓存类型，并将其存储到缓存中。 指定SDImageCacheTypeNone：禁用缓存存储; SDImageCacheTypeDisk：仅存储在磁盘缓存中; SDImageCacheTypeMemory：只存储在内存中；SDImageCacheTypeAll：存储在内存缓存和磁盘缓存中。如果没有提供或值无效，则使用SDImageCacheTypeAll |
| SDWebImageContextAnimatedImageClass      | Class（UIImage / NSImage的子类并采用了SDAnimatedImage协议 ） | 用于使用SDAnimatedImageView来改善动画图像渲染性能（尤其是大动画图像上的内存使用） |
| SDWebImageContextDownloadRequestModifier | id<SDWebImageDownloaderRequestModifier>                      |               用于在加载图片前修改NSURLRequest               |
| SDWebImageContextCacheKeyFilter          | id<SDWebImageCacheKeyFilter>                                 |                      指定图片的缓存key                       |
| SDWebImageContextCacheSerializer         | id<SDWebImageCacheSerializer>                                | 转换需要缓存的图片格式，通常用于需要缓存的图片格式与下载的图片格式不相符的时候，如：下载的时候为了节约流量、减少下载时间使用了WebP格式，但是如果缓存也用WebP，每次从缓存中取图片都需要经过一次解压缩，这样是比较影响性能的，就可以使用id |
| SDWebImageContextLoaderCachedImage       | UIImage/NSImage<SDAnimatedImage>                             |                       SDImageLoader.m                        |

1. `SDWebImageContextOption.setImageOperationKey` - 其实是用来保存当前图片的获取任务服务的;
    在当前UIView+WebCacheOperation中存在一个NSMapTable关联对象，保存多个图片的加载任务，通过当前的key可以获取到保存在NSMapTable中的加载任务，执行后续的cancel/remove操作。

2. `SDWebImageContextOption.customManager` - 自定义的SDWebImageManager，默认使用[SDWebImageManager sharedManager]

3. `SDWebImageContextOption.imageTransformer` - 处理加载出来的图片,比如翻转圆角等

4. `SDWebImageContextOption.imageScaleFactor` - 当图片解压缩完之后的图片放大比例

5. `SDWebImageContextOption.storeCacheType` - 图片的缓存规则

6. `SDWebImageContextOption.downloadRequestModifier` - 可以用于在加载图片前修改NSURLRequest

7. `SDWebImageContextOption.cacheKeyFilter` - 指定图片的缓存key

8. `SDWebImageContextOption.cacheSerializer `- 转换需要缓存的图片格式

9. `SDWebImageContextOption.loaderCachedImage` - 这个值定义在SDImageLoader.m中，传入一个UIImage的缓存对象,一般用不到.

这些contextOption可以灵活运用，实现我们想要的某些功能。



### SDWebImageCombinedOperation

当前类 `SDWebImageCombinedOperation` 遵循 `<SDWebImageOperation>` 协议。在当前类中，可以拿到cache Operation或者是load Operation任务，然后取消任务。

通过上面的`SDWebImageContextOption.setImageOperationKey` ，可以获取到NSMapTable的value，该value就是`SDWebImageCombinedOperation`类型

```objc
@interface SDWebImageCombinedOperation : NSObject <SDWebImageOperation>

/**
 取消当前的operation任务
 */
- (void)cancel;

/**
 表示当前的图片是从缓存中查找到的，将从缓存中获取任务放到cacheOperation属性中
 */
@property (strong, nonatomic, nullable, readonly) id<SDWebImageOperation> cacheOperation;

/**
 当前的图片需要下载,将下载任务放入loaderOperation属性中
 */
@property (strong, nonatomic, nullable, readonly) id<SDWebImageOperation> loaderOperation;

@end
```

其中

-  `cacheOperation` ：当执行完从cache中获取任务，会回调`SDWebImageOperation`当前的缓存任务。
-  `loaderOperation`：当执行下载任务，会返回当前的下载任务Operation，用于后续的取消操作。

```objc
//SDWebImageCombinedOperation
//当前对象中取消当前的下载任务
- (void)cancel {
    @synchronized(self) {
        if (self.isCancelled) {
            return;
        }
        self.cancelled = YES;
        //是从缓存中读取数据 SDImageCache
        if (self.cacheOperation) {
            [self.cacheOperation cancel];
            self.cacheOperation = nil;
        }
        //下载中读取数据 SDWebImageDownloadToken
        //取消NSOperation中的任务
        //取消SDWebImageDownloaderOperation中的自定义回调任务
        if (self.loaderOperation) {
            [self.loaderOperation cancel];
            self.loaderOperation = nil;
        }
        //从当前正在执行的operation列表中，移除当前的SDWebImageCombinedOperation任务
        [self.manager safelyRemoveOperationFromRunning:self];
    }
}
```



## SDWebImageManager

`SDWebImageManager`是当前`SDWebImage`的核心，通过当前的单例对象，可以合理的安排图片获取任务，以及图片的获取逻辑，看一看其`.h`中暴露了什么：

```objc
/**
 当前SDWebImageManagerDelegate代理方法,默认是nil
 */
@property (weak, nonatomic, nullable) id <SDWebImageManagerDelegate> delegate;

/**
 图片缓存,被用来去查询图片的缓存,默认是SDImageCache
 */
@property (strong, nonatomic, readonly, nonnull) id<SDImageCache> imageCache;

/**
 下载图片,被用来去下载图片,默认是SDWebImageDownloader
 */
@property (strong, nonatomic, readonly, nonnull) id<SDImageLoader> imageLoader;

/**
 被用来处理下载下来的图片,如果想要设置可以在UIView+webCache中的context中设置SDWebImageContextImageTransformer,来决定transform,包括去圆角/翻转等
 */
@property (strong, nonatomic, nullable) id<SDImageTransformer> transformer;

/**
 图片的缓存key默认是url,通过设置该选项重新指定图片的缓存key,相关实现代码
* @code
 SDWebImageManager.sharedManager.cacheKeyFilter =[SDWebImageCacheKeyFilter cacheKeyFilterWithBlock:^NSString * _Nullable(NSURL * _Nonnull url) {
    url = [[NSURL alloc] initWithScheme:url.scheme host:url.host path:url.path];
    return [url absoluteString];
 }];
 */
@property (nonatomic, strong, nullable) id<SDWebImageCacheKeyFilter> cacheKeyFilter;

/**
 * The default value is nil. Means we just store the source downloaded data to disk cache.
 默认值是nil,通过设置该值,可以将下载好的数据压缩成指定格式,保存到disk中
* @code
 SDWebImageManager.sharedManager.cacheSerializer = [SDWebImageCacheSerializer cacheSerializerWithBlock:^NSData * _Nullable(UIImage * _Nonnull image, NSData * _Nullable data, NSURL * _Nullable imageURL) {
    SDImageFormat format = [NSData sd_imageFormatForImageData:data];
    switch (format) {
        case SDImageFormatWebP:
            return image.images ? data : nil;
        default:
            return data;
    }
}];
 */
@property (nonatomic, strong, nullable) id<SDWebImageCacheSerializer> cacheSerializer;

/**
 //对所有的图片请求options设置,都可以统一放到当前的属性中进行
@code
 SDWebImageManager.sharedManager.optionsProcessor = [SDWebImageOptionsProcessor optionsProcessorWithBlock:^SDWebImageOptionsResult * _Nullable(NSURL * _Nullable url, SDWebImageOptions options, SDWebImageContext * _Nullable context) {
     // Only do animation on `SDAnimatedImageView`
     if (!context[SDWebImageContextAnimatedImageClass]) {
        options |= SDWebImageDecodeFirstFrameOnly;
     }
     // Do not force decode for png url
     if ([url.lastPathComponent isEqualToString:@"png"]) {
        options |= SDWebImageAvoidDecodeImage;
     }
     // Always use screen scale factor
     SDWebImageMutableContext *mutableContext = [NSDictionary dictionaryWithDictionary:context];
     mutableContext[SDWebImageContextImageScaleFactor] = @(UIScreen.mainScreen.scale);
     context = [mutableContext copy];
 
     return [[SDWebImageOptionsResult alloc] initWithOptions:options context:context];
 }];
 */
@property (nonatomic, strong, nullable) id<SDWebImageOptionsProcessor> optionsProcessor;

/**
 判断当前是否有正在执行的任务
 */
@property (nonatomic, assign, readonly, getter=isRunning) BOOL running;

/**
 默认是nil,设置完成之后可以通过SDImageCache.sharedImageCache获取
 */
@property (nonatomic, class, nullable) id<SDImageCache> defaultImageCache;

/**
 //默认是nil,设置完成之后可以通过SDWebImageDownloader.sharedDownloader获取
 */
@property (nonatomic, class, nullable) id<SDImageLoader> defaultImageLoader;

/**
 //放回当前SDWebImageManager单例对象
 */
@property (nonatomic, class, readonly, nonnull) SDWebImageManager *sharedManager;

/**
 //传入cache/loader初始化当前对象
 */
- (nonnull instancetype)initWithCache:(nonnull id<SDImageCache>)cache loader:(nonnull id<SDImageLoader>)loader NS_DESIGNATED_INITIALIZER;

/**
 *对外界暴露的加载图片接口1
 *
 * @param url           URL
 * @param options        图片的加载样式
 * @param progressBlock  进度block,默认在后台队列中
 * @param completedBlock  完成回调的block

 * @return Returns 返回SDWebImageCombinedOperation对象,用户直接操作当前的加载任务
 */
- (nullable SDWebImageCombinedOperation *)loadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageOptions)options
                                                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                                 completed:(nonnull SDInternalCompletionBlock)completedBlock;

/**
 *暴露给外界的加载图片接口2
 * @param url            URL
 * @param options        图片的加载样式
 * @param context        context内容枚举 `SDWebImageContextOption`设置图片样式/缓存格式/
cache的key
 * @param progressBlock  进度block,默认在后台队列中
 * @param completedBlock  完成回调的block
 *
 * @return Returns an instance of SDWebImageCombinedOperation, which you can cancel the loading process.
 */
- (nullable SDWebImageCombinedOperation *)loadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageOptions)options
                                                   context:(nullable SDWebImageContext *)context
                                                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                                 completed:(nonnull SDInternalCompletionBlock)completedBlock;

/**
 //取消所有的当前operations
 */
- (void)cancelAll;

/**
 //通过一个URL,返回一个cache中的key
 */
- (nullable NSString *)cacheKeyForURL:(nullable NSURL *)url;
```

在`.m`中又添加了几个私有属性

```objc
//修改为readwrite
@property (strong, nonatomic, readwrite, nonnull) SDImageCache *imageCache;
@property (strong, nonatomic, readwrite, nonnull) id<SDImageLoader> imageLoader;
//NSMutableSet 保存当前的请求中已经失败的URL
@property (strong, nonatomic, nonnull) NSMutableSet<NSURL *> *failedURLs;
//创建网络失败的信号量,防止数据冲突
@property (strong, nonatomic, nonnull) dispatch_semaphore_t failedURLsLock; 
//当前所有的加载图片任务
@property (strong, nonatomic, nonnull) NSMutableSet<SDWebImageCombinedOperation *> *runningOperations;
//创建加载中的信号量,防止数据冲突
@property (strong, nonatomic, nonnull) dispatch_semaphore_t runningOperationsLock;
```

我们来看一下当前类中的核心方法,这个方法是接收options类型,context信息,回调信息,url信息.
 主要判断当前的url是否合法;
 创建加载图片任务operation,用于以后对当前任务进行操作;
 从请求失败的set中查找当前的url，判断是否需要继续执行下面的步骤
 如果需要执行,那么将url放到加载中的operations里,以便对当前任务进行操作

```objc
- (SDWebImageCombinedOperation *)loadImageWithURL:(nullable NSURL *)url
                                          options:(SDWebImageOptions)options
                                          context:(nullable SDWebImageContext *)context
                                         progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                        completed:(nonnull SDInternalCompletionBlock)completedBlock {
    // 外部警告completedBlock为nil
    NSAssert(completedBlock != nil, @"If you mean to prefetch the image, use -[SDWebImagePrefetcher prefetchURLs] instead");

    // 对url的处理
    if ([url isKindOfClass:NSString.class]) {
        url = [NSURL URLWithString:(NSString *)url];
    }

   //如果为NSNull,那么可能会造成crash
    if (![url isKindOfClass:NSURL.class]) {
        url = nil;
    }

    //为了在当前的单例类中区分每次的加载图片任务，需要设置一个operation，表示一个加载任务
    SDWebImageCombinedOperation *operation = [SDWebImageCombinedOperation new];
    operation.manager = self;

    //从请求失败的set中查找当前的URL，并加锁
    BOOL isFailedUrl = NO;
    if (url) {
        //SD_LOCK 其实是一个信号量宏 - dispatch_semaphore_wait
        SD_LOCK(self.failedURLsLock);
        isFailedUrl = [self.failedURLs containsObject:url];
        SD_UNLOCK(self.failedURLsLock);
    }

    //假如从失败set中查找到了，并且没有设置失败之后的再次请求
    if (url.absoluteString.length == 0 || (!(options & SDWebImageRetryFailed) && isFailedUrl)) {
        //包装返回值，进行返回
        [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorInvalidURL userInfo:@{NSLocalizedDescriptionKey : @"Image url is nil"}] url:url];
        return operation;
    }
    
    //对当前正在加载中的operations的set加锁，并将当前的operation放入runningOperations
    SD_LOCK(self.runningOperationsLock);
    [self.runningOperations addObject:operation];
    SD_UNLOCK(self.runningOperationsLock);
    
    //将options和context放到统一的对象SDWebImageOptionsResult中进行管理
    SDWebImageOptionsResult *result = [self processedResultForURL:url options:options context:context];
    
    //开始去从cache/load加载图片
    [self callCacheProcessForOperation:operation url:url options:result.options context:result.context progress:progressBlock completed:completedBlock];

    return operation;
}
```

开始缓存查询UIImage或者下载UIImage

```objc
- (void)callCacheProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                 url:(nonnull NSURL *)url
                             options:(SDWebImageOptions)options
                             context:(nullable SDWebImageContext *)context
                            progress:(nullable SDImageLoaderProgressBlock)progressBlock
                           completed:(nullable SDInternalCompletionBlock)completedBlock {
    // Check whether we should query cache
    //首先判断是否需要去查询缓存
    BOOL shouldQueryCache = (options & SDWebImageFromLoaderOnly) == 0;
    if (shouldQueryCache) {
        //cache对应的url是否自定义
        id<SDWebImageCacheKeyFilter> cacheKeyFilter = context[SDWebImageContextCacheKeyFilter];
        //传入url,通过外部重新设置的格式,来重新生成cache key
        NSString *key = [self cacheKeyForURL:url cacheKeyFilter:cacheKeyFilter];
        @weakify(operation);
        
        //operation的当前缓存当前缓存查找顺序
        //在SDImageCache中,查找对应的缓存图片信息
        operation.cacheOperation = [self.imageCache queryImageForKey:key options:options context:context completion:^(UIImage * _Nullable cachedImage, NSData * _Nullable cachedData, SDImageCacheType cacheType) {
            @strongify(operation);
            if (!operation || operation.isCancelled) {
                //在runningOperations中移除operation
                [self safelyRemoveOperationFromRunning:operation];
                return;
            }
            // 拿到缓存data,进行下一步的操作
            [self callDownloadProcessForOperation:operation url:url options:options context:context cachedImage:cachedImage cachedData:cachedData cacheType:cacheType progress:progressBlock completed:completedBlock];
        }];
    } else {
         // 开始进行下载任务
        [self callDownloadProcessForOperation:operation url:url options:options context:context cachedImage:nil cachedData:nil cacheType:SDImageCacheTypeNone progress:progressBlock completed:completedBlock];
    }
}
```

方法中首先判断需不需要进行下载,如果不需要下载直接将缓存/nil返回,如果需要下载,判断当前是否设置了SDWebImageRefreshCached,如果设置了,那么先将缓存图片返回,并进行异步的图片加载.
 加载完毕之后,如果返回的code是SDWebImageErrorCacheNotModified,则说明后台的图片是没有更新的,不用处理;否则按照请求结果来回调image或者nil

```objc
//下载
- (void)callDownloadProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                    url:(nonnull NSURL *)url
                                options:(SDWebImageOptions)options
                                context:(SDWebImageContext *)context
                            cachedImage:(nullable UIImage *)cachedImage
                             cachedData:(nullable NSData *)cachedData
                              cacheType:(SDImageCacheType)cacheType
                               progress:(nullable SDImageLoaderProgressBlock)progressBlock
                              completed:(nullable SDInternalCompletionBlock)completedBlock {
    //判断是否只从缓存获取
    BOOL shouldDownload = (options & SDWebImageFromCacheOnly) == 0;
    //判断是否需要结合SDWebImageRefreshCached,来读取缓存
    shouldDownload &= (!cachedImage || options & SDWebImageRefreshCached);
    //self.delegate中是否能够响应响应的方法
    shouldDownload &= (![self.delegate respondsToSelector:@selector(imageManager:shouldDownloadImageForURL:)] || [self.delegate imageManager:self shouldDownloadImageForURL:url]);
    //判断下载url是否有效
    shouldDownload &= [self.imageLoader canRequestImageForURL:url];
    if (shouldDownload) {
        //设置了SDWebImageRefreshCached,那么需要去先将之前缓存的image回调
        if (cachedImage && options & SDWebImageRefreshCached) {
            //将缓存图片回调
            [self callCompletionBlockForOperation:operation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
            SDWebImageMutableContext *mutableContext;
            if (context) {
                mutableContext = [context mutableCopy];
            } else {
                mutableContext = [NSMutableDictionary dictionary];
            }
            mutableContext[SDWebImageContextLoaderCachedImage] = cachedImage;
            context = [mutableContext copy];
        }
       
        @weakify(operation);
        //在SDWebImageDownloader中,下载图片
        operation.loaderOperation = [self.imageLoader requestImageWithURL:url options:options context:context progress:progressBlock completed:^(UIImage *downloadedImage, NSData *downloadedData, NSError *error, BOOL finished) {
            @strongify(operation);
            //假如取消了网络图片下载请求,什么都不做
            if (!operation || operation.isCancelled) {
               
            } else if (cachedImage && options & SDWebImageRefreshCached && [error.domain isEqualToString:SDWebImageErrorDomain] && error.code == SDWebImageErrorCacheNotModified) {
                //设置了SDWebImageRefreshCached,并且cachedImage存在,如果error.code == SDWebImageErrorCacheNotModified,则说明图片没有更改,不用刷新
                //上面网络请求之前,已经将cacheImage返回到了completeBlock中
            } else if (error) {
                //网络请求失败
                [self callCompletionBlockForOperation:operation completion:completedBlock error:error url:url];
                BOOL shouldBlockFailedURL = [self shouldBlockFailedURLWithURL:url error:error];
                
                if (shouldBlockFailedURL) {
                    SD_LOCK(self.failedURLsLock);
                    [self.failedURLs addObject:url];
                    SD_UNLOCK(self.failedURLsLock);
                }
            } else {
                //如果设置了SDWebImageRetryFailed,那么以后多次对失败的URL进行网络请求
                if ((options & SDWebImageRetryFailed)) {
                    SD_LOCK(self.failedURLsLock);
                    [self.failedURLs removeObject:url];
                    SD_UNLOCK(self.failedURLsLock);
                }
                //将下载好的图片缓存
                [self callStoreCacheProcessForOperation:operation url:url options:options context:context downloadedImage:downloadedImage downloadedData:downloadedData finished:finished progress:progressBlock completed:completedBlock];
            }
            
            if (finished) {
                [self safelyRemoveOperationFromRunning:operation];
            }
        }];
    } else if (cachedImage) {
        //不用下载,但是有缓存,直接将缓存结果返回
        [self callCompletionBlockForOperation:operation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
        [self safelyRemoveOperationFromRunning:operation];
    } else {
        //图片没有缓存,并且又不能进行下载请求,直接将结果nil
        //比如设置了SDWebImageFromCacheOnly,但是第一次进入的时候没有图片,那么永远拿不到数据
        [self callCompletionBlockForOperation:operation completion:completedBlock image:nil data:nil error:nil cacheType:SDImageCacheTypeNone finished:YES url:url];
        [self safelyRemoveOperationFromRunning:operation];
    }
}
```

下载完图片之后将下载好的图片进行缓存,分为两种类型:
 缓存原始图片/缓存Transform处理的图片,context中包含的SDWebImageCacheKeyFilter/SDImageTransformer/SDWebImageCacheSerializerkey,在方法调用地方可以对context进行分别设置.
 最终进入imageCache进行图片的保存

```objc
- (void)callStoreCacheProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                      url:(nonnull NSURL *)url
                                  options:(SDWebImageOptions)options
                                  context:(SDWebImageContext *)context
                          downloadedImage:(nullable UIImage *)downloadedImage
                           downloadedData:(nullable NSData *)downloadedData
                                 finished:(BOOL)finished
                                 progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                completed:(nullable SDInternalCompletionBlock)completedBlock {
    //缓存的类型disk/cache
    SDImageCacheType storeCacheType = SDImageCacheTypeAll;
    if (context[SDWebImageContextStoreCacheType]) {
        storeCacheType = [context[SDWebImageContextStoreCacheType] integerValue];
    }
    //original存储图片类型SDImageCacheTypeNone
    SDImageCacheType originalStoreCacheType = SDImageCacheTypeNone;
    if (context[SDWebImageContextOriginalStoreCacheType]) {
        originalStoreCacheType = [context[SDWebImageContextOriginalStoreCacheType] integerValue];
    }
    //缓存中key的格式
    id<SDWebImageCacheKeyFilter> cacheKeyFilter = context[SDWebImageContextCacheKeyFilter];
    //转换相应格式的缓存key
    NSString *key = [self cacheKeyForURL:url cacheKeyFilter:cacheKeyFilter];
    //图片是进行处理
    id<SDImageTransformer> transformer = context[SDWebImageContextImageTransformer];
    //图片缓存格式
    id<SDWebImageCacheSerializer> cacheSerializer = context[SDWebImageContextCacheSerializer];
    
    //是否进行图片的处理(不是GIF/设置了SDWebImageTransformAnimatedImage)
    BOOL shouldTransformImage = downloadedImage && (!downloadedImage.sd_isAnimated || (options & SDWebImageTransformAnimatedImage)) && transformer;
    //是否缓存原始图片
    BOOL shouldCacheOriginal = downloadedImage && finished;
    
    //缓存原始图片
    if (shouldCacheOriginal) {
        //默认情况,如果图片,那么会SDImageCacheTypeAll
        SDImageCacheType targetStoreCacheType = shouldTransformImage ? originalStoreCacheType : storeCacheType;
        if (cacheSerializer && (targetStoreCacheType == SDImageCacheTypeDisk || targetStoreCacheType == SDImageCacheTypeAll)) {
            //对图片按照指定的格式jpg/png进行缓存
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
                @autoreleasepool {
                    NSData *cacheData = [cacheSerializer cacheDataWithImage:downloadedImage originalData:downloadedData imageURL:url];
                    [self.imageCache storeImage:downloadedImage imageData:cacheData forKey:key cacheType:targetStoreCacheType completion:nil];
                }
            });
        } else {
            //直接按照压缩后的图片格式进行缓存
            [self.imageCache storeImage:downloadedImage imageData:downloadedData forKey:key cacheType:targetStoreCacheType completion:nil];
        }
    }
    // if available, store transformed image to cache
    //对图片进行处理之后进行保存
    if (shouldTransformImage) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
            @autoreleasepool {
                //对下载的图片进行处理
                UIImage *transformedImage = [transformer transformedImageWithImage:downloadedImage forKey:key];
                if (transformedImage && finished) {
                    NSString *transformerKey = [transformer transformerKey];
                    //转化成transform对应的key
                    NSString *cacheKey = SDTransformedKeyForKey(key, transformerKey);
                    //如果处理过的图片和原始图片不同
                    BOOL imageWasTransformed = ![transformedImage isEqual:downloadedImage];
                    NSData *cacheData;
                    // pass nil if the image was transformed, so we can recalculate the data from the image
                    //cache
                    if (cacheSerializer && (storeCacheType == SDImageCacheTypeDisk || storeCacheType == SDImageCacheTypeAll)) {
                        //按照指定格式进行图片的压缩处理
                        cacheData = [cacheSerializer cacheDataWithImage:transformedImage  originalData:(imageWasTransformed ? nil : downloadedData) imageURL:url];
                    } else {
                        cacheData = (imageWasTransformed ? nil : downloadedData);
                    }
                    //对图片进行缓存
                    [self.imageCache storeImage:transformedImage imageData:cacheData forKey:cacheKey cacheType:storeCacheType completion:nil];
                }
                //图片处理完成的回调
                [self callCompletionBlockForOperation:operation completion:completedBlock image:transformedImage data:downloadedData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
            }
        });
    } else {
        //图片处理完成的回调
        [self callCompletionBlockForOperation:operation completion:completedBlock image:downloadedImage data:downloadedData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
    }
}
```



---

【完】