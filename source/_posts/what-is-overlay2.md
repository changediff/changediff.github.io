---
title: overlay2笔记
tags:
  - docker
  - overlay2
  - devicemapper
  - union filesystem
date: 2022-08-03 20:58:42
---


## overlay2文件系统的原理
overlay2的实现原理直观如下图：
![overlay2](overlay_constructs.jpeg)

可以使用下面的命令做简单的模拟：
```
mkdir -p /overlaytest/client_1/{upperdir,workdir} /overlaytest/client_2/{upperdir,workdir} lowerdir merged/{client_1,client_2}
mount -t overlay overlay -o lowerdir=/overlaytest/lowerdir -o upperdir=/overlaytest/client_1/upperdir -o workdir=/overlaytest/client_1/workdir /overlaytest/merged/client_1
mount -t overlay overlay -o lowerdir=/overlaytest/lowerdir -o upperdir=/overlaytest/client_2/upperdir -o workdir=/overlaytest/client_2/workdir /overlaytest/merged/client_2
```



## 对比

docker的文件驱动的发展路径是：`aufs --> devicemapper --> overlay --> overlay2`，目前生产中用的最多的是devicemapper和overlay2。

### devicemapper
devicemapper使用lvm作为底层实现技术，有thin pool空间满和inode占用高的问题。
![devicemapper](two_dm_container.jpeg)

### squashfs
只读文件系统，软路由系统openwrt推荐的文件系统格式。系统文件本身只读，变动通过overlay文件系统挂载，这样就实现了系统重置的功能。

### `mount --bind`
`mount --bind`可以实现类似于overlay2的功能，可以替换掉目录下的某一个文件或者文件夹


## 其他值得关注的点

### inode性能问题
哪些drive有inode问题？
devicemapper/overlay有inode问题，overlay中使用硬链接可能导致inode问题，overlay2的inode利用率更高，规避了该问题

### f_type
如果使用xfs文件系统，需要主要f_type选项设置为1，否则无法使用overlay2。具体可通过`xfs_info`查看

### pagecache
overlay2文件系统是共享pagecache的，这样对于底层文件系统中同一个文件的读取可以通过共用pagecache来提高效率，不过这也意味着容器化不能完全隔离pagecache

### copy on write
copy on write特性，导致如果先读取lowerdir的文件，会感知不到后续修改。该问题使用python脚本未复现，生产环境中通过挂载日志目录。

### rename
rename(2) systemcall的支持有差别，具体件参考文档

## 参考文档
https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt
https://docs.docker.com/storage/storagedriver/overlayfs-driver/
https://gdevillele.github.io/engine/userguide/storagedriver/overlayfs-driver/
https://gdevillele.github.io/engine/userguide/storagedriver/device-mapper-driver/
https://blog.k8s.li/Exploring-container-image.html
https://mp.weixin.qq.com/s/yacj95ltAZ3NfM-lv2NyXQ
https://blogs.cisco.com/developer/373-containerimages-03