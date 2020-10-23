### mac osX上使用latex 的技巧

来自：[texshop 使用技巧](http://www.mamicode.com/info-detail-2357498.html)

---



* 指定编译器， 通过宏定义， 也就是在文件开头， 加上类似命令 `% !TEX TS-program = pdflatex`
* 多个文件设定主文件， 通过宏定义， 加上主文件的路径， 比如 `% !TEX root =  ../main.tex`
* 命令补全， 用`esc` 键补全， 比如 输入`be` ， 然后点击 `esc` 键1次或者多次， 看看结果变化。
* 单词补全， 用F5补全
* `\ref{}` 补全， 用F5会出现label列表给选择
* `\cite{}` 补全， ？
* 编译快捷键 `command + T`



