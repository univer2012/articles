来自：[Python3 标准库概览](https://www.runoob.com/python3/python3-stdlib.html)



## 操作系统接口

os模块提供了不少与操作系统相关联的函数。

```python
>>> import os
>>> os.getcwd()
'/Users/yuanping/Documents'
>>> os.chdir('/server/accesslogs')
>>> os.system('mkdir today')
0
>>> 
```

建议使用 "import os" 风格而非 "from os import *"。这样可以保证随操作系统不同而有所变化的 os.open() 不会覆盖内置函数 open()。 

在使用 os 这样的大型模块时内置的 dir() 和 help() 函数非常有用:

```python
>>> import os
>>> dir(os)
<returns a list of all module functions>
>>> help(os)
<returns an extensive manual page created from the module's docstrings>
>>> 
```



针对日常的文件和目录管理任务，`:mod:shutil` 模块提供了一个易于使用的高级接口:

```python
>>> import shutil
>>> shutil.copyfile('data.db', 'archive.db')
>>> shutil.move('/build/executables', 'installdir')
```

//... ...

待续... ...