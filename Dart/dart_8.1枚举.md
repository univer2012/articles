## 枚举

1. 枚举是一种有穷序列集的数据类型
2. 使用关键字 `enum`定义一个枚举
3. 常用于代替常量，控制语句等

### Dart枚举特性
1. index从0开始，依次累加
2. 不能指定原始值
3. 不能添加方法

```dart
//const spring = 0;
//const summer = 1;
//const autumn = 2;
//const winter = 3;

void main() {
  var currentSeason = Season.spring;

  print(currentSeason.index);

  switch (currentSeason) {
    case Season.spring:
      print("1-3月");
      break;
    case Season.summer:
      print("4-6月");
      break;
    case Season.autumn:
      print("7-9月");
      break;
    case Season.winter:
      print("10-12月");
      break;
  }
}

enum Season {
  spring,
  summer,
  autumn,
  winter,
}
```