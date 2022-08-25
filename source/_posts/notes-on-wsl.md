---
title: "Windows上的shell环境折腾笔记"
date: 2022-08-25 22:52:38
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
3. 安装的【 Linux 内核更新包】：https://docs.microsoft.com/zh-cn/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package
4. 重启笔记本

至此，已经拥有了完整的Linux环境。然而容器环境启动和文件操作不方便，每次重启笔记本都需要重新启动容器，Docker本身的冷启动时间较长。

### Ubuntu-22.04 on WSL

既然已经安装好了`wsl`环境，那干脆尝试安装下其他发行版吧，考虑到使用便利性，我选择了`Ubuntu22.04`([下载地址](https://aka.ms/wslubuntu2204))
下载完成后使用一下命令安装，**注意，双击打开安装会报错**：
```
Add-AppxPackage .\app_name.appx
```
此时打开Windows-Terminal发现已经添加好了Ubuntu-22.04的启动配置，新建终端标签启动即可。第一次启动需要初始化，耗时较长。

其他发行版安装可参考官方教程：
https://docs.microsoft.com/en-us/windows/wsl/install-manual

至此，已经拥有了可以快速启动，无缝启动，丝般顺滑。然而WSL无法使用systemd，这就导致dockerd无法启动

### Ubuntu-22.04 on WSL2

于是我将目标瞄准了虚拟化更彻底的WSL2

将WSL的Linux发行版转换成WSL2很方便，只需要一条命令即可拥有，过程中不需要UAC提权：
```
wsl --set-version Ubuntu-22.04 2
```

使用wsl-distrod方案安装systemd，具体教程见官方repo的`Option 2: Make your Current Distro Run Systemd`部分：

https://github.com/nullpo-head/wsl-distrod

至此，我就在Windows上拥有了无缝的bash体验，使用过程中不需要UAC提权，还拥有WSL系统的root权限，完结，撒花！！！

