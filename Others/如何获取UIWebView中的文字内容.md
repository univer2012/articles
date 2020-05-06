业务场景：

 项目的资讯详情页面有一个分享，分享中要传的字段有一个`content`的字段，但是详情的内容使用`UIWebView`加载的，这就需要从`UIWebView`中拿到展示的文字内容。

 索引到`UIWebView`的`.h`文件中，发现没有直接获取内容的方法或者属性，看来只能执行js代码来拿到那些文字了。

```objc
 //
//  UIWebView.h
//  UIKit
//
//  Copyright (c) 2007-2015 Apple Inc. All rights reserved.
//
#import <Foundation/Foundation.h>
#import <UIKit/UIView.h>
#import <UIKit/UIKitDefines.h>
#import <UIKit/UIDataDetectors.h>
#import <UIKit/UIScrollView.h>
NS_ASSUME_NONNULL_BEGIN
typedef NS_ENUM(NSInteger, UIWebViewNavigationType) {
    UIWebViewNavigationTypeLinkClicked,
    UIWebViewNavigationTypeFormSubmitted,
    UIWebViewNavigationTypeBackForward,
    UIWebViewNavigationTypeReload,
    UIWebViewNavigationTypeFormResubmitted,
    UIWebViewNavigationTypeOther
} __TVOS_PROHIBITED;
typedef NS_ENUM(NSInteger, UIWebPaginationMode) {
    UIWebPaginationModeUnpaginated,
    UIWebPaginationModeLeftToRight,
    UIWebPaginationModeTopToBottom,
    UIWebPaginationModeBottomToTop,
    UIWebPaginationModeRightToLeft
} __TVOS_PROHIBITED;
typedef NS_ENUM(NSInteger, UIWebPaginationBreakingMode) {
    UIWebPaginationBreakingModePage,
    UIWebPaginationBreakingModeColumn
} __TVOS_PROHIBITED;
@class UIWebViewInternal;
@protocol UIWebViewDelegate;
NS_CLASS_AVAILABLE_IOS(2_0) __TVOS_PROHIBITED @interface UIWebView : UIView <NSCoding, UIScrollViewDelegate> 
@property (nullable, nonatomic, assign) id <UIWebViewDelegate> delegate;
@property (nonatomic, readonly, strong) UIScrollView *scrollView NS_AVAILABLE_IOS(5_0);
- (void)loadRequest:(NSURLRequest *)request;
- (void)loadHTMLString:(NSString *)string baseURL:(nullable NSURL *)baseURL;
- (void)loadData:(NSData *)data MIMEType:(NSString *)MIMEType textEncodingName:(NSString *)textEncodingName baseURL:(NSURL *)baseURL;
@property (nullable, nonatomic, readonly, strong) NSURLRequest *request;
- (void)reload;
- (void)stopLoading;
- (void)goBack;
- (void)goForward;
@property (nonatomic, readonly, getter=canGoBack) BOOL canGoBack;
@property (nonatomic, readonly, getter=canGoForward) BOOL canGoForward;
@property (nonatomic, readonly, getter=isLoading) BOOL loading;
- (nullable NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)script;
@property (nonatomic) BOOL scalesPageToFit;
@property (nonatomic) BOOL detectsPhoneNumbers NS_DEPRECATED_IOS(2_0, 3_0);
@property (nonatomic) UIDataDetectorTypes dataDetectorTypes NS_AVAILABLE_IOS(3_0);
@property (nonatomic) BOOL allowsInlineMediaPlayback NS_AVAILABLE_IOS(4_0); // iPhone Safari defaults to NO. iPad Safari defaults to YES
@property (nonatomic) BOOL mediaPlaybackRequiresUserAction NS_AVAILABLE_IOS(4_0); // iPhone and iPad Safari both default to YES
@property (nonatomic) BOOL mediaPlaybackAllowsAirPlay NS_AVAILABLE_IOS(5_0); // iPhone and iPad Safari both default to YES
@property (nonatomic) BOOL suppressesIncrementalRendering NS_AVAILABLE_IOS(6_0); // iPhone and iPad Safari both default to NO
@property (nonatomic) BOOL keyboardDisplayRequiresUserAction NS_AVAILABLE_IOS(6_0); // default is YES
@property (nonatomic) UIWebPaginationMode paginationMode NS_AVAILABLE_IOS(7_0);
@property (nonatomic) UIWebPaginationBreakingMode paginationBreakingMode NS_AVAILABLE_IOS(7_0);
@property (nonatomic) CGFloat pageLength NS_AVAILABLE_IOS(7_0);
@property (nonatomic) CGFloat gapBetweenPages NS_AVAILABLE_IOS(7_0);
@property (nonatomic, readonly) NSUInteger pageCount NS_AVAILABLE_IOS(7_0);
@property (nonatomic) BOOL allowsPictureInPictureMediaPlayback NS_AVAILABLE_IOS(9_0);
@property (nonatomic) BOOL allowsLinkPreview NS_AVAILABLE_IOS(9_0); // default is NO
@end
__TVOS_PROHIBITED @protocol UIWebViewDelegate <NSObject>
@optional
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;
- (void)webViewDidStartLoad:(UIWebView *)webView;
- (void)webViewDidFinishLoad:(UIWebView *)webView;
- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error;
@end
NS_ASSUME_NONNULL_END
```

但是自己又不懂js代码怎么写，只剩下网上查找了。百度中输入`iOS 获取UIWebView中的文字内容 2016`，找到了一篇博客:[ios通过UIWebView加载word文档，并获取word文档内容](http://blog.sina.com.cn/s/blog_13bc6705b0102wp8v.html)
 ，看到其中有一段是这样写的：

> 另外，通过webView的 `loadData: MIMEType: textEncodingName: baseURL:`这个方法也可以加载出来。
> 此外，我想拿到word显示的文字内容，可以通过执行js代码
>
> ```objc
> -(void)webViewDidFinishLoad:(UIWebView *)webView
> {
>  NSString *strings = [webView stringByEvaluatingJavaScriptFromString:@"document.body.innerText"];
>  NSLog(@"%@",strings);
> ｝
> ```
>
> 这里的strings就是word文档文字内容。
> 另外，通过`NSString *strings = [webView stringByEvaluatingJavaScriptFromString:@"document.body.innerHTML"];`这里获得的strings为word文档内容的html格式。



也就是说，可以通过这句代码

```objc
NSString *strings = [webView stringByEvaluatingJavaScriptFromString:@"document.body.innerText"];
```

拿到`UIWebView`中的文字内容。测试结果也证明了这一点。

另外，分享的内容是由长度限制的，不然就会报

```shell
2017-04-26 16:00:03.959998 SGHApp[699:389332] 分享失败,错误码:204,错误描述:Error Domain=ShareSDKErrorDomain Code=204 "(null)" UserInfo={user_data={
    error = " Text too long, please input text less than 140 characters!";
    "error_code" = 20012;
    request = "/2/statuses/upload.json";
}}
```

的错误。所以还要对拿到的内容做截取。

最终的代码如下：

```objc
//... ...
//获取UIWebView中的内容
    NSString *contentWebString = [_detailView.contentWebView stringByEvaluatingJavaScriptFromString:@"document.body.innerText"];
    if (contentWebString.length <= 0) {
        contentWebString = @"";
    }
    //截取80个字段
    NSInteger contentSubLength = contentWebString.length;
    contentWebString = [contentWebString substringWithRange:NSMakeRange(0, (contentSubLength > 80 ? 80 : contentSubLength))];
    //过滤字符串前后的空格
    contentWebString = [contentWebString stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]];
    contentWebString = [contentWebString stringByAppendingString:@"... ..."];
    NSDictionary *data=@{@"requestId":(self.requestId == nil ? @"" : self.requestId),
                         @"title" : [self.detailView.titleLabel.attributedText string] ? : @"",
                         @"type" : type,
                         @"content" : contentWebString};
                         
//... ...
```

其中的过滤字符串空格，还参考了这篇博客:[iOS 字符串过滤空格](http://www.jianshu.com/p/623c87407365)。关键代码如下：

```objc
//过滤空格
NSString * urlStr = @"  A as    dd  dmm ";
NSLog(@"urlStr = %@",urlStr);
//过滤字符串前后的空格
urlStr = [urlStr stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]];
NSLog(@"urlStr = %@",urlStr);
//过滤中间空格
urlStr = [urlStr stringByReplacingOccurrencesOfString:@" " withString:@""];
NSLog(@"urlStr = %@",urlStr);
```



---

【完】



