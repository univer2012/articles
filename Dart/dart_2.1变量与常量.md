## 变量与常量

#### 变量
1. 使用`var`声明变量，可 富余不同类型的值
2. 未初始化时，默认值为`null`
3. 使用`final`声明一个只能赋值一次的变量

```dart
	var a;
  print(a); // null

  a = 10;
  print(a); // 10

  a = "Hello Dart";
  print(a); //Hello Dart

  var b = 20;
  print(b); // 20 

  final c = 30;
  //c = 50; //'c', a final variable, can only be set once.
```

#### 常量
1. 使用const声明常量
2. 使用const声明的必须使编译期常量

```dart
const d = 20;
  // d = 50; //Constan variables can't be assigned a value.
```



