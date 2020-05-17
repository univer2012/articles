来自：[iOS底层原理探索 — 内存管理(二)](https://www.jianshu.com/p/c38b5741919b)



---



### 往期回顾

[iOS底层原理探索 — OC对象的本质](https://www.jianshu.com/p/ffd742041946)
 [iOS底层原理探索 — class的本质](https://www.jianshu.com/p/265412c910be)
 [iOS底层原理探索 — KVO的本质](https://www.jianshu.com/p/1ae6a02ffb42)
 [iOS底层原理探索 — KVC的本质](https://www.jianshu.com/p/91eb7eea2cdb)
 [iOS底层原理探索 — Category的本质(一)](https://www.jianshu.com/p/6735b7ec0fe5)
 [iOS底层原理探索 — Category的本质(二)](https://www.jianshu.com/p/bc3e9fa647cc)
 [iOS底层原理探索 — 关联对象的本质](https://www.jianshu.com/p/1227f7202665)
 [iOS底层原理探索 — block的本质(一)](https://www.jianshu.com/p/cebc6e8e7a2d)
 [iOS底层原理探索 — block的本质(二)](https://www.jianshu.com/p/26ff309f00a0)
 [iOS底层原理探索 — Runtime之isa的本质](https://www.jianshu.com/p/a3fb0919877c)
 [iOS底层原理探索 — Runtime之class的本质](https://www.jianshu.com/p/941cb11538aa)
 [iOS底层原理探索 — Runtime之消息机制](https://www.jianshu.com/p/c8d98d39f7ee)
 [iOS底层原理探索 — RunLoop的本质](https://www.jianshu.com/p/31237ae8c539)
 [iOS底层原理探索 — RunLoop的应用](https://www.jianshu.com/p/9475317619f7)
 [iOS底层原理探索 — 多线程的本质](https://www.jianshu.com/p/1faf2d78136c)
 [iOS底层原理探索 — 多线程的经典面试题](https://www.jianshu.com/p/3192a73fa7c2)
 [iOS底层原理探索 — 多线程的“锁”](https://www.jianshu.com/p/b33090b13d24)
 [iOS底层原理探索 — 多线程的读写安全](https://www.jianshu.com/p/aeae6e6d2ed5)
 [iOS底层原理探索 — 内存管理(一)](https://www.jianshu.com/p/70cc9623caa9)

# 前言

内存管理在APP开发过程中占据着一个很重要的地位，在iOS中，系统为我们提供了 `ARC` 的开发环境，帮助我们做了很多内存管理的内容，其实在`MRC`时代，内存管理对于开发者是个很头疼的问题。我们会通过几篇文章的分析，来帮助我们了解iOS中内存管理的原理，以及在`ARC`的开发环境下系统帮助我们做了哪些内存管理的操作。

在iOS中，系统为我们提供了一下三种内存管理方法：

- **Tagged Pointer技术**
- **NONPOINTER_ISA**
- **散列表**



其中，`Tagged Pointer` 技术和 `NONPOINTER_ISA` 我们在 [iOS底层原理探索 — 内存管理(一)](https://www.jianshu.com/p/70cc9623caa9) 一文中有简单介绍，这里不在赘述。本文主要针对散列表展开分析。

# 散列表

我们在[iOS底层原理探索 — 内存管理(一)](https://www.jianshu.com/p/70cc9623caa9)一文中介绍引用计数时讲到，在`NONPOINTER_ISA`中有两个成员变量`has_sidetable_rc`和`extra_rc` 。当 `extra_rc` 的19位内存不够存储引用计数时，`has_sidetable_rc`的值就会变为1，那么此时引用计数会存储在`SideTable`中。

系统在底层构建了`StripedMap`结构：  

![img](https:////upload-images.jianshu.io/upload_images/1760191-4a1071acb3db7772.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

StripedMap.png

> Striped   [straɪpt]  adj. 有条纹的；有斑纹的



`SideTables`可以理解为一个全局的`hash`数组，里面存储了`SideTable`类型的数据，其长度为64，就是里面有64个`SideTable`。

 我们来看`StripedMap`模板的定义：

![img](https:////upload-images.jianshu.io/upload_images/1760191-dcae4dea7896062d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

StripedMap<T>.png

 通过官方解释可以知道， `StripedMap` 是一个以`void *`为`key`， `T` 为 `vaule` 的 `hash` 表。
 在 `SideTables` 中，`T` 就是 `SideTable`类型的数据。
 我们来看一下 `SideTable` 的具体结构：



### SideTable

我们截取`SideTable`部分源码进行分析：

![img](https:////upload-images.jianshu.io/upload_images/1760191-38f4b4ccdc6ad5ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

SideTable.png


 可以看到，`SideTable` 的定义很简单，包括三个成员。接下来我们具体分析一下这三个成员。



#### 1.spinlock_t slock

`spinlock_t`的底层是 `os_unfair_lock` 类型的锁，主要作用就是防止线程冲突。这里不做过多分析。

#### 2.RefcountMap 引用计数表

`RefcountMap` 用来存储OC对象的引用计数。它实质上是一个以 `objc_object` 为 `key` 的散列表，其`vaule`就是OC对象的引用计数。同时，当OC对象的引用计数变为0时，会自动将相关的信息从散列表中剔除。`RefcountMap`的定义如下：

![img](https:////upload-images.jianshu.io/upload_images/1760191-1ee806e50549f8b4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

RefcountMap.png


 实质上是模板类型`objc::DenseMap`。模板的三个类型参数**`DisguisedPtr`、`size_t`、`true` 分别表示`DenseMap`的 `key` 类型、`value` 类型、是否需要在`value == 0` 的时候自动释放掉响应的`hash`节点**，这里是`true`。

>  dense  [dens]  adj. 稠密的；浓厚的；愚钝的
>
>  disguised 伪装的   
>  disguise [dɪsˈɡaɪz] vt. 掩饰；假装；隐瞒  n. 伪装；假装；用作伪装的东西



我们在 [iOS底层原理探索 — 内存管理(一)](https://www.jianshu.com/p/70cc9623caa9) 一文中讲到，底层调用`rootRetainCount`方法，内部会做一系列的引用计数操作。读者可以结合上文讲述的 `RefcountMap` 引用计数表回顾一下，本文就不在做赘述。

#### 3.weak_table_t 弱引用表

**弱引用表 `weak_table_t` 就是用来存储OC对象弱引用的相关信息**。我们知道，`SideTables`一共只有64个节点，而在我们的APP中，一般都会不只有64个对象。因此，**多个对象一定会重用同一个`SideTable`节点，也就是说，一个`weak_table`会存储多个对象的弱引用信息**。因此在一个`SideTable`中，又会通过`weak_table`作为`hash`表，再次分散存储每一个对象的弱引用信息。



我们来看一下 **`weak_table_t` 的结构：**

![img](https:////upload-images.jianshu.io/upload_images/1760191-e6668f60fb116311.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

weak_table_t.png

 通过官方注释可以看出，**`weak_table_t`是一个全局弱引用表。 将对象id存储为 `key` ，将 `weak_entry_t` 结构存储为 `value`。**

通过源码可以看到，**成员 `weak_entries`实质上是一个`hash`数组，数组中存储 `weak_entry_t` 类型的元素**。



我们重点来分析一下 `weak_entry_t`，我们先看一下源码：

```c
// The address of a __weak variable.
// These pointers are stored disguised so memory analysis tools
// don't see lots of interior pointers from the weak table into objects.
typedef DisguisedPtr<objc_object *> weak_referrer_t;

#if __LP64__
#define PTR_MINUS_2 62
#else
#define PTR_MINUS_2 30
#endif

/**
 * The internal structure stored in the weak references table. 
 * It maintains and stores
 * a hash set of weak references pointing to an object.
 * If out_of_line_ness != REFERRERS_OUT_OF_LINE then the set
 * is instead a small inline array.
 */
#define WEAK_INLINE_COUNT 4

// out_of_line_ness field overlaps with the low two bits of inline_referrers[1].
// inline_referrers[1] is a DisguisedPtr of a pointer-aligned address.
// The low two bits of a pointer-aligned DisguisedPtr will always be 0b00
// (disguised nil or 0x80..00) or 0b11 (any other address).
// Therefore out_of_line_ness == 0b10 is used to mark the out-of-line state.
#define REFERRERS_OUT_OF_LINE 2

struct weak_entry_t {
    DisguisedPtr<objc_object> referent; // 被弱引用的对象
    
    // 引用该对象的对象列表。 当引用个数小于4时，用inline_referrers数组。 引用个数大于4时，用动态数组weak_referrer_t *referrers
    union {
        struct {
            weak_referrer_t *referrers;                      // 弱引用该对象的对象列表的动态数组
            uintptr_t        out_of_line_ness : 2;           // 是否使用动态数组标记位
            uintptr_t        num_refs : PTR_MINUS_2;         // 动态数组中元素的个数
            uintptr_t        mask;                           // 用于hash确定动态数组index，值实际上是动态数组空间长度-1
            uintptr_t        max_hash_displacement;          // 最大的hash冲突次数
     
        };
        struct {
            // out_of_line_ness field is low bits of inline_referrers[1]
            weak_referrer_t  inline_referrers[WEAK_INLINE_COUNT];
        };
    };

    bool out_of_line() {
        return (out_of_line_ness == REFERRERS_OUT_OF_LINE);
    }
    //赋值方法
    weak_entry_t& operator=(const weak_entry_t& other) {
        memcpy(this, &other, sizeof(other));
        return *this;
    }
    // 构造方法，里面初始化了静态数组
    weak_entry_t(objc_object *newReferent, objc_object **newReferrer)
        : referent(newReferent)
    {
        inline_referrers[0] = newReferrer;
        for (int i = 1; i < WEAK_INLINE_COUNT; i++) {
            inline_referrers[i] = nil;
        }
    }
};
```

下面具体解释一下 `weak_entry_t` 中所有成员的含义：

> `DisguisedPtr referent`：**弱引用对象指针摘要**。其实可以理解为弱引用对象的指针，只不过这里使用了摘要的形式存储。（所谓摘要，其实是把地址取负）。

> `union` ：引用该对象的对象列表，有两种形式：
>
> - 定长数组`inline_referrers[WEAK_INLINE_COUNT]`
> - 动态数组 `referrers`。
>    这两个数组是用来存储 <u>弱引用该对象的指针的指针</u>，同样也**使用了指针摘要的形式存储**。当弱引用该对象的指针数目小于等于`WEAK_INLINE_COUNT`时，使用定长数组。当超过 `WEAK_INLINE_COUNT` 时，会将定长数组中的元素转移到动态数组中，并之后都是用动态数组存储。

> - `bool out_of_line()`： 该方法 **用来判断当前的 `weak_entry_t` 是使用的定长数组还是动态数组**。当返回`true`，此时使用的动态数组；当返回`false`，使用定长数组。



`referrers` 数组里面存储的元素是 `weak_referrer_t` 类型，弱引用该对象的指针的指针，以指针摘要的形式存储的。

至此，我们对`SideTables`的结构有了大致的了解，下面一张图来概括一下：

![img](https:////upload-images.jianshu.io/upload_images/1760191-a27a5ba16b04bac3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

SideTables结构示意图.png



---

【完】