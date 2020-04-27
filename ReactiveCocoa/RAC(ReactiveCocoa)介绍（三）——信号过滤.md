上一篇文章简要说明了映射的使用方法，这次介绍一下信号过滤。
 信号过滤，在RAC中会对RACSignal信号发送的信息进行过滤，只有符合判断要求的信号才能被订阅到。
 信号过滤有以下几种方法：`filter`、`ignore`、`ignoreValue`、`distinctUntilChanged`

#### `filter`方法：

 在filter的block代码块中，通过return一个BOOL值来判断是否过滤掉信号。直接上代码

```objc
//MARK: filter
- (void)RAC_filter {
    @weakify(self)
    [[self.testTextField.rac_textSignal filter:^BOOL(NSString * _Nullable value) {
        @strongify(self)
        //过滤判断条件
        if (self.testTextField.text.length >= 6) {
            self.testTextField.text = [self.testTextField.text substringToIndex:6];
            self.testLabel.text = @"已经到6位了";
            self.testLabel.textColor = [UIColor redColor];
        }
        return value.length <= 6;
    }] subscribeNext:^(NSString * _Nullable x) {
        //订阅逻辑区域
        NSLog(@"filter过滤后的订阅内容：%@",x);
    }];
}
```

在代码执行时订阅消息中打印出以下结果：

```objc
2019-11-19 09:28:58.456225+0800 ObjectiveCDemo160728[3947:80874] filter过滤后的订阅内容：1
2019-11-19 09:28:59.113794+0800 ObjectiveCDemo160728[3947:80874] filter过滤后的订阅内容：12
2019-11-19 09:28:59.669257+0800 ObjectiveCDemo160728[3947:80874] filter过滤后的订阅内容：123
2019-11-19 09:29:00.272909+0800 ObjectiveCDemo160728[3947:80874] filter过滤后的订阅内容：1234
2019-11-19 09:29:00.876184+0800 ObjectiveCDemo160728[3947:80874] filter过滤后的订阅内容：12345
2019-11-19 09:29:01.492708+0800 ObjectiveCDemo160728[3947:80874] filter过滤后的订阅内容：123456
```

从上面的代码中可以看出当filter方法中return NO时，并不会获取到任何订阅消息。

**查看filter方法实现**：

```objc
- (__kindof RACStream *)filter:(BOOL (^)(id value))block {
	NSCParameterAssert(block != nil);

	Class class = self.class;
	
	return [[self flattenMap:^ id (id value) {
		if (block(value)) {
			return [class return:value];
		} else {
			return class.empty;
		}
	}] setNameWithFormat:@"[%@] -filter:", self.name];
}
```

可以发现，<u>`filter`方法本身是通过`flattenMap`映射方法来实现过滤信号的</u>，也就意味着过滤出符合条件的值，变换出来新的信号并发送给订阅者;当block中的vlaue为`NO`时，将映射成一个空信号，订阅者不会受到空信号的订阅信号消息。



#### `ignoreValue`与`ignore`
<u> `ignoreValue`与`ignore`都是基于`filter`方法封装的。</u>

 `ignoreValue`是直接将指定的信号全部过滤掉，筛选条件全部为NO，将所有信号变为空信号。
 `ignore`是将符合指定字符串条件的信号过滤掉。

```objc
//MARK:ignoreValue_ignore
-(void)RAC_ignoreValue_ignore {
    [[self.testTextField.rac_textSignal ignoreValues] subscribeNext:^(id  _Nullable x) {
        ////将self.testTextField的所有textSignal全部过滤掉
        NSLog(@"ignoreValues:%@",x);        // --> 不会执行
    }];
    
    [[self.testTextField.rac_textSignal ignore:@"1"] subscribeNext:^(NSString * _Nullable x) {
        //将self.testTextField的textSignal中字符串为指定条件的信号过滤掉
        NSLog(@"ignore:%@",x);      //-->只忽略字符串「1」，不会忽略字符串「12」、「123」等
    }];
}
```

`ignoreValues`的实现：

```objc
- (RACSignal *)ignoreValues {
	return [[self filter:^(id _) {
		return NO;
	}] setNameWithFormat:@"[%@] -ignoreValues", self.name];
}
```

`ignore`的实现：

```objc
- (__kindof RACStream *)ignore:(id)value {
	return [[self filter:^ BOOL (id innerValue) {
		return innerValue != value && ![innerValue isEqual:value];
	}] setNameWithFormat:@"[%@] -ignore: %@", self.name, RACDescription(value)];
}
```



#### `distinctUntilChanged`

用于判断当前信号的值跟上一次的值相同，若相同时将不会收到订阅信号。

```objc
//MARK: distinctUntilChanged
- (void)RAC_distinctUntilChanged {
    RACSubject *subject = [RACSubject subject];
    [[subject distinctUntilChanged] subscribeNext:^(id  _Nullable x) {
        NSLog(@"distinctUntilChanged:%@",x);
    }];
    [subject sendNext:@1111];
    [subject sendNext:@2222];
    [subject sendNext:@2222];
}
```

打印：

```objc
2019-11-19 09:41:30.421297+0800 ObjectiveCDemo160728[4277:91571] distinctUntilChanged:1111
2019-11-19 09:41:30.421676+0800 ObjectiveCDemo160728[4277:91571] distinctUntilChanged:2222
```

可以发现上述代码执行时，当第二次发送一条@2222的信号时，并没有订阅打印出相关结果。



`distinctUntilChanged`的实现：

```objc
- (__kindof RACStream *)distinctUntilChanged {
	Class class = self.class;

	return [[self bind:^{
		__block id lastValue = nil;
		__block BOOL initial = YES;

		return ^(id x, BOOL *stop) {
			if (!initial && (lastValue == x || [x isEqual:lastValue])) return [class empty];

			initial = NO;
			lastValue = x;
			return [class return:x];
		};
	}] setNameWithFormat:@"[%@] -distinctUntilChanged", self.name];
}
```

上图为distinctUntilChanged方法实现，通过判断是否为指定的字符串值，当符合条件时，将信号变换为空信号，此时则不会接收到订阅信号。

## 文章中的代码已上传至GitHub，可从此处链接下载[demo链接](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FRoganZheng%2FReactiveObjC)

