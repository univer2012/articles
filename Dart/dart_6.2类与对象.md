## 类与对象
1. 使用关键字`class`声明一个类
2. 使用关键字`new`创建一个对象，`new`可省略
3. 所有对象都继承于`Object`类

#### 属性与方法
1. 属性默认会生成`getter`和`setter`方法
2. **使用final声明的属性只有`getter`方法**
3. 属性和方法通过 `.` 访问
4. 方法不能被重载

```dart
void main() {
  var person = new Person();
//  var person = Person();  //new可省略
  person.name = "Tom";
  person.age = 20;
  //person.address = "china";// 'address' can't be used as a setter because it is final.

  print(person.name); // output: Tom
  print(person.address);  //如果final的address没有初始化，获取address会崩溃
  person.work();  // output: Name is Tom, Age is 20,He is working...
}
class Person {
  String name;
  int age;
  final String address; //warnning: The final variable 'address' must be initialized.

  void work() {
    print("Name is $name, Age is $age,He is working...");
  }

// warnning: The name 'work'is already defined.
//void work(int a) {}
}
```

#### 类及成员可见性
1. Dart中的可见性以`library`（库）为单位
2. 默认情况下，每一个Dart文件就是一个库
3. 使用 `_` 表示库的私有性
4. 使用`import`导入库

如果把上面的实例代码中的`Person`类剪贴到新建的`Person.dart`文件中，
那么`main()`方法中Person就会出现`Undefined class 'Person'.`的警告。此时系统会提示你导入库文件：`import 'Person.dart';`。

如果把`Person.dart`文件中的`Person`类前面添加`_`：
```dart
class _Person {
  String name;
  int age;
  final String address; //warnning: The final variable 'address' must be initialized.

  void work() {
    print("Name is $name, Age is $age,He is working...");
  }

// warnning: The name 'work'is already defined.
//void work(int a) {}
}
```
即使已经导入了`import 'Person.dart';`，`main()`方法中`_Person`会出现`Undefined class '_Person'.`的警告。意思是`Person`类是一个私有类，别的库无法访问。

如果把`name`改为`_name`，`work`方法改为`_work`，那么外部做出如下操作：


* 设置`_name`的值 --->`The setter '_name' isn't defined for the class 'Person'.`
* 获取`_name`的值 ---> `The getter '_name' isn't defined for the class 'Person'.`
* 调用`_work`方法  ---> `The method '_work' isn't defined for the class 'Person'.`。


