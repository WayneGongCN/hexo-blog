---
title: 使用 Anaconda
date: 2018-07-11
tags: python
categories: notes
---


[Wiki](https://zh.wikipedia.org/wiki/Anaconda_(Python%E5%8F%91%E8%A1%8C%E7%89%88)) 介绍 [Anaconda](https://anaconda.org/) 是一种 Python 的免费增值开源发行版，用于进行大规模数据处理, 预测分析, 和科学计算, 致力于简化包的管理和部署。

简单来说，Anaconda 提供了一个包含 Ptyhon 、conda 以及常用第三方包的环境。

## 功能

在 Anaconda 下，不同的项目使用独立的开发环境，不同的开发环境下使用不同的 Python 版本，并且包互相隔离。

## 安装

[官方](https://www.anaconda.com/download/) 、[镜像](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/) 下载安装。

## conda

[conda](https://conda.io/docs/) 的主要功能就是 **环境管理** 与 **包管理**。

像是结合了 [virtualenv](https://virtualenv.pypa.io/en/stable/) 与 [pip](https://pypi.org/project/pip/)。

或者用熟悉的 node.js 来打比方， conda 结合了环境管理([nvm](https://github.com/creationix/nvm)) 与 包管理器([npm](https://www.npmjs.com/)、[yarn](https://yarnpkg.com/lang/en/) ...)的功能。

不同的是 conda 将几乎所有的工具、第三方包都当做 package 对待，比如 VScode 是一个 package，不同版本的 Python 是一个 package, conda 本身也是一个 package。

### 管理 conda

Windows 下如果已经将 `%Anaconda%/Scripts` 添加到环境变量，可以直接在 Cmd 下使用，
没有添加的可以从开始菜单找到 `Anaconda Prompt` 启动开始使用。

初次使用可以尝试更新：

```shell
conda --version
conda update conda
# Proceed([y]/n)? y
```

### 管理环境

可以使用 `conda info --envs` 查看所有的环境，conda 会有一个默认环境 `root`， 其中带 `*` 号的为当前环境。

创建一个 `snowflakes` 环境并安装 BioPython包：

1. `conda create --name snowflakes biopython` conda 会检查 biopython 的依赖，并提示是否继续。
2. `activate snowflakes` 激活环境之后，就处于 `snowflakes` 之下了。
3. `deactivate` 回到默认环境 `root`。

linux/macOs 下应使用 `source activate snowflakes` 或 `source deactivate`。

新环境中的 Python 版本与安装 Anaconda 的 Python 版本相同，
可以使用 `conda create --name snakes python=3.5` 指定所需的 Python 版本。

### 管理包

查看当前环境下的包：`conda list`。

检查是否可以从 Anaconda 源中获取 package：`conda search scrapy`。

安装到当前环境：`conda install scrapy`。

## 更多信息

- [conda](https://conda.io/docs/)
- [anaconda](https://docs.anaconda.com/anaconda-cloud/user-guide/)
