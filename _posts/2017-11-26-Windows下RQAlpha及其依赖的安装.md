---
layout:     post
title:      "Windows下RQAlpha及其依赖的安装"
date:       2017-11-26 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - RQALpha
---

| 主题     | 概要                                                                       |
| -------- | -------------------------------------------------------------------------- |
| 量化交易 | RQAlpha环境搭建                                                            |
| -------- | ---                                                                        |
| **编辑** | **时间**                                                                   |
| 新建     | 20171126                                                                   |
| 修改     | 添加anconda链接地址 20180624                                               |
| -------- | ---                                                                        |
| **序号** | **参考资料**                                                               |
| 1        | http://rqalpha.readthedocs.io/zh_CN/latest/intro/overview.html             |
| 2        | https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/                     |
| 3        | https://mrjbq7.github.io/ta-lib/install.html                               |
| 4        | https://estan.cn/2016/11/compiling-and-installation-details-of-ta-lib.html |
| 5        | http://rqalpha.readthedocs.io/zh_CN/latest/intro/install.html              |
| 6        | http://www.lfd.uci.edu/~gohlke/pythonlibs/#bcolz                           |


最近参考RQAlpha学习了下量化交易程序。RQAlpha 从数据获取、算法交易、回测引擎，实盘模拟，实盘交易到数据分析，为程序化交易者提供了全套解决方案（http://rqalpha.readthedocs.io/zh_CN/latest/intro/overview.html）。

搭建windows开发环境的过程是非常痛苦的过程，记录如下。

## 安装Anaconda ##
Anaconda 是一个用于科学计算的 Python 发行版，可以从清华源镜像站下载（https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/）。

当安装成功后，执行如下命令来查看是否安装成功:

```
conda -V
```
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQwOTU4Mjkx)

补充：
Anaconda3-4.4.0-Windows-x86_64下载地址：
https://pan.baidu.com/s/1o1D3y2yxZjHb2_pT_sU8Kw


## conda 虚拟环境 ##
新创建一个Python虚拟环境，使开发环境更加独立，不会因为安装包的不同而出现问题。

```
conda create --name stock python=3.6
```
其中，--name可以自己指定一个。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQxMDMzNzMz)

新建后，切换到虚拟环境：

```
activate stock
```
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQxMTMxNTAx)

## 安装 TA-Lib ##
假设已经切换到了虚拟环境，通常直接通过PIP是无法成功安装的。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQxMjE3NjM3)

提示没有找到模块numpy，则先安装numpy。

```
pip install numpy
```

安装成功后，再用pip 安装一次ta-lib。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQxMzI5MTEx)

还是build失败。

参考官方资料（https://mrjbq7.github.io/ta-lib/install.html），直接check out源码，通过命令

```
python setup.py install
```
安装。

假设源码下载后的路径为:

```
E:\Downloads\firefox\ta-lib-TA_Lib-0.4.10\ta-lib-TA_Lib-0.4.10
```

CD到该目录，并运行

```
python setup.py install
```
会出现无法打开包括文件: “ta_libc.h”错误提示。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQxNTUwNjkw)

是因为windows机子上没有ta-lib的C语言库。

参考官网上的资料：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQxNzE0OTEz)

把ta-lib下载并解压到C盘后，再次运行：

```
python setup.py install
```

这次会报error LNK2001: 无法解析的外部符号 TA_SetUnstablePeriod等错误。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQxODE0MzE0)

这种情况官网上没说明，凌乱了一会儿，查找资料说是下载的ta-lib只有32位版本，没有64位版本。

Ta-lib是用VS2005写的，如果要自己编译ta-lib 64位，ta则要搭建cmake，VS等环境，所幸找到了篇文章（https://estan.cn/2016/11/compiling-and-installation-details-of-ta-lib.html）：

从上面下载https://file.estan.cn/talib/ta-lib_x86-64.zip 64位编译好的静态库。

下载好后，
解压后用编辑器打开setup.py修改其中的52和53行为你下载的静态库对应的路径，将第15行的包导入复制粘贴到第6行的空行处，保存文件
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQyMDE3NDYy)

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQyMTM5ODkz)

运行命令：

```
python setup.py bdist_wheel
```

正常情况会在dist目录下出现TA_Lib-0.4.10-cp36-cp36m-win_amd64.whl。
运行：

```
pip install TA_Lib-0.4.10-cp36-cp36m-win_amd64.whl
```

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQyMzE5NTg0)

至此TA_Lib安装成功。

## 安装RQAlpha ##

RQAlpha主要依赖用到了cython, boclz等。

参考安装指南（http://rqalpha.readthedocs.io/zh_CN/latest/intro/install.html）。

### 安装Cython ###
通过pip命令直接安装：

```
pip install -U pip setuptools cython -i https://pypi.tuna.tsinghua.edu.cn/simple
```

会报错：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI2MTQyNjM1NjYy)

### 安装 boclz ##

```
pip install bcolz
```

同上，也会报错。

所幸，用到的模块都可以在扩展列表（http://www.lfd.uci.edu/~gohlke/pythonlibs/#bcolz
）里面找到，哪个模块报错，就下载自己适合好的whl文件，再进行安装就行。

最后，安装rqalpha:

```
pip install  rqalpha
```

很奇怪的是装rqalpha时总是提示logbook模块总是不成功，但早就已经装好了，
我准备重定向日志仔细查看时：

```
pip install rqalpha > c:\temp.txt
```

又莫名其妙的好了。


## Pycharm IDE

IDE选择Pycharm，2017.01发布的专业版：
https://pan.baidu.com/s/1jiHtuLKUGZg0XTLxZP1Vug

但需要注册，可以在本要安装一个破解服务：
链接：https://pan.baidu.com/s/1dn5Pu6LupUJ_sT-wwYO3Dg 密码：emmr

激活方法选为lisence server。


## 附件 ##

windows64位，python36版本所需的依赖文件，可以从这里下载：
[依赖文件](http://download.csdn.net/download/shaozhenged/10133607)


