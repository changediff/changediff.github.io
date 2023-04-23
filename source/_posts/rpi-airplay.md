---
title: 树莓派Airplay将有线音箱变成WiFi音箱
date: 2023-04-15 17:44:05
tags:
---

## 方案选择

因为是想要作为WiFi音箱使用，方案二shairport-sync更适合我
### 视频airplay
教程：https://zhuanlan.zhihu.com/p/340535327
项目地址：https://github.com/FD-/RPiPlay

### 音频airplay
教程：https://sspai.com/post/74318
项目地址：https://github.com/mikebrady/shairport-sync



## shairport-sync安装

### 树莓派系统选择
建议使用树莓派官方的[Raspberry Pi OS](https://www.raspberrypi.com/software/)，驱动全面少踩坑。之前将树莓派当做小服务器在用，一直跑的Rocky Linux，配置音箱的时候发现缺少驱动，找不到声卡。

### 编译安装

关于如何安装参考上面的[教程](https://sspai.com/post/74318)即可

1. nqptp是 Shairport Sync 监控时间的重要依赖
https://github.com/mikebrady/nqptp.git
2. sps-alsa-explore帮助配置Shairport Sync
https://github.com/mikebrady/sps-alsa-explore.git
3. Shairport Sync本体
https://github.com/mikebrady/shairport-sync


### 可选方案docker
项目官方提供了docker方案: https://hub.docker.com/r/mikebrady/shairport-sync

```
docker run -d --restart unless-stopped --net host --device /dev/snd \
    mikebrady/shairport-sync -a DenSystem -- -d hw:0 -c PCM
```


## shairport-sync配置

使用sps-alsa-explore的建议配置即可
### 踩坑记录
grep output_device查看配置选项为`hw:`后接字符串
```
root@rpi0:~# grep output_device ./shairport-sync/audio_alsa.c -C 2
    // this is very crude -- if the device is a hardware device, then it's assumed the delay is
    // precise
    const char *output_device_name = snd_pcm_name(alsa_handle);
    int is_a_real_hardware_device = 0;
    if (output_device_name != NULL)
      is_a_real_hardware_device = (strstr(output_device_name, "hw:") == output_device_name);

    // The criteria as to whether precision delay is available
--
        warn("The output device \"%s\" is busy and can't be used by Shairport Sync at present.",
             alsa_out_dev);
      debug(2, "the alsa output_device \"%s\" is busy.", alsa_out_dev);
    } else if (ret == -ENOENT) {
      die("the alsa output_device \"%s\" can not be found.", alsa_out_dev);
    } else {
      char errorstring[1024];
--

    /* Get the Output Device Name. */
    if (config_lookup_string(config.cfg, "alsa.output_device", &str)) {
      alsa_out_dev = (char *)str;
    }
```

aplay显示的设备和`hw:`样式对不上
```
root@rpi0:~# aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 1: vc4hdmi0 [vc4-hdmi-0], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: vc4hdmi1 [vc4-hdmi-1], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

查资料后确认，具体对应关系为`hw:card名称或编号:子设备编号`，而shairport-sync只需要用到card这一层，也即`hw:0`
https://superuser.com/questions/53957/what-do-alsa-devices-like-hw0-0-mean-how-do-i-figure-out-which-to-use

### 关闭WiFi节能模式
为了防止设备WiFi设备休眠，导致查找不到音箱，需要关闭节能模式
```
iw dev wlan0 set power_save off
```
### 其他小问题修复
#### 目录权限问题
`journal -u shairport-sync`发现有报错提示没有目录权限，不影响正常使用

```
mkdir -p /home/shairport-sync/.config/pulse
chown -R shairport-sync:shairport-sync /home/shairport-sync/
```

#### 最大音量调整
1. 为了避免扰民，可以使用`alsamixer`设置树莓派的最大输出音量
2. `/etc/shairport-sync.conf`中有音量相关的配置，有待尝试

## 后续计划
- [x] 使用单独usb声卡，解决自带3.5mm接口的底噪问题
- [ ] 重新连接后音量设置有时不会记忆的问题解决

### 底噪问题解决
京东18块购入海贝思USB声卡，插入后免驱运行，修改配置为`hw:3`和`Speaker`，重启服务后获得了更大的音量上限和更低的底噪（还是有，小了很多）
```
	output_device = "hw:3";
	mixer_control_name = "Speaker";
```