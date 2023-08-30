---
title: notes
date: 2022-03-28 17:37:28
---

> 好记性不如烂笔头
## HTTPS

- 根证书：https://www.cnblogs.com/429512065qhq/p/12581941.html
根证书是信任链的起点，CA(Certificate Authority)的公钥预埋在主流浏览器中，用来验证服务器证书是否合法
- 双向认证：https://help.aliyun.com/zh/api-gateway/user-guide/mutual-tls-authentication#title-3j6-y2z-1od
双向认证中，企业使用自签证书的原因之一是 client 端证书更新成本高，使用自签名证书可以将证书过期时间设置更长（例如 50 年），从而避免了频繁的客户端证书更新操作
- SSL 和 TLS: https://aws.amazon.com/cn/compare/the-difference-between-ssl-and-tls/
Taher Elgamal 领导了 SSL 的开发，并于 1995 年公开发布了 SSL 2.0。SSL 旨在确保万维网上的通信安全。在 SSL 经历多次迭代后，Tim Dierks 和 Christopher Allen 于 1999 年创建了 TLS 1.0，作为 SSL 3.0 的后继者。 
目前，所有 SSL 证书均已停用。TLS 证书是行业标准。但是，业界仍使用术语 SSL 来指代 TLS 证书。
TLS 证书在 SSL 证书基础上进行了迭代，并随着时间的推移对其进行了改进。SSL 证书和 TLS 证书的最终功能没有改变。 
- TCP over TLS
HTTPS is HTTP on top of TLS on top of TCP

##  负载均衡
- DR 模式不支持服务器同时作为客户端和服务端，因为 DR 模式会做 DNAT 转换，转换后的 ip 包源 ip 和目的 ip 都是本机 ip，包会被丢弃
- FULLNAT 模式，同时 RS 开启 `net.ipv4.tcp_tw_recycle=1` 和 `net.ipv4.tcp_timestamps=1` 会导致转发流量随机丢包，4.12 版本的内核中已经去移除了 `net.ipv4.tcp_tw_recycle` ，具体见[链接](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=4396e46187ca5070219b81773c4e65088dac50cc)

## Linux常用命令

### 查看文件描述符未释放的文件

```
lsof +L1 | sort -nk7
```

## macOS使用笔记

### 常见使用问题

#### macOS时间不同步问题解决
```
sudo sntp -sS time.apple.com
```

#### 触发角失效
```
defaults write com.apple.dock mcx-expose-disabled -bool FALSE; killall Dock
```

#### macOS版本升级
macOS有时会更新失败，可通过下载最新版本macOS的全量安装包来完成更新
https://support.apple.com/zh-cn/macos/upgrade

更新后可能需要重新安装developer tools
```
xcode-select --install
```

### 好用的软件
#### homebrew安装
```
iterm2
htop
karabiner-elements #键盘改键位
neofetch #screenfetch的跨平台版本
visual-studio-code
microsoft-remote-desktop
vlc
feishu
clipy
anaconda
pyenv
multipass #ubuntu虚拟机
wiznote
workflowy
vmware-fusion-tech-preview
racket --cask
md5sha1sum
bartender
mos #鼠标滚轮优化
balenaetcher #iso刻录到u盘
```

#### App Store安装
```
RunCat #状态栏负载监控
Hidden Bar #状态栏图标隐藏 
Xnip #截图
Magnet #窗口手势
MenuMate #顶栏菜单改为右键
Keka #压缩
```

#### 其他
```
Downie #在线视频下载
```

##### flash player

360极速浏览器是目前macOS下唯一可用的支持flash的浏览器
https://browser.360.cn/ee/mac/index.html

## Windows
```
CHKenPlayer #曾经最小的播放器
foobar2000
Ventory
```

## 搜索引擎
### Google搜索设置
1. 将语言改为English，这样就可以通过搜索关键字来区分中英文互联网内容