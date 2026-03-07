---
title: Windows10(2009)工作环境配置
date: 2022-10-06 23:59:59
categories:
  - 效率
tags: ["Windows", "WSL"]
---

最近工作电脑换成了`Windows 10`，也就重新开始了我的Windows折腾之旅。回顾下当年的操作系统折腾时间线

```
win98 --> winxp --> win vista（第一台自己的电脑） --> win7 --> win8 & windows phone 8 --> win10 & win10mobile --> win11 (Steam游戏机) --> win10(2009版本，工作使用)
                --> Ubuntu --> Debian --> Linux Mint --> Deepin --> Arch --> LFS --> Manjaro --> kali --> Arch / Debian / Ubuntu (个人服务器)
                                                                                             --> macOS10 --> macOS11 --> macOS12 (日常使用)
```

> 因为域管控的原因，工作电脑无法升级系统版本，所以以下的操作都是基于`win10(2009)`版本

## 包管理器

工欲善其事必先利其器，首先得有趁手的包管理器

### scoop
官方网址：https://scoop.sh/

受macOS上的homebrew启发的包管理，使用体验相近

```powershell
# 本体安装
> Set-ExecutionPolicy RemoteSigned -Scope CurrentUser # Optional: Needed to run a remote script the first time
> irm get.scoop.sh | iex
# 依赖安装
scoop install git
```

### winget
官方网址：https://github.com/microsoft/terminal

推荐使用scoop安装winget，然后根据实际需求选择scoop或者winget安装其他软件。Windows生态下另外一款包管理器[choco](https://chocolatey.org/)和公司安装的麦咖啡冲突导致蓝屏，此处略过。

#### scoop install winget
高版本的Windows10和Windows11自带winget，对于没有集成winget的版本，推荐使用scoop安装winget，这也是目前找到的唯一可行不升级操作系统安装winget的方式。

```
# 安装winget
scoop bucket add main
scoop install winget
# 安装c++库
scoop bucket add extras
scoop install vcredist
```

## Terminal 终端

### cmder
官方网址：https://cmder.app/

推荐使用scoop安装
```
scoop bucket add main
scoop install cmder-full
```

### Windows Terminal
官方网址：https://github.com/microsoft/terminal

推荐使用winget安装，方便后续更新
```
winget install --id=Microsoft.WindowsTerminal -e
```

#### 终端字体安装

官方网址：https://github.com/tonsky/FiraCode

```
scoop bucket add nerd-fonts
scoop install FiraCode
```

### Visual Studio Code
```
winget install --id=Microsoft.VisualStudioCode -e
```


### PowerShell配置
#### Shell工具安装(awk, grep, vim)
这些工具建议使用scoop安装，scoop安装的软件大多为portable模式，方便管理
```
# awk
scoop bucket add main
scoop install gawk

# grep
scoop bucket add main
scoop install grep

# vim
scoop bucket add main
scoop install vim
```

#### 安装新版本PowerShell
```
winget install Microsoft.PowerShell
```

#### PowerShell profile配置
`PowerShell`的`$profile`，相当于Linux的`.bashrc`
```powershell
## 参考博客
# https://blog.batkiz.com/posts/2020/some-pwsh-scripts/
# https://blog.batkiz.com/posts/2020/some-pwsh-scripts-2/

function nali {
    param (
        $Query = '',
        [Alias('l')]
        $Lang = 'zh'
    )

    if ($Lang.ToLower() -eq 'en' ) {
        $Lang = 'en'
    }
    else {
        $Lang = 'zh-CN'
    }

    $ApiUrl = "http://ip-api.com/json/{0}?lang={1}" -f $Query, $Lang

    $info = (Invoke-WebRequest $ApiUrl).Content | ConvertFrom-Json

    $printInfo = "{0}`t[{1} @ {2}, {3}]" -f $info.query, $info.isp, $info.city, $info.country

    $printInfo
}

function which {
    $results = New-Object System.Collections.Generic.List[System.Object];
    foreach ($command in $args) {
        $path = (Get-Command $command).Source
        if ($path) {
            $results.Add($path);
        }
    }
    return $results;
}

function time {
    $Command = "$args"

    $time = Measure-Command { Invoke-Expression $Command 2>&1 | out-default }

    $info = "{0:d2}:{1:d2}:{2:d2}.{3}" -f $time.Hours, $time.Minutes, $time.Seconds, $time.Milliseconds

    Write-Output $info
}

function Get-Size {
    param([string]$pth)
    "{0:n2}" -f ((Get-ChildItem -path $pth -recurse | measure-object -property length -sum).sum / 1mb) + " M"
}
```

## WSL(Windows Subsystem for Linux)

### WSL安装
1. WSL的安装可以参考之前的博文 [Windows上的shell环境折腾笔记](https://changediff.github.io/2022/08/25/notes-on-wsl/)
2. 同时附上官方教程：https://learn.microsoft.com/en-us/windows/wsl/install-manual

### 发行版选择
#### 图形界面kali
需要图形界面推荐`kali-linux`，图形界面配置参考：https://www.kali.org/docs/wsl/

```
> winget search kali
名称                ID                       版本    匹配      源
