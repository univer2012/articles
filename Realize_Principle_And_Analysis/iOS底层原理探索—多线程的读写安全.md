来自：[iOS底层原理探索—多线程的读写安全](https://www.jianshu.com/p/aeae6e6d2ed5)



---



### 前言

多线程是`iOS`开发中很重要的一个环节，无论是开发过程还是在面试环节中，多线程出现的频率都非常高。我们会通过几篇文章的探索，深入浅出的分析多线程技术。

我们通过前面三篇文章分析了一下多线程技术以及在应用过程中的问题和解决方案，我们今天继续分析多线程的读写安全问题。

# atomic

我们在实际开发过程中，声明属性的时候会经常用到`nonatomic`和`atomic`，它们实际的作用是什么呢？
 `atomic`：原子性，可以保证属性的`setter`和`getter`都是原子性操作，也就是保证`setter`和`gette`内部是`线程同步`的。`atomic`不能保证使用属性的过程是线程安全的。
 我们查看源码分析：

![set方法源码.png](https:////upload-images.jianshu.io/upload_images/1760191-f30a3b9f68633912.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

set方法源码.png


 通过源码可以看到，经过`nonatomic`和`atomic`修饰后，在赋值时分别进行了不同的操作。
 如果属性修饰词为`atomic`，则`setter`方法中会先添加自旋锁，再赋值，最后解锁；`nonatomic`直接赋值。
 再来看一下`getter`方法源码：

![getter方法源码.png](https:////upload-images.jianshu.io/upload_images/1760191-88486f1eb6528bd8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

getter方法源码.png


 <font color=#038103>**如果属性修饰词为`atomic`，则`getter`方法中会先添加自旋锁，再取值，最后解锁并返回取值。  `nonatomic` 则直接取值并返回取值。**</font>



源码中加的锁 `spinlock_t` 实际就是自旋锁`mutex`。

我们刚刚讲到，**使用`atomic`并不能保证使用属性的过程是线程安全的**。这是什么意思呢？下面一个例子来帮助我们理解：

```objectivec
// #pragma mark ----------------- Person类 -----------------
@interface Person : NSObject
@property (atomic, strong) NSMutableArray *dataArray;
@end

@implementation Person
@end

// #pragma mark ----------------- main -----------------
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *person = [[Person alloc] init];

        person.dataArray = [NSMutableArray array];
        [person.dataArray addObject:@"1"];
        [person.dataArray addObject:@"2"];

    }
    return 0;
}
```

`dataArray`属性是声明为`atomic` ，也只是在 `[person.dataArray addObject:@"1"];` 这句代码中，**`person.dataArray`(实际上调用了get方法)是线程安全的，`addObject:@"1"` 这句代码就不能保证线程安全** 。<font color=#FF0000>如果多条线程同时执行这句代码可能就会出现问题了。</font>



# iOS中的读写安全

我们考虑一下如何实现这样一个需求：

> 同一时间，只能有1个线程进行写的操作
> 同一时间，允许有多个线程进行读的操作
> 同一时间，不允许既有写的操作，又有读的操作

这个需求就是典型的**多读单写**，经常用于文件等数据的读写操作。iOS中提供了两种实现方案：

- `pthread_rwlock`——读写锁

- `dispatch_barrier_async`——异步栅栏



我们来具体分析一下这两种读写安全方案：

### 1、pthread_rwlock  读写锁

等待锁的线程会进入休眠
 需要导入`#import `头文件

```objectivec
// 初始化锁
pthread_rwlock_t lock;
pthread_rwlock_init(&lock, NULL);
// 读 - 加锁
pthread_rwlock_rdlock(&lock);
// 读 - 尝试加锁
pthread_rwlock_tryrdlock(&lock);
// 写 - 加锁
pthread_rwlock_wrlock(&lock);
// 写 - 尝试加锁
pthread_rwlock_trywrlock(&lock);
// 解锁
pthread_rwlock_unlock(&lock);
// 销毁
pthread_rwlock_destroy(&lock);
```

我们通过下面一个例子来展示读写锁的具体用法：

```objectivec
@interface ViewController ()
@property (nonatomic, assign) pthread_rwlock_t lock;
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];

    pthread_rwlock_init(&_lock, NULL);

    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    for (int i=0; i<5; i++) {
        dispatch_async(queue, ^{
            [self read];
        });
        dispatch_async(queue, ^{
            [self write];
        });
    }
}
// 允许多条线程同时 读取 操作
- (void)read {
    pthread_rwlock_rdlock(&_lock);
    sleep(1);
    NSLog(@"read");
    pthread_rwlock_unlock(&_lock);
}
// 不允许多条线程同时 写入 操作
- (void)write {
    pthread_rwlock_wrlock(&_lock);
    sleep(1);
    NSLog(@"write");
    pthread_rwlock_unlock(&_lock);
}
- (void)dealloc {
    pthread_rwlock_destroy(&_lock);
}
@end
```

我们看一下读和写的时间：

![img](https:////upload-images.jianshu.io/upload_images/1760191-734053a5265a3743.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

读写锁.png

 可以看到，在同一时间内可能进行`read`操作，但是`write`操作的时间肯定是不同的。
 说明同一时间，有多个线程进行读取的操作，只有1个线程进行写入的操作。保证了读取写入操作不会同时执行。



### 2、dispatch_barrier_async  异步栅栏

异步栅栏函数实现多读单写的原理是，将读和写操作放入同一个队列中， 将写操作放入栅栏函数中，当进入写操作时，会将写操作隔离开来，保证不会有读操作进入。



![img](https:////upload-images.jianshu.io/upload_images/1760191-d477a410ca5145d2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

异步栅栏函数.png

需要注意的是传入的并发队列必须是通过`dispatch_queue_cretate`创建的；
 如果传入的是一个串行或是一个全局的并发队列，那么异步栅栏函数便等同于`dispatch_async`函数的效果。

```objectivec
// 初始化队列
dispatch_queue_t queue = dispatch_queue_create("rw_queue", DISPATCH_QUEUE_CONCURRENT);
// 读
dispatch_async(self.queue, ^{
    [self read];
});
// 写
dispatch_barrier_async(self.queue, ^{
    [self write];
});
```

同样我们用一段示例代码来展示异步栅栏函数的用法：

```objectivec
@interface ViewController ()
@property (nonatomic, strong) dispatch_queue_t queue;
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];

    self.queue = dispatch_queue_create("rw_queue", DISPATCH_QUEUE_CONCURRENT);
    for (int i=0; i<5; i++) {
        [self read];
        [self read];
        [self write];
        [self write];
    }
}
- (void)read {
    dispatch_async(self.queue, ^{
        sleep(1);
        NSLog(@"read");
    });
}
- (void)write {
    dispatch_barrier_async(self.queue, ^{
        sleep(1);
        NSLog(@"write");
    });
}
@end
```

![异步栅栏函数测试.png](https://upload-images.jianshu.io/upload_images/1760191-e0bd6dcd6c7f99c2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



通过打印的时间可以看出，异步栅栏函数确实做到了多读单写。

---

【完】

