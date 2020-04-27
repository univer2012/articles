## 闭包

1. 闭包是一个方法（对象）
2. 闭包定义在其它方法内部
3. 闭包能够访问外部发发发内的局部变量，并持有其状态

```dart
void main() {
  var func = a();
  func(); //output: 0
  func(); //output: 1
  func(); //output: 2
  func(); //output: 3
}

a() {
  int count = 0;


//  printCount() {
//    print(count++);
//  }
//  return printCount;

  return () {
    print(count++);
  };
}
```