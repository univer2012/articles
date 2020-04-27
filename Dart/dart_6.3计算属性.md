## 计算属性

1. 顾名思义，计算属性的值是通过计算而来，本身不存储值
2. 计算属性赋值，其实是通过计算转换到其它实例变量

```dart

/*
void main() {

  var rect = new Rectangle();
  rect.height = 20;
  rect.width = 10;

  print(rect.area()); //output: 200

}
class Rectangle {
  num width, height;
  num area() {
    return width * height;
  }
}
*/

/// 上面的area 改为计算属性：

void main() {

  var rect = new Rectangle();
  rect.height = 20;
  rect.width = 10;

  print(rect.area); //output: 200

  rect.area = 300;
  print(rect.width);  // output: 15.0

}
class Rectangle {
  num width, height;

  //计算属性 写法1
//  num get area {
//    return width * height;
//  }
  //计算属性 写法2
  num get area =>  width * height;
      set area(value) {
        width = value / 20;
      }
}
```