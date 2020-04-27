## 静态成员

1. 使用 `static` 关键字来实现类级别的变量和函数
2. 静态成员不能访问非静态成员，非静态成员可以访问静态成员
3. 类中的常量需要使用 `static const` 声明

```dart
void main() {
  var page = new Page();
  page.scrollDown();  //output: ScrollDown...
}

class Page {
  static int currentPage = 1;

  //下滑
  void scrollDown() {
    currentPage = 1;
    print("ScrollDown...");
  }

  //上滑
 void scrollUp() {
    currentPage ++;
    print("scrollUp...");
 }
}
```

把`scrollDown`用`static`修饰，则`currentPage`会报错：`Instance members can't be accessed from a static method.`，因为`currentPage` 不是`static`的。

如果把`currentPage` 用`static`修饰，那么会发现`page.scrollDown(); `报错：`Static method 'scrolldown' can't be accessed through an instance.`，此时只能用`Page.scrollDown();`调用。


```dart
void main() {
  var page = new Page();
  //page.scrollDown();  //output: ScrollDown...

  Page.scrollDown();
}

class Page {

  //const int maxPage = 10;//报错：Only static fields can be declared as const.
  static const int maxPage = 10;
  static int currentPage = 1;

  //下滑
  static void scrollDown() {
  //void scrollDown() {

    //如果方法被static修饰，会报错：Instance members can't be accessed from a static method.
    //因为currentPage 不是static的。
    currentPage = 1;
    print("ScrollDown...");
  }

  //上滑
 void scrollUp() {
    currentPage ++;
    print("scrollUp...");
 }
}
```
