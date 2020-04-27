## 列表List


### List（数组）创建
1. 创建List： `var list = [1,2,3];`
2. 创建不可变的List：`var list = const [1,2,3];`
3. 构造创建： `var list = new List();`


#### 常用操作
1. []、length
2. add()、insert()
3. remove()、clear()
4. indexOf()、lastIndexOf()
5. sort()、sublist()
6. shuffle()、asMap()、forEach()

```dart
void main() {
  var list1 = [1, 2, 3, "Dart", true];
  print(list1); // [1, 2, 3, Dart, true]
  print(list1[2]); // 3
  list1[1] = "Hello";
  print(list1); // [1, Hello, 3, Dart, true]

  var list2 = const [1, 2, 3];
  //list2[0] = 5; // Unsupported operation: Cannot modify an unmodifiable list

  var list3 = new List();
  print(list3); // []



  var list = ["Hello", "dart"];
  print(list.length);
  list.add("New");
  print(list);  // [Hello, dart, New]

  list.insert(1,  "Java");
  print(list);  // [Hello, Java, dart, New]

  list.remove("Java");
  print(list);  // [Hello, dart, New]

//  list.clear();
//  print(list);  // []

  print(list.indexOf("dart1")); // -1

  list.sort();
  print(list);  // [Hello, New, dart]

  print(list.sublist(1)); // [New, dart]

  list.forEach(print);
  /*
  * Hello
  * New
  * dart
  * */
}
```