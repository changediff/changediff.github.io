---
title: etcd运维踩坑记录
date: 2023-03-19 18:47:10
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
+------------------+------------------+---------+---------+-----------+-----------+------------+-----------------+------------------+------------+
|      ENDPOINT     |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX | RAFT APPLIED IDX |      ERRORS      | PROXY ERROR |
+------------------+------------------+---------+---------+-----------+-----------+------------+-----------------+------------------+------------+
| http://10.0.0.1: | d4f945f9af97d6c1 |  3.5.3  |  35 MB  |     true  |      5    |   243336   |      243336     |                  |            |
|      2379        |                  |         |         |           |           |            |                 |                  |            |
| http://10.0.0.2: | 7aa2e8e903c2a2d9 |  3.5.3  |  35 MB  |     false |      5    |   243336   |      243336     |                  |            |
|      2379        |                  |         |         |           |           |            |                 |                  |            |
| http://10.0.0.3: | 04a31cf28c38af8e |  3.5.3  |  35 MB  |     false |      5    |   243336   |      243336     |                  |            |
|      2379        |                  |         |         |           |           |            |                 |                  |            |
+------------------+------------------+---------+---------+-----------+-----------+------------+-----------------+------------------+------------+
```

2. 在主节点上操作压缩
```
ver=$(etcdctl --endpoints=http://10.0.0.1:2379 --command-timeout=300s endpoint status --write-out="json" | egrep -o '"revision":[0-9]*' | egrep -o '[0-9].*')
#确认获取的reversion, 获取出来的值 -10000 ，保留前10000个历史记录
echo $ver
etcdctl --endpoints=http://10.0.0.1:2379 --command-timeout=300s compact $(($ver-10000))
```

3. 碎片整理
```
# 清理第一个从节点（dbsize较大时，命令执行时间比较长，请耐心等待）, 确认集群状态
etcdctl  --command-timeout=300s defrag   --endpoints='http://10.0.0.2:2379'
etcdctl --endpoints=http://10.0.0.1:2379,http://10.0.0.2:2379,http://10.0.0.3:2379 endpoint status -w table
etcdctl --endpoints=http://10.0.0.1:2379,http://10.0.0.2:2379,http://10.0.0.3:2379 endpoint health -w table

# 清理第二个从节点, 确认集群状态
etcdctl  --command-timeout=300s defrag   --endpoints='http://10.0.0.3:2379'
etcdctl --endpoints=http://10.0.0.1:2379,http://10.0.0.2:2379,http://10.0.0.3:2379 endpoint status -w table
etcdctl --endpoints=http://10.0.0.1:2379,http://10.0.0.2:2379,http://10.0.0.3:2379 endpoint health -w table

# 清理主节点, 确认集群状态
etcdctl  --command-timeout=300s defrag   --endpoints='http://10.0.0.1:2379'
etcdctl --endpoints=http://10.0.0.1:2379,http://10.0.0.2:2379,http://10.0.0.3:2379 endpoint status -w table
etcdctl --endpoints=http://10.0.0.1:2379,http://10.0.0.2:2379,http://10.0.0.3:2379 endpoint health -w table
```

### 开启定期压缩

注意etcd配置生效的优先级
1. 配置文件最高优，并且如果传入了配置文件则环境变量和命令行参数都会失效
2. 命令行参数的生效优先级高于环境变量

具体配置参考[官方文档](https://etcd.io/docs/v3.5/op-guide/configuration/)和这篇[腾讯云博客](https://cloud.tencent.com/developer/article/1917040)
```
ETCD_AUTO_COMPACTION_RETENTION=periodic
ETCD_AUTO_COMPACTION_RETENTION=12
```


## 2. etcd 使用鉴权访问的性能风险

### 运维踩坑案例

公司使用了apisix作为内网服务代理，apisix的路由数据是保存在etcd中的，在压测场景下，apisix会启动多个线程频繁读取etcd。如果使用了密码鉴权认证的方式，密码认证操作会成为性能瓶颈。etcd的表现为频繁的心跳超时，发生切主操作，导致集群状态不可用。针对该种情况，加大心跳超时时间到1s，选举超时到5s可以缓解。

### 密码鉴权
密码鉴权流程可参考下图，详细可参考[etcd鉴权](https://time.geekbang.org/column/article/338524)
![密码鉴权流程](https://static001.geekbang.org/resource/image/8d/8a/8d18f8877ea7c8fbyybebae236a8688a.png?wh=1920*1125)

### 证书鉴权
证书认证在稳定性、性能上都优于密码认证。稳定性上，它不存在 Token 过期、使用更加方便、会让你少踩坑，避免了不少 Token 失效而触发的 Bug。性能上，证书认证无需像密码认证一样调用昂贵的密码认证操作 (Authenticate 请求)，
，前者更安全但是维护成本稍高需要定期替换，适合外网访问；后者使用简便适合内网，但是鉴权开销稍大，

## 参考资料

[官方文档](https://etcd.io/docs/v3.5/op-guide/configuration/)
[腾讯云博客](https://cloud.tencent.com/developer/article/1917040)
[etcd压缩](https://time.geekbang.org/column/article/342891)
[etcd鉴权](https://time.geekbang.org/column/article/338524)