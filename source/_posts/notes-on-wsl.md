---
title: "Windows上的shell环境折腾笔记"
date: 2022-08-25 22:52:38
categories:
  - 效率
tags: ["WSL", "Linux", "cygwin", "Shell"]
---

最近工作电脑从MacOS换到了Windows 10，shell环境的缺失让我很不习惯，于是便开启我的折腾之旅。

工作电脑的Windows系统没有管理员UAC权限，没有微软应用商店，只能从内网软件中心安装软件，这也就导致操作有很多限制。

下面按照时间顺序记录下尝试过的方案

## Cygwin类

都是基于Cygwin的方案，使用mingw64编译器移植的Linux系工具

### git for windows

软件中心有git，windows版本的git是自带一个mintty的bash环境的，可以当做简单的shell环境使用。然而该方案有很大的缺点：

1. 没有包管理器，无法安装Linux生态工具
2. 快捷键不方便

### cmder

https://cmder.app/

优点：
1. 有绿色Portable版本，解压即用，不需要提权
2. 终端界面更美观了，也可以用来调用PowerShell

缺点
1. 没法使用`ctrl+w`等快捷键
2. 没有包管理器

### mobaxterm

https://mobaxterm.mobatek.net/

优点：
1. 有绿色Portable版本，解压即用，不需要提权
2. 有自己的包管理器，可以安装一些常见的软件包
3. 兼容常见的终端快捷键

缺点：
1. 源的使用体验一般，部分软件包的链接已失效
2. 界面复杂，个人更喜欢简洁风格

## PowerShell

Windows自带的PowerShell同样很大

### PowerShell5 + cmder/Windows-Terminal

Shell环境不好用之后，也考虑过使用PowerShell作为替代，不过系统自带的版本不好用。其和bash的功能和使用习惯差异巨大，例如没法`ctrl+r`搜索历史命令，没法使用`&&`等等。

不过在折腾PowerShell期间发现了另外的神器`Windows-Terminal`。

https://github.com/microsoft/terminal/releases

从上面的链接下载最新版本的`msixbundle`安装包，双击打开安装即可拥有。或者使用PowerShell命令安装：
```
Add-AppxPackage .\app_name.appx
```

如果是低版本的Win10，可能需要参考以下教程去先安装`VC++ v14 Desktop Framework Package`

https://docs.microsoft.com/troubleshoot/cpp/c-runtime-packages-desktop-bridge#how-to-install-and-update-desktop-framework-packages

上述安装操作都**不需要UAC提权**。

### PowerShell7 + cmder/Windows-Terminal

https://github.com/PowerShell/PowerShell

PowerShell的使用体验和bash更加接近，支持了`ctrl+w`, `ctrl+r`, `&&`等功能，且脚本可全平台运行。

安装方式双击打开或者使用下述命令都可以，同样不需要UAC提权
```
Add-AppxPackage .\app_name.appx
```

## WSL

之所以一开始不选择WSL，是因为没有应用商店安装比较麻烦，且安装过程中**需要申请UAC提权**。

### Docker Desktop

上面折腾了一圈，总觉得用着不爽，所以考虑了容器方案，申请了临时管理员权限，使用了官方推荐的`WSL2`后端。

安装步骤如下，以下操作**需要UAC提权**：

1. docker官网下载安装包并安装：https://www.docker.com/products/docker-desktop/
2. 按照教程加的docker-user用户组权限`C:\WINDOWS\system32\compmgmt.msc`：https://blog.csdn.net/fragrant_no1/article/details/84256775
3. 安装的【 Linux 内核更新包】：https://docs.microsoft.com/zh-cn/windows/wsl/install-manual#step-4