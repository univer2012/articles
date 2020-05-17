来自：[swift中Class和Struct的区别](https://www.jianshu.com/p/170895a5f4fb)



类（class）和结构体（struct），不止在iOS开发中，在其他各种语言开发中都属于老生常谈的问题了，再看class和struct之前，我们先看一下引用类型和结构体的概念。

引用类型：将一个对象赋值给另一个对象时，系统不会对此对象进行拷贝，而会将指向这个对象的指针赋值给另一个对象，当修改其中一个对象的值时，另一个对象的值会随之改变。

值类型：将一个对象赋值给另一个对象时，会对此对象进行拷贝，复制出一份副本给另一个对象，在修改其中一个对象的值时，不影响另外一个对象。

------

在swift中，类属于引用类型，结构体属于值类型。相对于其他语言来说，swift的结构体功能更加强大，它除了支持在结构体中声明基础变量之外，它还支持在结构体中声明方法，这相对于其他语言来说，是swift的一个特性之一。此外，除了引用类型和值类型的区别之外，他们还有其他的不同点。



下面总结一下在swift中类和结构体的**不同点**：

1. <font color=#FF0000>类属于引用类型，结构体属于值类型；</font>

2. <font color=#FF0000>类允许被继承，结构体不允许被继承；</font>

3. <font color=#FF0000>类中的每一个成员变量都必须被初始化，否则编译器会报错；而结构体不需要，编译器会自动帮我们生成 `init` 函数，给变量赋一个默认值。</font>

------

下面我们通过代码来看一下，在swift中类（引用类型）和结构体（值类型）具体表现出有何不同：

首先我们声明一个ClassTest类，其中声明了两个变量number和name

```swift
fileprivate class ClassTest {
    
  var number: Int = 0
    
  var name: String = "test"
}
```



声明一个aTest，输出aTest的值，之后将aTest赋值给bTest，并改变bTest的值，相对应的，aTest的值也会被改变。

```swift
@objc func sec1demo1() {
    
    print("class ==========")
    let aTest = ClassTest()
    
    print("number is \(aTest.number)")
    print("name is \(aTest.name)")
    /*output:
     number is 0
     name is test
     */

    let bTest = aTest
    bTest.number = 5
    bTest.name = "testAAA"
    //改变了bTest中的值。由于类是引用类型，相对于的aTest中的值也会被改变
    
    print("number is \(aTest.number)")
    print("number is \(aTest.name)")
    /*output:
     number is 5
     number is testAAA
     */    
}
```



如果我们将类换成结构体，那会是什么情况呢？很明显，根据值类型的特性，当我们改变b的值时，不会影响到a的值。

```swift
fileprivate struct StructTest {
  var number:Int = 1
  var name:String = "struct"
}
```



声明aStruct，打印其中的值为默认值，声明bStruct，并且将aStruct赋值给bStruct，改变b的值并不会影响a中的值。

```swift
@objc func sec1demo1() {
    print("struct ==========")
    print("改变前：")
    let aStruct = StructTest()
    print(aStruct.number)
    print(aStruct.name)
    /*output:
     改变前：
     1
     struct
     */
    
    var bStruct = aStruct
    bStruct.number = 10
    bStruct.name = "myTestStruct"
    
    print("改变后：bStruct")
    print(bStruct.number)
    print(bStruct.name)
    /*output:
     改变后：bStruct
     10
     myTestStruct
     */
    
    print("改变后：aStruct")
    print(aStruct.number)
    print(aStruct.name)
    /*output:
     改变后：aStruct
     1
     struct
     */
}
```



以上关于类和结构体的异同就讲到这里，感兴趣的同学可以在深究，在swift中，许多新特性相对于其他语言来说更有意思，比如结构体中也可以声明方法，这意味着我们在使用结构体时，可以更加灵活。

