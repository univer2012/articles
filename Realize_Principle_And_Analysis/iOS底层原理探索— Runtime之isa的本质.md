来自：[iOS底层原理探索— Runtime之isa的本质](https://www.jianshu.com/p/a3fb0919877c)



---



### 往期回顾

[iOS底层原理探索—OC对象的本质](https://www.jianshu.com/p/ffd742041946)
 [iOS底层原理探索—class的本质](https://www.jianshu.com/p/265412c910be)
 [iOS底层原理探索—KVO的本质](https://www.jianshu.com/p/1ae6a02ffb42)
 [iOS底层原理探索— KVC的本质](https://www.jianshu.com/p/91eb7eea2cdb)
 [iOS底层原理探索— Category的本质(一)](https://www.jianshu.com/p/6735b7ec0fe5)
 [iOS底层原理探索— Category的本质(二)](https://www.jianshu.com/p/bc3e9fa647cc)
 [iOS底层原理探索— 关联对象的本质](https://www.jianshu.com/p/1227f7202665)
 [iOS底层原理探索— block的本质(一)](https://www.jianshu.com/p/cebc6e8e7a2d)
 [iOS底层原理探索— block的本质(二)](https://www.jianshu.com/p/26ff309f00a0)

今天继续带领大家探索iOS之`Runtime`的本质。

# 前言

OC是一门动态性比较强的编程语言，它的动态性是基于`Runtime`的`API`。`Runtime`在我们的实际开发中占据着重要的地位，在面试过程中也经常遇到`Runtime`相关的面试题，我们在之前几期的探索分析时也经常会到`Runtime`的底层源码中查看相关实现。`Runtime`对于`iOS`开发者的重要性不言而喻，想要学习和掌握`Runtime`的相关技术，就要从`Runtime`底层的一些常用数据结构入手。掌握了它的底层结构，我们学习起来也能达到事半功倍的效果。今天先学习`isa`。

# isa

我们在[iOS底层原理探索—OC对象的本质](https://www.jianshu.com/p/ffd742041946)一文中讲解OC对象本质的时候提到，每个OC对象的底层结构体中都包含一个`isa`指针：

```objectivec
struct NSObject_IMPL {
    Class isa;
};
```

在`arm64`架构之前，`isa`仅是一个指针，保存着类对象(Class)或元类对象(Meta-Class)的内存地址，在`arm64`架构之后，苹果对`isa`进行了优化，变成了一个`isa_t`类型的**共用体(union)**结构，同时使用**位域**来存储更多的信息：

![img](https:////upload-images.jianshu.io/upload_images/1760191-c64f1f043b9a5538.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

isa.h中objc_object部分代码.png


 也就是说，我们之前熟知的OC对象的`isa`指针并不是直接指向类对象或者元类对象的内存地址，而是需要`&ISA_MASK`通过位运算才能获取到类对象或者元类对象的地址。



现在大家可能心存疑问，什么是共用体？什么是位域？位运算又是什么？不要着急，接下来一一为大家解答。

### 1、位域

位域是指信息在存储时，并不需要占用一个完整的字节， 而只需占一个或几个二进制位。例如生活中的电灯开关，它只有“开”、“关”两种状态，那我们就可以用 `1` 和 `0` 来分别代表这两种状态，这样我们就仅仅用了一个二进制位就保存了开关的状态。这样一来不仅节省存储空间，还使处理更加简便。

### 2、位运算符

在计算机语言中，除了加、减、乘、除等这样的算术运算符之外还有很多运算符，这里只为大家简单讲解一下位运算符。
 位运算符用来对二进制位进行操作，当然，操作数只能为整型和字符型数据。`C`语言中六种位运算符： `&` 按位与、 `|` 按位或、 `^` 按位异或、 `~` 非、 `<<` 左移和 `>>` 右移。
 我们依旧引用上面的电灯开关论，只不过现在我们有两个开关，`1`代表开，`0`代表关。

#### 1) 按位与 &

有0出0，全1为1。

![img](https:////upload-images.jianshu.io/upload_images/1760191-fa0399d81d860d8b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

按位与.png


 我们可以理解为在按位与运算中，两个开关是串联的，如果我们想要灯亮，需要两个开关都打开灯才会亮，所以是`1 & 1 = 1`。如果任意一个开关没打开，灯都不会亮，所以其他运算都是0。



#### 2) 按位或 |

有1出1，全0出0。



![img](https:////upload-images.jianshu.io/upload_images/1760191-2fe3a269cbd55c91.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

按位或.png



在按位或运算中，我们可以理解为两个开关是并联的，即一个开关开，灯就会亮。只有当两个开关都是关的，灯才不会亮。

#### 3) 按位异或 ^

相同为0，不同为1。



![img](https:////upload-images.jianshu.io/upload_images/1760191-0ea2ac264a0e107a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

按位异或.png

#### 4) 非 ~

非运算即取反运算，在二进制中 1 变 0 ，0 变 1。例如`110101`进行非运算后为`001010`，即`1010`。

#### 5) 左移 <<

左移运算就是把`<<`左边的运算数的各二进位全部左移若干位，移动的位数即`<<`右边的数的数值，高位丢弃，低位补0。
 左移n位就是乘以2的n次方。例如：`a<<4`是指把a的各二进位向左移动4位。如a=00000011(十进制3)，左移4位后为00110000(十进制48)。

#### 6) 右移 >>

右移运算就是把`>>`左边的运算数的各二进位全部右移若干位，`>>`右边的数指定移动的位数。例如：设 a=15，a>>2 表示把00001111右移为00000011(十进制3)。

简单了解了位运算符后，下面为大家介绍位运算符的两种运用场景。

### 位运算符的运用

#### 1、取值

可以利用`按位与 &` 运算取出指定位的值，具体操作是想取出哪一位的值就将那一位置为1，其它位都为0，然后同原数据进行按位与计算，即可取出特定的位。

> 例：`0000 0011`取出倒数第三位的值



```c
// 想取出倒数第三位的值，就将倒数第三位的值置为1，其它位为0，跟原数据按位与运算
  0000 0011
& 0000 0100
------------
  0000 0000  // 得出按位与运算后的结果，即可拿到原数据中倒数第三位的值为0
```

上面的例子中，我们从`0000 0011`中取值，则`0000 0011`被称之为**源码**。进行按位与操作设定的`0000 0100`称之为**掩码**

#### 2、设值

可以通过`按位或 |`运算符将某一位的值设为1或0。具体操作是：
 想将某一位的值置为1的话，那么就将掩码中对应位的值设为1，掩码其它位为0，将源码与掩码进行按位或操作即可。

> 例：将`0000 0011`倒数第三位的值改为1



```c
// 改变倒数第三位的值，就将掩码倒数第三位的值置为1，其它位为0，跟源码按位或运算
  0000 0011
| 0000 0100
------------
  0000 0111  // 即可将源码中倒数第三位的值改为1
```

想将某一位的值置为0的话，那么就将掩码中对应位的值设为0，掩码其它位为1，将源码与掩码进行按位或操作即可。

> 例：将`0000 0011`倒数第二位的值改为0

```c
// 改变倒数第二位的值，就将掩码倒数第二位的值置为0，其它位为1，跟源码按位或运算
  0000 0011
| 1111 1101
------------
  0000 0001  // 即可将源码中倒数第二位的值改为0
```



到这里相信大家对位运算符有了一定的了解，下面我们通过OC代码的一个例子，来将位运算符运用到实际代码开发中。

我们声明一个`Man`类，类中有三个`BOOL`类型的属性，分别为`tall`、`rich`、`handsome`，通过这三个属性来判断这个人是否高富帅。

![img](https:////upload-images.jianshu.io/upload_images/1760191-bb827c99ab964afd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1176)

Man类.png


 然后我们查看一下一个`Man`类对象所占据的内存大小：

```objectivec
NSLog(@"SGHisaMan_size: %zd", class_getInstanceSize([SGHisaMan class]) );
```

打印如下：

```
SGHisaMan_size: 16
```



![img](https:////upload-images.jianshu.io/upload_images/1760191-00af85b2256bd701.png?imageMogr2/auto-orient/strip|imageView2/2/w/1068)

Man类对象占据内存.png


 我们看到，一个`Man`类的对象占16个字节。其中包括一个`isa`指针和三个`BOOL`类型的属性，8+1+1+1=11，根据**内存对齐**原则所以一个`Man`类的对象占16个字节。



我们知道，`BOOL`值只有两种情况:`0` 或 `1`，占据一个字节的内存空间。而<font color=#FF0000>一个字节的内存空间中又有8个二进制位，并且二进制同样只有 `0` 或 `1` </font>，那么我们完全可以使用1个二进制位来表示一个`BOOL`值。也就是说我们上面声明的三个`BOOL`值最终只使用3个二进制位就可以，这样就节省了内存空间。那我们如何实现呢？

想要实现三个`BOOL`值存放在一个字节中，我们可以通过 `char` 类型的成员变量来实现。<font color=#FF0000>`char`类型占一个字节内存空间，也就是8个二进制位。</font> 可以使用其中最后三个二进制位来存储3个`BOOL`值。

**当然我们不能把 `char` 类型写成属性，因为一旦写成属性，系统会自动帮我们添加成员变量，自动实现`set`和`get`方法。**



```objectivec
@interface Man()
{
    char _tallRichHandsome;
}
```

如果我们赋值`_tallRichHansome`为`1`，即`0b 0000 0001` ，只使用8个二进制位中的最后3个分别用`0`或者`1`来代表`tall`、`rich`、`handsome`的值。那么此时`tall`、`rich`、`handsome`的状态为：

![img](https:////upload-images.jianshu.io/upload_images/1760191-6a6853df0e35f41c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1088)

char类型含义.png


 结合我们上文将的6中位运算符以及使用场景，我们可以分别声明`tall`、`rich`、`handsome`的掩码，来方便我们进行下一步的位运算取值和赋值：

```objectivec
#define Tall_Mask       0b00000100 //此二进制数对应十进制数为 4
#define Rich_Mask       0b00000010 //此二进制数对应十进制数为 2
#define Handsome_Mask   0b00000001 //此二进制数对应十进制数为 1
```

通过对位运算符的`左移 <<`和`右移 >>`的了解，我们可以将上面的代码优化成：

```objectivec
#define Tall_Mask       (1<<2) //0b00000100 
#define Rich_Mask       (1<<1) //0b00000010 
#define Handsome_Mask   (1<<0) //0b00000001 
```

自定义的`set`方法如下：

```objectivec
- (void)setTall:(BOOL)tall {
    if (tall) { // 如果需要将值置为1，将源码和掩码进行按位或运算
        _tallRichHandsome |= Tall_Mask;
    } else { // 如果需要将值置为0 // 将源码和按位取反后的掩码进行按位与运算
        _tallRichHandsome &= ~Tall_Mask;
    }
}
- (void)setRich:(BOOL)rich {
    if (rich) {
        _tallRichHandsome |= Rich_Mask;
    } else {
        _tallRichHandsome &= ~Rich_Mask;
    }
}
- (void)setHandsome:(BOOL)handsome {
    if (handsome) {
        _tallRichHandsome |= Handsome_Mask;
    } else {
        _tallRichHandsome &= ~Handsome_Mask;
    }
}
```

自定义的`get`方法如下：

```objectivec
- (BOOL)isTall {
    return !!(_tallRichHandsome & Tall_Mask);
}
- (BOOL)isRich {
    return !!(_tallRichHandsome & Rich_Mask);
}
- (BOOL)isHandsome {
    return !!(_tallRichHandsome & Handsome_Mask);
}
```

此处需要注意的是，代码中`!`为逻辑运算符`非`，因为`_tallRichHandsome & Tall_Mask`代码执行后，返回的肯定是一个整型数，如当`tall`为`YES`时，说明二进制数为`0b 0000 0100`，对应的十进制数为4，那么进行一次逻辑非运算后，`!(4)`的值为`0`，对`0`再进行一次逻辑非运算`!(0)`，结果就成了`1`，那么正好跟`tall`为`YES`对应。所以此处进行两次逻辑非运算，`!!`。

当然，还要实现初始化方法：

```objectivec
- (instancetype)init {
    if (self = [super init]) {
        _tallRichHandsome = 0b00000100;
    }
    return self;
}
```

通过测试验证，我们完成了取值和赋值：



![img](https:////upload-images.jianshu.io/upload_images/1760191-fee23d37c84b9d24.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

测试代码.png



## 使用结构体位域优化代码

我们上文讲到了`位域`的概念，那么我们就可以使用`结构体位域`来优化一下我们的代码。这样就不用再额外声明上面代码中的掩码部分。位域声明的格式是`位域名 : 位域长度`。
 在使用`位域`的过程中需要注意以下几点：

> 1.如果一个字节所剩空间不够存放另一位域时，应从下一单元起存放该位域。
>  2.位域的长度不能大于数据类型本身的长度，比如`int`类型就不能超过32位二进位。
>  3.位域可以无位域名，这时它只用来作填充或调整位置。无名的位域是不能使用的。

使用位域优化以后：

```objectivec
///==============================SGHisaMan2.h
@interface SGHisaMan2 : NSObject
{
    //位域
    struct {
        char tall : 1;
        char rich : 1;
        char handsome : 1;
    } _tallRichHandsome;
}
@property (nonatomic, assign, getter = isTall) BOOL tall;
@property (nonatomic, assign, getter = isRich) BOOL rich;
@property (nonatomic, assign, getter = isHandsome) BOOL handsome;

@end

///==============================SGHisaMan2.m
@implementation SGHisaMan2

- (void)setTall:(BOOL)tall {
    _tallRichHandsome.tall = tall;
}
- (BOOL)isTall {
    return !!_tallRichHandsome.tall;
}

- (void)setRich:(BOOL)rich {
    _tallRichHandsome.rich = rich;
}
- (BOOL)isRich {
    return !!_tallRichHandsome.rich;
}

- (void)setHandsome:(BOOL)handsome {
    _tallRichHandsome.handsome = handsome;
}
- (BOOL)isHandsome {
    return !!_tallRichHandsome.handsome;
}

@end
```


![img](https:////upload-images.jianshu.io/upload_images/1760191-d4badd28e69948af.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

使用位域优化后的代码.png

 测试看一下是否正确，这次我们将`tall`设为`YES`、`rich`设为`NO`、`handsome`设为`YES`：

```objectivec
SGHisaMan2 *man = [[SGHisaMan2 alloc] init];
man.tall = NO;
man.rich = NO;
man.handsome = YES;
NSLog(@"tall:%d,\n rich:%d, \n handsome:%d",man.isTall, man.isRich, man.isHandsome);
```

![img](https:////upload-images.jianshu.io/upload_images/1760191-0f741682bcb98add.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

优化后测试.png

依旧完成赋值和取值。

但是**代码这样优化后，我们去掉了掩码和初始化的代码，可读性很差，我们继续使用<u>共用体</u>进行优化**：



## 使用共用体优化代码

我们可以使用比较高效的位运算来进行赋值和取值，使用`union`共用体来对数据进行存储。这样不仅可以增加读取效率，还可以增强代码可读性。

```objectivec
#define Tall_Mask (1<<2) // 0b00000100
#define Rich_Mask (1<<1) // 0b00000010
#define Handsome_Mask (1<<0) // 0b00000001

@interface SGHisaMan3 : NSObject
{
    union {
        char bits;
       // 结构体仅仅是为了增强代码可读性
        struct {
            char tall : 1;
            char rich : 1;
            char handsome : 1;
        };
    } _tallRichHandsome;
}

@property (nonatomic, assign, getter = isTall) BOOL tall;
@property (nonatomic, assign, getter = isRich) BOOL rich;
@property (nonatomic, assign, getter = isHandsome) BOOL handsome;

@end

@implementation Man

- (void)setTall:(BOOL)tall {
    if (tall) {
        _tallRichHandsome.bits |= Tall_Mask;
    } else {
        _tallRichHandsome.bits &= ~Tall_Mask;
    }
}

- (void)setRich:(BOOL)rich {
    if (rich) {
        _tallRichHandsome.bits |= Rich_Mask;
    } else {
        _tallRichHandsome.bits &= ~Rich_Mask;
    }
}
- (void)setHandsome:(BOOL)handsome {
    if (handsome) {
        _tallRichHandsome.bits |= Handsome_Mask;
    } else {
        _tallRichHandsome.bits &= ~Handsome_Mask;
    }
}

- (BOOL)isTall {
    return !!(_tallRichHandsome.bits & Tall_Mask);
}
- (BOOL)isRich {
    return !!(_tallRichHandsome.bits & Rich_Mask);
}
- (BOOL)isHandsome {
    return !!(_tallRichHandsome.bits & Handsome_Mask);
}

@end
```

**其中 `_tallRichHandsome` 共用体只占用一个字节**。因为结构体中`tall`、`rich`、`handsome`都只占一位二进制空间，所以结构体只占一个字节，而`char`类型的`bits`也只占一个字节，他们都在共用体中，因此共用一个字节的内存即可。
而且我们在`set`、`get`方法中的赋值和取值通过使用掩码进行位运算来增加效率，整体逻辑也就很清晰了。



但是，如果我们在日常开发中这样写代码的话，很可能会被同事打死。虽然代码已经很清晰了，但是整体阅读起来是很吃力的。我们在这里学习位运算以及共用体这些知识，更多的是为了方便我们阅读OC底层的代码。

下面我们就回到本文主题，查看一下 `isa_t` 共用体的源码。



# isa_t共用体



![img](https:////upload-images.jianshu.io/upload_images/1760191-7e45e5e5da40e5f5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1032)

isa_t源码.png


 我们发现在 `isa_t` 共用体内用宏 `ISA_BITFIELD` 定义了位域，我们进入位域内查看源码：

![img](https:////upload-images.jianshu.io/upload_images/1760191-daeecbb3979e552e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

ISA_BITFIELD内部源码.png

 我们看到，在内部分别定义了`arm64`位架构和`x86_64`架构的掩码和位域。我们只分析`arm64`位架构下的部分内容(红色标注部分)。

可以清楚看到`ISA_BITFIELD`位域的内容以及掩码`ISA_MASK`的值：`0x0000000ffffffff8ULL`。

我们重点看一下 `uintptr_t shiftcls : 33;`，在 `shiftcls` 中存储着类对象和元类对象的内存地址信息，我们上文讲到，**对象的 `isa` 指针需要同 `ISA_MASK` 经过一次按位与运算才能得出真正的类对象地址**。那么我们将`ISA_MASK`的值`0x0000000ffffffff8ULL`转化为二进制数分析一下：

![img](https:////upload-images.jianshu.io/upload_images/1760191-ed453749423bc286.png?imageMogr2/auto-orient/strip|imageView2/2/w/794)

ISA_MASK转化为二进制.png


从图中可以看到 `ISA_MASK` 的值转化为二进制中有33位都为`1`，上文讲到按位与运算是可以取出这33位中的值。那么就说明，**同 `ISA_MASK `进行按位与运算就可以取出类对象和元类对象的内存地址信息。**



我们继续分析一下 **结构体位域中** 其他的内容代表的含义：

```c
struct {
    // 0代表普通的指针，存储着类对象、元类对象的内存地址。
    // 1代表优化后的使用位域存储更多的信息。
    uintptr_t nonpointer        : 1; 

   // 是否有设置过关联对象，如果没有，释放时会更快
    uintptr_t has_assoc         : 1;

    // 是否有C++析构函数，如果没有，释放时会更快
    uintptr_t has_cxx_dtor      : 1;

    // 存储着类对象、元类对象对象的内存地址信息
    uintptr_t shiftcls          : 33; 

    // 用于在调试时分辨对象是否未完成初始化
    uintptr_t magic             : 6;

    // 是否有被弱引用指向过。
    uintptr_t weakly_referenced : 1;

    // 对象是否正在释放
    uintptr_t deallocating      : 1;

    // 引用计数器是否过大无法存储在isa中
    // 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中
    uintptr_t has_sidetable_rc  : 1;

    // 里面存储的值是引用计数器减1
    uintptr_t extra_rc          : 19;
};
```



至此我们已经对`isa`指针有了新的认识，**`__arm64__` 架构之后，`isa` 指针不单单只存储了类对象和元类对象的内存地址，而是使用共用体的方式存储了更多信息。其中 `shiftcls` 存储了类对象和元类对象的内存地址，需要同 `ISA_MASK` 进行 `按位与 &` 运算才可以取出其内存地址值。**



---



【完】