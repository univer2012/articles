## Mixins

1. Mixins 类似于多继承，是在多类继承中重用一个类代码的方式
2. 作为Mixin的类不能有显示声明构造方法
3. 作为Mixin的类只能继承自Object
4. 使用关键字 `with` 连接一个或多个mixin

```dart

/*
void main() {
  var d = new D();
  d.a();
}

class A {
  void a() {
    print("A.a()...");
  }
}

class B {
  void a() {
    print("B.a()...");
  }
  void b() {
    print("B.b()...");
  }
}

class Test {}

//报错: The class 'C' can't be used as a mixin because it extends a class other than Object.
//class C extends Test {

class C {

  // 报错：The class 'C' can't be used as a mixin because it declares a constructor.
  //C() { }


  void a() {
    print("C.a()...");
  }
  void b() {
    print("C.b()...");
  }
  void c() {
    print("C.c()...");
  }
}

//class D extends A with B,C {
class D extends A with C,B {

}

*/







abstract class Engine {
  void work();
}

class OilEngine implements Engine {
  @override
  void work() {
    print("Work with oil...");
  }
}

class ElectricEngine implements Engine {
  @override
  void work() {
    print("Work with Electric ...");
  }
}

class Tyre {
  String name;
  void run() {}
}

class Car = Tyre with ElectricEngine;

//class Car extends Type with ElectricEngine {
//
//}

class Bus = Tyre with OilEngine;
```