---
title: "WIN 上常使用的软件"
date: 2022-12-12
tags: ["软件", "win"]
description : "搭建开发环境--windows篇"
---

该篇主要是记录了我 win 上经常使用的软件。

# 系统
在[`MSDN,我告诉你`](https://msdn.itellyou.cn/)中下载所需要的`win10`系统

# 开发环境
安装完系统后需要安装以下环境：
- `windows`自带的软件包：`telnet客户端`，`Hyper-V`，`用于 Linux 的 Windows 子系统`；安装完以上后重启一次。
- [vscode](https://code.visualstudio.com/)
- [git](https://git-scm.com/)
- [docker](https://www.docker.com/products/docker-desktop)，由于`win`上的docker是基于 `Hyper-V` 的。
- [google chrome](https://www.google.cn/intl/zh-CN/chrome/)
- [postman](https://www.getpostman.com/downloads/)
- [DBeaver](https://dbeaver.io/)
- [Tabby](https://tabby.sh/), 控制台终端，能记录 SSH
- [Idea](https://www.jetbrains.com/idea/download/#section=windows)
- Mysql ：[Mysql Win上免安装版本](https://www.aliyundrive.com/drive/folder/60814f51f21cdcd4e87941eea010beaf250c36ef)
- 
下载解压后将 ${mysql_path}/bin 放入系统变量中
重启后需要执行 `mysqld --initialize-insecure`


#### 1. `vscode`的配置
终端配置：由于之前安装了`git`,可以将vscode的默认终端改成git的；个人比较喜欢`bash`,所以在设置的时候，我的配置如下：

```json
"terminal.integrated.shell.windows": "C:\\Windows\\System32\\bash.exe"
```
;具体参考资料如下：[终端shell配置](https://code.visualstudio.com/docs/editor/integrated-terminal)
`bash`是通过windows的商店搜索linux，下载而来的

通过终端安装`zsh`,安装[`Oh-My-zsh`](https://ohmyz.sh/)

主题：`Linux Themes for VS Code`和`VsCode Great Icons`
字体：[`Source Code Pro`](http://www.googlefonts.cn/specimen/Source+Code+Pro)

其他详细配置：请参考`vscode`的常用插件和配置

### 基础软件
- [TIM](https://office.qq.com/download.html)
- [wechat](https://pc.weixin.qq.com/)
- [foxmail](https://www.foxmail.com/win/)
- [xmind](https://www.xmind.net/download/xmind8)
- [PeaZip](https://peazip.github.io/) , 开源的压缩工具