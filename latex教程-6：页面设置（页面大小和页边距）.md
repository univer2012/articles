来自：[Latex教程: [6]页面设置(页面大小和页边距)](https://jingyan.baidu.com/article/ab0b563092c6b7c15afa7db8.html)

---



## 页面大小设置

默认的页面大小是A4纸，输入以下内容，如果不进行设置的话，Latex采用默认的页面大小，也即A4纸。

```latex
\documentclass{article}

\begin{document}
Latex

\end{document} 
```

如果想要在其他大小的纸张上进行文档编辑，例如A5的纸张，只需要在\documentclass[]{}的[]里面加入纸张大小的值a5paper即可。代码如下：

```latex
\documentclass[a5paper]{article}

\begin{document}
Latex
\end{document} 
```

注意，这个[]中不仅可以设置页面大小，也可以设置文档的字体。这在前面的一篇经验里面已经讲过，代码如下，大家可以自己试试。

```latex
\documentclass[12pt,a5paper]{article}

\begin{document}
Latex
\end{document} 
```

可用的页面设置以及尺寸如下。大家可以根据需求自行设置。

```
a4paper 				---> (297mm 	* 210mm)
a5paper 				---> (210mm 	* 148mm)
b5paper 				---> (250mm 	* 176mm)
letterpaper 		---> (11in 		* 8.5in)
legalpaper 			---> (14in 		* 8.5in)
executivepaper 	---> (10.5in 	* 7.25in)
```



## geometry宏包设置页边距

如果想要自由设置页面属性的话，可以使用geometry宏包。geometry宏包可以让我们自由设置上下左右的页边距。输入如下代码

```latex
\documentclass[12pt,a5paper]{article}

\usepackage{geometry}

\geometry{left=2.0cm,right=2.0cm,top=2.5cm,bottom=2.5cm}

\begin{document}
Latex
\end{document} 
```

其中left, right, top, bottom的值分别代表左右上下的页边距。作为演示，我们将左页边距分别设置成1.0和3.5。

```latex
\documentclass[12pt,a5paper]{article}

\usepackage{geometry}

\geometry{left=1.0cm,right=2.0cm,top=2.5cm,bottom=2.5cm}

\begin{document}
Latex
\end{document} 
```



```latex
\documentclass[12pt,a5paper]{article}

\usepackage{geometry}

\geometry{left=3.5cm,right=2.0cm,top=2.5cm,bottom=2.5cm}

\begin{document}
Latex
\end{document} 
```

编译后可以看到正文中的字体的位置随着页边距的变化也在变化。你可以根据自己的需要任意设置文档的页边距。

---

【完】