## 条件表达式

1. 三目运算符：`condition ? expr1 : expr2`
2. `??`运算符： `expr1 ?? expr2`

```dart
void main() {
  int gender = 1;
  String str = gender == 0 ? "Male=$gender" : "Female=$gender";
  print(str); //Male=0    Female=1


//  String a;
//  String b = "Java";
//  String c = a ?? b;
//  print(c); // Java

  String a = "Dart";
  String b = "Java";
  String c = a ?? b;  //??    a有值，就取a的值；否则取b的值
  print(c); // Dart
}
```