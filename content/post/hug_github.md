---
title: "利用 Hugo 搭建 github 博客"
date: 2021-04-16
tags: ["blog", "hugo"]
description : "该篇文章主要是讲述怎么利用 mac 上怎么使用 hugo 和 github 搭建自己的博客。"
---
该篇文章主要是讲述 mac 上怎么使用 hugo 和 github 搭建自己的博客。

## 新建 github 博客仓库

新建 github 仓库, 名称为 xxx.github.io,  xxx 是你的用户名，例如我的用户名为: `plutokaito`, 那么仓库名就是 `plutokaito.github.io`, 具体操作如下图所示操作：

![新建博客仓库](/images/post/1.png)

由于我之前已经存在了该仓库，所以不能新建，没有的话，就可以新建仓库， 新建项目后 clone 到本地。

## 安装 hugo 项目
在 [github releases 页面](https://github.com/gohugoio/hugo/releases) 中找到相应的版本, 本篇文章以 macos 为例， 找到 `hugo_0.82.0_macOS-64bit.tar.gz`, 进行下载，当然也可以参照[官网](https://gohugo.io/getting-started/installing/)其他的方式进行安装，如下图所示:

![下载包](/images/post/2.png)

加载完保存完后，解压缩，后台能看到
![文件结构](/images/post/3.jpg)

使用命令
```sh
echo ”export PATH=$PATH:{DOWNLOAD_PATH}/hugo_0.82.0_macOS-64bit >> ~/.zhsrc“
source ~/.zshrc
```
更改 DOWNLOAD_PATH 路径成 hugo 解压后的路径，`~/.zshrc` 为我本地的路径，可以更改成 `~/.bashrc` 或者 `~/.bash_profile`。 source 命令使命令立即生效。 使用 `hugo version` 检验是否安装成功。
```sh
$ hugo version
hugo v0.82.0-9D960784 ...
```

## 新建 hugo 项目
使用命令
```sh
$ hugo new site myblog
Congratulations! Your new Hugo site is created in ....
```
myblog 为你的文件夹名，当出现命令行中出现 Configuration·时，说明已经新建成功了，将 myblog 目录中的内容拷贝到之前从 github 仓库 clone 到本地的目录中。

### 设置 hugo 主题
进入博客仓库目录后， 利用 `git submodule` 命令安装主题，

```sh
$ git submodule add https://github.com/lxndrblz/anatole.git themes/anatole
Cloning into '../themes/anatole'...
remote: Enumerating objects: 1586, done.
remote: Counting objects: 100% (167/167), done.
remote: Compressing objects: 100% (156/156), done.
remote: Total 1586 (delta 71), reused 91 (delta 11), pack-reused 1419
Receiving objects: 100% (1586/1586), 4.78 MiB | 729.00 KiB/s, done.
Resolving deltas: 100% (826/826), done.
```
这样就安装好了主题，在此项目的根目录中的 `config.toml` 进行配置， 配置 ok， 使用命令 `hugo server -D` 验证本地是否ok, 根据命令提示，打开预览地址，默认 `http://localhost:1313`


## 配置 github action
### 配置 yml 文件
在项目根目录中新建文件
`.github/workflows/.gp-pages.yml`, 配置如下：
```yml
name: github pages

on:
  push:
    branches:
      - main  # Set a branch name to trigger deployment

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.82.0'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

### 新建 GITHUB_TOKEN
在本地机子上，使用命令
```sh
$ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```
生成秘钥对。
1. 在 github 项目 Tab 页 Settings 中 Deploy Keys 设置新增 deploy key，将生成好的 public key 中的内容贴上去，保存即可。如下图所示：
![新增 deploy key](/images/post/4.png)

2. 在 github 项目 Tab 页 Settings 中的 Secrets 中新建名字为 : `ACTIONS_DEPLOY_KEY` 的 secret.
如下图所示：
![新增 secret](/images/post/5.png)

3. 上传代码后，项目会有一个 `gh-pages` 的分支，将 `Setting > Pages` 中的分支更改成 gp-pages。即可。目录为 /(root) 目录。 具体配置如下：
![配置访问分支](/images/post/6.png)


## 验收成果
打开网址 https://plutokaito.github.io，访问成功。
![成功访问](/images/post/7.png)

## 自定义域名
### 设置域名解析
在域名购买商设置好域名解析，主域名用 A 即可，以下是我的设置。
![域名解析](/images/post/hugo_github/dns.png)

然后因为我要访问的 www.kaitoshy.com 作为我的博客域名地址，因此我又设置 CNAME。

设置好使用 dig 工具，查看：
```powershell
>  dig www.kaitoshy.com +nostats +nocomments +nocmd
;www.kaitoshy.com.              IN      A
www.kaitoshy.com.       600     IN      CNAME   plutokaito.github.io.
plutokaito.github.io.   3600    IN      A       185.199.109.153
plutokaito.github.io.   3600    IN      A       185.199.108.153
plutokaito.github.io.   3600    IN      A       185.199.111.153
plutokaito.github.io.   3600    IN      A       185.199.110.153
```

### 增加 CNAME 文件
在 static 中增加 CNAME 文件， 编译后 CNAME 就会放到 root 的目录中， 在 root 中的 CNAME 文件会被 gtihub-pages 自动解析到。

### 验证自定义域名
推送到远程仓库后，查看自定义域名，www.kaitoshy.com.访问浏览器，出现以下画面就表示成功了。


![成功](/images/post/hugo_github/success.png)

## favicon.ico
利用网站 [figma](https://www.figma.com/) 手绘了一个符合网站个性的图片，生成后利用 png 转 ico 的工具转成后缀为 ico 的格式的文件，放在 static 下。 重启后便能在浏览器上的标签页上看到相应的图片了。


> 参考资料
- [GitHub Pages action](https://github.com/marketplace/actions/github-pages-action#%EF%B8%8F-first-deployment-with-github_token)
- [gohu 部署在 github 上](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
- [gohu 快速入门](https://gohugo.io/getting-started/quick-start/)
- [anantole 主题](https://github.com/lxndrblz/anatole)
- [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
