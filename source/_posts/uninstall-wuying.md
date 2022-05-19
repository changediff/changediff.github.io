---
title: 卸载无影云桌面以及相关插件
date: 2022-05-18 21:00:54
tags: ["macOS", "日常折腾"]
---
最近阿里云的无影云桌面在做免费推广，出于好奇试用了下。分别尝试了mac桌面端和网页版，发现网页版已经满足使用需求了。于是决定卸载桌面端。

## round 1
在访达finder的应用程序文件夹中找到对应的app执行删除
![finder](app.png)

然而今天重启笔记本之后发现有个**Elastic Desktop Service**还是会开机自启
![finder](bar.png)

## round 2
猜测和第一次启动的时候安装的插件有关，但是程序弹窗要求提权
![plugin](plugin.png)

查看任务管理器和程序关于，基本确认是Citrix插件没有卸载干净
![citrix](citrix.png)
![citrix2](citrix2.png)

怀疑Library下面有部分文件没有删除，删除之
![find-citrix](find-citrix.png)

## round 3
然而重启之后**Elastic Desktop Service**还是出现了！！！
![finder](bar.png)

重新查文档，发现无影云桌面可能是使用的这家公司的服务，[Citrix官网](https://docs.citrix.com/en-us/citrix-workspace-app-for-mac/install-uninstall.html)是提供了卸载程序的

![citrix-uninstaller](citrix-uninstaller.png)

最后别忘了将Library下面的文件再删一遍

## end

折腾就像生活，不放弃多学习多尝试，总能有所收获