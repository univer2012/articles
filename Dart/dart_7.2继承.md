## 继承

1. 使用关键字`extends`继承一个类
2. 子类会继承父类课件的属性和方法，不会继承构造方法
3. 子类能够复写父类的方法，getter和setter
4. 单继承，多态性

```dart
import 'person.dart';

void main() {

//  var student = new Student();
//  student.study();  // output: Student study...
//
//  student.name = "Tom";
//  student.age = 16;
//
//  print(student.isAdult);
//  student.run();  // output: Person run...



  Person person = new Student();
  person.name = "Tom";
  person.age = 18;


  if (person is Student) {
    person.study(); // output: Student study...
  }

  print(person);
  // 没有重写时：output: Instance of 'Student'
  //重写后：output: Name is Tom, Age is 18

}

class Student extends Person {
  void study() {
    print("Student study...");
  }

  @override
  bool get isAdult => age > 15;

  @override
  void run() {
    //super.run();
    print("Student run...");
  }

  @override
  String toString() {
    return "Name is $name, Age is $age";
  }
}

```

