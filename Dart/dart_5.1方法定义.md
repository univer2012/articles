## Function

#### 方法定义
```
返回类型 方法名 (参数1, 参数2,...) {
	方法体...
	return 返回值
}
```

#### 方法特性

1. 方法也是对象，并且有具体类型Function
2. 返回值类型、参数类型都可省略
3. 箭头语法：`=> expr` 是 `{return expr;}` 缩写。只适用于**一个表达式**
4. 方法都有返回值。如果没有指定，默认 `return null` 最后一句执行

```dart
void main(List args) {
  print(args);    // []

  print(getPerson("张三", 18)); //output: name = 张三, age = 18

//  printPerson("李四", 20);  //output: name = 李四, age = 20

  print(printPerson("李四", 20));
  //output:
  //name = 李四, age = 20
  //null

}
//写法1
//String getPerson (String name, int age) {
//  return "name = $name, age = $age";
//}
//写法2
//getPerson (name, age) => "name = $name, age = $age";
//写法3
int gender = 2;
getPerson (name, age) =>  gender == 1 ? "name = $name, age = $age" : "Test";

//写法1
//void printPerson(String name, int age) {
//  print("name = $name, age = $age");
//}
//写法2
printPerson(name, age) {
  print("name = $name, age = $age");
}
```