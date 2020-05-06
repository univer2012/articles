在上架华安项目前，测试人员反馈，点击app会莫名的闪退。于是查看崩溃手机的`View Device Logs`，找到对应项目最接近崩溃的时间的日志，发现了这么一段：
[![img](https://univer2012.github.io/2017/05/15/14NSAttributedString-initWithData-blocking-App-problem/pic1.png)](https://univer2012.github.io/2017/05/15/14NSAttributedString-initWithData-blocking-App-problem/pic1.png)

于是把这断复制，百度了下，说`NSAttributedString`的操作很耗时，需要放在异步线程，于是自己在demo中实践了一把，点击进去和快速滚动时，确实会发生卡顿，而且内存会暴涨。测试代码如下：

```objc
#import "SGH0515AttibuteStringInitViewController.h"
static NSString *kCellIndentifier=@"kCellIndentifier";
@interface SGH0515AttibuteStringInitViewController ()<UITableViewDelegate,UITableViewDataSource>
@property(nonatomic,strong)UITableView *tableView;
@end
@implementation SGH0515AttibuteStringInitViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    self.tableView=({
        UITableView *tableView=[UITableView new];
        [self.view addSubview:tableView];
        tableView.frame=CGRectMake(0, 0, CGRectGetWidth(self.view.frame), CGRectGetHeight(self.view.frame));
        tableView.estimatedRowHeight = 80;
        tableView.rowHeight = UITableViewAutomaticDimension;
        tableView.delegate=self;
        tableView.dataSource=self;
        tableView;
    });
    [_tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:kCellIndentifier];
    
}
-(NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 1;
}
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return 1000;
}
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:kCellIndentifier];
    NSString *testString = @"<p>盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升 盘后分析（5月15日）: 午后市场维持震荡，中央政务区概念受消息刺激集体拉升</p>";
    cell.textLabel.numberOfLines = 0;
#if 1
    NSData *summaryData = [testString  dataUsingEncoding:NSUnicodeStringEncoding];
    NSAttributedString *muatbleAttrStr=[[NSAttributedString alloc]initWithData:summaryData options:@{NSDocumentTypeDocumentAttribute:NSHTMLTextDocumentType} documentAttributes:nil error:nil];
    
    NSMutableAttributedString *subMuatbleAttrStr = [[muatbleAttrStr attributedSubstringFromRange:NSMakeRange(0, muatbleAttrStr.length)] mutableCopy];
    
    NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
    paragraphStyle.lineSpacing = 1;//行间距
    paragraphStyle.lineBreakMode = NSLineBreakByTruncatingTail;
    [subMuatbleAttrStr addAttributes:@{NSFontAttributeName : cell.textLabel.font, NSForegroundColorAttributeName : cell.textLabel.textColor, NSParagraphStyleAttributeName : paragraphStyle} range:NSMakeRange(0, subMuatbleAttrStr.length)];
    cell.textLabel.attributedText = subMuatbleAttrStr;
    
    
#else
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        
        NSData *summaryData = [testString  dataUsingEncoding:NSUnicodeStringEncoding];
        NSAttributedString *muatbleAttrStr=[[NSAttributedString alloc]initWithData:summaryData options:@{NSDocumentTypeDocumentAttribute:NSHTMLTextDocumentType} documentAttributes:nil error:nil];
        
        NSMutableAttributedString *subMuatbleAttrStr = [[muatbleAttrStr attributedSubstringFromRange:NSMakeRange(0, muatbleAttrStr.length)] mutableCopy];
        
        NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
        paragraphStyle.lineSpacing = 1;//行间距
        paragraphStyle.lineBreakMode = NSLineBreakByTruncatingTail;
        [subMuatbleAttrStr addAttributes:@{NSFontAttributeName : cell.textLabel.font, NSForegroundColorAttributeName : cell.textLabel.textColor, NSParagraphStyleAttributeName : paragraphStyle} range:NSMakeRange(0, subMuatbleAttrStr.length)];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            cell.textLabel.attributedText = subMuatbleAttrStr;
        });
        
    });
#endif
    
    return cell;
}
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    
}
- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}
@end
```

改为异步后，点击后可以马上进去，但是无论等多久都不会有数据显示出来！只有滚动后才会填充数据。同时，快速滚动不会有卡顿。

