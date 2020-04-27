## 接口

1. 类和接口是统一的，**类就是接口**
2. 每个类都隐式的定义了一个包含所有实例成员的接口
3. 如果是复用已有类的实现，使用继承（`extends`）
4. 如果只是使用已有类的外在行为，使用接口（`implements`）


```dart
void main() {
  var student = new Student();
}

class Person {
  String name;

  int get age => 18;

  void run() {
    print("Person run...");
  }
}

class Student implements Person {
  @override
  String name;

  @override
  int get age => 15;

  @override
  void run() {
  }

}
```




 ====== 抽象类
```dart
void main() {
  var student = new Student();
  student.run();
}

abstract class Person {
  void run();
}

class Student implements Person {

  @override
  void run() {
    print("Student run...");
  }

}
```