---
title: Windows10(2009)工作环境配置
date: 2022-10-06 22:35:52
tags: ["Windows", "WSL"]
---

最近工作电脑换成了`Windows 10`，也就重新开始了我的Windows折腾之旅。回顾下当年的操作系统折腾时间线

```
win98 --> winxp --> win vista（第一台自己的电脑） --> win7 --> win8 & windows phone 8 --> win10 & win10mobile --> win11 (Steam游戏机) --> win10(2009版本，工作使用)
                --> Ubuntu --> Debian --> Linux Mint --> Deepin --> Arch --> LFS --> Manjaro --> kali --> Arch / Debian / Ubuntu (个人服务器)
                                                                                             --> macOS10 --> macOS11 --> macOS12 (日常使用)
```

## Terminal 终端
> 因为域管控的原因，工作电脑无法升级系统版本，所以以下的操作都是基于`win10(2009)`版本

### scoop
官方网址：https://scoop.sh/

```powershell
> Set-ExecutionPolicy RemoteSigned -Scope CurrentUser # Optional: Needed to run a remote script the first time
> irm get.scoop.sh | iex
```
### Windows Terminal
官方网址：https://github.com/microsoft/terminal

推荐先试用scoop安装winget，再使用winget安装Windows Terminal。Windows生态下另外一款包管理器[choco](https://chocolatey.org/)和公司安装的麦咖啡冲突导致蓝屏，所以略过。

#### scoop install winget
高版本的Windows10和Windows11自带winget，对于没有集成winget的版本，推荐使用scoop安装winget，这也是目前找到的唯一可行不升级操作系统安装winget的方式。

#### winget install 
```
# 安装winget
scoop bucket add main
scoop install winget
# 安装c++库
scoop bucket add extras
scoop install vcredist
```

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

推荐使用Fira Code https://github.com/tonsky/FiraCode

### Visual Studio Code
```
winget install --id=Microsoft.VisualStudioCode -e
```


### PowerShell配置
#### Shell工具安装(awk, grep, vim, git)
以下4个工具建议使用scoop安装，scoop安装的软件大多为portable模式，方便管理
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

# git
scoop bucket add main
scoop install git
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
1. WSL的安装可以参考之前的博文 [Windows上的shell环境折腾笔记](https://changediff.github.io/2022/08/25/notes-on-wsl/)
2. 同时附上官方教程：https://learn.microsoft.com/en-us/windows/wsl/install-manual

## 快捷键 PowerToys

官方网址：https://github.com/microsoft/PowerToys

从macOS切到Windows最不习惯的是快捷键，如果说是只使用一个操作系统，那可以重新记忆习惯，然而我是macOS和Windows切换使用，就有快捷键打架的问题。

最近新发现的`PowerToys`的能够


推荐使用winget安装，方便后续更新
```
winget install Microsoft.PowerToys -s winget
```

### 参考键盘配置

![powertoy1](powertoy1.png)
![powertoy2](powertoy2.png)
![powertoy3](powertoy3.png)