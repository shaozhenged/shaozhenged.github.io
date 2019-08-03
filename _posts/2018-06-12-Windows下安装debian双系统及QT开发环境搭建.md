---
layout:     post
title:      "Windows下安装debian双系统及QT开发环境搭建"
date:       2018-06-12 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - QT
    - debian
---

| 主题     | 概要                     |
| -------- | ------------------------ |
| 环境搭建 | debian系统及开发环境配置 |
| -------- | ---                      |
| **编辑** | **时间**                 |
| 新建     | 20180612                 |
| -------- | ---                      |
| **序号** | **参考资料**             |
| 1        |

在windows下，安装debian9.3系统，及QT5.10等开发环境配置。

# 1. Debian系统安装
## 1.1. 启动U盘制作

事先在windows系统下
debian9.3镜像下载地址：
https://pan.baidu.com/s/1UcHx1ZgiFlvMn_IzUaLONw

U盘制作工具下载地址：
https://pan.baidu.com/s/1othQ-Ezg1WnxiUv8t5TLuA

准备一个不小于2G的U盘。
**注意提前备份该U盘上的数据**，制作安装盘的过程将格式化U盘。

将U盘插入电脑，直接启动下载的 Universal-USB-Installer软件，选择Agree:

![这里写图片描述](https://img-blog.csdn.net/20180612202524361?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

此处在提供的Debain选项中，只有Live 和Netinst两个选项，并非想安装的amd64:

![这里写图片描述](https://img-blog.csdn.net/20180612202605844?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

往下拉滚动条，直接拉到最后，选择Try Unlisted Linux ISO:

![这里写图片描述](https://img-blog.csdn.net/20180612202628389?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

然后选择下载的ISO镜像文件，选择U盘，点击Create按钮:

![这里写图片描述](https://img-blog.csdn.net/20180612202650837?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


 接下来将解压ISO文件，制作安装U盘，此过程时间较长，大约15分钟左右。
出现以下提示，表示安装启动盘已经成功制作完成:

![这里写图片描述](https://img-blog.csdn.net/20180612202715970?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 1.2. 磁盘分区

应用windows磁盘管理工具，根据各人情况压缩出150G以上的磁盘空间，注意不要分区，也不要格式化。

压缩卷：

![这里写图片描述](https://img-blog.csdn.net/20180612202736416?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

压缩后，保持未分配：

![这里写图片描述](https://img-blog.csdn.net/20180612202753772?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

Linux系统将安装在这个未分配的磁盘上。

## 1.3. BIOS设置

重启电脑，当出现开机界面时按F1(Thinkpad)，其它机型可能是F2，进入BIOS：
-   Security → Secure Boot ，设置为：Disabled；

-   Startup → UEFI/Legacy Boot ，设置为：Both（原选项为：UEFI Only）；

-   在新显示的 UEFI/Legacy Boot Priority 设置为：UEFI First；

-   CSM 设置为 Yes；
-   F10保存并退出；


## 1.4. USB镜像启动

-   插入USB设备；
-   重启电脑；
-   当出现开机界面时，长按F12，进入启动项选择；
-   选择含有USB-HDD字样的引导项；

## 1.5. 基础设置
-   选择“Graphical Install”；
-   选择“English”,"Other","Asia","China","en_US.UTF-8","American English";
-   出现"Detect Network Hardware"选项时，点击"No"，并接上有线网络；
-   输入主机名，须在全局域网内保持唯一，推荐姓.名缩写-操作系统形式，如"shao.z-debian"，"liu.y.y-debian";
-   输入域名，没有建域，可任写；
-   设置root密码；
-   设置full name（可任取）;
-   设置user name（可任取）;
-   设置用户密码；

## 1.6. 系统分区
-   选择Manual,列出的磁盘中选择前面压缩出来的未分区的磁盘，FREE SPACE;
-   选择"Automatic partition"；
-   选择"All files in one partition";
-   确认分区信息，并选择"Yes"，开始进行分区；

## 1.7. 安装系统
-   选择包管理器镜像，"China"；
-   选择镜像源"ftp.cn.debian.org"（如没有，任选一个）；
-   HTTP代理保持空白；
-   直接保持默认选择"Debian desktop environment",debian已把Gnome作为默认界面;
-   重启电脑，安装成功，注意重启前移除U盘；

# 2. 系统配置

## 2.1. 无线网卡配置

该linux内核没自带无线网卡驱动，通过以下方式安装：

```
su
nano /etc/apt/sources.list

替换：deb http://ftp.cn.debian.org/debian/ stretch main
为：deb http://ftp.cn.debian.org/debian/ stretch main non-free

apt-get update
apt-get install linux-image-amd64 linux-headers-amd64 broadcom-sta-dkms firmware-iwlwifi

modprobe -r b44 b43 b43legacy ssb brcmsmac bcma  iwlwifi
modprobe iwlwifi
modprobe wl  (可能会报错，不用管)

```

## 2.2. 输入法设置

debian不自带中文输入法，一般的输入法又太古老，可通过以下方式安装搜狗输入法：

先安装fcitx:
```
su

apt-update
apt-get install fcitx
apt-get install fcitx-ui-classic && apt-get install fcitx-ui-light
```

下载sougou-for-linux:

https://pan.baidu.com/s/1oXtAWS_QPib_d8pXOUZnKQ

切换到下载目录，root用户：

```
执行：
dpkg -i sogoupinyin_2.2.0.0108_amd64.deb 

可能会有依赖安装不成功，执行：
apt-get install -f

完成后，再次执行：
dpkg -i sogoupinyin_2.2.0.0108_amd64.deb 
```

重启电脑，正常通过中英切换按键进行切换。

## 2.3. GDB/GCC/G++/CMAKE安装

可通过类似
```
apt-cache madison gdb
```
查看版本库中的可用版本，我们只需安装现在的最新稳定版本:

```
sudo apt-get install cmake gcc g++ gdb
```


## 2.4. BOOST库安装

执行：
```
sudo apt-get install libboost-dev
```


## 2.5. QT5.10安装

### 2.5.1. 准备工作

下载QT-for-linux安装包:

https://pan.baidu.com/s/15UXLm91nbGESjRMsRuyolA

QT安装包中没有包含安装环境，需要分别安装gcc/g++/make等开发软件，另外需要OpenGL库。对于ubuntue与基于debian的系统，通过下列方法获取OpenGL库和最小开发工具集：

```
sudo apt-get install build-essential libgl1-mesa-dev
```


### 2.5.2. 安装步骤

```
cd到安装包下载目录：
如:
cd /home/shao/Downloads

chmod a+x qt-opensource-linux-x64-5.10.0.run
```

接着执行安装包，可以以user用户和以root用户执行，user用户默认安装到user用户的home目录，当前用户可用。root用户默认安装到/opt/QT目录，所有用户可用。

我们统一安装成root用户。

```
sudo ./qt-opensource-linux-x64-5.10.0.run 
```

默认安装目录：

![这里写图片描述](https://img-blog.csdn.net/20180612202937340?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

选中QT选项框：

![这里写图片描述](https://img-blog.csdn.net/20180612202951345?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW96aGVuZ2Vk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


安装成功后，就可启动qt collector。


## 2.6. Rabbitcvs安装

安装SVN与Git客户端：

```
sudo apt-get install rabbitvcs-core rabbitvcs-nautilus3 rabbitvcs-gedit rabbitvcs-cli
```

重启有效。
