## latex等号和不等号垂直左对齐

来自：[LaTeX_fleqn参数时，多行公式对齐居中的同时选择性的加编号](https://www.cnblogs.com/mashiqi/p/5979446.html)

---

有些时候，我们的latex文档是这样开头的

```
\documentclass[fleqn,leqno]{article}
```

其中 leqno 参数的作用是使得公式的编号显示在左边。当参数 fleqn 存在时，公式往往不能正确的居中。这时，我们就需要特别小心的使用equation、align、flalign、alignat等命令。下面详细讲讲。

### 1、多行公式在一个地方对齐
如何居中多行的公式呢？我试过很多种方法后，觉得下面这个最好用：

```
\begin{flalign*}
    % In this way (this arrange of &), the equation will in the center and align at the third &. If use this method for 'split', equations will not be centered
    % However, 'flalign' will give each line a separate number. It cannot number the whole equations in one number.
    && a & = b & \\
    && a & = b & \\
    && a & = b & \\
    && a & = b &
\end{flalign*}
```

其中的多个&符号是专门安排好的。只用往第二个和第三个&符号后面添加代码就行了，它们就能对齐  并且所有公式作为一个整体进行居中。但这里有个问题，就是编号的问题。这里没有公式的编号。如果把flalign后面的*号去掉的话，那么就有编号了，但是是每一行公式一个编号，有些情况下这不太合适。因此，为了编号，我们要对上述代码做如下更改：

```
\begin{flalign}
    % In this way (this arrange of &), the equation will in the center and align at the third &. If use this method for 'split', equations will not be centered
    % However, 'flalign' will give each line a separate number. It cannot number the whole equations in one number.
    && a & = b & \nonumber\\
    && a & = b & \\
    && a & = b & \nonumber\\
    && a & = b & \label{eq2}\\
    && a & = b & \nonumber
\end{flalign}
Now we refer equation (\ref{eq2}).
```

现在去掉了flalign后面的*号，那么给不想编号的行的结尾加上 \nonumber，给想加编号的行的后面 不添加任何东西 或者加上 \label{...} 。当 不添加任何东西 时，此行是有编号的，但这个编号将不能被引用到；当加上了 \label{...} 时，可以通过 \ref{...} 来引用这个编号。

下图就是上面两段代码的输出结果：

![图片.png](https://upload-images.jianshu.io/upload_images/843214-a3a871ccd5f22343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上就基本解决了：多行公式、居中、对齐、加编号、引用编号这些问题。唯一的小缺点就是编号并不在整个公式的中间。不过这个小缺点可以容忍。

### 2、多行公式在多个地方对齐

为了达到多行公式在多个地方对齐的目的，我们必须取消最开始的 fleqn 命令。

有了上面使用 flalign 的经验之后，我们可以使用 \begin{alignat*}{·} 进行多行公式在多个地方对齐的操作。“alignat”后面的那个括号里的数字是每行的“&”符号的个数加1后除以2。举例如下：

```
\begin{alignat*}{2}
    a & = b &\quad c & = d \\
    a & = b &      c & = d
\end{alignat*}
```

注意：为了让各个公式之间能有一个间隔，要在分隔各个公式的&符号之后加一个 \quad 。其实 flalign 也可以通过上面的方法实现多个地方对齐，但是当每一行的公式不多时，为了故意填满一整行，公式间的间距会很大。而 alignat 命令不会可以去填满一整行，而是会使得各个公式间尽量靠拢，所以各个公式间的水平距离会看起来更舒服一些。 alignat 和 flalign 之间的区别举例如下：

```
\begin{alignat*}{2}
    a & = b &\quad c & = d \\
    a & = b &      c & = d
\end{alignat*}

\begin{flalign*}
    && a & = b & c & = d && \\
    && a & = b & c & = d &&
\end{flalign*}
```

![图片.png](https://upload-images.jianshu.io/upload_images/843214-61d518a2f2f6e82f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、 其他命令加编号

另外，使用

```
$$...$$
```

命令也是可以加编号的，方法是使用 \eqno{...} 命令。举例如下：

```
$$a=b \eqno{(1)}$$
```

但这时的编号 (1) 要手动输入，而且不能被引用。

---

【完】