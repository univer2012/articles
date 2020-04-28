一言以蔽之：为了安全！

当一个可变字符串（NSMutableString）赋值给一个字符串属性（无论这个字符串是NSString还是NSMutableString），

如果此属性是retain修饰的，就是浅拷贝，引用计数加1。赋值后源字符串改变，这个属性值也跟着改变。

如果此属性是copy修饰的，就是深拷贝，引用计数等于1（因为从堆里新分配一个内存块）。赋值后源字符串改变，这个属性值不会改变。（保证了安全）

假设对象有下面4个属性：

```objc
@property (retain, nonatomic) NSString *retainStr;
@property (copy, nonatomic)   NSString *copyStr;
@property (retain, nonatomic) NSMutableString *retainMStr;
@property (copy, nonatomic)   NSMutableString *copyMStr;
```

```objc
NSMutableString *mStr = [NSMutableString string];
    
    [mStr setString:@"我没变-------"];
    self.retainStr   = mStr;  // 浅拷贝，引用计数加1，
    NSLog(@"%ld",[self.retainStr retainCount]);// 2
    
    self.copyStr     = mStr;    // 深拷贝，
    NSLog(@"%ld",[self.copyStr retainCount]);// 1
    
    self.retainMStr = mStr;   // 浅拷贝，引用计数加1，
    NSLog(@"%ld",[self.retainMStr retainCount]);// 3
    
    self.copyMStr   = mStr;   // 深拷贝
    NSLog(@"%ld",[self.copyMStr retainCount]);// 1
    
    
    
    NSLog(@"retainStr:%@",  self.retainStr);
    NSLog(@"copyStr:%@",    self.copyStr);
    NSLog(@"retainMStr:%@", self.retainMStr);
    NSLog(@"copyMStr:%@",   self.copyMStr);
    
    NSLog(@"\n");
    
    [mStr setString:@"我变了--------"];
    
    NSLog(@"retainStr:%@",  self.retainStr);// 浅拷贝
    NSLog(@"%ld",[self.retainStr retainCount]);// 3
    
    NSLog(@"copyStr:%@",    self.copyStr);// 深拷贝
    NSLog(@"%ld",[self.copyStr retainCount]);// 1
    
    NSLog(@"retainMStr:%@", self.retainMStr);// 浅拷贝
    NSLog(@"%ld",[self.retainMStr retainCount]);// 3
    
    NSLog(@"copyMStr:%@",   self.copyMStr);// 深拷贝
    NSLog(@"%ld",[self.copyMStr retainCount]);// 1
    
    NSLog(@"\n");
```



当一个不可变字符串（NSString）赋值给一个字符串属性（无论这个字符串是NSString还是NSMutableString），就不存在安全性问题，都是深拷贝。此时无论retain还是copy都无所谓。

```objc
NSString *str = @"我来了";//[[NSString alloc] initWithString:@"我来了"];//两种方式都一样。都在常量区
    
    self.retainStr  = str;
    self.copyStr    = str;
    self.retainMStr = [str mutableCopy];
    self.copyMStr   = [str mutableCopy];
    
    NSLog(@"retainStr:%@",  self.retainStr);
    NSLog(@"copyStr:%@",    self.copyStr);
    NSLog(@"retainMStr:%@", self.retainMStr);
    NSLog(@"copyMStr:%@",   self.copyMStr);
    
    NSLog(@"\n");
    
    str =@"我走了";//[[NSStringalloc] initWithString:@"我走了"];//两种方式都一样
    
    NSLog(@"retainStr:%@",  self.retainStr);
    NSLog(@"copyStr:%@",    self.copyStr);
    NSLog(@"retainMStr:%@", self.retainMStr);
    NSLog(@"copyMStr:%@",   self.copyMStr);
    
    NSLog(@"\n");
```

打印结果如下：

```objc
2020-04-28 16:35:20.274732+0800 OCDemo20200321[54481:842017] retainStr:我来了
2020-04-28 16:35:20.275087+0800 OCDemo20200321[54481:842017] copyStr:我来了
2020-04-28 16:35:20.275198+0800 OCDemo20200321[54481:842017] retainMStr:我来了
2020-04-28 16:35:20.275291+0800 OCDemo20200321[54481:842017] copyMStr:我来了
2020-04-28 16:35:20.275375+0800 OCDemo20200321[54481:842017] 
2020-04-28 16:35:20.275458+0800 OCDemo20200321[54481:842017] retainStr:我来了
2020-04-28 16:35:20.275540+0800 OCDemo20200321[54481:842017] copyStr:我来了
2020-04-28 16:35:20.275658+0800 OCDemo20200321[54481:842017] retainMStr:我来了
2020-04-28 16:35:20.276175+0800 OCDemo20200321[54481:842017] copyMStr:我来了
2020-04-28 16:35:20.276441+0800 OCDemo20200321[54481:842017] 
```



---

【完】