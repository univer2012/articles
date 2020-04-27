## 继承中的构造方法

1. 子类的构造方法默认会调用父类的无名无参构造方法
2. 如果父类没有无名无参构造方法，则需要显示调用父类构造方法
3. 在构造方法参数后使用  `:`  显示调用父类构造方法


### 1 子类的构造方法默认会调用父类的无名无参构造方法
```dart
void main() {
  var student = new Student();
}

class Person {
  String name;

  Person() {
    print("Person...");
  }
}

class Student extends Person {
  int age;
}
```
上面的代码会输出：`Person...`，因为子类实例化时，会调用父类的无名无参构造方法。

#### 2. 如果父类没有无名无参构造方法，则需要显示调用父类构造方法
```dart
class Person {
  String name;

  Person(this.name);

  Person.withName(this.name);
}

class Student extends Person {
  int age;
}
```
代码如上面所示，那么`Student`会报错：`The superclass 'Person' doesn't have a zero argument constructor.` 这时`Student`需要创建对应的构造方法：
```dart
class Student extends Person {
  int age;

  Student(String name) : super(name);
}
```

### 构造方法执行顺序
1. 父类的构造方法在子类构造方法体开始执行的位置调用
2. 如果有初始化列表，初始化列表会在父类构造方法之前执行


```dart
void main() {
  //子类初始化会调用父类的无名无参构造方法，所以打印：Person...
  var student = new Student("Tom", "Male");

  print(student.name);  // output: Tom
}

class Person {
  String name;

  Person(this.name);

  Person.withName(this.name);
}

class Student extends Person {
  int age;

  final String gender;

  //Student(String name) : super(name);
  //Student(String name) : super.withName(name); //或者调用父类的`withName`方法


  //下面构造方法会报错：super call must be last in an initializer list (see https://goo.gl/EY6hDP): 'super.withName(name)'.
  //Student(String name, String g) : super.withName(name), gender = g; //错误的写法

  Student(String name, String g) : gender = g, super.withName(name); //正确的写法
}
```
