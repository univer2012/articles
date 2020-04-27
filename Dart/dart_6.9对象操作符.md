## 对象操作符

1. 条件成员访问：`?.`
2. 类型转换： `as`
3. 是否指定类型：`is`、`is!`
4. 级联操作： `..`

```dart
void main() {
  Person person;
  person.work();
}

class Person {
  String name;
  int age;

  void work() {
    print("Work...");
  }
}
```

运行上面代码，会报错：`NoSuchMethodError: The method 'work' was called on null.`，此时如果把`person.work();`改为`person?.work();`，则运行正常，但是也不会打印，也就是没有执行`work()`方法。

如果实例化`person`，那么就会打印，即执行`work()`方法。
```dart
void main() {
  Person person = new Person();
  person?.work();//output: Work...
}
```

如果是用下面的方式实例化`person`，
```dart
void main() {
  var person;
  person = "";
	person = new Person();

  (person as Person).work();  //output: Work...
}
```
运行可以正常打印`Work...`。但是如果注释掉`person = new Person();`:

```dart
void main() {
  var person;
  person = "";
//  person = new Person();

  (person as Person).work();  //output: Work...
}
```
则会打印报错：
```
Unhandled exception:
type 'String' is not a subtype of type 'Person' in type cast
```


```dart
  var person = new Person();
  //person.name = "Tom";
  //person.age = 20;
  //person.work();
  //与下面的级联操作等价：
  person..name = "Tome"
        ..age = 20
        ..work();
```
级联操作可以写成同一行：
```dart
person..name = "Tome"..age = 20 ..work();
```
也可以实例化后直接写级联操作：
```dart
new Person()..name = "Tome"..age = 20 ..work();
//output: Work...Tome, 20
```