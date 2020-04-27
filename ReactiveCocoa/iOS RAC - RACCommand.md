来自：[iOS RAC - RACCommand](https://www.jianshu.com/p/baa5fe76191c)



Command翻译过来就是命令，RACCommand是用来干啥子的呢？我们来简单的看看。

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    return nil;
}];

[command execute:@"开始飞起来"];
```

然后就愉快的运行了，然后就是愉快的**奔溃**了。然后我们查看一下奔溃日志

```objc
2020-04-01 09:22:23.443115+0800 OCDemo20200321[5208:216024] *** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'nil signal returned from signal block for value: 开始飞起来'

```



在log中很明确的告诉我们，返回的信号不能为空，既然如此我们就放回一个信号给他呗。

于是代码变成了这样子的：

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        return nil;
    }];
}];
[command execute:@"开始飞起来"];
```

再次愉快的运行，这次没有奔溃，啥都没有，那我们发送的数据呢？`[command execute:@"开始飞起来"];`
 没错就是在创建command的block中的input参数
 我们可以打印一下

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    NSLog(@"%@",input);
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        return nil;
    }];
}];
[command execute:@"开始飞起来"];
```

```objc
2020-04-02 14:48:31.890199+0800 OCDemo20200321[46077:1582794] 开始飞起来
```



既然返回的是一个信号，那我们就尝试着发布信息

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    NSLog(@"%@",input);
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"大佬大佬放过我"];
        
        return nil;
    }];
}];
[command execute:@"开始飞起来"];
```



这个时候问题来了，发送的信息谁去接收呢？？？

这个时候我们注意一下`execute`这个方法

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2020/RACCommand_tu1.png)

![](https://upload-images.jianshu.io/upload_images/1940927-3451d8f32c4eddbe.png?imageMogr2/auto-orient/strip|imageView2/2/w/1176)



把代码改成这个样子，大胆的猜测一下，在创建的方法中返回的信号就是他

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    NSLog(@"%@",input);
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"大佬大佬放过我"];
        
        return nil;
    }];
}];
[command execute:@"开始飞起来"];

RACSignal * signal = [command execute:@"开始飞起来"];

[signal subscribeNext:^(id  _Nullable x) {
    NSLog(@"接收数据 - %@",x);
}];
```

运行查看结果

```objc
2020-04-02 14:53:20.430980+0800 OCDemo20200321[46220:1588261] 开始飞起来
2020-04-02 14:53:20.433549+0800 OCDemo20200321[46220:1588261] 接收数据 - 大佬大佬放过我
```



还有没有别的方法可以拿到呢？肯定有啦，不然就打这行字啦

```objc
[command.executionSignals subscribeNext:^(id  _Nullable x) {
    NSLog(@"接收数据 - %@",x);
}];
```

通过这个也可以获取到数据，executionSignals就是用来发送信号的信号源，需要注意的是这个方法一定要在执行`execute`方法之前，否则就不起作用了，然后就运行程序发现，给我的竟然是一个信号？？？？黑人问号？？

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    NSLog(@"%@",input);
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"大佬大佬放过我"];
        
        return nil;
    }];
}];

[command.executionSignals subscribeNext:^(id  _Nullable x) {
    NSLog(@"接收数据 - %@",x);
}];

[command execute:@"开始飞起来"];
```

```objc
2020-04-02 14:54:12.041239+0800 OCDemo20200321[46263:1589640] 开始飞起来
2020-04-02 14:54:12.043056+0800 OCDemo20200321[46263:1589640] 接收数据 - <RACDynamicSignal: 0x6000012e2180> name: 
```



事已至此，既然是信号我们就在订阅一次吧，看看会拿到什么值。

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    NSLog(@"%@",input);
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"大佬大佬放过我"];
        
        return nil;
    }];
}];

[command.executionSignals subscribeNext:^(id  _Nullable x) {
    NSLog(@"接收数据 - %@",x);
}];

[command execute:@"开始飞起来"];
```

```objc
2020-04-02 14:54:12.041239+0800 OCDemo20200321[46263:1589640] 开始飞起来
2020-04-02 14:54:12.043056+0800 OCDemo20200321[46263:1589640] 接收数据 - <RACDynamicSignal: 0x6000012e2180> name: 
2020-04-02 14:54:12.045376+0800 OCDemo20200321[46263:1589640] 这里会是什么呢？ - 大佬大佬放过我
```



好吧，一波三折终于拿到了值，现在我们要去看看`execute`这个方法里面到底做了什么骚操作，不然心有不甘啊……

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2020/RACCommand_tu2.png)

![](https://upload-images.jianshu.io/upload_images/1940927-1c84e0a73803440a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1183)

就是这个家伙我们才可以先发送后订阅啊

上面的操作是不是很繁琐？没关系，RAC肯定给了你更好的骚操作
 除了上面那个双层订阅，我们还可以用这个`switchToLatest`

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    NSLog(@"%@",input);
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"大佬大佬放过我"];
        
        return nil;
    }];
}];

[command.executionSignals.switchToLatest subscribeNext:^(id _Nullable x) {
    NSLog(@"接收数据 - %@",x);
}];

[command execute:@"开始飞起来"];
```

```objc
2020-04-02 14:55:39.554049+0800 OCDemo20200321[46306:1591578] 开始飞起来
2020-04-02 14:55:39.556801+0800 OCDemo20200321[46306:1591578] 接收数据 - 大佬大佬放过我
```



其中`switchToLatest`表示的是最新发送的信号，验证一下看他是不是最新的信号吧。

1、先创建5个RACSubject，其中第一个为信号中的信号（也就是发送的数据是信号）

```objc
RACSubject *signalofsignal = [RACSubject subject];
RACSubject *signal1 = [RACSubject subject];
RACSubject *signal2 = [RACSubject subject];
RACSubject *signal3 = [RACSubject subject];
RACSubject *signal4 = [RACSubject subject];
```



2、然后我们就订阅信号中的信号（因为我们约定了，发送的是信号，所以接收到的也是信号，既然是信号，那就可以订阅）

```objc
[signalofsignal subscribeNext:^(id  _Nullable x) {
    [x subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x);
    }];
}];
```

3、发送数据

```objc
[signalofsignal sendNext:signal1];
[signal1 sendNext:@"1"];
```

现在我们查看log吧

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2020/RACCommand_tu3.png)

![](https://upload-images.jianshu.io/upload_images/1940927-afea02c72813f4c3.png?imageMogr2/auto-orient/strip|imageView2/2/w/874)



现在我们在用`switchToLatest`在去订阅看看

```objc
RACSubject *signalofsignal = [RACSubject subject];
RACSubject *signal1 = [RACSubject subject];
RACSubject *signal2 = [RACSubject subject];
RACSubject *signal3 = [RACSubject subject];
RACSubject *signal4 = [RACSubject subject];

[signalofsignal.switchToLatest subscribeNext:^(id _Nullable x) {
    NSLog(@"%@",x);
}];

[signalofsignal sendNext:signal1];
[signal1 sendNext:@"1"];
```

```objc
2020-04-02 14:57:11.365797+0800 OCDemo20200321[46373:1593551] 1
```



可以看到一样可以拿到数据，现在我们开始测试发送多个信号，看拿到是不是最后一个信号

```objc
RACSubject *signalofsignal = [RACSubject subject];
RACSubject *signal1 = [RACSubject subject];
RACSubject *signal2 = [RACSubject subject];
RACSubject *signal3 = [RACSubject subject];
RACSubject *signal4 = [RACSubject subject];

[signalofsignal.switchToLatest subscribeNext:^(id _Nullable x) {
    NSLog(@"%@",x);
}];

[signalofsignal sendNext:signal1];
[signalofsignal sendNext:signal2];
[signalofsignal sendNext:signal3];
[signalofsignal sendNext:signal4];

[signal1 sendNext:@"1"];
[signal2 sendNext:@"2"];
[signal3 sendNext:@"3"];
[signal4 sendNext:@"4"];
```

```objc
///打印：
2020-04-01 21:36:01.746549+0800 OCDemo20200321[6444:301993] 4
```



OK，会到command中，我们现在已经验证了之前的推测，现在如果想监听到执行完成或者还在执行就可以这样子

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    NSLog(@"input - %@",input);
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"6666666666"];
        
        return nil;
    }];
}];

[command.executionSignals.switchToLatest subscribeNext:^(id _Nullable x) {
    NSLog(@"%@",x);
}];

[command.executing subscribeNext:^(NSNumber * _Nullable x) {
    if ([x boolValue]) {
        NSLog(@"还在执行");
    } else {
        NSLog(@"执行结束了");
    }
}];

[command execute:@"9999999999"];
```

```objc
2020-04-01 21:39:20.207941+0800 OCDemo20200321[6492:305596] 执行结束了
2020-04-01 21:39:20.209383+0800 OCDemo20200321[6492:305596] input - 9999999999
2020-04-01 21:39:20.211573+0800 OCDemo20200321[6492:305596] 还在执行
2020-04-01 21:39:20.211940+0800 OCDemo20200321[6492:305596] 6666666666
```



在上面的过程中，我们发现有两点不太对：

- 1、刚运行的时候就来了一次执行结束，这个不是我们想要的
- 2、并没有结束，但其实我们已经完成了

我们先解决第2个问题，在command的block我们可以注意到，我们在signal的block中只发送了数据，并没有告诉外界发送完成了，所以就导致了，一直没发送完成，所以我们在发送数据之后加上`[subscriber sendCompleted];`

```objc
2020-04-01 21:39:20.207941+0800 OCDemo20200321[6492:305596] 执行结束了
2020-04-01 21:39:20.209383+0800 OCDemo20200321[6492:305596] input - 9999999999
2020-04-01 21:39:20.211573+0800 OCDemo20200321[6492:305596] 还在执行
2020-04-01 21:39:20.211940+0800 OCDemo20200321[6492:305596] 6666666666
```



然后我们在来看第1个问题，为什么第一次就执行结束了，这次的判断不是我们想要的， 那我们可不可以跳过呢？
 答案肯定是可以的啦
 这个时候我需要用到一个方法`skip`，这个方法后面有一个参数，填的就是忽略的次数，我们这个时候只想忽略第一次， 所以就填1

```objc
[[command.executing skip:1] subscribeNext:^(NSNumber * _Nullable x) {
    if ([x boolValue]) {
        NSLog(@"还在执行");
    } else {
        NSLog(@"执行结束了");
    }
}];
```



这个时候我们在运行，然后看log

```objc
RACCommand * command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
    
    NSLog(@"input - %@",input);
    
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        
        [subscriber sendNext:@"6666666666"];
        
        return nil;
    }];
}];

[command.executionSignals.switchToLatest subscribeNext:^(id _Nullable x) {
    NSLog(@"%@",x);
}];

[[command.executing skip:1] subscribeNext:^(NSNumber * _Nullable x) {
    if ([x boolValue]) {
        NSLog(@"还在执行");
    } else {
        NSLog(@"执行结束了");
    }
}];

[command execute:@"9999999999"];
```

```objc
2020-04-02 14:58:11.983964+0800 OCDemo20200321[46420:1595007] input - 9999999999
2020-04-02 14:58:11.987464+0800 OCDemo20200321[46420:1595007] 还在执行
2020-04-02 14:58:11.988916+0800 OCDemo20200321[46420:1595007] 6666666666
```

既然提到了`skip`那就随便可以提提其它的类似的方法
 `filter`过滤某些
 `ignore`忽略某些值
 `startWith`从哪里开始
 `skip`跳过（忽略）次数
 `take`取几次值 正序
 `takeLast`取几次值 倒序



---

【完】

