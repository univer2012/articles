来自：[iOS 打印对象内存地址的正确姿势](http://www.jianshu.com/p/d2d248e63ff1)

1. 地址有两种情况：

   - 指针指向的对象的内存地址，也就是这个指针保存的内容；
   - 指针自己的内存地址。

2. 打印的正确姿势：

   ```objc
   NSString *string = @"immutableObject";
   NSString *stringCopy = [string copy];
   NSMutableString *stringMutableCopy = [string mutableCopy];
   //打印对象的内存地址
   NSLog(@"\n string: %p,\n stringCopy: %p,\n stringMutableCopy: %p", string, stringCopy, stringMutableCopy);
   //打印指针自己的内存地址 d：十进制，x：16进制
   NSLog(@"\n string: %x,\n stringCopy: %x,\n stringMutableCopy: %x", &string, &stringCopy, &stringMutableCopy);
   ```

打印如下：

```objc
string: 0x102c8b3b8,
stringCopy: 0x102c8b3b8,
stringMutableCopy: 0x604000443fc0

string: 5cf786b8,
stringCopy: 5cf786b0,
stringMutableCopy: 5cf786a8
```

