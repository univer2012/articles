来自：[iOS 使用dispatch_group_enter使多次网络请求依次执行](https://www.jianshu.com/p/71fbc0415b5d)



---



### 常用的几个方法

- dispatch_group_enter :通知 group,下个任务要放入 group 中执行了
- dispatch_group_leave: 通知 group,任务成功完成,要移除,与 enter成对出现
- dispatch_group_wait: 在任务组完成时调用，或者任务组超时是调用（完成指的是enter和leave次数一样多）
- dispatch_group_notify: 只要任务全部完成了,就会在最后调用

### 具体情况分析

详见:[dispatch_group_enter 使用与讲解](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fpangshishan1%2Farticle%2Fdetails%2F79611284)

### 实际应用

我们就可以使用dispatch_group_enter了,在执行了多段之后再在 notify 中执行另一个,类似于栅栏的效果.但是如果是网络请求,需要达到网络请求嵌套的效果,A网络请求完之后再请求 B,需要添加`dispatch_group_wait`,让线程等待 A 执行完成之后再执行 B.

```objc
// A 请求数据 - 异步
- (void)loadADataFinished:(void(^)(BOOL success))finished {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(3);
        NSLog(@"A");
        finished(YES);
    });
    
        
}
// B 请求数据 - 异步
- (void)loadBDataFinished:(void(^)(BOOL success))finished {
    
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(2);
        NSLog(@"B");
        finished(YES);
    });
    
}
// C 请求数据 - 异步
- (void)loadCDataFinished:(void(^)(BOOL success))finished {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(1);
        NSLog(@"C");
        finished(YES);
    });
}

// 请求是否全部完成
- (void)finishedDataFinished:(void(^)(BOOL success))finished {
  dispatch_group_t group = dispatch_group_create();
  dispatch_group_enter(group);
  [self loadADataFinished:^(BOOL success){
    if (success){
      dispatch_group_leave(group);
    }else{
      finished(NO);
    }
  }];
  dispatch_group_enter(group);
  [self loadBDataFinished:^(BOOL success){
    if (success){
      dispatch_group_leave(group);
    }else{
      finished(NO);
    }
  }];

 dispatch_group_wait(group, DISPATCH_TIME_FOREVER);// A,B同时执行, 执行完才会执行下面的 C
 dispatch_group_enter(group);
 [self loadCDataFinished:^(BOOL success){
    if (success){
      dispatch_group_leave(group);
    }else{
      finished(NO);
    }
  }];
 
  //  group 中的任务都成功完成后,才会返回 YES
  dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        finished(YES);
   });
}

//调用
- (void)sec2demo1 {
    [self finishedDataFinished:^(BOOL success) {
        if (success) {
            NSLog(@"执行完成");
        } else {
            NSLog(@"执行中...");
        }
    }];
}
```

在我们的项目中,在一个 VC 中会有多个网络请求A,B.现在要实现的是:A 请求数据成功之后,再执行 B 的网络请求.这时候因为网络请求是异步的,所以我们要达到效果,需要在子线程中加入信号量`dispatch_semaphore_t`,在网络请求内部标记信号量,请求完成之后将信号量清为 0.

```objc
// A 请求数据 - 异步
- (void)loadADataFinished:(void(^)(BOOL success))finished {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(3);
        NSLog(@"A");
        finished(YES);
    });
    
        
}
// B 请求数据 - 异步
- (void)loadBDataFinished:(void(^)(BOOL success))finished {
    
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(2);
        NSLog(@"B");
        finished(YES);
    });
    
}

- (void)sec3demo2 {
    [self finishedSemDataFinished:^(BOOL success) {
        if (success) {
            NSLog(@"执行完成");
        } else {
            NSLog(@"执行中...");
        }
    }];
}

- (void)finishedSemDataFinished:(void(^)(BOOL success))finished {
    NSLog(@"开始");
    dispatch_async(dispatch_get_global_queue(0, 0), ^{

        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
        [self loadADataFinished:^(BOOL success){
            if (success){
            }else{
                finished(NO);
            }
            dispatch_semaphore_signal(semaphore);
        }];
    
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER); // A请求完成之后,请求 B
    
        [self loadBDataFinished:^(BOOL success){
            if (success){
            }else{
                finished(NO);
            }
            dispatch_semaphore_signal(semaphore);
        }];
        
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        
        NSLog(@"结束");
        
    });
}
```

其他情况可以参考:[GCD 的控制使用](https://www.jianshu.com/p/92eed31b7421)



## 友信智投的上传图片


```objc
//上传图片
- (void)uploadPicture{
    dispatch_group_t group = dispatch_group_create();
    for (int i = 0; i<self.imgLocationUrlArr.count; i++) {
        
        dispatch_group_enter(group);
        NSURL *url = [NSURL fileURLWithPath:self.imgLocationUrlArr[i]];
        QCloudCOSXMLUploadObjectRequest *put = [QCloudCOSXMLUploadObjectRequest new];
        put.object = [NSString stringWithFormat:@"feedback/iOS_feedback_%llu_%ld.jpg",  [YXUserManager userUUID], (long)([NSDate date].timeIntervalSince1970*1000)];
        put.bucket = YXUrlRouterConstant.suggestionBucket;
        put.body = url;
        [put.customHeaders setObject:@"www.yxzq.com" forKey:@"Referer"];
        
        [put setSendProcessBlock:^(int64_t bytesSent, int64_t totalBytesSent, int64_t totalBytesExpectedToSend) {
            LOG_VERBOSE(kOther, @"upload %lld totalSend %lld aim %lld", bytesSent, totalBytesSent, totalBytesExpectedToSend);
        }];
        
        @weakify(self)
        [put setFinishBlock:^(QCloudUploadObjectResult *outputObject, NSError* error) {
            @strongify(self)
            
            if (error) {
                self.isUploadSuccess = NO;
                dispatch_group_leave(group);
                
            }else{
                
                [self.imageUrlArr addObject:outputObject.location];
                dispatch_group_leave(group);
            }
        }];
        [[QCloudCOSTransferMangerService costransfermangerServiceForKey:YXQCloudService.keyQCloudGuangZhou] UploadObject:put];
        
        //存储在数组中
        [self.putRequestsArr addObject:put];
    }
    
    @weakify(self)
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        @strongify(self)
        if (self.isUploadSuccess) {
            //开始提交
            [self suggestionSubmit];
        }else{
            [self.imageUrlArr removeAllObjects];
            self.isUploadSuccess = YES;
            UIViewController *vc = [UIViewController currentViewController];
            [YXConstant.rootViewController hideHud];
            [YXConstant.rootViewController showError:kLang(@"mine/upload_picture_failure") inView:vc.view];
        }
    });
    
}
```



使用 **GCD + group enter/leave 方式，实现「异步执行、统一通知完成」**

跟下面的代码一样：

```objc
- (void)concurrentTest2 {
    dispatch_group_t group = dispatch_group_create();
    for (int i = 0 ; i < 5; i++) {
        dispatch_group_enter(group);
        NSLog(@"开始%d",i);
          // 模拟请求 ↓
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            sleep(3);
            NSLog(@"任务%d完成",i);
            dispatch_group_leave(group);
        });
          // 模拟请求 ↑
    }
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"全部搞完了");
    });
}
```

执行结果：

```
29.154449+0800 ObjectiveCDemo20200321[5515:266078] 开始0
29.154823+0800 ObjectiveCDemo20200321[5515:266078] 开始1
29.155029+0800 ObjectiveCDemo20200321[5515:266078] 开始2
29.155200+0800 ObjectiveCDemo20200321[5515:266078] 开始3
29.155420+0800 ObjectiveCDemo20200321[5515:266078] 开始4
32.156916+0800 ObjectiveCDemo20200321[5515:268752] 任务1完成
32.156915+0800 ObjectiveCDemo20200321[5515:268168] 任务0完成
32.156943+0800 ObjectiveCDemo20200321[5515:268754] 任务3完成
32.156943+0800 ObjectiveCDemo20200321[5515:268753] 任务2完成
32.157938+0800 ObjectiveCDemo20200321[5515:268755] 任务4完成
32.158313+0800 ObjectiveCDemo20200321[5515:266078] 全部搞完了
```





---

【完】