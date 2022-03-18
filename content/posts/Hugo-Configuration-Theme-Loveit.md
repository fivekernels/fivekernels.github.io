---
title: Hugo 主题配置——loveit
date: 2022-01-25T19:06:24+08:00
tags:
- Hugo
categories:
- Hugo
draft: true
---

## 站点配置

&emsp;&emsp;从[Hugo主题网站上](https://themes.gohugo.io/)找一个喜欢的主题，使用git同步到本地代码仓库的themes文件夹。下面挑了两个比较喜欢的：

```bash
git submodule add git@github.com:dillonzq/LoveIt.git themes/LoveIt
git submodule add git@github.com:google/docsy.git themes/docsy
```

&emsp;&emsp;将主题信息写入hugo配置文件config.toml，在config.toml添加一行：

```toml
theme = "LoveIt"
```

&emsp;&emsp;同时可以配置一下其他信息，某些主题会有特定的规范。以[LoveIt主题](https://hugoloveit.com/zh-cn/theme-documentation-basics/#basic-configuration)为例

```toml
baseURL = "http://example.org/"

defaultContentLanguage = "zh-cn" # 默认语言 [en, zh-cn, fr, ...] 
languageCode = "zh-CN" # 网站语言, 仅在这里 CN 大写
hasCJKLanguage = true # 是否包括中日韩文字
title = "我的全新 Hugo 网站" # 网站标题

# 更改使用 Hugo 构建网站时使用的默认主题
theme = "LoveIt"

[params]
  # LoveIt 主题版本
  version = "0.2.X"

[menu]
  [[menu.main]]
    identifier = "posts"
    # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
    pre = ""
    # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
    post = ""
    name = "文章"
    url = "/posts/"
    # 当你将鼠标悬停在此菜单链接上时, 将显示的标题
    title = ""
    weight = 1
  [[menu.main]]
    identifier = "tags"
    pre = ""
    post = ""
    name = "标签"
    url = "/tags/"
    title = ""
    weight = 2
  [[menu.main]]
    identifier = "categories"
    pre = ""
    post = ""
    name = "分类"
    url = "/categories/"
    title = ""
    weight = 3

# Hugo 解析文档的配置
[markup]
  # 语法高亮设置 (https://gohugo.io/content-management/syntax-highlighting)
  [markup.highlight]
    # false 是必要的设置 (https://github.com/dillonzq/LoveIt/issues/158)
    noClasses = false
```

&emsp;&emsp;待补充...
