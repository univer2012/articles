参考：
1. [KVC和KVO简单的区别](https://blog.csdn.net/u013983033/article/details/84071530)
2. [iOS KVC和KVO详解](https://www.jianshu.com/p/b9f020a8b4c9)

### KVC
KVC：键值赋值，使用最多的即使字典转模型。利用runtime获取对象的所有成员变量， 在根据kvc键值赋值，进行字典转模型

- `setValue: forKey:` 只查找本类里面的属性
- `setValue: forKeyPath:`会查找本类里面属性，没有会继续查找父类里面属性。

#### KVO
KVO：键值观察，根据键对应的值的变化，来调用方法。

- 注册观察者：`addObserver:forKeyPath:options:context:`
- 实现观察者：`observeValueForKeyPath:ofObject:change:context:`
- 移除观察者：`removeObserver:forKeyPath:`(对象销毁，必须移除观察者)

### 注意

使用kvo监听A对象的时候，监听的本质不是这个A对象，而是系统创建的一个中间对象`NSKVONotifying_A`并继承A对象，并且A对象的isa指针指向的也不是A的类，而是这个`NSKVONotifying_A`对象.