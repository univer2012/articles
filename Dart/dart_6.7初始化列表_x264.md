## 初始化列表_x264

1. 初始化列表会在构造方法体执行之前执行
2. 使用逗号分隔初始化表达式
3. 初始化列表常用于设置 `final` 变量的值

```dart
void main() {
  var person = new Person("Tom", 20, "Male");
}

class Person {
  String name;
  int age;

  final String gender;

  //构造方法 写法4 当有final的属性要在构造方法中初始化时，需要使用下面的构造方法
  Person(this.name, this.age, this.gender);

  Person.withMap(Map map): name = map["name"], gender = map["gender"] {
    age = map["age"];
  }

  void work() {
    print("Work...");
  }
}
```