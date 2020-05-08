来自：[iOS 使用GCD控制网络请求顺序](https://www.jianshu.com/p/92eed31b7421)



---



多个网络请求同时执行，等所有网络请求完成，再统一做其他操作，我们可能会想到`dispatch_group_async`、`dispatch_group_notify`结合使用。

```objc
- (void)demo1 {
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();
    dispatch_group_async(group, queue, ^{
        NSLog(@"任务一完成");
    });
    
    dispatch_group_async(group, queue, ^{
        NSLog(@"任务二完成");
    });
    
    dispatch_group_async(group, queue, ^{
        NSLog(@"任务三完成");
    });
    //在分组的所有任务完成后触发
    dispatch_group_notify(group, queue, ^{
        NSLog(@"所有任务完成");
    });

}
```

或者使用栅栏

```objc
- (void)barrier {
//    dispatch_queue_t queue = dispatch_queue_create("com.lai.www", DISPATCH_QUEUE_CONCURRENT);
    dispatch_queue_t queue =dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);
    
    dispatch_async(queue, ^{
        NSLog(@"任务1-1完成");
    });
    dispatch_async(queue, ^{
        NSLog(@"任务1-2完成");
    });
    
    dispatch_async(queue, ^{
        NSLog(@"任务1-3完成");
    });
    
    dispatch_barrier_async(queue, ^{
        NSLog(@"以上任务都完成 dispatch_barrie完成");
    });
}
```

比如上述写法，**内部执行的是同步操作**没有问题。如果以上三个任务都是异步的，比如是网络请求，那么就达不到我们想要的效果。因为异步，请求没有回来，`dispatch_group_notify` 或者 `dispatch_barrier_async` 已经执行了。

我们可以采用`信号量`或者`dispatch_group_enter、dispatch_group_leave`实现。



# 核心思想：将异步变成同步

下述描述了常见场景下的代码实现：包括**顺序执行**和**同时执行异步操作**

### 一、 顺序执行 ：

##### 方式一 : 信号量semaphore   (必须放在子线程，因为 `dispatch_semaphore_wait`会卡死主线程) （例：1执行完了执行2）

```objc
- (void)serialBySemaphore {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{

        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);

        [self requestOneWithSuccessBlock:^{
            dispatch_semaphore_signal(semaphore);
        }];

        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

        [self requestTwoWithBlock:^{
        }];
    });
}

- (void)requestTwoWithBlock:(void(^)(void))finished {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(2);
        NSLog(@"Two");
        finished();
    });
}

- (void)requestOneWithSuccessBlock:(void(^)(void))finished {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(3);
        NSLog(@"One");
        finished();
    });
}
```

执行到`dispatch_semaphore_wait`时，由于信号量为0，进行等待。请求1完成后调用`dispatch_semaphore_signal` ，信号量不再为0，接着执行请求2。



##### 方式二：  GCD   `dispatch_group_enter` + `dispatch_group_leave ` （例： 1、2 同时执行，执行完了再执行3）

```objc
- (void)serialByGroupWait {
    
    dispatch_group_t group = dispatch_group_create();
    
    dispatch_group_enter(group);
    [self requestOneWithSuccessBlock:^{
        dispatch_group_leave(group);
    }];
    
    dispatch_group_enter(group);
    [self requestTwoWithBlock:^{
        dispatch_group_leave(group);
    }];
    // 1  2同时执行
    
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);// 1 2 执行完 下面才会执行
    
    dispatch_group_enter(group);
    [self requestThreeWithBlock:^{
        dispatch_group_leave(group);
    }];
    
  // 1 2 3 都完成 才会执行
    dispatch_group_notify(group, dispatch_get_global_queue(0, 0), ^{
        NSLog(@"all request  done!");
    });
}
- (void)requestThreeWithBlock:(void(^)(void))finished {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(1);
        NSLog(@"Three");
        finished();
    });
}
```

执行到`dispatch_group_wait`时，由于enter数不等于leave数，进行等待。请求1，2都完成后调用`dispatch_group_leave` ，enter数等于leave数，接着执行请求3。 请求1，2，3都执行后，执行 `dispatch_group_notify` 。




##### 方式三：回调中执行

```objc
- (void) serialByCallBack {
    [self requestOneWithSuccessBlock:^{
        [self requestTwoWithBlock:^{
        }];
    }];
}
```

low方法，请求一多，嵌套恶心



###二、 同时执行 ：

##### 方式一：使用信号量 dispatch_semaphore_t

```objc
- (void)concurrentBySemaphore {
    dispatch_group_t group = dispatch_group_create();
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_t sema = dispatch_semaphore_create(0);
        [self requestOneWithSuccessBlock:^{
            dispatch_semaphore_signal(sema);
        }];
        dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_t sema = dispatch_semaphore_create(0);
        [self requestTwoWithBlock:^{
            dispatch_semaphore_signal(sema);
        }];
        dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    });
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"全部搞完了");
    });
}
```



##### 方法二、使用GCD的 `dispatch_group_enter` + `dispatch_group_leave`

```objc
- (void)concurrentByGroup {
    
    dispatch_group_t group = dispatch_group_create();
    
    dispatch_group_enter(group);
    [self requestOneWithSuccessBlock:^{
        dispatch_group_leave(group);
    }];
    
    dispatch_group_enter(group);
    [self requestTwoWithBlock:^{
        dispatch_group_leave(group);
    }];
    
  // 1 2  都完成 才会执行
    dispatch_group_notify(group, dispatch_get_global_queue(0, 0), ^{
        NSLog(@"all request  done!");
    });
}
```



## 扩充：循环请求情况 顺序请求/同时请求

### 一、模拟循环网络请求 - 异步执行、统一通知完成

##### 方式一：GCD + 信号量方式

```objc
- (void)concurrentTest1 {
    dispatch_group_t group = dispatch_group_create();
    for (int i = 0 ; i < 5; i++) {
        dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            dispatch_semaphore_t sema = dispatch_semaphore_create(0);
            NSLog(@"开始%d",i);
             // 模拟请求 ↓
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
                sleep(3);
                NSLog(@"任务%d完成",i);
                 dispatch_semaphore_signal(sema);
            });
             // 模拟请求 上
            dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
        });
    }
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"全部搞完了");
    });
}
```

执行结果：

```
18.534208+0800 ObjectiveCDemo20200321[5515:267973] 开始0
18.534267+0800 ObjectiveCDemo20200321[5515:268168] 开始1
18.534460+0800 ObjectiveCDemo20200321[5515:268170] 开始3
18.534496+0800 ObjectiveCDemo20200321[5515:268169] 开始2
18.534820+0800 ObjectiveCDemo20200321[5515:268171] 开始4
21.538946+0800 ObjectiveCDemo20200321[5515:268173] 任务1完成
21.538965+0800 ObjectiveCDemo20200321[5515:268175] 任务2完成
21.538977+0800 ObjectiveCDemo20200321[5515:268176] 任务4完成
21.538997+0800 ObjectiveCDemo20200321[5515:268172] 任务0完成
21.539007+0800 ObjectiveCDemo20200321[5515:268174] 任务3完成
21.539422+0800 ObjectiveCDemo20200321[5515:266078] 全部搞完了
```



##### 方式二：GCD + group  enter/leave  方式

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

### 二、模拟循环网络请求 - 异步中顺序执行

##### 方式一：GCD + 信号量方式

```objc
- (void)serialTest1 {
    dispatch_semaphore_t sema = dispatch_semaphore_create(0);
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        for (int i = 0 ; i < 5; i++) {
            NSLog(@"开始%d",i);
            // 模拟请求 ↓
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
                sleep(3);
                NSLog(@"任务%d完成",i);
                dispatch_semaphore_signal(sema);
            });
            // 模拟请求 上
           dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
        }
        
       NSLog(@"全部搞完了");
    });
}
```

执行结果

```objc
25:03.612131+0800 ObjectiveCDemo20200321[5515:268754] 开始0
25:06.612751+0800 ObjectiveCDemo20200321[5515:268998] 任务0完成
25:06.613019+0800 ObjectiveCDemo20200321[5515:268754] 开始1
25:09.618478+0800 ObjectiveCDemo20200321[5515:268998] 任务1完成
25:09.618757+0800 ObjectiveCDemo20200321[5515:268754] 开始2
25:12.619812+0800 ObjectiveCDemo20200321[5515:268998] 任务2完成
25:12.620069+0800 ObjectiveCDemo20200321[5515:268754] 开始3
25:15.621084+0800 ObjectiveCDemo20200321[5515:268998] 任务3完成
25:15.621348+0800 ObjectiveCDemo20200321[5515:268754] 开始4
25:18.626789+0800 ObjectiveCDemo20200321[5515:268998] 任务4完成
25:18.627045+0800 ObjectiveCDemo20200321[5515:268754] 全部搞完了
```



##### 方式二：GCD + group enter/leave 方式

```objc
- (void)serialTest2 {
    dispatch_group_t group = dispatch_group_create();
    for (int i = 0 ; i < 5; i++) {
        dispatch_group_enter(group);
        NSLog(@"开始%d",i);
        // 模拟请求 ↓
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            sleep(6 - i);
            NSLog(@"任务%d完成",i);
            dispatch_group_leave(group);
        });
        // 模拟请求 ↑
        dispatch_group_wait(group, DISPATCH_TIME_FOREVER); // 顺序执行与同步执行的不同点
    }
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"全部搞完了");
    });
}
```

执行结果

```objc
26:14.858291+0800 ObjectiveCDemo20200321[5515:266078] 开始0
26:20.863823+0800 ObjectiveCDemo20200321[5515:268754] 任务0完成
26:20.864132+0800 ObjectiveCDemo20200321[5515:266078] 开始1
26:25.868070+0800 ObjectiveCDemo20200321[5515:268754] 任务1完成
26:25.868347+0800 ObjectiveCDemo20200321[5515:266078] 开始2
26:29.870906+0800 ObjectiveCDemo20200321[5515:268754] 任务2完成
26:29.871172+0800 ObjectiveCDemo20200321[5515:266078] 开始3
26:32.872810+0800 ObjectiveCDemo20200321[5515:268754] 任务3完成
26:32.873108+0800 ObjectiveCDemo20200321[5515:266078] 开始4
26:34.878482+0800 ObjectiveCDemo20200321[5515:268754] 任务4完成
26:34.880741+0800 ObjectiveCDemo20200321[5515:266078] 全部搞完了
```



---

【完】