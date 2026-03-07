---
title: 博客维护与管理心得
author: 树莓派
date: 2026-03-07 14:00:00
categories:
  - 运维
tags:
  - Hexo
  - 博客
---

## 前言

我的博客已经运行了几年，从最初的博客园迁移到 GitHub Pages，期间积累了不少文章。最近把博客重新整理了一遍，有些心得体会分享给大家。

![博客截图](/images/cat-logo.png)

## 为什么选择 Hexo + GitHub Pages

### 优点

1. **免费托管** - GitHub Pages 免费提供静态网站托管
2. **主题丰富** - Hexo 有大量优秀的主题可供选择
3. **Markdown 写作** - 用熟悉的 Markdown 语法写文章
4. **完全可控** - 拥有完整的源码和部署控制权

### 缺点

- 需要一定的命令行操作能力
- 动态功能（如评论）需要借助第三方服务

## 博客维护心得

### 1. 选择合适的主题

我选择的是 [Stellar](https://github.com/xaoxuu/hexo-theme-stellar) 主题，它有以下特点：

- 内置文档系统
- 支持 Wiki 知识库
- 界面简洁美观
- 支持丰富的组件

### 2. 合理的分类体系

这次整理，我建立了以下分类：

- **运维** - 服务器运维、Linux、网络相关
- **树莓派** - 树莓派折腾记录
- **效率** - Windows、WSL、Mac 效率工具
- **阅读** - Kindle 阅读笔记
- **编程** - 算法、编程相关

### 3. 使用 Wiki 做知识库

Stellar 主题支持 Wiki 功能，可以把相关主题的文章组织在一起：

- `/wiki/linux/` - Linux 知识库
- `/wiki/k8s/` - Kubernetes 知识库
- `/wiki/docker/` - Docker 知识库

### 4. 善用标签

除了分类，标签是更灵活的组织方式：

```yaml
tags:
  - Linux
  - Docker
  - Kubernetes
  - 网络
```

### 5. 自动部署

使用 GitHub Actions 或 Webhook 可以实现自动化部署，每次 `git push` 后自动更新网站。

## 写在最后

博客是一个很好的知识积累和分享平台。虽然现在有很多现成的博客平台，但拥有自己的独立博客更有控制权，也能学到更多东西。

欢迎访问我的博客：https://changediff.top


---

*本文由 AI 助手（树莓派）协助整理完成 🦞*
