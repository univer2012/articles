来自：[Python3 实例](https://www.runoob.com/python3/python3-examples.html)



## 2.Python 数字求和

以下实例为通过用户输入两个数字，并计算两个数字之和：

```python
#用户输入数字
num1 = input('输入第一个数字：')
num2 = input('输入第二个数字：')
# 求和
sum = float(num1) + float(num2)
# 显示计算结果
print('数字{0}和{1}相加结果为：{2}'.format(num1, num2, sum))
```

执行以上代码输出结果为：

```
输入第一个数字：3.14
输入第二个数字：6.28
数字3.14和6.28相加结果为：9.42
```

 在该实例中，我们通过用户输入两个数字来求和。使用了内置函数 input() 来获取用户的输入，input() 返回一个字符串，所以我们需要使用 float() 方法将字符串转换为数字。

两数字运算，求和我们使用了加号 (+)运算符，除此外，还有 减号 (-), 乘号 (*), 除号 (/), 地板除 (//) 或 取余 (%)。更多数字运算可以查看我们的[Python 数字运算](https://www.runoob.com/python3/python3-number.html)。 

我们还可以将以上运算，合并为一行代码：

```python
print('两数之和为 %.1f ' %(float(input('输入第一个数字：')) + float(input('输入第二个数字：'))) )
```

执行以上代码输出结果为：

```
(venv) yuanpingdeAir:venv yuanping$ python test.py
输入第一个数字：1.5
输入第二个数字：2.5
两数之和为 4.0 
```

## 3.Python 平方根

平方根，又叫二次方根，表示为〔√￣〕，如：数学语言为：√￣16=4。语言描述为：根号下16=4。

以下实例为通过用户输入一个数字，并计算这个数字的平方根：

```python
num = float(input('请输入一个数字：'))
num_sqrt = num ** 0.5
print(' %0.3f 的平方根为 %0.3f' %(num,num_sqrt))
```

执行以上代码输出结果为：

```
请输入一个数字：4
 4.000 的平方根为 2.000
```

 在该实例中，我们通过用户输入一个数字，并使用指数运算符 ** 来计算该数的平方根。 

 该程序只适用于正数。负数和复数可以使用以下的方式：

```python
# 计算实数和复数平方根
# 导入复数数学模块

import cmath
num = int(input('请输入一个数字: '))
num_sqrt = cmath.sqrt(num)
print('{0} 的平方根为 {1:0.3f} + {2:0.3f}j'.format(num, num_sqrt.real, num_sqrt.imag))
```

执行以上代码输出结果为：

```
请输入一个数字: -8
-8 的平方根为 0.000 + 2.828j
```

该实例中，我们使用了  `cmath` (complex math) 模块的 `sqrt()` 方法。 





## 4.Python 二次方程

以下实例为通过用户输入数字，并计算二次方程：

```python
# 二次方程式 ax**2 + bx + c = 0
# a、b、c 用户提供，为实数，a ≠ 0

# 导入cmath（复杂数学运算）模块

import  cmath
a  = float(input('输入 a: '))
b  = float(input('输入 b: '))
c  = float(input('输入 c: '))

#计算
d = (b**2) - (4*a*c)
#两种求解方式
sol1 = (-b-cmath.sqrt(d))/(2*a)
sol2 = (-b+cmath.sqrt(d))/(2*a)

print('结果为 {0} 和 {1}'.format(sol1, sol2))
```

执行以上代码输出结果为：

```
输入 a: 1
输入 b: 3
输入 c: 2
结果为 (-2+0j) 和 (-1+0j)
```

该实例中，我们使用了  `cmath` (complex math) 模块的 `sqrt()` 方法 来计算平方根。 



## 5.Python 计算三角形的面积

以下实例为通过用户输入三角形三边长度，并计算三角形的面积：

```python
a = float(input(('输入三角形第一边长: ')))
b = float(input(('输入三角形第二边长: ')))
c = float(input(('输入三角形第三边长: ')))
# 计算半周长
s = (a + b + c) / 2
# 计算面积
area = (s*(s-a)*(s-b)*(s-c)) ** 0.5
print('三角形面积为 %0.2f' %area)
```

执行以上代码输出结果为：

```
输入三角形第一边长: 4
输入三角形第二边长: 5
输入三角形第三边长: 6
三角形面积为 9.92
```



## 6.Python 计算圆的面积

圆的面积公式为 ：

![img](https://www.runoob.com/wp-content/uploads/2019/05/pirr.png)

公式中 r 为圆的半径。

```python
# 定义一个方法来计算圆的面积
def findArea(r):
    PI = 3.142
    return  PI * (r*r)

# 调用方法
print('圆的面积为 %.6f' %findArea(5))
```

以上实例输出结果为：

```
圆的面积为 78.550000
```



## 7.Python 随机数生成

以下实例演示了如何生成一个随机数：

```python
# 生成 0 ~ 9 之间的随机数

# 导入 random(随机数) 模块
import  random
print(random.randint(0, 9))
```

执行以上代码输出结果为：

```
8
```



以上实例我们使用了 random 模块的 randint() 函数来生成随机数，你每次执行后都返回不同的数字（0 到 9），该函数的语法为： 

```python
random.randint(a,b)
```

函数返回数字 N ，N 为 a 到 b 之间的数字（a <= N <= b），包含 a 和 b。



## 8.Python 摄氏温度转华氏温度

以下实例演示了如何将**摄氏温度转华氏温度**：

```python
# 用户输入摄氏温度
# 接收用户输入
celsius = float(input('输入摄氏温度: '))
# 计算华氏温度
fahrenheit = (celsius * 1.8) + 32
print('%0.1f 摄氏温度转为华氏温度为 %0.1f' %(celsius, fahrenheit))
```

执行以上代码输出结果为：

```
输入摄氏温度: 28
28.0 摄氏温度转为华氏温度为 82.4
```



以上实例中，摄氏温度转华氏温度的公式为 celsius * 1.8 = fahrenheit - 32。所以得到以下式子： 

```python
celsius = (fahrenheit - 32) / 1.8
```



## 9.Python 交换变量

以下实例通过用户输入两个变量，并相互交换：

```python
# 用户输入
x = input('输入 x 值: ')
y = input('输入 y 值: ')
# 创建临时变量，并交换
temp = x
x = y
y = temp

print('交换后 x 的值为: {}'.format(x))
print('交换后 y 的值为: {}'.format(y))
```

执行以上代码输出结果为：

```
输入 x 值: 34
输入 y 值: 54
交换后 x 的值为: 54
交换后 y 的值为: 34
```

 以上实例中，我们创建了临时变量 temp ，并将 x 的值存储在 temp 变量中，接着将 y 值赋给 x，最后将 temp 赋值给 y 变量。 



### 9.1 不使用临时变量

我们也可以不创建临时变量，用一个非常优雅的方式来交换变量：

```python
x,y = y,x
```

所以以上实例就可以修改为：

```python
# 用户输入
x = input('输入 x 值: ')
y = input('输入 y 值: ')
# 不使用临时变量
x, y = y, x

print('交换后 x 的值为: {}'.format(x))
print('交换后 y 的值为: {}'.format(y))
```

执行以上代码输出结果为：

```
输入 x 值: 2
输入 y 值: 3
交换后 x 的值为: 3
交换后 y 的值为: 2
```



## 10Python if 语句

```python
# 用户输入数字
num = float(input('输入一个数字: '))
if num>0:
    print('正数')
elif num == 0:
    print('零')
else:
    print("负数")
```

执行以上代码输出结果为：

```
输入一个数字: 5
正数
```



我们也可以使用内嵌 if 语句来实现：

```python
num = float(input('输入一个数字: '))
if num>=0:
    if num == 0:
        print('零')
    else:
        print('正数')
else:
    print("负数")
```

执行以上代码输出结果为：

```
输入一个数字: 0
零
```



## 11.Python 判断字符串是否为数字

以下实例通过创建自定义函数 **is_number()** 方法来判断字符串是否为数字：

```

```

