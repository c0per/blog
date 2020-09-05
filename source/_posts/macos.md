---
title: OI with macOS
date: 2018-03-14 23:44:12
tags:
- 技术
- macOS
categories:
- 技术
---

# OI with macOS

## 前言

本蒟蒻从学校处获得MacBook Pro 2014一台，遂开始使用macOS作为主力系统。

当然，遇到了很多很多困难~~辣鸡苹果~~，特地写了一篇博客，帮助萌新顺利进场。

{% asset_image high_sierra.png 系统信息 %}

<!-- more -->

## Homebrew

在Ubuntu，Arch Linux上，有apt和pacman等好用的软件包管理器。在macOS中，有homebrew作为替代，但需要我们手动安装。


## gdb

## 安装gdb8.0.1

这是tm的巨坑啊！本蒟蒻搞了一天啊。

网上的教程都是Sierra的，而High Sierra就是很难受了。

首先，需要使用homebrew安装gdb**8.0.1**，最新的版本gdb8.1会瞎鸡儿报错。

```
brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/c3128a5c335bd2fa75ffba9d721e9910134e4644/Formula/gdb.rb
```

## 处理证书权限

找到“钥匙串访问”的应用，点开菜单中的钥匙串访问=>证书助理=>创建证书。

如图所示。

{% asset_image gdb-cert.png %}

然后一路确定。

成功创建后，将你的证书从“登录钥匙串”拖动到“系统钥匙串”。

双击，将“信任”改为“始终信任”。

关机，开机时按住command+R，进入恢复模式。

在终端中执行：

```
csrutil enable --without debug
```

重启，分别在终端执行：

```
codesign -fs gdb-cert /usr/local/bin/gdb
echo "set startup-with-shell off" >> ~/.gdbinit
```

现在，打开你的gdb，应该可以正常运行。

## 使用gcc中的g++

macOS会预装来自clang的g++，并占了g++的命令。

我们可以用brew来安装gcc中的g++。

```
brew install gcc
```

为了使用g++命令来代替g++-7，执行：

```
echo "alias g++=/usr/local/bin/g++-7" >> ~/.bash_profile
```

## 安装MacVim

MacOS自带的vim各种难用，我们用MacVim来代替。

使用brew来安装：

```
brew install macvim
```

接着就能正常使用啦～

用vim命令来代替mvim：

```
echo "alias vim=/usr/local/bin/mvim" >> ~/.bash_profile
```

**以上**
