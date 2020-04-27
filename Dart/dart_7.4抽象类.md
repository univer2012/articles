## 抽象类

1. 抽象类使用  `abstract` 表示，不能直接被实例化
2. 抽象方法不用 `abstract` 修饰，无实现
3. 抽象类可以没有抽象方法
4. 有抽象方法的类一定得声明为抽象类

```dart
void main() {
  //抽象类不能实例化。下面代码会报错： Abstract classes can't be created with a 'new' expression.
  //var person = new Person();

  var person = new Student();
  person.run(); // output: run ...

}

abstract class Person {
  void run();
}

class Student extends Person {
  @override
  void run() {
    print("run ...");
  }
}
```