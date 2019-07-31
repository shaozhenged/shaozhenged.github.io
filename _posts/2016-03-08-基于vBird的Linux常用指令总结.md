---
layout:     post
title:      "基于vBird的Linux常用指令总结"
date:       2016-03-08 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - Linux    
  
---

| 主题      | 概要                                                      |
| --------- | --------------------------------------------------------- |
| Linux指令 | Linux常用指令，包括XWindow快捷键，shell命令和常用网络命令 |
| --------  | ---                                                       |
| **编辑**  | **时间**                                                  |
| 新建      | 20160308                                                  |
| --------  | ---                                                       |
| **序号**  | **参考资料**                                              |
| 1         | 鸟哥的linux私房菜                                         |

## X Windows快捷键  ## 

**[Alt] + [Ctrl] + [Backspace]：** 重启X
**[Ctrl] + [Alt] + [F1] ~ [F6]  ：**文字接口登陆 tty1 ~ tty6 终端机
**[Ctrl] + [Alt] + [F7]   ：**图形接口壁纸
**startx：**启动X窗口界面
**exit:**注销Linux（管理员账户）
## shell命令（按章节顺序） ##
### 第五章  首次登陆与在线求助 man page###
**LANG：**语系支持
**date：**显示日期与时间的命令 
**cal：**显示日历的命令 
**bc：**简单好用的计算器
**[Tab]按键：**『命令补全』与『文件补齐』
**[Ctrl]-c 按键：**中断目前程序
**[Ctrl]-d 按键：**键盘输入结束(End Of File, EOF 或 End Of Input)』，另外，也可以用来取代exit
**帮助文档：**
**man page：** man+command
如『DATE(1)』，中1代表：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTcyOTA2MjA1)
当按下『/』之后，光标就会移动到屏幕的最下面一行， 并等待你输入搜寻的字符串。
**man page** 常用的按键：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTczMDEzNTQ5)

**whatis**  [命令或者是数据]== **man -f** [命令或者是数据]：搜寻相关命令
**apropos** [命令或者是数据] == **man -k** [命令或者是数据]：搜寻关键字

与**man page**类似，linux独有的**info page**,增强了可读性
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTczMTM5NzY5)

**who：**查看当前在线用户
 **netstat -a ：**查看网络联机状态
**ps –aux：**查看背景运行程序

**sync：**将数据同步写入硬盘中的命令
**shutdown：**惯用的关机命令 
**reboot, halt, poweroff：**重新启动，关机
**init ：**切换运行等级
**init 0：** 关机

### 第六章 Linux 的文件权限与目录配置###
**chgrp ：**改变文件所属群组
**chown ：**改变文件拥有者
**chmod ：**改变文件的权限, SUID, SGID, SBIT等等的特性
**etc：**配置文件
**/bin：**重要执行档
**/dev：**所需要的装置文件
**/lib：**执行档所需的函式库与核心所需的模块
**/sbin：**重要的系统执行文件

### 第七章 Linux 文件与目录管理###
特殊目录：
**.**         代表此层目录，也可以使用 ./ 来表示
**..**        代表上一层目录，也可以 ../ 来代表
**-**         代表前一个工作目录
**~**         代表『目前使用者身份』所在的家目录
**~account**  代表 account 这个使用者的家目录(account是个帐号名称)

**cd：**变换目录
**pwd：**显示目前的目录
**mkdir：**创建一个新的目录
**rmdir：**删除一个空的目录
**ls：**文件与目录的检视
**cp, rm, mv：**复制、删除与移动 

文件内容查阅：
**cat ：** 由第一行开始显示文件内容
**tac ：**从最后一行开始显示，可以看出 tac 是 cat 的倒著写
**nl：** 显示的时候，顺道输出行号
**more：** 一页一页的显示文件内容
**less ：**与 more 类似，但是比 more 更好的是，他可以往前翻页
**head：** 只看头几行
**tail：** 只看尾巴几行
**od ：**  以二进位的方式读取文件内容

**touch：**修改文件时间或建置新档
touch最常被使用的情况是：
创建一个空的文件；
将某个文件日期修订为目前 (mtime 与 atime)

### 第八章 Linux 磁盘与文件系统管理###

待续。。。

### 第九章 文件与文件系统的压缩与打包###

待续。。。

### 第十章 vim 程序编辑器###

一般模式可用的按钮说明，光标移动、复制贴上、搜寻取代等
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MDA4ODc3)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MDM0MDk2)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MTI5Njkw)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MTUzNzc0)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MjMxNzg0)

一般模式切换到编辑模式的可用的按钮说明
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MjU5Mjkw)

一般模式切换到指令列模式的可用的按钮说明
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MzM2OTYz)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTg1MzUwODMy)

### 第十一章 认识与学习 BASH ###
**alias:**命令别名配置功能
**[Ctrl] + c ：**停掉程序
**echo：**变量的取用，只是需要在变量名称前面加上 $ ， 或者是以 ${变量} 的方式来取用
**export：** 自定义变量转成环境变量
**read:** 变量键盘读取
**declare / typeset:** 宣告变量的类型
**history:**历史命令

**很多环境配置的内容，暂时不用理会**

**输入与输出：**
**1.	标准输入　　(stdin) ：**代码为 0 ，使用 < 或 << ；
**2.	标准输出　　(stdout)：**代码为 1 ，使用 > 或 >> ；
**3.	标准错误输出(stderr)：**代码为 2 ，使用 2> 或 2>> ；

**分号 ；: ** cmd ; cmd  不考虑命令相关性的连续命令下达
**$?：**命令回传值
**&& 或 ||:**

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkwMDM4MzAx)

**撷取命令：** cut, grep
**cut:** 从一行数据中“切”取想要的信息
**grep:**从文档中提取想要的行
**排序命令：** sort, wc, uniq
**双向重导向：** tee，把输出同时输入到屏幕和文件

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkwMTMwMTYz)

**字符转换命令：** tr, col, join, paste, expand
**tr：** 可以用来删除一段信息当中的文字，或者是进行文字信息的替换
**col：**常用来简单的处理将 [tab] 按键取代成为空格键
**join：**把两个文件当中有"相同数据" 的那一行连接在一起
**paste：**将两行贴在一起，且以[tab] 键隔开
**expand：**将 [tab] 按键转成空格键
**split：**分割命令，将一个大文件，依据文件大小或行数来分割，分割成为小文件
**参数代换：** xargs (没搞懂)

### 第十二章 正规表示法与文件格式化处理###
**特殊符号意义：**

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkwODE3NjU1)

**grep的用法：**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkwODQ3NDgw)

 **[]:**代表某『一个』字节
**[^] ：**代表反向选择
**^ ,$:**行首与行尾字节, 注意：^ 符号在 [] 内代表『反向选择』，在 [] 之外则代表定位在行首的意义
**. (小数点)：**代表『一定有一个任意字节』的意思 
**•：**代表『重复前一个字节， 0 到无穷多次』的意思，为组合形态

基础正规表示法字符汇整：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYxODA5MDc2)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkxMDExMzg3)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkxMDI1MjY4)

**sed ：**	以行为单位的新增/删除功能
	                 以行为单位的取代与显示功能
			部分数据的搜寻并取代的功能
			直接修改文件内容(危险动作)
**printf：**格式化打印
**awk：**数据处理工具，主要处理『每一行的栏位内的数据』，而默认的『栏位的分隔符号为 "空白键" 或 "[tab]键" 
**diff：**比对两个文件之间的差异，并且是以行为单位来比对
**cmp：**也是比对两个文件，主要利用『位组』单位去比对
**patch：**文件升级还原
**pr:**文件列印准备

### 第十三章 学习 Shell Scripts###

**test测试命令：**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkxOTQwMDkx)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkyMDA2MDEz)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkyMTI3OTM3)

**特殊符号：**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkyOTU3MDY2)

**shift：**使参数变量号码偏移
**shell script的跟踪与调试：**可以直接以 bash 的相关参数来进行
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkzMDM4MDkw)

### 第十四章 Linux帐号管理与ACL权限配置###
**/etc/passwd:** 每一行都代表一个账号,UID为0是系统管理员帐号，1~499系统账号，500~65535为可登陆帐号
**/etc/shadow：**存放口令限制参数，如登陆密码、账号失效日期等
**/etc/group：**记录群组的配置，组名、口令、GID、此群组支持的账号名称
**有效群组(effective group)与初始群组(initial group)：**
/etc/passwd 里面的第四栏GID 就是所谓的『初始群组 (initial group) 』，也就是说，当用户一登陆系统，立刻就拥有这个群组的相关权限的意思
有效群组是用户已加入的群组
**groups:** 有效与支持群组的观察，用来观察某个用户所支持的群组
**newgrp:** 有效群组的切换
**/etc/gshadow：**创建群组管理员
**useradd：**新增与移除使用者
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTkzOTU1Nzc0)

**useradd 参考档：**useradd –D
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MDIxOTAw)

**passwd：**配置口令
**usermod：**调整账号相关参数
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MTAwOTE0)
**userdel：**删除用户的相关数据
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MTMzNDQ2)
**finger：**查阅用户相关信息
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MjAzNDMz)
**id：**查询某人或自己的相关 UID/GID 等等的信息
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MjI1NTcx)
**groupadd：**新增群组
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MjUwNjIx)
**groupmod：**修改相关group参数
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MzEzOTMx)
**groupdel：**删除群组
**gpasswd：**群组管理员功能
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0MzQxMDEz)

**主机的细部权限规划：ACL 的使用**
**ACL ：** Access Control List 的缩写，主要的目的是在提供传统的 owner,group,others 的 read,write,execute 权限之外的细部权限配置。ACL 可以针对单一使用者，单一文件或目录来进行 r,w,x 的权限规范，对于需要特殊权限的使用状况非常有帮助。
**setfacl：**配置某个目录/文件的 ACL 规范
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0OTA1MjIy)
**例子：**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk0OTQzNDU3)

**getfacl 命令用法**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk1MDEyMDgz)

**su：** 身份切换命令
单纯使用『 su 』切换身份时，读取的变量配置方式为 non-login shell 的方式，这种方式很多原本的变量不会被改变
**su  -,**读取的变量配置方式为 non-login shell 的方式，会改变整个环境变量
**su - -c：**只有 root 才能进行的命令，且运行完毕就恢复原本的身份
如：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk1MTAyOTAy)

**sudo：**为避免root口令泄露，sudo则仅需要自己的口令，就能够以其他用户的身份运行命令。并非所有人都能够运行 sudo ，而是仅有规范到 /etc/sudoers 内的用户才能够运行 sudo 这个命令
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk1MTM1MjQx)

**visudo 与 /etc/sudoers：**除了 root 之外的其他账号，若想要使用 sudo 运行属于 root 的权限命令，则 root 需要先使用 visudo 去修改 /etc/sudoers ，让该账号能够使用全部或部分的 root 命令功能

**PAM 模块：**PAM (Pluggable Authentication Modules, 嵌入式模块)，PAM 可以说是一套应用程序编程接口 (Application Programming Interface, API)，他提供了一连串的验证机制，只要使用者将验证阶段的需求告知 PAM 后， PAM 就能够回报使用者验证的结果 (成功或失败)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA4MTk1NzE3NTE0)

**查询使用者：** w, who, last, lastlog
**使用者对谈：** write, mesg, wall,mail

## Linux网络常用指令 ##

**ifconfig**
**ifconfig** 主要是可以手动的启动、观察与修改网络接口的相关参数，包括 IP 参数以及 MTU 等等都可以修改，他的语法如下：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYxOTM0ODMz)

常用操作：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyMDM3MDM2)

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyMTAzNDM5)

**ifup,ifdown**
实时的手动修改一些网络接口参数，可以利用 ifconfig 来达成，如果是要直接以配置文件，亦即是在 /etc/sysconfig/network-scripts 里面的 ifcfg-ethx 等档案的设定参数来启动的话， 那就得要透过 ifdown 或 ifup 来达成了。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyMjE3Mzgx)
ifup与ifdown两支程序其实是 script 而已，他会直接到/etc/sysconfig/network-scripts 目录下搜寻对应的配置文件，例如 ifup eth0 时，他会找出 ifcfg-eth0 这个档案的内容，然后来加以设定。

**route：路由修改**

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyMzE2ODAx)
路由的新增与删除
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyMzU0MDgz)

**ip:网络参数综合指令**
ip指令综合了ifconfig 与 route 这两个指令，而且ip 可以达成的功能却又更多，主要有ip link,ip addr,ip route三个部分。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyNTAxOTU5)

**ip link**
 可以设定与装置 (device) 有关的相关参数，包括 MTU 以及该网络接口的 MAC 等等，当然也可以启动 (up) 或关闭 (down) 某个网络接口啦！整个语法是这样的：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyNjE4MTMz)
示例：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyNjQzMjc0)

**ip address**
如果说 ip link是与第二层资料连阶层有关的话，那么ip address (ip addr) 就是与第三层网络层有关的参数啦，主要是在设定与 IP 有关的各项参数，包括 netmask, broadcast 等等。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyODA2NjMz)

**ip route** 的功能几乎与 route 这个指令差不多，但是，他还可以进行额外的参数设计，例如 MTU 的规划等等，相当的强悍啊！
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyODU3MTAy)

示例：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYyOTMxNDMw)

**dhclient**
如果你是使用 DHCP 协议在局域网络内取得 IP 的话，那么是否一定要去编辑 ifcfg-eth0 内的 BOOTPROTO 呢？ 嘿嘿！有个更快速的作法，那就是利用 dhclient 这个指令～因为这个指令才是真正发送 dhcp 要求工作的程序啊！那要如何使用呢？很简单！如果不考虑其他的参数，使用底下的方法即可：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzMDU3Njg3)
够简单吧！这样就可以立刻叫我们的网络卡以 dhcp 协议去尝试取得 IP 喔！

**ping**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzMTU5MjM1)

**traceroute**
追踪两部主机之间通过的各个节点 (node) 通讯状况。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzMjU1NDU1)

**netstat**
查询自己的网络接口所监听的端口 (port) 。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzMzQzNDU1)

**host**
这个指令可以用来查出某个主机名的 IP 喔！
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzNDIzOTQ4)

**nslookup**
这玩意儿的用途与 host 基本上是一样的，就是用来作为 IP 与主机名对应的检查， 同样是使用 /etc/resolv.conf 这个档案来作为 DNS 服务器的来源选择。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzNTIyMjM5)

**telnet**
远程连接。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzNTUzNDMz)

**ftp**
ftp 这个指令很简单，用在处理 FTP 服务器的下载数据啦。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzNjUwNjQ3)

**lftp (自动化脚本)**
单纯使用 ftp 总是觉得很麻烦，有没有更快速的 ftp 用户软件呢？让我们可以使用类似网址列的方式来登入 FTP 服务器啊？有的，那就是 lftp 的功能了！ lftp 预设使用匿名登录 FTP 服务器，可以使用类似网址列的方式取得数据，使用上比单纯的 ftp 要好用些。此外，由于可在指令列输入账号/密码，可以辅助进行程序脚本的设计喔！
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzNzMxMjcy)

**tcpdump:** 文字接口封包撷取器
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwMzA5MTYzODA2OTQ1)

