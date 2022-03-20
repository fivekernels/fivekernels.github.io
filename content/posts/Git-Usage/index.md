---
title: Git Usage
date: 2022-03-20T14:34:46+08:00
lastmod: 2022-03-20T16:17:46+08:00
tags: 
- Git
categories: 
- Git
draft: true
---

## 清理提交历史中的敏感信息和大文件

参考：

&emsp;&emsp;[知乎 [彼得三三]：GitHub清除敏感信息操作](https://zhuanlan.zhihu.com/p/297695945)

&emsp;&emsp;[[dotnet 职业技术学院]：清理 git 仓库太繁琐？试试 bfg！删除敏感信息删除大文件一句命令搞定（比官方文档还详细的使用说明）](https://dotnet-campus.github.io/post/clean-up-git-repo-using-bfg.html)

&emsp;&emsp;[简书 [不忘初心2017]：BFG Repo-Cleaner - 从 Git 历史中真正删除文件](https://www.jianshu.com/p/a9caf4b3100e)

使用[BFG Repo-Cleaner by rtyley](https://rtyley.github.io/bfg-repo-cleaner/)，需要java运行环境。

- BFG 默认不会 touch最新的commit，即如果你要删除的文件在最新的 commit 中，则不会删除之。为什么？因为最新的 commit 很可能是已上线在产品中的~
- BFG 用的是仓库镜像 git clone --mirror。
- 如果要删除的文件在受保护的commit中，BFG 不会删，不建议删。至于commit什么情况下会受保护，我还不太清楚。

&emsp;&emsp;首先将待清理的仓库克隆到本地，记得clone后备份一份xxx.git到其他位置，以免操作失误。

```bash
git clone --mirror git://example.com/xxx.git
```

&emsp;&emsp;将下载的bfg.jar与xxx.git放到同一个同目录。

### 清理敏感信息

&emsp;&emsp;准备一个文本文件passwords.txt，名称任意，里面包含要要替换的文本：

```txt
PASSWORD1                       # Replace literal string 'PASSWORD1' with '***REMOVED***' (default)
PASSWORD2==>examplePass         # replace with 'examplePass' instead
PASSWORD3==>                    # replace with the empty string
regex:password=\w+==>password=  # Replace, using a regex
regex:\r(\n)==>$1               # Replace Windows newlines with Unix newlines
```

&emsp;&emsp;执行敏感信息替换：

```bash
java -jar bfg.jar --replace-text passwords.txt xxx.git
```

&emsp;&emsp;回收已经没有引用的旧提交，这可以减小本地仓库的大小：

```bash
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

&emsp;&emsp;推回远端仓库：

```bash
git push
```

&emsp;&emsp;更新本地代码推荐删除原有工程，重新clone仓库后配置工程，注意原有工程下.gitignore的重要文件（敏感信息等）。

### 清理大文件

```bash
java -jar bfg.jar --strip-blobs-bigger-than 500M xxx.git # 将仓库历史中大于 500M 的文件都删除掉
java -jar bfg.jar --delete-files walterlv.snk xxx.git # 删除特定文件 walterlv.snk
java -jar bfg.jar --delete-files {walterlv,lindexi}.snk xxx.git # 删除 walterlv.snk 或 lindexi.snk 文件
java -jar bfg.jar --delete-folders walterlv xxx.git # 删除名字为 walterlv 的文件夹
```

&emsp;&emsp;回收已经没有引用的旧提交，这可以减小本地仓库的大小：

```bash
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

&emsp;&emsp;推回远端仓库：

```bash
git push
```

&emsp;&emsp;更新本地代码推荐删除原有工程，重新clone仓库后配置工程，注意原有工程下.gitignore的重要文件（敏感信息等）。

## 子模块submodule的使用

参考：

&emsp;&emsp;[CSDN [guotianqing]：git中submodule子模块的添加、使用和删除](https://blog.csdn.net/guotianqing/article/details/82391665)

&emsp;&emsp;[简书 [thinker_g]：[Git] 如何优雅的删除子模块(submodule)或修改Submodule URL](https://www.jianshu.com/p/ed0cb6c75e25)

### 添加

```bash
git submodule add <url> <path>
git commit # 提交？
```

其中，url为子模块的路径，path为该子模块存储的目录路径。

### 修改模块url

1. 修改'.gitmodules'文件中对应模块的”url“属性;
1. 使用git submodule sync命令，将新的URL更新到文件.git/config；
1. git commit -am "Update submodule url." # 提交变更

### 删除

```bash
git submodule deinit <MOD_NAME> # 逆初始化模块，其中{MOD_NAME}为模块目录，执行后可发现模块目录被清空
git rm --cached <MOD_NAME> # 删除.gitmodules中记录的模块信息（--cached选项清除.git/modules中的缓存）
# rm -rf <子模块目录> # 删除子模块目录及源码
# vi .gitmodules # 删除项目目录下.gitmodules文件中子模块相关条目
vi .git/config # 删除配置项中子模块相关条目
# rm .git/module/* # 删除模块下的子模块目录，每个子模块对应一个目录，注意只删除对应的子模块目录即可
git commit -am "Remove a submodule." 
```
