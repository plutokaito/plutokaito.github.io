---
title: " ShenYu 后台前端页面部署方式"
date: 2021-06-08
tags: ["shenyu", "gateway", "java"]
description : "该篇文章主要是讲述 shenyu 网关怎么集成静态文件进行部署"
---

# 前言

由于之前的接触的到的管理程序都是后端代码和前端分别打包放在，然后利用 Nginx 做代理，将两个代码进行访问。但是看 ShenYu 启动时是直接能将前端代码进行访问，是有什么黑魔法么？
还是将前端代码直接打包好放入 ShenYu Admin 中呢？还是有其他方法呢？

# Static 静态文件

## 验证是否含有静态文件

查看源代码，确实有静态文件，
![static](https://upload-images.jianshu.io/upload_images/4058540-f9028a5753c2ca12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

我们先验证一下，是不是这里，删除这块代码然后再启动，果然页面就办法访问了。
![访问页面](https://upload-images.jianshu.io/upload_images/4058540-be604aae6b4c1287.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 总结

利用编译好的前端静态文件，减少了使用者的两次编译部署(前端编译打包一次，后端代码打包一次)的麻烦，但这样对于前端代码的更新，static 目录就要随着更新，那这样的话，`shenyu-dashboard` 的目录不就没什么作用了么？那他有扮演着什么样的角色呢？

# shenyu-dashboard 的作用

查看 shenyu-admin 的 `pom.xml` 时发现这么一个插件
![erislett插件](https://upload-images.jianshu.io/upload_images/4058540-cd981a4a6875c212.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看 eirslett 插件的 [github 地址](https://github.com/eirslett/frontend-maven-plugin), 介绍有这么一段话：
`this plugin downloads/installs Node and NPM locally for your project`  
这个意思就是说，在本地执行前端代码的编译的工作。安装 npm 的教程，可以参考
[安装Node.js和npm](https://www.liaoxuefeng.com/wiki/1022910821149312/1023025597810528), 如果没安装 npm 和 nodejs 也没关系，该插件会帮你进行安装。

根据 shenyu-admin 中的 `pom.xml` 上的注释，我们修改项目中的 pom 文件，结果如下图所示：
![image.png](https://upload-images.jianshu.io/upload_images/4058540-30386cbea34c6950.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再次重启。发现还是无法访问，再次查看 `pom.xml` 文档，发现前端代码的工作目录为：`shenyu-dashboard`, 如下图所示
![工作目录为 shenyu-dashboard](https://upload-images.jianshu.io/upload_images/4058540-9751115f0bf2acb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看源代码中 `shenyu-dashboard` 文件，发现为空。查看源代码，发现 [git 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)的设置，如下图所示：
![前端子模块代码](https://upload-images.jianshu.io/upload_images/4058540-9c7cc21bef4c214b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用命令 `git submodule update --init --recursive`, 更新代码，成功后如下图所示
![更新子模块](https://upload-images.jianshu.io/upload_images/4058540-c8915754b165c413.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个时候会很慢，要耐心等待，没有进度条。

再次启动。又发现报错了
![npm报错](https://upload-images.jianshu.io/upload_images/4058540-67b8029dab878adc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改如下：
![pom.xml](https://upload-images.jianshu.io/upload_images/4058540-dbb626a592c6d2fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注释掉为了防止再次安装，更改 `--registry` 成淘宝镜像，加速安装

好了，再次执行命令`mvn clean install`后发现，之前删掉的文件又回来了。
![static 中的文件](https://upload-images.jianshu.io/upload_images/4058540-1d5e818cd59cebc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 为了防止每次都编译前端代码，可以将项目根目录下的 pom 文件中配置 `frontend.plugin.skip` 修改成 `true`
> 启动程序，有可以愉快开心的玩耍了。

# 总结：

1.  `frontend-maven-plugin` 用在管理后台上还是很不错的。

> 拓展阅读：
[Shenyu 网关 Github](https://github.com/dromara/shenyu)
