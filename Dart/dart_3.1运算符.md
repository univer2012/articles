## 运算符

1. 加减乘除：`+`、`-`、`*`、`/`、`~/`、`%`
2. 递增递减：`++var`、`var++`、`--var`、`var--`


```dart
void main() {
  int a = 10;
  int b = 2;

  print(a + b); // 12
  print(a - b); // 8
  print(a * b); // 20
  print(a / b); // 5.0
  print(a ~/ b); // 5 //求商
  print(a % b); // 0  //求余

  print(a++); // output: 10
  print(++a); // output: 12

  print(a--); // output: 12
  print(--a); // output: 10
}
```