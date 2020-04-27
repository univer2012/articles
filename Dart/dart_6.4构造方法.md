## 构造方法

1. 如果没有自定义构造方法，则会有个默认构造方法
2. 如果存在自定义构造方法，则默认构造方法无效

```dart
void main() {
  var person = new Person();
  person.name = "Tom";
  person.age = 20;

}

class Person {
  String name;
  int age;

  final String gender = "Female";

  void work() {
    print("Work...");
  }
}
```

上面代码的`Person`类，如果添加了构造方法：
```dart
class Person {
  String name;
  int age;

  final String gender = "Female";

  Person(String name, int age) {
    this.name = name;
    this.age = age;
  }

  void work() {
    print("Work...");
  }
}
```

那么实例化时的`new Person();`会报错：`2 required argument(s) expected, but 0 found`，这时要在实例化时带上参数：`new Person("Tom", 20);`。

3. 构造方法不能重载

## 命名构造方法
1. 使用命名构造方法，可以实现多个构造方法
2. 使用 `类名.方法` 的形式实现

```dart
class Person {
  String name;
  int age;

  //final String gender;

  // ==== 类名构造方法 写法1
  Person.withName(String name) {
    this.name = name;
  }

  // ==== 类名构造方法 写法2
  Person.withAge(this.age);

  void work() {
    print("Work...");
  }
}
```