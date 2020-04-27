## switch ... case 语句

1. 比较类型：`num`, `String`, 编译期常量, 对象, 枚举
2. 非空`case`必须有一个`break`
3. `default`处理默认情况
4. `continue`跳转标签

```dart
void  main() {
  String language = "Java";
  switch (language) {
    case "Dart":
      print("Dart is my favrite");
      break;
    case "Java":
      print("Java is my favrite");
      break;
    case "Python":
      print("Python is my favrite");
      break;
    default:
      print("None");
  }


  // ======= `continue`跳转标签
  switch (language) {
    Test:
    case "Dart":
      print("Dart is my favrite");
      break;
    case "Java":
      print("Java is my favrite");
      continue Test;
//      break;
    case "Python":
      print("Python is my favrite");
      break;
    default:
      print("None");
  }
  /*output:
  Java is my favrite
  Dart is my favrite   */
}
```

