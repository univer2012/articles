
## 控制流语句
### 条件语句

1. if 语句
2. if ... else if 语句
3. if ... else if ... else 语句

```dart
void main() {
  int score = 100;
  if (score >= 90) {
    if (score == 100) {
      print("完美");
    } else {
      print("优秀");
    }
  } else if (score > 60) {
    print("良好");
  } else if (score == 60) {
    print("及格");
  } else {
    print("不及格");
  }

  //output: 完美
}
```