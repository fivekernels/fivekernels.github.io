---
title: Python Usage
date: 2022-03-21T20:15:47+08:00
lastmod: 2022-03-21T20:15:47+08:00
tags: 
- Python
- pip
categories: 
- Python
draft: false
---

## pip换源

```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple/ # 清华源
pip config list # 查看当前源
```

## 使用virtualenvwrapper创建虚拟环境

参考：  
&emsp;&emsp;[virtualenvwrapper官方文档](https://virtualenvwrapper.readthedocs.io/en/latest/#)  
&emsp;&emsp;[CSDN [团子大圆帅]：Python 虚拟环境管理工具介绍（virtualenv、virtualenvwrapper、pipenv）](https://blog.csdn.net/jpch89/article/details/89478614)  
&emsp;&emsp;[CSDN [zhicai_liu]：【virtualenvwrapper】重命名虚拟环境](https://blog.csdn.net/zhicai_liu/article/details/110518521)

```bash
pip install virtualenvwrapper-win # 安装virtualenvwrapper，会自动安装virtualenv
```

&emsp;&emsp;安装好后默认创建的虚拟环境会位于"C:\Users\<username>\envs\\"目录下;可以在环境变量-系统变量中创建WORKON_HOME，并将其值设定为需要存放的路径(如：D:\envs)以实现修改。

&emsp;&emsp;常用命令：

```bash
lsvirtualenv # 查看虚拟环境
workon       # 查看虚拟环境
mkvirtualenv <venv_name>                      # 创建名为venv_name的虚拟环境，默认python解释器继承当前环境
mkvirtualenv <venv_name> --python=<解释器路径> # 创建名为venv_name的虚拟环境，指定python解释器版本
workon <venv_name>       # 激活虚拟环境
deactivate               # 退出当前虚拟环境
rmvirtualenv <venv_name> # 删除虚拟环境
cdvirtualenv <venv_name> # 进入虚拟环境所在目录

cpvirtualenv <old_venv_name> <new_env_name> # 重命名虚拟环境(1/2)
rmvirtualenv <old_venv_name>                # 重命名虚拟环境(2/2)
```

## 使用miniconda创建虚拟环境

参考：[CSDN [有马白颠]：conda、miniconda 、anaconda、 virtualenv的区别与miniconda的安装配置](https://blog.csdn.net/qq_36782182/article/details/111922349)

&emsp;&emsp;conda虚拟环境是独立于操作系统解释器环境的，即无论操作系统解释器什么版本（哪怕2.7），我也可以指定虚拟环境python版本为3.6，而venv是依赖主环境的。  
&emsp;&emsp;安装miniconda时，勾选“为所有用户安装”。

### conda换源

参考：  
&emsp;&emsp;[CSDN [哈！小白要成长！]：miniconda 换源（添加镜像）](https://blog.csdn.net/qq_41956139/article/details/122748299)  
&emsp;&emsp;[简书 [谢小帅]：Anaconda 删除自己配置的镜像源](https://www.jianshu.com/p/39819bcb889f)

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2/
conda config --set show_channel_urls yes # 设置安装时显示源地址, 方便安装时知道包来自哪个源
```

查看添加结果：

```bash
conda info
```

删除指定源：

```bash
conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```

换回默认源：

```bash
conda config --remove-key channels
```

### 虚拟环境管理

&emsp;&emsp;conda create -n默认将虚拟环境创建在"$HOME/.conda/envs/env_name"目录下

```bash
conda env list                            # 列出所有环境
conda create --name <env_name> python=3.6 # 创建虚拟环境
conda create -n <env_name> python=3.6     # 创建虚拟环境
conda activate <env_name>                 # 激活虚拟环境
conda deactivat                           # 表示退出当前环境
conda remove -n <env_name> --all          # 删除虚拟环境
```

```bash
conda create -n <new_env_name> --clone <old_env_name> # 重命名(1/2) 复制
conda remove -n <old_env_name> --all                  # 重命名(2/2) 删除旧环境
```

&emsp;&emsp;此外，可以通过\-p \<path>或\-\-prefix \<path>的方式在指定路径下创建虚拟环境，但此时不允许使用-n \<env_name>的方式指定名称：

```bash
conda create  -p </path/to/environment_dir> python=3.6       # 在指定路径下创建虚拟环境
conda create  --prefix </path/to/environment_dir> python=3.6 # 在指定路径下创建虚拟环境
conda activate </path/to/environment-dir>                    # 激活指定路径下的虚拟环境
conda remove --prefix=</path/to/environment_dir> --all       # 删除指定路径下的虚拟环境
```

&emsp;&emsp;虚拟环境草导出与导入：

```bash
conda env export > environment.yaml  # 生成一个包含环境名称、环境源、环境依赖包、通过pip安装的包的配置文件
conda env create -f environment.yaml # 使用配置文件创建虚拟环境
```

&emsp;&emsp;清理conda缓存：

```bash
conda clean -p    # 删除没有用的包 --packages
conda clean -t    # 删除tar打包 --tarballs
conda clean --all # 删除所有的安装包及cache(索引缓存、锁定文件、未使用过的包和tar包)
```

### conda仓库简介

参考：[霍小强的博客：conda、miniconda、anaconda、仓库的详解](https://www.huoxiaoqiang.com/experience/pythone/6033.html)

&emsp;&emsp;repo.anaconda.com/pkgs/是 由Anaconda®公司自己构建的包的公开仓库，您仅仅具有包的使用权。此仓库仅可用于个人、教育机构学习用，如果用于商用需付费购买商业产品。

**main channel**  
&emsp;&emsp;大多数由 Anaconda, Inc. 使用新编译器堆栈构建的包都托管在这里。此频道作为最高优先级别频道已包含在 conda 的默认频道中。

**free channel**  
&emsp;&emsp;在没有新编译器堆栈的情况下构建的包。这些软件包中的大多数与 pkgs/main 中的软件包兼容。已包含在 conda 的默认频道中。

**r channel**  
&emsp;&emsp;Microsoft R Open conda 包和 Anaconda, Inc. 的 R conda 包。已包含在 conda 的默认频道中。

**mro channel**  
&emsp;&emsp;这是一个空频道。此频道中的软件包已移至 pkgs/mro-archive。新的 MRO 软件包位于 pkgs/r 频道中。

**pro channel**  
&emsp;&emsp;现在已弃用——尽管仍包含在 conda 的默认频道中。最后一个软件包于 2017 年 2 月更新。包括 Anaconda, Inc. 的各个商业软件包，所有这些现在都是开源的。（MKL 优化、IOPro、加速）

**archive channel**  
&emsp;&emsp;有时，发布到其他频道之一的软件包会出现问题，迫使 Anaconda, Inc. 将其从频道中删除。在这些情况下，程序包会存档到此频道，供任何仍然需要它的人使用。

**mro-archive channel**  
&emsp;&emsp;pkgs/mro 频道的全部内容存档，Microsoft 的 MRO 和 MRAN 的过时版本。

**msys2 channel**  
&emsp;&emsp;仅限 Windows。已包含在 conda 的默认频道中。Anaconda, Inc. 的 R conda 软件包以及 pkgs/main 和 pkgs/free 中的其他一些软件包是必需的。它提供了 bash shell、Autotools、修订控制系统等，用于使用 MinGW-w64 工具链构建原生 Windows 应用程序。

## penv pyenv pipenv

*好像很厉害，待更新...*
