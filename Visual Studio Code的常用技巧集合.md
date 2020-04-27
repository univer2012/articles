# Visual Studio Code的常用技巧集合

### 1、问：设置为中文

答：

（1）关闭工程，重新打开VS Code，打开命令行：Mac OS：`command + shift + P`，win：`Ctrl + Shift + P`

（2）输入configure language，选择`configure display language`

（3）修改语言为`zh-CN`

答案来自：[VScode设置环境为中文](https://blog.csdn.net/menglongfc/article/details/88639346)

#### 2、问：VS Code跳转到函数之后怎么跳转回之前的位置？

答：MacOS：后退：`control + _`，前进：`control + shift + _`

菜单的 “转到” 的 “前进”， “后退”，就是做这个用的。

答案来自：[vscode 跳转到函数之后怎么跳转回之前的位置](https://www.v2ex.com/amp/t/385490)

#### 3、VS Code怎么清空终端？
答：MacOS：`command + k` ，win：`ctrl + k`
答案来自：[VS Code 添加清空控制台Ctrl+K快捷键](https://blog.csdn.net/u011199063/article/details/90902925)

#### 4、VS Code中怎么调试flutter代码？
答：
答案来自：[vscode中调试flutter非常实用方便](https://segmentfault.com/a/1190000020416153?utm_source=tag-newest)

#### 4、VS Code调试Flutter（检查用户页面）

答：
* 1、使用VS Code打开你的项目，并且运行（菜单栏 Debug-->Start Debugging(F5)），运行项目前要打开模拟器或使用真机。
* 2、在VS Code中通过F1或者Win:`Ctrl+Shift+P`，MacOS:`command+shift+P`打开命令面板，输入：`Dart:OPen DevTools`

![](https://img2018.cnblogs.com/blog/1481953/201903/1481953-20190322114025959-863047807.png)

* 3、点击`Activate`/`Upgrade Dart DevTools`按钮，激活调试工具，该调试工具会自动打开浏览器（建议使用Chrome浏览器为默认浏览器，如果不是，打开网页后，将网页地址复制到Chrome浏览器中打开）

![](https://img2018.cnblogs.com/blog/1481953/201903/1481953-20190322114104692-1043579611.png)

* 4、通过点击页面上不同的位置，可以在调试工具上显示出详情。，下面是我写的微信页面demo，github地址为https://github.com/zhaoqiyin/wechat_clone

  欢迎大家clone，也希望大家点赞。

![](https://img2018.cnblogs.com/blog/1481953/201903/1481953-20190322114545793-2116743705.png)

来自：[使用VS Code调试Flutter（检查用户页面）](https://blog.csdn.net/a50270/article/details/101161718)

