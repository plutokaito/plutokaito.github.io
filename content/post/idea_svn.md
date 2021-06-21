---
title: "Win 上 idea ce 版本集成 svn 使用"
date: 2021-06-18
tags: ["工具"]
description : "该篇文章主要是 idea ce 版本集成 svn"
---

本文主要是讲述怎么在 Windows 上的 intellij Idea 集成 SVN, 虽然现在用 Git 占大多数，但是不免有些情况下还是需要用到 SVN 的。 集成的好处就是不用在进入文件资源管理器进行更新，提交什么的。在 IDE 便能方便的完成此项操作。又有哪些坑需要注意的呢？


## 1. 下载 intellij Idea 
进入 idea 的 [windows 下载页面](https://www.jetbrains.com/idea/download/#section=windows)， 下载社区版本便能满足日常的开发。当然可以支持一下 Ultimate 的版本。 点击下载好的程序安装即可。

## 2. 安装 TortoiseSVN
在 windows 上，我们常用的 svn 客户端软件为 [TortoiseSVN](https://tortoisesvn.net/downloads.zh.html), 根据系统选择相应的版本。 点击下载好的安装程序。 当进入以下界面时: ![svn](/images/post/svn/svn.png) 需要注意的两个点：
- command line client tools # 这个默认是未安装的，需要将其安装，这是安装命令行的 svn， 这个是 idea 集成 svn 的关键所在。
- Localtion # 默认是安装在系统盘目录下的， 更改成其他的，可以防止软件安装在系统上。

然后点击下一步安装即可。


## 3. idea 集成 svn
这里有两种方式：
1. 利用外部的 TortoiseSVN 或者命令行检出相应仓库到指定的文件夹。以 TortoiseSVN 为例， 在指定的文件夹下面点击鼠标右键， 会出现 SVN CheckOut 的选项，在 checkout 界面输入 svn 的地址，点击确定。 操作如下图所示
![鼠标右键](/images/post/svn/t-svn.png) 鼠标右键
![chekcout](/images/post/svn/checkout.png) 检出
打开 idea, 菜单上有 svn 有字样，说明已经离成功很近了。可以使用快捷键 CTRL + T 或者点击图片所指位置。 ![更新](/images/post/svn/update.png)进行更新项目。出现 Update 的提示框点击确定后，有项目更新或者提示没有更新，说明已经成功了。



2. 打开 idea， 此时没有任何项目，菜单上会出现 VCS ， 点击 "Browse VCS Respository" 中的 ”Browse Subversion Repository“。 底部会出现增加 repo 的提示。点击 Add 按钮， 如下图所示: ![cvs](/images/post/svn/csv.png), 输入仓库地址。
底部出现仓库时，右键 checkout 到相应的位置， idea 会提示是新窗口打开还是本窗口，按照自己喜欢的选择就好了。验证是否集成成功与上述一致。


综上：win 上使用 svn 有界面的话还是在乌龟 svn 中使用方便，在苹果上虽然也有 svn 的图形化，还是没有 win 上来的方便，所以苹果上的话可以参照这个第二种方式来使用。
