## 对象call方法

1. 如果类实现了`call()` 方法，则该类的对象可以作为方法使用。

```dart
void main() {
  var person = new Person();
  person.name = "Tom";
  person.age = 20;

//  person.work();  // output: Name is Tom, Age is 20

  // 类的对象作为方法使用：
//  person(); // output: Name is Tom, Age is 20

  print(person("Test", 30));  // output: Name is Test, Age is 30
}


class Person {
  String name;
  int age;

//  void work() {
//    print("Name is $name, Age is $age");
//  }

  //写法1
//  void call() {
//    print("Name is $name, Age is $age");
//  }

  //写法2
  String call(String name, int age) {
    return "Name is $name, Age is $age";
  }
}
```
