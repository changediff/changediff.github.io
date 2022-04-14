---
title: 从外接USB设备启动树莓派4b
date: 2021-02-28 20:46:00
tags: ["rpi", "Linux"]
---

树莓派默认从tf卡启动系统，io性能太弱了。最近入手了Argon ONE外壳，可以通过usb外接一个m.2 sata接口的固态硬盘；那么，折腾一下从ssd吧。

# 方案调查
一番查资料，目前支持两种启动方案：
1. 升级固件，这也是网上推荐的主流方案。这个方案需要先用原版的raspbian升级固件，这样就可以直接设置从USB设备引导。
找到的靠谱教程如下：
[1. New Raspberry Pi 4 Bootloader USB / Network Boot Guide](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)
[2. Raspberry Pi 4 Ubuntu USB Boot (No SD Card)](https://www.instructables.com/Raspberry-Pi-4-USB-Boot-No-SD-Card/)

然鹅，我现在用的是Ubuntu系统，这个方案折腾起来比较麻烦，可能还需要重装系统。pass

2. 从tf卡引导，将根目录替换成ssd的分区。这样理论上兼容性更好，而且可以在现有的系统上升级。不犹豫，马上开搞。
参考教程：[Raspberry Pi 4 USB Boot Config Guide for SSD / Flash Drives](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/)

# 方案实施

1. 复制现有系统到ssd，==注意，这个操作会清空SSD上面的数据==
```
dd bs=4M if=/dev/mmcblk0 of=/dev/sda
```

2. 确认usb设备id，我的是`174c:55aa`
```
lsusb

Bus 002 Device 003: ID 174c:55aa ASMedia Technology Inc. Name: ASM1051E SATA 6Gb
```

3. 修改cmdline.txt，树莓派是通过这个文件来确认系统启动目录的，直接修改fstab无效
```
# 备份
cp cmdline.txt cmdline.txt.bak
# 修改为如下内容：
## 1. 注意将XXXX:XXXX替换为上一步获取的usb id
## 2. 注意root=的配置，需要和硬盘对应的LABEL或者UUID一致（如果是dd复制的数据，这块应该不用改）
usb-storage.quirks=XXXX:XXXX:u net.ifnames=0 dwc_otg.lpm_enable=0 console=serial
0,115200 console=tty1 root=LABEL=writable rootfstype=ext4 elevator=deadline
rootwait fixrtc
```

4. 更新`/etc/fstab`，这一步其实不是必须的，为了不造成迷惑，还是和`cmdline.txt`的配置保持一致了。
5. reboot之后就可以看到`/`目录已经切换到ssd上面了
```
ubuntu@rpi:~$ findmnt -n -o SOURCE /
/dev/sdb2
```
6. 不服跑个分，io速度提升10倍，哈哈：
```
sudo curl https://raw.githubusercontent.com/TheRemote/PiBenchmarks/master/Storage.sh | sudo bash

     Category                  Test                      Result
HDParm                    Disk Read                 185.42 MB/s
HDParm                    Cached Disk Read          185.55 MB/s
DD                        Disk Write                92.6 MB/s
FIO                       4k random read            4429 IOPS (17716 KB/s)
FIO                       4k random write           5109 IOPS (20439 KB/s)
IOZone                    4k read                   21790 KB/s
IOZone                    4k write                  19337 KB/s
IOZone                    4k random read            16226 KB/s
IOZone                    4k random write           20809 KB/s

                          Score: 4777
```