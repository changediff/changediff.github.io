---
title: etcd运维踩坑记录
date: 2023-03-19 18:47:10
categories:
  - 运维
tags: ["etcd"]
---
etcd运维踩坑记录

## 1. 没有开启压缩导致 dbsize 一直增长

etcd 的默认启动参数不会开启定期数据压缩 compact，一般程序使用 etcd 时会主动调用etcd 接口做压缩，例如 kubernetes。如果没有开启定期压缩，并且程序也没有主动调用压缩接口，那么就会有dbsize增长到上限（默认2GiB），etcd无法写入的风险。

### etcd手动压缩SOP
假设etcd为3节点部署，节点ip为`10.0.0.1`, `10.0.0.2`, `10.0.0.3`
1. 确认etcd主节点，压缩操作在主节点进行
```
# etcd 3.3及之前的版本，需要指定ETCDCTL_API变量版本
export ETCDCTL_API=3
etcdctl --endpoints=http://10.0.0.1:2379,http://10.0.0.2:2379,http://10.0.0.3:2379 endpoint status -w table

# 当前主节点为10.0.0.1
+