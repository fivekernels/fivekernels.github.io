---
title: "Hugo 主题配置——Papermod"
date: 2022-01-25T19:12:11+08:00
tags: ["Hugo"]
draft: true
---

## Hugo 主题配置——Papermod

官方链接：

github: <https://github.com/adityatelange/hugo-PaperMod>

github wiki:<https://github.com/adityatelange/hugo-PaperMod/wiki>

demo code: <https://github.com/adityatelange/hugo-PaperMod/tree/exampleSite/>

demo: <https://adityatelange.github.io/hugo-PaperMod//>

&emsp;&emsp;Papermod 主题默认使用 .yaml 格式作为配置文件，因此新建网站时可以使用 -f 设置 yml 格式：

```bash
hugo new site <name of site> -f yml
```

&emsp;&emsp;将主题添加到网站文件夹中：

```bash
git submodule add git@github.com:adityatelange/hugo-PaperMod.git themes/PaperMod --depth=1
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```

## 导航菜单与页面配置

## 文章基础配置

## 首页样式配置

