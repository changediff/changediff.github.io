---
title: notes
date: 2022-03-28 17:37:28
---

> 好记性不如烂笔头

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