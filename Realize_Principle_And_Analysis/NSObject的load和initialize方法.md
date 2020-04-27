# 0. 概述
就像Application有生命周期回调方法一样，在Objective-C的类被加载和初始化的时候，也可以收到方法回调，可以在适当的情况下做一些定制处理。而这正是load和initialize方法可以帮我们做到的。

```
+ (void)load;
+ (void)initialize;
```
可以看到这两个方法都是以“+”开头的类方法，返回为空。通常情况下，我们在开发过程中可能不必关注这两个方法。**如果有需要定制，我们可以在自定义的NSObject子类中给出这两个方法的实现，这样在类的加载和初始化过程中，自定义的方法可以得到调用。**

从如上声明上来看，也许这两个方法和其它的类方法相比没什么特别。但是，这两个方法具有一定的“特殊性”，这也是这两个方法经常会被放在一起特殊提到的原因。详细请看如下几小节的整理。

# 1. load和initialize的共同特点
load和initialize有很多共同特点，下面简单列一下：
* 在不考虑开发者主动使用的情况下，系统最多会调用一次
* **如果父类和子类都被调用，父类的调用一定在子类之前**
* 都是为了应用运行提前创建合适的运行环境
* **在使用时都不要过重地依赖于这两个方法**，除非真正必要

# 2. load方法相关要点
废话不多说，直接上要点列表：
* 调用时机比较早，运行环境有不确定因素。具体说来，在iOS上通常就是App启动时进行加载，但当load调用的时候，并不能保证所有类都加载完成且可用，必要时还要自己负责做auto release处理。
* 补充上面一点，**对于有依赖关系的两个库中，被依赖的类的load会优先调用。但在一个库之内，调用顺序是不确定的。**
* 对于一个类而言，没有load方法实现就不会调用，不会考虑对NSObject的继承。
* **一个类的load方法不用写明`[super load]`，父类就会收到调用，并且在子类之前。**
* <font color=#ff0000 size=3>Category的load也会收到调用，但顺序上在主类的load调用之后</font>。
* 不会直接触发 initialize 的调用。


# 3. initialize方法相关要点
同样，直接整理要点：
* <font color=#ff0000 size=3>initialize 的自然调用是在第一次主动使用当前类的时候</font>（lazy，这一点和Java类的“clinit”的很像）。
* 在 initialize 方法收到调用时，运行环境基本健全。
* **initialize 的运行过程中是能保证线程安全的。**
* 和load不同，**即使子类不实现 initialize 方法，也会把父类的实现继承过来调用一遍**。~~注意的是在此之前，父类的方法已经被执行过一次了，同样不需要super调用。~~

由于 initialize 的这些特点，使得其应用比load要略微广泛一些。**可用来做一些初始化工作，或者单例模式的一种实现方案。**

# 4. 原理
“源码面前没有秘密”。最后，我们来看看苹果开放出来的部分源码。从中我们也许能明白为什么load和initialize及调用会有如上的一些特点。

**其中load是在objc库中一个load_images函数中调用的，先把二进制映像文件中的头信息取出，再解析和读出各个模块中的类定义信息，把实现了load方法的类和Category记录下来，最后统一执行调用。**

其中的prepare_load_methods函数实现如下：
 ```
 void prepare_load_methods(header_info *hi)
{
    Module mods;
    unsigned int midx;
    if (_objcHeaderIsReplacement(hi)) {
        return;
    }
  
    mods = hi->mod_ptr;
    for (midx = 0; midx < hi->mod_count; midx += 1)
    {
        unsigned int index;
  
        if (mods[midx].symtab == nil)
            continue;
  
        for (index = 0; index < mods[midx].symtab->cls_def_cnt; index += 1)
        {
            Class cls = (Class)mods[midx].symtab->defs[index];
            if (cls->info & CLS_CONNECTED) {
                schedule_class_load(cls);
            }
        }
    }
    mods = hi->mod_ptr;
  
    midx = (unsigned int)hi->mod_count;
    while (midx-- > 0) {
        unsigned int index;
        unsigned int total;
        Symtab symtab = mods[midx].symtab;
  
        if (mods[midx].symtab == nil)
            continue;
        total = mods[midx].symtab->cls_def_cnt +
            mods[midx].symtab->cat_def_cnt;
  
        index = total;
        while (index-- > mods[midx].symtab->cls_def_cnt) {
            old_category *cat = (old_category *)symtab->defs[index];
            add_category_to_loadable_list((Category)cat);
        }
    }
}
 ```
这大概就是**主类中的load方法先于category**的原因。

再看下面这段：
```
static void schedule_class_load(Class cls)
{
    if (cls->info & CLS_LOADED) return;
    if (cls->superclass) schedule_class_load(cls->superclass);
    add_class_to_loadable_list(cls);
    cls->info |= CLS_LOADED;
}
```
**这正是父类load方法优先于子类调用的原因。**

再来看下initialize调用相关的源码。**objc的库里有一个`_class_initialize`方法实现**，如下：
```
void _class_initialize(Class cls)
{
    assert(!cls->isMetaClass());
  
    Class supercls;
    BOOL reallyInitialize = NO;
  
    supercls = cls->superclass;
    if (supercls  &&  !supercls->isInitialized()) {
        _class_initialize(supercls);
    }
  
    monitor_enter(&classInitLock);
    if (!cls->isInitialized() && !cls->isInitializing()) {
        cls->setInitializing();
        reallyInitialize = YES;
    }
    monitor_exit(&classInitLock);
  
    if (reallyInitialize) {
        _setThisThreadIsInitializingClass(cls);
  
        if (PrintInitializing) {
            _objc_inform("INITIALIZE: calling +[%s initialize]",
                         cls->nameForLogging());
        }
  
        ((void(*)(Class, SEL))objc_msgSend)(cls, SEL_initialize);
  
        if (PrintInitializing) {
            _objc_inform("INITIALIZE: finished +[%s initialize]",
                         cls->nameForLogging());
        }
  
        monitor_enter(&classInitLock);
        if (!supercls  ||  supercls->isInitialized()) {
            _finishInitializing(cls, supercls);
        } else {
            _finishInitializingAfter(cls, supercls);
        }
        monitor_exit(&classInitLock);
        return;
    }
  
    else if (cls->isInitializing()) {
        if (_thisThreadIsInitializingClass(cls)) {
            return;
        } else {
            monitor_enter(&classInitLock);
            while (!cls->isInitialized()) {
                monitor_wait(&classInitLock);
            }
            monitor_exit(&classInitLock);
            return;
        }
    }
  
    else if (cls->isInitialized()) {
        return;
    }
  
    else {
        _objc_fatal("thread-safe class init in objc runtime is buggy!");
    }
}
```
在这段代码里，**我们能看到initialize的调用顺序和线程安全性。**