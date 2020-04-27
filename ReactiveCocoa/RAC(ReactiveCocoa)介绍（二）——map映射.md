来自：[RAC(ReactiveCocoa)介绍（二）——map映射](https://www.jianshu.com/p/a61cb45a9594)



上一篇简单介绍了一下RAC，这次探究RAC中的map映射用法。

Map(映射)
 RAC中包含两种映射方法map、flattenMap，映射方法是将原信号中的内容映射成新的指定内容。
 通过对比，从map的实现方法中可以看出是基于flattenMap方法的一层封装，但同时又有不同之处。

```objc
- (__kindof RACStream *)map:(id (^)(id value))block {
	NSCParameterAssert(block != nil);

	Class class = self.class;
	
	return [[self flattenMap:^(id value) {
		return [class return:block(value)];
	}] setNameWithFormat:@"[%@] -map:", self.name];
}
```



从上图的实现方法中可以看出，`flattenMap`方法和`map`方法都有一个带参数value的block作为这个方法的参数。不同的是：`flattenMap`方法通过调用`block（value）`来创建一个新的方法。它可以灵活的定义新创建的信号。而`map`方法，将会创建一个和原来一模一样的信号，只不过新的信号传递的值变成了block（value）

`flattenMap`作用：把原信号的内容映射成一个新信号，并`return`返回给一个`RACStream`类型数据。实际上是根据前一个信号传递进来的参数重新建立了一个信号，这个参数，可能会在创建信号的时候用到，也有可能用不到。

```objc
- (void)RAC_flattenMap {
    [[self.testTextField.rac_textSignal map:^id _Nullable(NSString * _Nullable value) {
        return [NSString stringWithFormat:@"map自定义:%@",value];
    }] subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x);
    }];
    
    [[self.testTextField.rac_textSignal flattenMap:^__kindof RACSignal * _Nullable(NSString * _Nullable value) {
        
        //自定义返回内容
        return [RACSignal return:[NSString stringWithFormat:@"自定义返回信号：%@",value]];
    }] subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x);
    }];
}
```

进入`flattenMap`方法查看，可以发现是调用`bind`方法实现。而`flattenMap`中的block返回值，将作为`bind`中`bindBlock`返回值处理。
 当订阅绑定信号时，生成bindBlock；
 源信号发送内容时，会去调用bindBlock，而bindBlock内部就会调用flattenMap的block，把数据封装成`RACSignal`类型信号；
 flattenMap return返回的信号成为bindBlock中返回信号，当订阅bindBlock返回信号时，就会得到绑定信号的订阅者，将处理完成的信号内容send出来。

```objc
- (__kindof RACStream *)flattenMap:(__kindof RACStream * (^)(id value))block {
	Class class = self.class;

	return [[self bind:^{
		return ^(id value, BOOL *stop) {
			id stream = block(value) ?: [class empty];
			NSCAssert([stream isKindOfClass:RACStream.class], @"Value returned from -flattenMap: is not a stream: %@", stream);

			return stream;
		};
	}] setNameWithFormat:@"[%@] -flattenMap:", self.name];
}
```



map作用：是将原信号的值自定义为新的值，不需要再返回RACStream类型，value为源信号的内容，将value处理好的内容直接返回即可。map方法将会创建一个一模一样的信号，只修改其value。

```objc
- (void)RAC_map {
    [[self.testTextField.rac_textSignal map:^id _Nullable(NSString * _Nullable value) {
        return [NSString stringWithFormat:@"map自定义：%@",value];
    }] subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x);
    }];
}
```

总结一下，**同样作为映射命令，在实际开发过程中，如果使用`map`命令，则block代码块中return的是对象类型；而`flattenMap`命令block代码块中return的是一个新的信号。**

对于map与flattenMap区别的理解，参考于：
 [https://blog.csdn.net/abc649395594/article/details/46552865](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fabc649395594%2Farticle%2Fdetails%2F46552865)