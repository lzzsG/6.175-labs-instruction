---
layout: page
title: 实验0：入门
permalink: /Lab0/
nav_order: 0
---


# 实验 0: 入门

在本课程中，你将使用共享机器来完成实验。这些机器包括从 vlsifarm-03.mit.edu 到 vlsifarm-08.mit.edu。你可以通过使用你的Athena用户名和密码通过ssh登录这些机器。

本文档将指导你完成实验所需的一些操作，如获取每个实验的初始代码。首先使用ssh客户端登录上述任一服务器。

## 设置工具链

执行以下命令来设置你的环境并访问工具链：

```shell
add 6.175
source /mit/6.175/setup.sh
```

第一个命令使你能够访问课程锁定目录 /mit/6.175，并且每台电脑只需运行一次。第二个命令配置你当前的环境以包括实验所需的工具，每次登录工作时都需要运行。

## 使用Git获取和提交实验代码

参考设计提供在Git仓库中。你可以使用以下命令将它们克隆到你的工作目录中（用实验编号替换 labN，例如 lab1 和 lab2）：

```shell
git clone $GITROOT/labN.git
```

**注意**：如果 "git clone" 失败，可能是因为我们没有你的Athena用户名。请给我发送电子邮件（至 qmn mit），我将为你创建一个远程仓库。

此命令在你当前目录中创建一个 labN 目录。$GITROOT 环境变量是唯一的，因此这个仓库将是你个人的仓库。在该目录中，可以使用实验讲义中指定的指令运行测试台。

讨论问题应该在提供的代码中的 discussion.txt 文件中回答。

如果你想添加任何新文件，除了助教提供的文件外，你需要使用以下命令添加新文件（在这个例子中，是 git 中的 newFile）：

```shell
git add newFile
```

你可以在达到一个里程碑时本地提交你的代码：

```shell
git commit -am "Hit milestone"
```

通过添加任何必要的文件然后使用以下命令提交你的代码：

```shell
git commit -am "Finished lab"
git push
```

如有必要，你可以在截止日期前多次提交。

## 编写实验的Bluespec SystemVerilog（BSV）代码

### 在 vlsifarm-0x 上

如果你还不熟悉Linux命令行环境，6.175将是一个很好的学习机会。测试你的BSV代码，你需要在Linux环境下运行bsc，即BSV编译器。在同一台机器上编写BSV代码是有意义的。

虽然你可以使用许多文本编辑器，但只有Vim和Emacs为Bluespec提供了BSV语法高亮。Vim语法高亮文件可以通过运行以下命令安装：

```shell
/mit/6.175/vim_copy_BSV_syntax.sh
```

Emacs语法高亮文件可以在[课程资源](http://csg.csail.mit.edu/6.175/archive/2016/resources.html)页面找到。你的助教曾经使用Emacs，但后来转用了Vim。他无法声称知道如何安装高亮模式文件，甚至是否有效。如果你是Emacs用户并愿意就此事贡献文档，请发邮件给课程工作人员。

### 在 Athena 集群上

你在 vlsifarm 机器上的家目录与任何 Athena 机器上的家目录相同。因此，你可以在Athena机器上使用gedit或其他图形文本编辑器编写代码，然后登录到vlsifarm机器上运行它。

### 在你自己的机器上

你也可以使用文件传输程序在

你的Athena家目录和你自己的机器之间移动文件。MIT在网上提供了关于安全传输文件的帮助，网址为 <http://ist.mit.edu/software/filetransfer。>

## 在其他机器上编译BSV

BSV也可以在非vlsifarm机器上编译。这在实验截止日期临近时vlsifarm机器繁忙时可能很有用。

### 在 Athena 集群上

用于vlsifarm机器的指令也适用于基于Linux的Athena机器。只需打开一个终端，像在vlsifarm机器上一样运行命令即可。

### 在你自己的基于Linux的机器上

要在你自己的基于Linux的机器上运行6.175实验，你需要在计算机上安装以下软件：

- OpenAFS以访问课程锁定目录
- Git以访问和提交实验
- [GMP (libgmp.so.3)](https://gmplib.org/)以运行BSV编译器
- Python以运行构建脚本

**旁注**：类似的设置也可能适用于Mac OS X / macOS。如果你让这样的设置工作，请向助教提供详细信息。

#### OpenAFS

在你的本地机器上安装OpenAFS将使你能够访问包含所有课程锁定目录的目录 /afs/athena.mit.edu。你将需要在根目录中创建一个名为/mit的文件夹，并在其中使用符号链接指向必要的课程锁定目录。

CSAIL TIG有一些关于如何为Ubuntu安装OpenAFS的信息，网址为 <http://tig.csail.mit.edu/wiki/TIG/OpenAFSOnUbuntuLinux。这些指令用于访问> /afs/csail.mit.edu，但你需要访问 /afs/athena.mit.edu 来进行实验，所以无论何时看到csail都用athena替换。当你在你的机器上安装OpenAFS时，它会给你一个包含许多域的 /afs 文件夹。此网站还包含了使用你的用户名和密码登录以获取需要身份验证的文件的指令。你需要每天工作或每次重置计算机时都执行此操作，以便访问6.175课程锁定目录。

接下来你需要在根目录创建一个名为mit的文件夹，并在其中填充指向课程仓库的符号链接。在Ubuntu和类似的分发版上，命令如下：

```shell
cd /
sudo mkdir mit
cd mit
sudo ln -s /afs/athena.mit.edu/course/6/6.175 6.175
```

现在你可以在 /mit/6.175 文件夹中访问课程锁定目录了。

#### Git

在Ubuntu和类似的分发版上，你可以用以下命令安装Git：

```shell
sudo apt-get install git
```

#### GMP (libgmp.so.3)

BSV编译器使用libgmp来处理无界整数。在Ubuntu和类似的分发版上安装它，使用命令：

```shell
sudo apt-get install libgmp3-dev
```

如果你的机器上安装了libgmp，但你没有libgmp.so.3，你可以创建一个名为libgmp.so.3的符号链接，指向不同版本的libgmp。

#### Python

在Ubuntu和类似的分发版上，你可以使用以下命令安装Python：

```shell
sudo apt-get install python
```

#### 在你基于Linux的机器上设置工具链

原始的 setup.sh 脚本在你的机器上不会工作，所以你将需要使用

```shell
source /mit/6.175/local_setup.sh
```

来设置工具链。完成这些后，你应该能够像在你自己的机器上

一样正常使用这些工具。

------

© 2016 [麻省理工学院](http://web.mit.edu/)。版权所有。
