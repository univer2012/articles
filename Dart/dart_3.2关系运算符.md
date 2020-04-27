## 关系运算符

1. 运算符：`==`、`!=`、 `>`、`<`、`>=`、`<=`
2. 判断内容是否相同使用`==`

```dart
void main() {
  int a = 5;
  int b = 3;
  
  print(a == b);  // false
  print(a != b);  // true

  print(a > b); // true
  print(a < b); // false
  print(a >= b); // true
  print(a <= b); // false


  String strA = "123";
  String strB = "321";
  print(strA == strB);  // false

}
```