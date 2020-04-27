来自：[swift  map flatMap  Filter Reduce](https://www.jianshu.com/p/1dcbc0a53ea2)

### 1、Map(映射)

map函数 类似for循环做的一些事 只是以闭包的形式表达出来 map闭包函数的功能就是对数组中的每一项进行遍历，然后通过映射规则对数组中的每一项进行处理，最终的返回结果是处理后的数组（以一个*新的数组*形式出现）。当然，*原来数组中的元素值是保持不变的*，这就是map闭包函数的用法与功能。

```swift
var arr = [1,2,3,4,5,6]
var temparr = arr.map { (item: Int) -> Int in
    return item + 1
}
print(temparr)
///打印：
///[2, 3, 4, 5, 6, 7]
```



2、Filter (过滤器) 用法和map形式一样就是返回值是bool 函数体加过滤条件 

```swift
var arr = [1,2,3,4,5,6]
let temparr = arr.filter { (item: Int) -> Bool in
    return item >= 3
}
print(temparr)
///打印：
///[3, 4, 5, 6]
```



3、Reduce 求和 

```swift
var arr = [1,2,3,4,5,6]
let temparr = arr.reduce(0) { (item1: Int, item2:Int) -> Int in
    return item1 + item2
}
print(temparr)
///打印：
///21
```

4、sorted (排序)

```swift
let arrs = [2,4,5,1,7,1,8,9]
//官方函数体写法
let temparr = arrs.sorted{ (num1:Int, num2:Int) -> Bool in
    return num1 > num2
}

//sort函数包含函数体
let temparr1 = arrs.sorted { (num1:Int, num2:Int) -> Bool in
    return num1 > num2
}

//省略形参名和返回值和in，用 $0 和 $1 代替形参
let temparr2 = arrs.sorted(by: {$1 < $0})

//swift中运算符号也是一种特殊的函数
let temparr3 = arrs.sorted(by: < )

print(" temparr=\(temparr)\n temparr1=\(temparr1)\n temparr2=\(temparr2)\n temparr3=\(temparr3)")
/*打印：
 temparr=[9, 8, 7, 5, 4, 2, 1, 1]
 temparr1=[9, 8, 7, 5, 4, 2, 1, 1]
 temparr2=[9, 8, 7, 5, 4, 2, 1, 1]
 temparr3=[1, 1, 2, 4, 5, 7, 8, 9]
 */
```



总结 `map` `filter` `reduce` 函数 可将后面函数体 看做闭包 省略其形参和返回体和in 只写实现如下

```swift
let arrs = [2,4,5,1,7,1,8,9]

let temparr2 = arrs.map{$0 + 2}

let temparr = arrs.filter{$0 != 1}

let temp3 = arrs.reduce(1) { $0 * $1 }

print(temp3)
///打印：
///20160
```



#### map 、flatMap、 compactMap 区别

1. `map` 映射的是本身是什么类型新的数组就是什么类型 不会解包

2. `flatMap` 会把`optional`类型解包，并且会过滤掉 nil， 还可把多维数组展开、合并成一个数组（遇到nil类型除外 eg [[1,3],[2,5],**nil**]）。 ps： swift 4.1之后`flatmap`会报警告（解决方法下篇文章给出）


3. `compactMap`： swift4.1以后出现的新特性 为了解决多维数组nil问题 功能类似`flatmap` 

4. 遇到` [[1,3],[2,5], nil]` 这种数组 目前方法是先用`compactMap`进行去除nil 再进行`flatmap`展开。

