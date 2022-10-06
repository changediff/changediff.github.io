---
title: ubi真香
tags: ["ubi", "RHEL", "docker", "image"]
---
最近刚接触到红帽的ubi镜像，Universal Base Image. ubi是基于RHEL制作的用于容器镜像的发行版，其软件源和RHEL通用，且可免费使用。最近制作的镜像都切到了ubi8，用了一段时间发现真香，值得记录一下。

说起大家更熟悉的应该是Alpine, 但是从维护成本、系统稳定性、安全等维度来看，ubi镜像更有优势，具体可以看Reddit上的讨论：[Current thoughts on UBI?](https://www.reddit.com/r/redhat/comments/s2jz7y/current_thoughts_on_ubi/)

## Entrypoint
coreutils


## ubi-init

