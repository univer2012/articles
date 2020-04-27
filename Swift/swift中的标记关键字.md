## swift中的标记关键字

来自：[[Swift]swift中使用MARK,TODO,FIXME](https://www.jianshu.com/p/307427e6d990)

##### 1. MARK

我们知道,在OC中我们可以使用pragma mark添加一些说明,能够快速定位到相应的代码,

> 例如: #pragma mark -说明文字

那么在swift中怎么实现类似的功能呢?
其实也很简单,只要在需要添加说明的地方加上如下格式的注释:

> // MARK: - 说明文字,带分割线
> // MARK: 说明文字,不带分割线

MARK一定要大写,例如:

```
// MARK: - 实例方法
public func someFunc1()  {
        
    }
```

在类视图中就会出现刚刚添加的文本注释,并且有一条分割线隔开



![img](https://upload-images.jianshu.io/upload_images/1928848-d8e93289f0c02cf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)



##### 2. TODO

是在编写一个类的时候,一些功能可能需要以后添加或者补全,这时需要一个标记,用于提醒,可使用TODO关键字添加注释:

> TODO: 需要提醒的文字

这时,在类视图中就会出现需要提醒的文字:



![img](https://upload-images.jianshu.io/upload_images/1928848-0be1e021397c185f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)



##### 3. FIXME

有时候在编程的过程中,会有一些小的bug,不紧急也不影响程序的运行,可以暂时不予处理,稍后需要修改,其用法类似TODO,只是语义有所区别:

> FIXME: 需要修改bug的相关说明



![img](https://upload-images.jianshu.io/upload_images/1928848-49de243f820a1184.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)



以上就是在swift中经常使用的在类视图添加

