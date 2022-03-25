---
title: 使用hexo搭博客
tags: hexo
date: 2022-03-26 01:32:18
---

hexo的[官网文档](https://hexo.io/zh-cn/docs/setup)很详细，这里主要记载使用`hexo+Github Pages`过程中遇到的坑

1. GitHub项目创建可参考[《超详细Hexo+Github博客搭建小白教程》](https://zhuanlan.zhihu.com/p/35668237)
2. 如果是按照步骤1新建的项目，`_config.yml`中的`url`只需要填写网址即可，不用加上project
```yaml
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: https://changediff.github.io
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```