## 默认参数

1. 使用 = 在可选参数指定默认值
2. 默认值只能是编译时常量

```dart
void main() {
  printPerson("张三");
  printPerson("李四",age: 20);
  printPerson("李四",age: 20, gender: "Male");

  //output:
  //name=张三,age=30, gender= Female
  //name=李四,age=20, gender= Female
  //name=李四,age=20, gender= Male
}

printPerson(String name, {int age = 30, String gender = "Female"}) {
  print("name=$name,age=$age, gender= $gender");
}

```