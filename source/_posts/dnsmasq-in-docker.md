---
title: dnsmasq容器部署折腾
date: 2022-09-20 21:31:46
tags: ["dnsmasq", "docker"]
---
## 蛋疼的华为云K8s的DNS问题
最近换工作，接触各个云厂商的服务比较多，忍不住想吐槽下华为云。

华为云不支持云DNS转发，也就是说如果想要解析公司其他DNS域名的内网域名就得更换华为云的默认DNS（阿里云的Private Zone功能支持将云DNS的上游加上自建DNS，直接解决了这个问题）。如果只是用云服务器，那么换成自建DNS也不用影响，然而华为云的CCE（托管版K8s）的Node节点依赖华为云的DNS做一些劫持（主要有dockerhub劫持、华为repo内网解析、华为容器镜像仓库地址解析），将Node的`/etc/resolv.conf`改到了自建DNS之后，上面的劫持就失效了，也就导致Node节点没法kube-system相关镜像，也没法yum安装软件包。

所以，华为云的CCE的node节点的DNS是不能直接更换的，然而我又不得不换，因为普通网络模式的pod可以通过coredns配置将DNS请求转发到自己DNS，*但是host网络模式的pod使用的还是node上`/etc/resolv.conf`*。

于是，我的需求就变成了，换掉node的DNS，但是不直接换，DNS请求先走华为云的DNS，找不到记录之后再去查找自建DNS（其实就是阿里云的Private Zone功能之一）

## 折腾解决
最终，通过在Node上部署dnsmasq，在dnsmasq中指定DNS服务器顺序，本机的DNS指向本机内网ip的方式解决了这个问题。这样相当于给K8s引入了外部依赖，不过可以通过初始化脚本解决这个问题

### 容器部署
每个节点都需要部署的服务，使用DaemonSet部署是最方便的，配置可以使用configmap管理。既然折腾了就折腾导致，看看容器里面怎么跑dnsmasq（这其实会导致鸡生蛋蛋生鸡的问题，初始化node的时候需要先将dnsmasq的镜像放到node上才好启动服务）

#### 踩坑
一开始的启动参数是`ENTRYPOINT ["dnsmasq", "-k"]`，容器启动直接退出了，没有任何报错；改为debug模式`ENTRYPOINT ["dnsmasq", "-d"]`启动，服务则没有任何问题，然而线上服务不能跑在debug模式。反复尝试不同base镜像和版本，都其实失败，还没有任何日志。

最后尝试在物理机上使用`dnsmasq -k`正常启动，观察发现`-k`启动的进程运行用户是`nobody`，`-d`启动的进程用户是`root`，怀疑是容器中fork切换用户之后容器认为主进程退出运行了。`dnsmasq --help`查看帮助，发现可以指定用户。
```
-u, --user=<username>   Change to this user after startup. (defaults to nobody).
```

最终使用`ENTRYPOINT ["dnsmasq", "-k", "-u", "root"]`在容器中成功启动进程

#### 最终配置
最终的可运行的容器配置如下：
```
# 使用centos镜像方便调试，最终部署可以换成alpine优化镜像大小，红帽的ubi8镜像也是不错的选择
FROM centos:7
RUN yum install -y dnsmasq
ADD ./main.conf /etc/dnsmasq.d/
ADD ./dns.conf /etc/dnsmasq.d/
ENTRYPOINT ["dnsmasq", "-k", "-u", "root"]
```

```
# cat ./main.conf
no-resolv
domain-needed
no-negcache
max-cache-ttl=1
enable-dbus
dns-forward-max=10000
cache-size=10000
bind-dynamic
min-port=1024
except-interface=lo
# End of config
```

```
# cat ./dns.conf
server=114.114.114.114  #自建DNS
server=100.125.1.250
server=100.125.64.250
```

### systemd部署
使用systemd部署，服务启动方便，避免和容器自身形成循环依赖
```
yum install -y dnsmasq

```

## 未完待续
为什么`dnsmasq -k`在容器中会退出？