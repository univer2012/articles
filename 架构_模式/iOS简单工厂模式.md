参考：
1. [iOS 简单工厂模式](https://www.jianshu.com/p/6356748e8f33)

# 一、基础（多态）
## 1.什么是多态

1. 本质是父类指针指向子类指针；
2. 面向对象三大特性之一；
3. 同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果；


## 2.OC中的多态
这里举一个例子来介绍一下OC中的多态：使用一个父类Animal，动态新建相应的子类Dog、Cat和Sheep

#### 创建一个Animal类
```
---------- AnimalModel.h ----------
#import <Foundation/Foundation.h>

typedef NS_ENUM(NSInteger, AnimalType) {
    AnimalTypeDog,//狗
    AnimalTypeCat,//猫
    AnimalTypeSheep,//羊
};

@interface AnimalModel : NSObject

//构造器 类初始化（根据不同的枚举类型初始化不同的动物）
+ (AnimalModel *)initWithType:(AnimalType)type;

//动物叫
- (void)shout;

@end

---------- AnimalModel.m ----------
#import "AnimalModel.h"
#import "DogModel.h"
#import "CatModel.h"
#import "SheepModel.h"

@implementation AnimalModel

+ (AnimalModel *)initWithType:(AnimalType)type {
    AnimalModel *model = nil;
    if (type == AnimalTypeDog) {//狗
        model = [[DogModel alloc] init];
    } else if (type == AnimalTypeCat) {
        model = [[CatModel alloc] init];
    } else if (type == AnimalTypeSheep) {
        model = [[SheepModel alloc] init];
    }
    return model;
}

- (void)shout {
    NSLog(@"动物叫");
}

@end
```
#### 新建Dog类，继承自Animal
```
---------- DogModel.h ----------
#import "AnimalModel.h"

@interface DogModel : AnimalModel

@end

---------- DogModel.m ----------
#import "DogModel.h"

@implementation DogModel

- (void)shout {
    NSLog(@"汪");
}

@end
```
#### 新建Cat类，继承自Animal
```
---------- CatModel.h ----------
#import "AnimalModel.h"

@interface CatModel : AnimalModel

@end

---------- CatModel.m ----------
#import "CatModel.h"

@implementation CatModel

- (void)shout {
    NSLog(@"喵");
}

@end
```

#### 新建Sheep类，继承自Animal
```
---------- SheepModel.h ----------
#import "AnimalModel.h"

@interface SheepModel : AnimalModel

@end

---------- SheepModel.m ----------
#import "SheepModel.h"

@implementation SheepModel

- (void)shout {
    NSLog(@"咩");
}

@end
```
#### 在ViewController中进行使用
```
#import "ViewController.h"
#import "AnimalModel.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    AnimalModel *dog = [AnimalModel initWithType:AnimalTypeDog];
    [dog shout];
    AnimalModel *cat = [AnimalModel initWithType:AnimalTypeCat];
    [cat shout];
    AnimalModel *sheep = [AnimalModel initWithType:AnimalTypeSheep];
    [sheep shout];
}

@end
```
#### 得到的打印值为
```
2017-05-24 16:35:18.333 Demo_GaoQiang[8211:3992590] 汪
2017-05-24 16:35:18.334 Demo_GaoQiang[8211:3992590] 喵
2017-05-24 16:35:18.334 Demo_GaoQiang[8211:3992590] 咩
```
#### 结论
ViewController中并没有import Dog、Cat或者Sheep（抽象子类），它们本身的对象是通过它们的父类Animal（抽象类）初始化完成，在程序运行时才会确定对象的实际类型。 在父类(抽象类)指针指向不同的对象的时候，通过父类指针调用被重写的方法，会执行该指针所指向的那个对象（抽象子类）的方法。这就是OC中的多态。


# 二、简单工厂介绍
## 1.多态和简单工厂方法的关系
第一节介绍了多态，因为简单工厂模式本质上就是基于多态的实现：**简单工厂实际上就是在抽象类和抽象子类的两者关系中加入了一个第三者——工厂类Factory，这个Factory将原本在抽象类（父类）中的接口、构造器方法写在了自己内部，从而将抽象类和抽象子类之间的耦合度降低，在需求的更改后，只需要对工厂类进行修改就可以**。

## 2.实现条件

1. 存在继承关系
2. 运用多态的特性

## 3.优点

1. 比较适合一些简单的、Model中存在大量继承情况的项目
2. 在一定程度上优化了代码，提高了代码的复用性
3. 将大量的操作放到工厂类中去处理，业务类中只负责创建需要的对象，降低了对象初始化和业务类之间逻辑的耦合

## 4.缺点
1. 必须基于继承关系
2. 工厂类中集中了大量的创建逻辑，但是当所有的子类（抽象子类）不是继承自同一个父类（抽象类）时，它的扩展比较困难


# 三、简单工厂模式的实现

基于第一节的代码，将AnimalModel中的构造器方法取出，放入新建的工厂类AnimalFactory中（这里只展示有修改的代码，没有修改的代码和原来一样）
```
---------- AnimalModel.h ----------
#import <Foundation/Foundation.h>

@interface AnimalModel : NSObject

//动物叫
- (void)shout;

@end

---------- AnimalModel.m ----------
#import "AnimalModel.h"

@implementation AnimalModel

- (void)shout {
    NSLog(@"动物叫");
}
@end
```
#### 新建一个AnimalFactory类
```
---------- AnimalFactory.h ----------
#import <Foundation/Foundation.h>
@class AnimalModel;

typedef NS_ENUM(NSInteger, AnimalType) {
    AnimalTypeDog,//狗
    AnimalTypeCat,//猫
    AnimalTypeSheep,//羊
};

@interface AnimalFactory : NSObject

//构造器 类初始化（根据不同的枚举类型初始化不同的动物）
+ (AnimalModel *)initWithType:(AnimalType)type;

@end

---------- AnimalFactory.m ----------
#import "AnimalFactory.h"
#import "AnimalModel.h"
#import "DogModel.h"
#import "CatModel.h"
#import "SheepModel.h"

@implementation AnimalFactory

+ (AnimalModel *)initWithType:(AnimalType)type {
    AnimalModel *model = nil;
    if (type == AnimalTypeDog) {//狗
        model = [[DogModel alloc] init];
    } else if (type == AnimalTypeCat) {
        model = [[CatModel alloc] init];
    } else if (type == AnimalTypeSheep) {
        model = [[SheepModel alloc] init];
    }
    return model;
}
@end
```
#### 在ViewController中进行使用
```
#import "ViewController.h"
#import "AnimalModel.h"
#import "AnimalFactory.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    AnimalModel *dog = [AnimalFactory initWithType:AnimalTypeDog];
    [dog shout];
    AnimalModel *cat = [AnimalFactory initWithType:AnimalTypeCat];
    [cat shout];
    AnimalModel *sheep = [AnimalFactory initWithType:AnimalTypeSheep];
    [sheep shout];
}
@end
```
#### 得到的打印值为
```
2017-05-24 17:17:34.646 Demo_GaoQiang[8337:4159039] 汪
2017-05-24 17:17:34.646 Demo_GaoQiang[8337:4159039] 喵
2017-05-24 17:17:34.646 Demo_GaoQiang[8337:4159039] 咩
```
#### 结论
注意对比简单工厂模式的实现和多态的实现，两者的区别其实仅仅只是一个工厂类Factory而已