来自：

1. [Objective-C学习备忘录：Clang编译器编译运行Objective－C代码](https://www.cnblogs.com/wzk89/p/4650637.html)
2. [iOS 终端使用Clang编译 重写观察Runtime源码](http://blog.csdn.net/zy_flyway/article/details/72846019)

# 一、在终端使用Clang命令 编译（相比于Xcode运行，可以单独的编译文件并运行）

我们都知道可以通过Apple公司的Xcode工具来学习Objective-C编程语言，但是能不能脱离XCode这个IDE进行Objective-C学习呢？当然是可以的。
首先作为计算机科班出身的程序员都应该知道任何一门编程语言都离不开编译器，OC也不例外，我们可以通过度娘搜索发现，XCode的默认编译器是clang，那么问题来了，我能不能通过clang命令直接编译并运行一段OC代码呢？当然是可以的。

> 注意：关于XCode编译器详细介绍可以参考该文章：[编译器](http://objccn.io/issue-6-2/)

下面将叙述一下如何通过Mac OS中文本编辑器创建一个Hello Word的程序，并通过clang命令编译运行。

#### 1、打开“文本编辑”工具，输入以下代码，并保存为纯文本格式，文件名命名为`helloword.m`。

路径随意放：

```objc
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[])
{
    @autoreleasepool
    {
        NSLog(@"Hello, OC!");
    }

    return 0;
}
```



#### 2、接下来可以利用“终端”将helloword.m文件编译成可执行文件。

具体步骤：

1. 打开“终端”
2. 通过cd命令进入helloword.m文件所在目录
3. 使用clang命令对helloword.m文件进行编译，最后生成helloword可执行文件。

如下图所示。
[![img](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/171045579858583.png)](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/171045579858583.png)

```
clang -fobjc-arc -framework Foundation HelloWord.m -o HelloWord
```



有几个地方需要注意一下：

- `$`符号是终端命令提示符，不是需要输入的内容；
- `-fobjc-arc`表示编译器需要支持ARC特性；
- `-framework Foundation`表示引用Foundation框架；
- `HelloWord.m`为需要进行编译的源代码文件；
- `-o HelloWord`表示输出的可执行文件的文件名；

#### 3、生成可执行文件后，就可以在终端中执行该文件了

输入的命令如下：

```
$ ./HelloWord
```



执行结果如下图：
[![img](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/171058297826638.png)](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/171058297826638.png)

另外也可以**直接双击运行刚才生成的HelloWord可执行文件，运行结果和上面运行结果一样**。
[![img](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/171103378916108.png)](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/171103378916108.png)

至此通过几个简单的clang命令，就可以编译运行一段简单的Objective-C代码了。

# 二、Clang重写m文件为cpp文件  （重点说下，在学习Runtime时候很有用，可以逆向观察学习）

1. 进入文件目录，找到你要重写的文件
2. `clang -rewrite-objc xxxx.m`
3. 然后你目录下就会从写一个cpp文件，内容比较多你可以搜索关键方法对照查看。cpp为runtime代码，学习runtime感觉非常实。

下面是操作图：

1. 进入操作目录，执行clang命令：
   [![img](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/20170602171723371.png)](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/20170602171723371.png)
2. 生成结果cpp：
   [![img](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/20170602171708855.png)](https://univer2012.github.io/2017/12/14/41Clang编译器编译运行Objective-C代码/20170602171708855.png)



​	  