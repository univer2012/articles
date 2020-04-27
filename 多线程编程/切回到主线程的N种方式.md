参考：

1. [IOS基础之切回到主线程的N种方式](https://www.jianshu.com/p/6f1a6d8f9f1d)

### 第1种

```
    NSLog(@"开启一个异步线程");
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"正在执行子线程中的代码");
        sleep(1);
        NSLog(@"准备回到主线程");
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"开始执行main线程中的代码");
            sleep(1);
            NSLog(@"继续执行main线程中的代码");
            sleep(1);
            NSLog(@"继续执行main线程中的代码");
            sleep(1);
            NSLog(@"继续执行main线程中的代码");
            NSLog(@"main线程中的代码执行完毕");
        });
        NSLog(@"继续执行子线程中的代码");
        sleep(1);
        NSLog(@"继续执行子线程中的代码");
        sleep(1);
        NSLog(@"继续执行子线程中的代码");
        sleep(1);
        NSLog(@"继续执行子线程中的代码");
        sleep(1);
        NSLog(@"子线程中代码执行完成");
    });
    NSLog(@"完成");
```


### 第2种
```
[[NSOperationQueue mainQueue] addOperationWithBlock:^{
            //需要在主线程执行的代码
}];
```
### 第3种
```
[self performSelectorOnMainThread:@selector(WantToGoBackMianThread:) withObject:@"1" waitUntilDone:YES];
```

### 第4种
```objc
[[NSThread mainThread] performSelector:@selector(updateData) withObject:nil];

- (void)updateData {
    
}
```

