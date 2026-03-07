---
title: Debian SSH 配置密钥 + MFA 双重认证指南
date: 2026-03-07 21:45:00
categories:
  - 运维
tags:
  - SSH
  - 安全
  - Debian
  - MFA
---

以下是在 Debian 上为 SSH 配置密钥 + MFA（Google Authenticator）双重认证的正确完整步骤，避免之前遇到的密码提示问题。

## 📋 步骤概览

1. 安装必要软件
2. 为用户生成 MFA 密钥
3. 配置 PAM（禁用密码，启用 MFA 模块）
4. 修改 SSH 服务端配置（启用 keyboard-interactive、设置认证方法）
5. 检查语法并重载 SSH 服务
6. 测试新登录流程
7. 可选：防火墙、Fail2ban 等加固

## 🔧 详细操作

### 1. 安装 Google Authenticator PAM 模块

```bash
sudo apt update
sudo apt install libpam-google-authenticator
```

### 2. 为您的普通用户生成 MFA 密钥

**切换到要配置的用户**（不要用 root）：

```bash
su - your_username
google-authenticator
```

按照交互提示全部选择 `y`（允许更新配置文件、禁止令牌重用、启用速率限制等）。

**务必保存好显示的应急恢复码**（每个码仅能使用一次）。

### 3. 配置 PAM

编辑 PAM 配置文件：

```bash
sudo nano /etc/pam.d/sshd
```

- 注释掉 `@include common-auth` 这一行（禁用密码验证）
- 在文件**开头**添加一行：

```bash
auth required pam_google_authenticator.so
```

保存并退出。

### 4. 修改 SSH 服务端配置

编辑 `/etc/ssh/sshd_config`：

```bash
sudo nano /etc/ssh/sshd_config
```

确保以下设置存在且正确：

```ssh-config
# 修改默认端口（建议高位端口，例如 23456）
Port 23456

# 禁止 root 直接登录
PermitRootLogin no

# 启用公钥认证（必须）
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# 启用键盘交互认证（用于 MFA 验证码输入）
KbdInteractiveAuthentication yes

# 兼容旧版 OpenSSH（可选）
ChallengeResponseAuthentication yes

# 必须使用 PAM
UsePAM yes

# 认证方法顺序：先公钥，后 keyboard-interactive（即 MFA 验证码）
AuthenticationMethods publickey,keyboard-interactive

# 禁用密码认证
PasswordAuthentication no

# 可选：限制登录用户
AllowUsers your_username

# 可选：安全加固
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 0
```

### 5. 检查语法并重新加载服务

```bash
sudo sshd -t  # 无输出表示语法正确
sudo systemctl reload ssh  # 平滑重载
```

### 6. 测试新登录流程

**保持当前终端窗口不关闭**，另开一个终端测试登录：

```bash
ssh -p 23456 your_username@your_server_ip
```

- 若您已配置 SSH 密钥，第一步会自动验证密钥。
- 密钥验证通过后，会提示 `Verification code:`，输入手机 Google Authenticator 的 6 位动态码。
- 输入正确后登录成功。

**如果仍然提示密码**：
- 检查 `/etc/ssh/sshd_config` 中 `KbdInteractiveAuthentication` 是否为 `yes`。
- 检查 `/etc/pam.d/sshd` 中是否已注释 `@include common-auth`。
- 查看日志：`sudo journalctl -u ssh -f`

### 7. 确认无误后彻底禁用密码认证

```bash
PasswordAuthentication no
sudo systemctl reload ssh
```

### 8. 可选：防火墙与 Fail2ban 加固

```bash
# 开放新 SSH 端口
sudo ufw allow 23456/tcp
sudo ufw enable

# 安装 Fail2ban
sudo apt install fail2ban
sudo systemctl enable fail2ban --now
```

## ⚠️ 重要注意事项

- **始终保留一个已登录的终端会话**，以便在配置错误时回滚。
- **应急恢复码**请妥善保存，丢失后若手机不可用将无法登录。
- 用户家目录的 `.ssh` 权限应为 `700`，`authorized_keys` 权限应为 `600`。
- 若您的 OpenSSH 版本低于 8.7（如 Debian 10），请使用 `ChallengeResponseAuthentication yes` 代替 `KbdInteractiveAuthentication`。

按照以上步骤，您将获得一个**密钥 + 动态验证码双重保护**的安全 SSH 环境，有效抵御密码泄露和暴力破解。
