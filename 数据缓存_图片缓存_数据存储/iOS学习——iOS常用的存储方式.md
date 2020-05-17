来自：[iOS学习——iOS常用的存储方式](https://cloud.tencent.com/developer/article/1047702)



---



 不管是在iOS还是Android开发过程中，我们都经常性地需要存储一些状态和数据，比如用户对于App的相关设置、需要在本地缓存的数据等等。根据要存储的的数据的大小、存储性质以及存储类型，在iOS和Android中哪个都有多种存储方式。其中，iOS中的存储方式主要包括以下六类：

- **plist文件（属性列表）**
- **preference（偏好设置）**
- **NSKeyedArchiver（归档）**
- **SQLite 3**
- **CoreData**
- **手动存放沙盒**

# **一、沙盒机制**

 在研究存储方式之前，我们有必要先研究下这些文件会存储到什么地方去，这就需要我们了解iOS App特有的沙盒机制了。iOS程序默认情况下只能访问程序自己的目录，这个目录被称为“沙盒”，即沙盒其实就是一个App特有的一个文件夹，iOS下每个App都有自己特有的一个沙盒，其结构和目录特性都是一样的。

## 1.1 沙盒结构

　　既然沙盒就是一个文件夹，那就看看里面有什么吧。沙盒的目录结构如下图所示，每个App的沙盒都是由下图所示的四部分组成，每一部分中存放的数据和内容都是有一定的规范和性质的。该目录路径的获取方法是直接通过 NSHomeDirectory() 就得到和应用沙盒的路径。

![mi8rx3okq4.png](https://upload-images.jianshu.io/upload_images/843214-da37f139532611bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　此外，每一个App还有一个**Bundle目录**，即“**应用程序包**”，该目录下 存放的是应用程序的源文件，包括资源文件和可执行文件。

## 1.2 沙盒目录特性

　　虽然沙盒中有这么多文件夹，但是没有文件夹都不尽相同，都有各自的特性。所以在选择存放目录时，一定要认真选择适合的目录。

- **应用程序包**：存放的是应用程序的源文件，包括资源文件和可执行文件。如果你要仿写某一个App或借用某个App的应用图标，可以在该App的应用程序包中找到其.app结尾的源文件，然后显示报内容即可直接获取到其所有的图标和应用切图。在开发中获取其bundle（应用程序包）路径的方法是：  

```objectivec
NSString *path = [[NSBundle mainBundle] bundlePath];
NSLog(@"%@", path);
```

- **Documents**: 最常用的目录，iTunes同步该应用时会同步此文件夹中的内容，**适合存储重要数据**。如果自己存储log数据到本地，一般是保存到该路径下。获取路径下的方法是：  

```objectivec
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
NSLog(@"%@", path);
```

- **Library/Caches**: iTunes不会同步此文件夹，**适合存储体积大，不需要备份的非重要数据**。  

```objectivec
SString *path = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject;
NSLog(@"%@", path);
```

- **Library/Preferences**: iTunes同步该应用时会同步此文件夹中的内容，**通常保存应用的设置信息**。

- **tmp**：iTunes不会同步此文件夹，系统可能在应用没运行时就删除该目录下的文件，所以**此目录适合保存应用中的一些临时文件，用完就删除。**  

```objectivec
NSString *path = NSTemporaryDirectory();
NSLog(@"%@", path);
```



# 二、存储方式

 在文章的开始已经讲到了，iOS中本地存储的方式一般有6种。下面我们将一个个来进行学习和研究。

## 2.1 **plist文件(属性列表)**

 plist文件是将某些特定的类，通过XML文件的方式保存在目录中。可以被序列化的类型只有如下几种：

- `NSArray`
- `NSMutableArray`
- `NSDictionary`
- `NSMutableDictionary`
- `NSData`
- `NSMutableData`
- `NSString`
- `NSMutableString`
- `NSNumber`
- `NSDate`

**1. 获得文件路径**

 项目中**plist文件是存储在沙盒的documents**中，所以要获取某个plist文件，只需要知道其文件名就可以了，如下方式就好可以获取并读取其中的内容，读取时通过对应类型的方式来获取plist的数据。一般plist中的内容都是以NSArray或`NSDictionary的形式保存。`

```objectivec
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
NSString *fileName = [path stringByAppendingPathComponent:@"123.plist"];
NSArray *result = [NSArray arrayWithContentsOfFile:fileName];
```

**2. 存储**

往plist中写内容也非常简单，直接用相应类型的writeToFile方法即可。

```objectivec
NSArray *array = @[@"123", @"456", @"789"];
[array writeToFile:fileName atomically:YES];
```

**3. 注意**

-  只有以上列出的类型才能使用plist[文件存储](https://cloud.tencent.com/product/cfs?from=10680)。 
-  存储时使用writeToFile: atomically:方法。 其中atomically表示是否需要先写入一个辅助文件，再把辅助文件拷贝到目标文件地址。这是更安全的写入文件方法，一般都写YES。 
-  读取时使用arrayWithContentsOfFile:方法 

## 2.2 **preference（偏好设置）**

 preefrence（偏好设置）顾名思义就是用户在使用过程中对App的一些状态和自定义设置状态的保存，例如App的皮肤样式、游戏时是否屏蔽电话和聊天、界面显示格式等等。一般对于一些基本的用户设置，因为数据量很小，我们可以使用OC语言中的NSUserDefaults类来进行处理。使用方法很简单，只需要调用类中的方法即可。此外，NSUserDefaults 创建的数据其实也是一个plist文件，其中数据保存格式是键值对形式，即NSDictionary形式，该文件存放在**沙盒 Library/Preferences/ 目录**下，一个以你包名命名的.plist文件。

**1. 使用方法**

```objectivec
//1.获得NSUserDefaults文件
NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
//2.向文件中写入内容
[userDefaults setObject:@"AAA" forKey:@"a"];
[userDefaults setBool:YES forKey:@"sex"];
[userDefaults setInteger:21 forKey:@"age"];
//2.1立即同步
[userDefaults synchronize];
//3.读取文件
NSString *name = [userDefaults objectForKey:@"a"];
BOOL sex = [userDefaults boolForKey:@"sex"];
NSInteger age = [userDefaults integerForKey:@"age"];
NSLog(@"%@, %d, %ld", name, sex, age);
```

**2. 注意**

-  偏好设置是专门用来保存应用程序的配置信息的，一般不要在偏好设置中保存其他数据。 
-  如果没有调用synchronize方法，系统会根据I/O情况不定时刻地保存到文件中。所以如果需要立即写入文件的就必须调用synchronize方法。 
-  偏好设置会将所有数据保存到同一个文件中。即preference目录下的一个以此应用包名来命名的plist文件。 

## 2.3  **NSKeyedArchiver（归档）**

 之前说了，不管是NSUserDefaults 或者是 plist 都不能对自定义的对象进行存储，OC提供了解归档恰好解决这个问题。归档在iOS中是另一种形式的序列化，只要遵循了NSCoding协议的对象都可以通过它实现序列化。由于决大多数支持存储数据的Foundation和Cocoa Touch类都遵循了NSCoding协议，因此，对于大多数类来说，归档相对而言还是比较容易实现的。

**1. 遵循NSCoding协议**

 NSCoding协议声明了两个方法，这两个方法都是必须实现的。一个用来说明如何将对象编码到归档中，另一个说明如何进行解档来获取一个新对象。

- **遵循协议和设置属性**

```objectivec
//1.遵循NSCoding协议 
@interface Person : NSObject   //2.设置属性
@property (strong, nonatomic) UIImage *avatar;
@property (copy, nonatomic) NSString *name;
@property (assign, nonatomic) NSInteger age;
@end
```

**实现协议方法**

```objectivec
//解档
- (id)initWithCoder:(NSCoder *)aDecoder {
    if ([super init]) {
        self.avatar = [aDecoder decodeObjectForKey:@"avatar"];
        self.name = [aDecoder decodeObjectForKey:@"name"];
        self.age = [aDecoder decodeIntegerForKey:@"age"];
    }
    return self;
}
//归档
- (void)encodeWithCoder:(NSCoder *)aCoder {
    [aCoder encodeObject:self.avatar forKey:@"avatar"];
    [aCoder encodeObject:self.name forKey:@"name"];
    [aCoder encodeInteger:self.age forKey:@"age"];
}
```

- **特别注意**

 如果需要归档的类是某个自定义类的子类时，就需要在归档和解档之前先实现父类的归档和解档方法。即 [super encodeWithCoder:aCoder] 和 [super initWithCoder:aDecoder] 方法。

**2. 使用**

　　需要把对象归档是调用NSKeyedArchiver的工厂方法 archiveRootObject: toFile: 方法。

```objectivec
NSString *file = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.data"];
Person *person = [[Person alloc] init];
person.avatar = self.avatarView.image;
person.name = self.nameField.text;
person.age = [self.ageField.text integerValue];
[NSKeyedArchiver archiveRootObject:person toFile:file];
```

需要从文件中解档对象就调用NSKeyedUnarchiver的一个工厂方法 unarchiveObjectWithFile: 即可。

```objectivec
NSString *file = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.data"];
Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:file];
if (person) {
     self.avatarView.image = person.avatar;
     self.nameField.text = person.name;
     self.ageField.text = [NSString stringWithFormat:@"%ld", person.age];
}
```

**3. 注意**

-  必须遵循并实现NSCoding协议 
-  保存文件的扩展名可以任意指定 
-  继承时必须先调用父类的归档解档方法 
- 扩展：[iOS开发基础-数据存储方式（归档）](https://www.jianshu.com/p/94657b7d31d0)

## 2.4 手动存放沙盒

 手动将数据存放到沙盒，其实就是自己在沙盒的某一个指定路径（第一部分介绍了沙盒各目录路径的获取方式）下新建一个保存数据的文件（.txt、.plist、.data等格式的文件），然后向其中写我们需要保存的数据即可。但是沙盒中只能保存OC中的基本数据，自定义的对象不能直接存入，但是可以通过归档存为.data文件。 

```objectivec
//假设我们需往cache 存入数据，并命名为test的txt格式文件中
NSString *filePath = [cachesDir stringByAppendingPathComponent:@"test.txt"];
NSArray *dic = [[NSArray alloc] initWithObjects:@"test",@"test1" ,nil];
    
if([dic writeToFile:filePath atomically:YES]){
    NSLog(@"存入成功");
}
//取出数据 打印
NSLog(@"%@",[NSArray arrayWithContentsOfFile:filePath]); 
```

## 2.5 CoreData

 Core Date是ios3.0后引入的数据持久化解决方案，它是是苹果官方推荐使用的，不需要借助第三方框架。Core Date实际上是对SQLite的封装，提供了更高级的持久化方式。在对数据库操作时，不需要使用sql语句，也就意味着即使不懂sql语句，也可以操作数据库中的数据。

　　在各类应用开发中使用数据库操作时通常都会用到 (ORM) “对象关系映射”，Core Data就是这样的一种模式。ORM是将关系数据库中的表，转化为程序中的对象，但实际上是对数据中的数据进行操作。

　　在使用Core Data进⾏行数据库存取并不需要手动创建数据库，创建数据库的过程完全由Core Data框架自动完成，开发者需要做的就是把模型创建起来，具体数据库的创建不需要管。简单点说，Core Data实际上是将数据库的创建、表的创建、对象和表的转换等操作封装起来，极大的简化了我们的操作。

　　Core Date与SQLite相比较，SQLite比较原始，操作比较复杂，使用的是C的函数对数据库进行操作，但是SQLite可控性更强，并且能够跨平台。

　　关于Core Date的具体使用方法参见：[IOS 数据存储之 Core Data详解](http://www.cnblogs.com/jerehedu/p/4607368.html)



## 2.6 SQLite 3

　　iOS系统自带`Core Data`来进行持久化处理，而且`Core Data`可以使用图形化界面来创建对象，但是`Core Data`不是[关系型数据库](https://cloud.tencent.com/product/cdb-overview?from=10680)，对于`Core Data`来说比较擅长管理在设备上创建的数据持久化存储用户创建的对象，但是要处理大量的数据时就应该优先选择`SQL`关系型数据库来存储这些数据。 `　　Core Data`在后台也是使用SQLite来存储数据的，但是开发人员不能直接访问这些数据，只能通过`Core Data`提供的API来操作，如果一旦人为的通过`SQLite`修改这些数据那么使用`Core Data`再次访问这些数据时就会发生错误。

 SQLite是使用`C`语言写的开源库，实现了一个自包含的`SQL关系型数据库引擎`，可以使用`SQLite`存储操作大量的数据，作为关系型数据库我们可以在一个数据库中建立多张相关联的表来解决大量数据重复的问题。而且`SQLite`库也针对移动设备上的使用进行了优化。 因为`SQLite`的接口使用`C`写的，而且`Objective-C`是`C`的超集所以可以直接在项目中使用`SQLite`。 

　　关于SQLite的详细使用方法详见：[**iOS开发数据库篇—SQLite的应用**](https://www.cnblogs.com/wendingding/p/3870289.html)



---

【完】