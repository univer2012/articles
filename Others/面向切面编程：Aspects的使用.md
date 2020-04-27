参考：
1. [iOS三方库Aspects的使用](https://www.jianshu.com/p/f508ab4b0b75)
2. [iOS架构-view视图层的组织和调用方案](https://www.jianshu.com/p/41ab86897d65)
3. [Aspects使用](https://www.jianshu.com/p/10d342ae6ccd)


#### Aspects的使用实例：
2.1 勾取UIViewController 类所有实例的viewWillAppear:方法

2.2 勾取UIControl 类的所有实例的sendAction:to:forEvent:方法

2.3 勾取RootViewController类的rootVC 实例的viewDidLoad 方法

代码如下
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    
    UIWindow *window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    self.window = window;
    
    RootViewController *rootVC = [[RootViewController alloc] init];
    UINavigationController *rootNav = [[UINavigationController alloc] initWithRootViewController:rootVC];
    self.window.rootViewController = rootNav;
    [self.window makeKeyAndVisible];
    
    // 勾取 UIViewController 类所有实例的viewWillAppear: 方法
    [UIViewController aspect_hookSelector:@selector(viewWillAppear:) withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo> info){
        UIViewController *tempVC = (UIViewController *)info.instance;
        NSLog(@"%@ viewWillAppear",[tempVC class]);
    } error:nil];
    
    [UIControl aspect_hookSelector:@selector(sendAction:to:forEvent:) withOptions:AspectPositionBefore usingBlock:^(id<AspectInfo> info){
        UIControl *control = (UIControl *)info.instance;
        if ([control isKindOfClass:[UIButton class]]) {
            UIButton *btn = (UIButton *)control;
            NSLog(@"%@",btn.titleLabel.text);
        }
    } error:nil];
    
    // 勾取rootVC 实例的viewDidLoad 方法
    [rootVC aspect_hookSelector:@selector(viewDidLoad) withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo> info){
        NSLog(@"RootViewController viewDidLoad");
    } error:nil];
    
    return YES;
}
```


### 对于 UI 界面的编写工作，到底应该用 xib/storyboard 完成，还是用手写代码来完成？

1. 对于复杂的、动态生成的界面，建议使用手工编写界面。
2. 对于需要统一风格的按钮或UI控件，建议使用手工用代码来构造。方便之后的修改和复用。
3. 对于需要有继承或组合关系的 UIView 类或 UIViewController 类，建议用代码手工编写界面。
4. 对于那些简单的、静态的、非核心功能界面，可以考虑使用 xib 或 storyboard 来完成。

### 是否有必要让业务方统一派生ViewController
有的时候我们出于记录用户操作行为数据的需要，或者统一配置页面的目的，会从UIViewController里面派生一个自己的ViewController，来执行一些通用逻辑。比如天猫客户端要求所有的ViewController都要继承自TMViewController（其实我们同花顺也是这样的所有ViewController都要继承自HXViewController）。这个统一的父类里面针对一个ViewController的所有生命周期都做了一些设置，至于这里都有哪些设置对于本篇文章来说并不重要。

在这里我想讨论的是，在设计View架构时，如果为了能够达到统一设置或执行统一逻辑的目的，使用派生的手段是有必要的吗？

我觉得没有必要，为什么没有必要？

1. 使用派生比不使用派生更容易增加业务方的使用成本
2. 不使用派生手段一样也能达到统一设置的目的

### 为什么使用了派生，业务方的使用成本会提升？
其实不光是业务方的使用成本，架构的维护成本也会上升。那么具体的成本都来自于哪里呢？

#### 1.集成成本
这里讲的集成成本是这样的：如果业务方自己开了一个独立demo，快速完成了某个独立流程，现在他想把这个现有流程集合进去。那么问题就来了，他需要把所有独立的UIViewController改变成TMViewController。

那为什么不是一开始就立刻使用TMViewController呢？因为要想引入TMViewController，就要引入整个天猫App所有的业务线，所有的基础库，因为这个父类里面涉及很多天猫环境才有的内容，所谓拔出萝卜带出泥，你要是想简单继承一下就能搞定的事情，搭环境就要搞半天，然后这个小Demo才能跑得起来。

对于业务层存在的所有父类来说，它们是很容易跟项目中的其他代码纠缠不清的，这使得业务方开发时遇到一个两难问题：要么把所有依赖全部搞定，然后基于App环境（比如天猫）下开发Demo，要么就是自己Demo写好之后，按照环境要求改代码。这里的两难问题都会带来成本，都会影响业务方的迭代进度。

#### 2.上手接受成本
新来的业务工程师有的时候不见得都记得每一个ViewController都必须要派生自TMViewController而不是直接的UIViewController。新来的工程师他不能直接按照苹果原生的做法去做事情，他需要额外学习，比如说：所有的ViewController都必须继承自TMViewController。

#### 3.架构的维护难度
尽可能少地使用继承能提高项目的可维护性，具体内容见Casa《[跳出面向对象思想（一） 继承](https://casatwy.com/tiao-chu-mian-xiang-dui-xiang-si-xiang-yi-ji-cheng.html)》

### 那么如果不使用派生，我们应该使用什么手段？
我的建议是使用AOP。

在架构师实现具体的方案之前，必须要想清楚几个问题，然后才能决定采用哪种方案。是哪几个问题？


1. 方案的效果，和最终要达到的目的是什么？
2. 在自己的知识体系里面，是否具备实现这个方案的能力？
3. 在业界已有的开源组件里面，是否有可以直接拿来用的轮子？

这三个问题按照顺序一一解答之后，具体方案就能出来了。

#### 我们先看第一个问题：方案的效果，和最终要达到的目的是什么？

其实就是要实现不通过业务代码上对框架的主动迎合，使得业务能够被框架感知
这样的功能。细化下来就是两个问题，框架要能够拦截到ViewController的生命周期，另一个问题就是，拦截的定义时机。

对于方法拦截，很容易想到Method Swizzling，那么我们可以写一个实例，在App启动的时候添加针对UIViewController的方法拦截，这是一种做法。还有另一种做法就是，使用NSObject的load函数，在应用启动时自动监听。使用后者的好处在于，这个模块只要被项目包含，就能够发挥作用，不需要在项目里面添加任何代码。

然后另外一个要考虑的事情就是，原有的TMViewController（所谓的父类）也是会提供额外方法方便子类使用的，Method Swizzling只支持针对现有方法的操作，拓展方法的话，嗯，当然是用Category啦。
我本人不赞成Category的过度使用，但鉴于Category是最典型的化继承为组合的手段，在这个场景下还是适合使用的。还有的就是，关于Method Swizzling
手段实现方法拦截，业界也已经有了现成的开源库：[Aspects](https://github.com/steipete/Aspects)，我们可以直接拿来使用。

#### 那么什么是AOP？
```
AOP（Aspect Oriented Programming），面向切片编程,但它跟我们熟知的面向对象编程没什么关系。
```

**那切片又是什么？**

程序要完成一件事情，一定会有一些步骤，1，2，3，4这样。这里分解出来的每一个步骤我们可以认为是一个切片。

**为什么会出现面向切片编程？**

你要想做到在每一个步骤中间做你自己的事情，不用AOP也一样可以达到目的，直接往步骤之间塞代码就好了。**但是事实情况往往很复杂，直接把代码塞进去，主要问题就在于：塞进去的代码很有可能是跟原业务无关的代码，在同一份代码文件里面掺杂多种业务，这会带来业务间耦合。为了降低这种耦合度，我们引入了AOP。**

#### 如何实现AOP？

AOP一般都是需要有一个拦截器，然后在每一个切片运行之前和运行之后（或者任何你希望的地方），通过调用拦截器的方法来把这个jointpoint扔到外面，在外面获得这个jointpoint的时候，执行相应的代码。

在iOS开发领域，objective-C的runtime有提供了一系列的方法，能够让我们拦截到某个方法的调用，来实现拦截器的功能，这种手段我们称为`Method Swizzling`。[Aspects](https://github.com/steipete/Aspects)通过这个手段实现了针对某个类和某个实例中方法的拦截。

另外，也可以使用protocol的方式来实现拦截器的功能.

### 设计心法
针对View层的架构不光是看重如何合理地拆分MVC来给UIViewController减负，另外一点也要照顾到业务方的使用成本。最好的情况是业务方什么都不知道，然后他把代码放进去就能跑，同时还能获得框架提供的种种功能。

#### 第一心法：尽可能减少继承层级，涉及苹果原生对象的尽量不要继承

**继承是罪恶，尽量不要继承**。就我目前了解到的情况看，除了安居客的Pad App没有在框架级针对UIViewController有继承的设计以外，其它公司或多或少都针对UIViewController有继承，包括安居客iPhone app（那时候我已经对此无能为力，可见View的架构在一开始就设计好有多么重要）。甚至有的还对UITableView有继承，这是一件多么令人发指，多么惨绝人寰，多么丧心病狂的事情啊。虽然不可避免的是有些情况我们不得不从苹果原生对象中继承，比如UITableViewCell。但我还是建议尽量不要通过继承的方案来给原生对象添加功能，前面提到的Aspect方案和Category方案都可以使用。用Aspect＋load来实现重载函数，用Category来实现添加函数，当然，耍点手段用Category来添加property也是没问题的。这些方案已经覆盖了继承的全部功能，而且非常好维护，对于业务方也更加透明，何乐而不为呢。

不用继承可能在思路上不会那么直观，但是对于不使用继承带来的好处是足够顶得上使用继承的坏处的。顺便在此我要给Category正一下名：业界对于Category的态度比较暧昧，在多种场合（讲座、资料文档）都宣扬过尽可能不要使用Category。它们说的都有一定道理，但我认为Category是苹果提供的最好的使用集合代替继承的方案，但针对Category的设计对架构师的要求也很高，请合理使用。而且苹果也在很多场合使用Category，来把一个原本可能很大的对象，根据不同场景拆分成不同的Category，从而提高可维护性。

不使用继承的好处我在[这里](https://casatwy.com/tiao-chu-mian-xiang-dui-xiang-si-xiang-yi-ji-cheng.html)已经说了，放到iOS应用架构来看，还能再多额外两个好处：

1. 在业务方做业务开发或者做Demo时，可以脱离App环境，或花更少的时间搭建环境。
2. 对业务方来说功能更加透明，也符合业务方在开发时的第一直觉。

#### 第二心法：做好代码规范，规定好代码在文件中的布局，尤其是ViewController

这主要是为了提高可维护性。在一个文件非常大的对象中，尤其要限制好不同类型的代码在文件中的布局。比如在写ViewController时，我之前给团队制定的规范就是前面一段全部是getter setter，然后接下来一段是`life cycle`，viewDidLoad之类的方法都在这里。然后下面一段是各种要实现的`Delegate`，再下面一段就是`event response`，Button的或者GestureRecognizer的都在这里。然后后面是`private method`。一般情况下，如果做好拆分，ViewController的private method那一段是没有方法的。后来随着时间的推移，我发现开头放`getter和setter`太影响阅读了，所以后面改成全放在ViewController的最后。

```
#pragma mark - life cycle
#pragma mark - Delegate
#pragma mark - event response
#pragma mark - private method
#pragma mark - getter和setter
```

#### 第三心法：能不放在Controller做的事情就尽量不要放在Controller里面去做
Controller会变得庞大的原因，一方面是因为Controller承载了业务逻辑，MVC的总结者（在正式提出MVC之前，或多或少都有人这么设计，所以说MVC的设计者不太准确）对Controller下的定义也是承载业务逻辑，所以Controller就是用来干这事儿的，天经地义。另一方面是因为在MVC中，关于Model和View的定义都非常明确，很少有人会把一个属于M或V的东西放到其他地方。然后除了Model和View以外，还会剩下很多模棱两可的东西，这些东西从概念上讲都算Controller，而且由于M和V定义得那么明确，所以直觉上看，这些东西放在M或V是不合适的，于是就往Controller里面塞咯。

正是由于上述两方面原因导致了Controller的膨胀。我们再细细思考一下，Model膨胀和View膨胀，要针对它们来做拆分其实都是相对容易的，Controller膨胀之后，拆分就显得艰难无比。所以如果能够在一开始就尽量把能不放在Controller做的事情放到别的地方去做，这样在第一时间就可以让你的那部分将来可能会被拆分的代码远离业务逻辑。所以我们要稍微转变一下思路：模棱两可的模块，就不要塞到Controller去了，塞到V或者塞到M或者其他什么地方都比塞进Controller好，便于将来拆分。

所以关于前面我按下不表的关于胖Model和瘦Model的选择，我的态度是更倾向于胖Model。客观地说，业务膨胀之后，代码规模肯定少不了的，不管你技术再好，经验再丰富，代码量最多只能优化，该膨胀还是要膨胀的，而且优化之后代码往往也比较难看，使用各种奇技淫巧也是有代价的。所以，针对代码量优化的结果，往往要么就是牺牲可读性，要么就是牺牲可移植性（通用性），Every magic always needs a pay, you have to make a trade-off.

那么既然膨胀出来的代码，或者将来有可能膨胀的代码，不管放在MVC中的哪一个部分，最后都是要拆分的，既然迟早要拆分，那不如放Model里面，这样将来拆分胖Model也能比拆分胖Cotroller更加容易。在我还在安居客的时候，安居客Pad app承载最复杂业务的ViewController才不到600行，其他多数Controller都是在300-400行之间，这就为后面接手的人降低了非常多的上手难度和维护复杂度。拆分出来的东西都是可以直接迁移给iPhone app使用的。现在看天猫的ViewControler，动不动就几千行，看不了多久头就晕了，问了一下，大家都表示很习惯这样的代码长度，摊手。

#### 第四心法：架构师是为业务工程师服务的，而不是去使唤业务工程师的

架构师在公司里的职级和地位往往都是要高于业务工程师的，架构师的技术实力和经验往往也都是高于业务工程师的。所以你值得在公司里获得较高的地位，但是在公司里的地位高不代表在软件工程里面的角色地位也高。架构师是要为业务工程师服务的，是他们使唤你而不是你使唤他们。

另外，制定规范一方面是起到约束业务工程师的代码，但更重要的一点是，这其实是利用你的能力帮助业务工程师避免他无法预见的危机，所以地位高有一定的好处，毕竟夏虫不可语冰，有的时候不见得能够解释得通，因此高地位随之而来的就是说服力会比较强。但在软件工程里，一定要保持谦卑，一定要多为业务工程师考虑。

一个不懂这个道理的架构师，设计出来的东西往往复杂难用，因为他只愿意做核心的东西，周边不愿意做的都期望交给业务工程师去做，甚至有的时候就只做了个Demo，然后就交给业务工程师了，业务工程师变成给他打工的了。但是一个懂得这个道理的架构师，设计出来的东西会非常好用，业务方只需要扔很少的参数然后拿结果就好了，这样的架构才叫好的架构。

举一个保存图片到本地的例子，一种做法是提供这样的接口：
```
- (NSString *)saveImageWithData:(NSData *)imageData
```
另一种是
```
- (NSString *)saveImage:(UIImage *)image
```
后者更好，原因自己想。

**你的态度越谦卑，就越能设计出好的架构，这是我设计心法里的最后一条，也是最重要的一条**。即使你现在技术实力不是业界大牛级别的，但只要保持这个心态去做架构，去做设计，就已经是合格的架构师了，要成为业界大牛也会非常快。