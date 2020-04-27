学习地址：[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013744142037508cf42e51debf49668810645e02887691000)

# 1、在Mac OSX上安装Git
如果你正在使用Mac做开发，有两种安装Git的方法。

一是安装homebrew，然后通过homebrew安装Git，具体方法请参考homebrew的文档：http://brew.sh/。

第二种方法更简单，也是推荐的方法，就是直接从AppStore安装Xcode，Xcode集成了Git，不过默认没有安装，你需要运行Xcode，选择菜单“Xcode”->“Preferences”，在弹出窗口中找到“Downloads”，选择“Command Line Tools”，点“Install”就可以完成安装了。

![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Git%E6%95%99%E7%A8%8B_1.png)


# 2、创建版本库
第一步，选择一个合适的地方，创建一个空目录：
```
mkdir LearnGit
cd LearnGit
pwd
/Users/huangaengoln/Documents/LearnGit
```
第二步，通过`git init` 命令把这个目录变成Git 可以管理的仓库：
```
git init
Initialized empty Git repository in /Users/huangaengoln/Documents/LearnGit/.git/
```
瞬间Git就把仓库建好了，而且告诉你是一个空的仓库（empty Git repository），细心的读者可以发现当前目录下多了一个`.git`的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。

如果你没有看到`.git`目录，那是因为这个目录默认是隐藏的，用`ls -ah`命令就可以看见。

## 2.1 把文件添加到版本库

 首先这里再明确一下，<font color=#ff0000 size=3>所有的版本控制系统，其实只能跟踪文本文件的改动</font>，比如TXT文件，网页，所有的程序代码等等，Git也不例外。版本控制系统可以告诉你每次的改动，比如在第5行加了一个单词“Linux”，在第8行删了一个单词“Windows”。**而图片、视频这些二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化，只能把二进制文件每次改动串起来，也就是只知道图片从100KB改成了120KB，但到底改了啥，版本控制系统不知道，也没法知道。**

<u>**不幸的是，Microsoft的Word格式是二进制格式，因此，版本控制系统是没法跟踪Word文件的改动的**</u>，前面我们举的例子只是为了演示，如果要真正使用版本控制系统，就要以纯文本方式编写文件。

因为文本是有编码的，比如中文有常用的GBK编码，日文有Shift_JIS编码，如果没有历史遗留问题，<font color=#ff0000 size=3>强烈建议使用标准的UTF-8编码，所有语言使用同一种编码，既没有冲突，又被所有平台所支持。</font>

 编写一个readme.txt文件，内容如下：
 ```
 Git is a version control system.
Git is free software.
 ```

 一定要放到`LearnGit`目录下（子目录也行），因为这是一个Git仓库，放到其他地方Git再厉害也找不到这个文件。

 和把大象放到冰箱需要3步相比，把一个文件放到Git仓库只需要两步。

第一步，用命令`git add`告诉Git，把文件添加到仓库：
```
git add readme.txt
```
执行上面的命令，没有任何显示，这就对了，Unix的哲学是“没有消息就是好消息”，说明添加成功。
        
第二步，用命令git commit告诉Git，把文件提交到仓库：
```
$ git commit -m "wrote a readme file"
[master (root-commit) cb926e7] wrote a readme file
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt
```
简单解释一下`git commit`命令，`-m`后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。

嫌麻烦不想输入`-m "xxx"`行不行？确实有办法可以这么干，但是强烈不建议你这么干，因为输入说明对自己对别人阅读都很重要。实在不想输入说明的童鞋请自行Google，我不告诉你这个参数。

`git commit`命令执行成功后会告诉你，1个文件被改动（我们新添加的readme.txt文件），插入了两行内容（readme.txt有两行内容）。

为什么Git添加文件需要`add`，`commit`一共两步呢？因为`commit`可以一次提交很多文件，所以你可以多次`add`不同的文件，比如：
```
git add file1.txt
git add file2.txt file3.txt
git commit -m "add 3 files."
```
继续修改readme.txt文件，改成如下内容：
```
Git is a distributed version control system.
Git is free software.
```

现在，运行`git status`命令看看结果：
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
`git status`命令可以**让我们时刻掌握仓库当前的状态**，上面的命令告诉我们，readme.txt被修改过了，但还没有准备提交的修改。

 虽然Git告诉我们readme.txt被修改了，但如果能**看看具体修改了什么内容**，自然是很好的。比如你休假两周从国外回来，第一天上班时，已经记不清上次怎么修改的readme.txt，所以，**需要用`git diff`这个命令看看**：

我们已经成功地添加并提交了一个readme.txt文件，现在，是时候继续工作了，于是，我们继续修改readme.txt文件，改成如下内容：
```
Git is a distributed version control system.
Git is free software.
```

现在，运行`git status`命令看看结果：
```
$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#    modified:   readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
```
`git status`命令可以让我们时刻掌握仓库当前的状态，上面的命令告诉我们，readme.txt被修改过了，但还没有准备提交的修改。
虽然Git告诉我们readme.txt被修改了，但如果能看看具体修改了什么内容，自然是很好的。比如你休假两周从国外回来，第一天上班时，已经记不清上次怎么修改的readme.txt，所以，需要用`git diff`这个命令看看：
```
$ git diff readme.txt 
diff --git a/readme.txt b/readme.txt
index 46d49bf..9247db6 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.
```
**`git diff`顾名思义就是查看difference，显示的格式正是Unix通用的diff格式**，可以从上面的命令输出看到，我们在第一行添加了一个“distributed”单词。

 知道了对readme.txt作了什么修改后，再把它提交到仓库就放心多了，提交修改和提交新文件是一样的两步，第一步是`git add`：
```
$ git add readme.txt
```
 同样没有任何输出。在执行第二步`git commit`之前，我们再运行`git status`看看当前仓库的状态：
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   readme.txt

```
`git status`告诉我们，将要被提交的修改包括readme.txt，下一步，就可以放心地提交了：
```
$ git commit -m "add distributed" 
[master b491306] add distributed
 1 file changed, 2 insertions(+), 2 deletions(-)
```
提交后，我们再用`git status`命令看看仓库的当前状态：
```
$ git status
On branch master
nothing to commit, working tree clean
```
 Git告诉我们当前没有需要提交的修改，而且，工作目录是干净（working directory clean）的。

 # 3. 版本回退
 现在，你已经学会了修改文件，然后把修改提交到Git版本库，现在，再练习一次，修改readme.txt文件如下：
 ```
 Git is a distributed version control system.
Git is free software distributed under the GPL.﻿
 ```
 然后尝试提交：
 ```
 $ git add readme.txt
$ git commit -m "append GPL"
[master 787b30e] append GPL
 1 file changed, 1 insertion(+), 1 deletion(-)
 ```
 像这样，你不断对文件进行修改，然后不断提交修改到版本库里，就好比玩RPG游戏时，每通过一关就会自动把游戏状态存盘，如果某一关没过去，你还可以选择读取前一关的状态。有些时候，在打Boss之前，你会手动存盘，以便万一打Boss失败了，可以从最近的地方重新开始。Git也是一样，每当你觉得文件修改到一定程度的时候，就可以“保存一个快照”，这个快照在Git中被称为`commit`。一旦你把文件改乱了，或者误删了文件，还可以从最近的一个`commit`恢复，然后继续工作，而不是把几个月的工作成果全部丢失。

现在，我们回顾一下readme.txt文件一共有几个版本被提交到Git仓库里了：

版本1：wrote a readme file
```
Git is a version control system.
Git is free software.
```
版本2：add distributed
```
Git is a distributed version control system.
Git is free software.
```
版本3：append GPL
```
Git is a distributed version control system.
Git is free software distributed under the GPL.
```
当然了，在实际工作中，我们脑子里怎么可能记得一个几千行的文件每次都改了什么内容，不然要版本控制系统干什么。版本控制系统肯定有某个命令可以**告诉我们历史记录**，在Git中，我们用`git log`命令查看：
```
$ git log
commit 787b30ea924fb5218c7e84197a389ef979bf5010
Author: univer2012 <1054880335@qq.com>
Date:   Tue Nov 28 13:19:47 2017 +0800

    append GPL

commit b49130686e56b873aa9f4e6f8ffcad2edf9e6170
Author: univer2012 <1054880335@qq.com>
Date:   Tue Nov 28 11:51:42 2017 +0800

    add distributed

commit f10039adae16c72dd3a83148f4917e28ba9d63c1
Author: univer2012 <1054880335@qq.com>
Date:   Tue Nov 28 11:44:47 2017 +0800

    wrote a readme file

```
 `git log`命令显示从最近到最远的提交日志，我们可以看到3次提交，最近的一次是`append GPL`，上一次是`add distributed`，最早的一次是`wrote a readme file`。


**如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数：**
```
$ git log --pretty=oneline
787b30ea924fb5218c7e84197a389ef979bf5010 append GPL
b49130686e56b873aa9f4e6f8ffcad2edf9e6170 add distributed
f10039adae16c72dd3a83148f4917e28ba9d63c1 wrote a readme file
```

需要友情提示的是，你看到的一大串类似`3628164...882e1e0`的是`commit id`（版本号），和SVN不一样，Git的`commit id`不是1，2，3……递增的数字，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示，而且你看到的`commit id`和我的肯定不一样，以你自己的为准。为什么`commit id`需要用这么一大串数字表示呢？因为Git是分布式的版本控制系统，后面我们还要研究多人在同一个版本库里工作，如果大家都用1，2，3……作为版本号，那肯定就冲突了。

每提交一个新版本，实际上Git就会把它们自动串成一条时间线。如果使用可视化工具查看Git历史，就可以更清楚地看到提交历史的时间线：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Git%E6%95%99%E7%A8%8B_2.png)


好了，现在我们启动时光穿梭机，**准备把readme.txt回退到上一个版本，也就是“add distributed”的那个版本，怎么做呢？**

首先，Git必须知道当前版本是哪个版本，在Git中，用`HEAD`表示当前版本，也就是最新的提交`3628164...882e1e0`（注意我的提交ID和你的肯定不一样），上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

现在，我们要把当前版本“append GPL”回退到上一个版本“add distributed”，就可以使用`git reset`命令：
```
$ git reset --hard HEAD^
HEAD is now at b491306 add distributed
```
`--hard`参数有啥意义？这个后面再讲，现在你先放心使用。

看看readme.txt的内容是不是版本`add distributed`：
```
$ cat readme.txt
Git is a distributed version control system.
Git is free software.
```
果然。

还可以继续回退到上一个版本`wrote a readme file`，不过且慢，然我们用`git log`再看看现在版本库的状态：
```
$ git log
commit b49130686e56b873aa9f4e6f8ffcad2edf9e6170
Author: univer2012 <1054880335@qq.com>
Date:   Tue Nov 28 11:51:42 2017 +0800

    add distributed

commit f10039adae16c72dd3a83148f4917e28ba9d63c1
Author: univer2012 <1054880335@qq.com>
Date:   Tue Nov 28 11:44:47 2017 +0800

    wrote a readme file

```

**最新的那个版本`append GPL`已经看不到了！好比你从21世纪坐时光穿梭机来到了19世纪，想再回去已经回不去了，肿么办？**

**办法其实还是有的，只要上面的命令行窗口还没有被关掉，你就可以顺着往上找啊找啊，找到那个`append GP`L的`commit id`是`fa800ec...`**，于是就可以指定回到未来的某个版本：
```
$ git reset --hard fa800ec
HEAD is now at fa800ec append GPL
```
**版本号没必要写全，前几位就可以了，Git会自动去找**。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了。


再小心翼翼地看看readme.txt的内容：
```
cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
```

果然，我胡汉三又回来了。

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的`HEAD`指针，当你回退版本的时候，Git仅仅是把HEAD从指向`append GPL`：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Git%E6%95%99%E7%A8%8B_3.png)

改为指向`add distributed`：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Git%E6%95%99%E7%A8%8B_4.png)
然后顺便把工作区的文件更新了。所以你让`HEAD`指向哪个版本号，你就把当前版本定位在哪。

 现在，**你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的`commit id`怎么办？**

在Git中，总是有后悔药可以吃的。当你用`$ git reset --hard HEAD^`回退到`add distributed`版本时，再想恢复到`append GPL`，就必须找到`append GPL`的commit id。Git提供了一个命令`git reflog`用来记录你的每一次命令：

```
$ git reflog
9278195 HEAD@{1}: reset: moving to HEAD^
fa800ec HEAD@{2}: commit: append GPL
9278195 HEAD@{3}: commit: add distributed
6ccfe68 HEAD@{4}: commit (initial): wrote a readme file
```
终于舒了口气，第二行显示`append GPL`的commit id是`3628164`，现在，你又可以乘坐时光机回到未来了。

# 4.工作区和暂存区
Git和其他版本控制系统如SVN的一个不同之处就是有暂存区的概念。

### 工作区（Working Directory）
就是你在电脑里能看到的目录，比如我的`learngit`文件夹就是一个工作区：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Git%E6%95%99%E7%A8%8B_5.png)

### 版本库（Repository）
工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/Git%E6%95%99%E7%A8%8B_6.png)

分支和`HEAD`的概念我们以后再讲。

前面讲了我们把文件往Git版本库里添加的时候，是分两步执行的：

第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

因为我们创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`master`分支上提交更改。


你可以简单理解为，**需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。**

俗话说，实践出真知。现在，我们再练习一遍，先对`readme.txt`做个修改，比如加上一行内容：
```
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
```
然后，在工作区新增一个`LICENSE`文本文件（内容随便写）。
先用`git status`查看一下状态：
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	LICENSE

no changes added to commit (use "git add" and/or "git commit -a")
```
Git非常清楚地告诉我们，`readme.txt`被修改了，而`LICENSE`还从来没有被添加过，所以它的状态是`Untracked`。

 现在，使用两次命令`git add`，把readme.txt和LICENSE都添加后，用`git status`再查看一下：
```
$ git add readme.txt 
$ git add LICENSE 
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   LICENSE
	modified:   readme.txt

```