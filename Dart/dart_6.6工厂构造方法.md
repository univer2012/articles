## 工厂构造方法

1. 工厂构造方法类似于设计模式中的工厂模式
2. 在构造方法前添加关键字 `factory` 实现一个工厂构造方法
3. 在工厂构造方法中可返回对象

```dart
void main() {

}

class Logger {
  final String name;

  static final Map<String, Logger> _cache = <String, Logger>{};

  factory Logger (String name) {
    if (_cache.containsKey(name)) {
      return _cache[name];
    } else {
      final logger = Logger._internal(name);
      _cache[name] = logger;
      return logger;
    }
  }
  //私有 构造方法
  Logger._internal(this.name);

  void log(String msg) {
    print(msg);
  }
}
```