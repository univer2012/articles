



### 类比1



### 1.没有回调

`SHRxswift_3ViewModel.swift`代码：

```swift
import UIKit
import RxSwift

class SHRxswift_3ViewModel: NSObject {
    
    let disposeBag = DisposeBag()
    
    
    func didClickOpen(_ text: String) -> Single<String> {
        print("执行操作:\(text)__success")
        
        
        return Single.create { (singleFunc) -> Disposable in
            singleFunc(.success("OK"))
            
            return Disposables.create()
        }
    }
    
    override init() {
        super.init()
    }   
}
```

`SHRxswift_3ViewController.swift`的代码：


```swift
import UIKit

import RxSwift
import RxCocoa

class SHRxswift_3ViewController: SHBaseTableViewController {
		let viewModel = SHRxswift_3ViewModel()
    let disposeBag = DisposeBag()
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.priceTextFld.frame = CGRect(x: 0, y: 200, width: self.view.frame.size.width, height: 50)
        self.priceTextFld.backgroundColor = UIColor.gray
        self.view.addSubview(priceTextFld)
      
        
        self.priceTextFld.rx.text.asObservable().subscribe(onNext: {[weak self] (text) in
            guard let `self` = self else { return }
            
            print("callBack_succeed_observer:\(String(describing: text))__OK")
            let _ = self.viewModel.didClickOpen("没有回调")
            
        }).disposed(by: self.disposeBag)
      
    }
	
}
```



##### 对应RAC中的代码

`SHRAC12ViewModel`代码：

```objc
///.h
@interface SHRAC12ViewModel : NSObject

@property (nonatomic, strong) RACCommand *subscribeCommand; //订阅长链接

@end


///.m
#import "SHRAC12ViewModel.h"

@implementation SHRAC12ViewModel

- (instancetype)init
{
    self = [super init];
    if (self) {
        
        @weakify(self)
        self.subscribeCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(NSString * _Nullable input) {
            @strongify(self)
            NSLog(@"执行操作:%@__success",input);
            
            //return [RACSignal empty];
            //或者
            return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
                [subscriber sendNext:@"OK"];
                [subscriber sendCompleted];
                return nil;
            }];
        }];
        
        
    }
    return self;
}

@end
```



```objc
#import "SHRAC12ViewController.h"
#import "SHRAC12ViewModel.h"

@interface SHRAC12ViewController ()

@property(nonatomic, strong)SHRAC12ViewModel *viewModel;
@property(nonatomic, strong)UITextField *priceTextFld;

@end

@implementation SHRAC12ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.viewModel = [[SHRAC12ViewModel alloc] init];
    
    self.priceTextFld = [[UITextField alloc]initWithFrame:CGRectMake(0, 300, self.view.frame.size.width, 40)];
    self.priceTextFld.backgroundColor = UIColor.grayColor;
    [self.view addSubview:self.priceTextFld];
    
    @weakify(self)
    //监听价格输入
    [[RACObserve(self.priceTextFld, text) merge:self.priceTextFld.rac_textSignal] subscribeNext:^(id  _Nullable x) {
        @strongify(self)
        NSLog(@"callBack_succeed_observer:%@__OK",x);
        [self.viewModel.subscribeCommand execute:@"没有回调"];
    }];
}
@end
```

通知执行相关操作，不需要回调。





### 2.有回调

```swift
import UIKit
import RxSwift

class SHRxswift_3ViewModel: NSObject {
    
    let disposeBag = DisposeBag()
    
    func didClickOpen(_ text: String) -> Single<String> {
        print("执行操作:\(text)__success")
        
        
        return Single.create { (singleFunc) -> Disposable in
            singleFunc(.success("OK"))
            
            return Disposables.create()
        }
    }
    
    override init() {
        super.init()
    }
    
    
}
```

`SHRxswift_3ViewController.swift`的代码：

```swift
import UIKit

import RxSwift
import RxCocoa

class SHRxswift_3ViewController: SHBaseTableViewController {
		let viewModel = SHRxswift_3ViewModel()
    let disposeBag = DisposeBag()
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.priceTextFld.frame = CGRect(x: 0, y: 200, width: self.view.frame.size.width, height: 50)
        self.priceTextFld.backgroundColor = UIColor.gray
        self.view.addSubview(priceTextFld)
      
        
        self.priceTextFld.rx.text.asObservable().subscribe(onNext: {[weak self] (text) in
            guard let `self` = self else { return }
            
            print("callBack_succeed_observer:\(String(describing: text))__OK")
            self.viewModel.didClickOpen("有回调").subscribe(onSuccess: { (text) in
                print("callBack_succeed：\(text)__haode")
            }).disposed(by: self.disposeBag)
            
        }).disposed(by: self.disposeBag)
      
    }
	
}
```



##### 对应RAC中的代码

`SHRAC12ViewModel`代码：

```objc
///.h
@interface SHRAC12ViewModel : NSObject

@property (nonatomic, strong) RACCommand *subscribeCommand; //订阅长链接

@end


///.m
#import "SHRAC12ViewModel.h"

@implementation SHRAC12ViewModel

- (instancetype)init
{
    self = [super init];
    if (self) {
        
        @weakify(self)
        self.subscribeCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(NSString * _Nullable input) {
            @strongify(self)
            NSLog(@"执行操作:%@__success",input);
            
            //return [RACSignal empty];
            //或者
            return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
                [subscriber sendNext:@"OK"];
                [subscriber sendCompleted];
                return nil;
            }];
        }];
        
        
    }
    return self;
}

@end
```



```objc
#import "SHRAC12ViewController.h"
#import "SHRAC12ViewModel.h"

@interface SHRAC12ViewController ()

@property(nonatomic, strong)SHRAC12ViewModel *viewModel;
@property(nonatomic, strong)UITextField *priceTextFld;

@end

@implementation SHRAC12ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.viewModel = [[SHRAC12ViewModel alloc] init];
    
    self.priceTextFld = [[UITextField alloc]initWithFrame:CGRectMake(0, 300, self.view.frame.size.width, 40)];
    self.priceTextFld.backgroundColor = UIColor.grayColor;
    [self.view addSubview:self.priceTextFld];
    
    @weakify(self)
    //监听价格输入
    [[RACObserve(self.priceTextFld, text) merge:self.priceTextFld.rac_textSignal] subscribeNext:^(id  _Nullable x) {
        @strongify(self)
        NSLog(@"callBack_succeed_observer:%@__OK",x);
        [[self.viewModel.subscribeCommand execute:@"有回调"] subscribeNext:^(NSString * _Nullable text) {
            NSLog(@"callBack_succeed：%@__haode",text);
        }];
    }];
}
@end
```

也就是直接通知执行，有回调。



### 3.不调用，只监听






##### 对应RAC中的代码
`SHRAC12ViewModel`代码：

```objc
///.h
@interface SHRAC12ViewModel : NSObject

@property (nonatomic, strong) RACCommand *subscribeCommand; //订阅长链接

@end


///.m
#import "SHRAC12ViewModel.h"

@implementation SHRAC12ViewModel

- (instancetype)init
{
    self = [super init];
    if (self) {
        
        @weakify(self)
        self.subscribeCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(NSString * _Nullable input) {
            @strongify(self)
            NSLog(@"执行操作:%@__success",input);
            
            //return [RACSignal empty];
            //或者
            return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
                [subscriber sendNext:@"OK"];
                [subscriber sendCompleted];
                return nil;
            }];
        }];
        
        
    }
    return self;
}

@end
```



```objc
#import "SHRAC12ViewController.h"
#import "SHRAC12ViewModel.h"

@interface SHRAC12ViewController ()

@property(nonatomic, strong)SHRAC12ViewModel *viewModel;
@property(nonatomic, strong)UITextField *priceTextFld;

@end

@implementation SHRAC12ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.viewModel = [[SHRAC12ViewModel alloc] init];
    
    self.priceTextFld = [[UITextField alloc]initWithFrame:CGRectMake(0, 300, self.view.frame.size.width, 40)];
    self.priceTextFld.backgroundColor = UIColor.grayColor;
    [self.view addSubview:self.priceTextFld];
    
    //不主动调用，只监听
    [self.viewModel.subscribeCommand.executionSignals.switchToLatest subscribeNext:^(id  _Nullable x) {
        NSLog(@"监听后执行：%@",x);
        
    }];
    
    @weakify(self)
    //监听价格输入
    [[RACObserve(self.priceTextFld, text) merge:self.priceTextFld.rac_textSignal] subscribeNext:^(id  _Nullable x) {
        @strongify(self)
        NSLog(@"callBack_succeed_observer:%@__OK",x);
        [self.viewModel.subscribeCommand execute:@"不主动调用，只监听"];
    }];
}
@end
```

