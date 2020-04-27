## while语句

1. while 循环
2. do ... while 循环


```dart
void main() {

  int count = 0;
   while (count < 5) {
     print(count ++);
   }

   /*output:
   * 0
   * 1
   * 2
   * 3
   * 4*/
   print("------------")
;
   do {
     print(count --);
   } while (count > 0 && count < 5);
   /*output:
   * 5
   * 4
   * 3
   * 2
   * 1*/
}
```