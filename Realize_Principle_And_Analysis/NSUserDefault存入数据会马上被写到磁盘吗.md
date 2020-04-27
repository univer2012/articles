问题：如果往NSUserDefault存入数据，数据是会马上被写到磁盘吗？如果是，是通过什么方式实现？如果不是，是什么时候写入磁盘？


参考：
[NSUserDefault 如何及时存储 (synchronize)](https://blog.csdn.net/weixin_33946605/article/details/87594522)


需要注意的是，==`NSUserDefaults`是定时把缓存中的数据写入磁盘的，而不是即时写入，为了防止在写完`NSUserDefaults`后程序退出导致的数据丢失，可以在写入数据后使用`synchronize`强制立即将数据写入磁盘==：
```
[mySetting synchronize];
```

运行上面的语句后，`NSUserDefaults`中的数据立即被写入到`.plist`文件中，如果是在模拟器上运行程序，可以在Mac`的/Library/Prefereces`目录下面找到一个文件名为`com.apple.PeoplePicker.plist`的plist文件，用Xcode打开该文件，可以看到刚才写入的数据。