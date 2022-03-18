---
title: "Hugo 文件管理"
tags: ["Hugo"]
categories: [ "Hugo" ]
date: 2021-10-28T11:10:22+08:00
draft: true
---

&emsp;&emsp;hey

## Hugo 目录结构

```txt
.(hugo-root)
|-- archetypes/ (内容模板文件)
|-- assets/     (not created by default)
|   |-- css/    (adding custom css to replace css in themes)
|   |   |-- *.*
|   |   `-- *.*
|   `-- ***/
|-- content/    (存放文章内容，通过"hugo new"创建的文件以此为根目录)
|   `-- posts/
|       `-- *.md
|-- data/       (存储网站用到一些配置、数据文件。文件类型可以是yaml|toml|json等格式，生成网站时使用)
|-- docs/       (github pages可以选用docs作为网站根目录，添加publishdir: 'docs'将网站生成在该目录下)
|-- layouts/    (渲染文章内容的模板文件，相较于themes/<theme-name>/layouts目录下的同名文件具有更高优先级)
|-- resources/
|-- static/     (存储图片、css、js等静态资源文件；md中引用图片的根目录位于此文件夹下)
|-- themes/     (存储不同主题，可以方便的切换网站的风格样式)
`-- config.yml  (或config.toml，config.json，网站默认配置文件)
```

tree --dirsfirst --charset=ascii /path/to/directory将生成一个很好的ASCII树，可以将其整合到文档中：

```txt
(example fire tree)
|   |-- de-DE
|   |   |-- art.mshc
|   |   |-- artnoloc.mshc
|   |   |-- clientserver.mshc
|   |   |-- noarm.mshc
|   |   |-- resources.mshc
|   |   `-- windowsclient.mshc
|   `-- en-US
|       |-- art.mshc
|       |-- artnoloc.mshc
|       |-- clientserver.mshc
|       |-- noarm.mshc
|       |-- resources.mshc
|       `-- windowsclient.mshc
`-- IndexStore
    |-- de-DE
    |   |-- art.mshi
    |   |-- artnoloc.mshi
    |   |-- clientserver.mshi
    |   |-- noarm.mshi
    |   |-- resources.mshi
    |   `-- windowsclient.mshi
    `-- en-US
        |-- art.mshi
        |-- artnoloc.mshi
        |-- clientserver.mshi
        |-- noarm.mshi
        |-- resources.mshi
        `-- windowsclient.mshi
```

## 静态资源（图片等）存放方式

...待更新...
