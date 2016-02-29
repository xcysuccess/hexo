---
layout: post
title: "Mac shell入门"
date: 2015-12-25 21:07:43 +0800
comments: true
categories: git_svn_shell
description: "这里有一些常用的Shell脚本可以参考" 
keywords: shell
---

## 一. 工具篇
工欲善其事，必先利其器，以下推荐工具建议大家都安装一下.  

| 作用 | 名称 |
| --- | --- |
| 命令行工具 | [item2](https://www.iterm2.com/) |
| 配色 | [在 Mac OS X 终端里使用 Solarized 配色方案](http://get.ftqq.com/991.get) |
| html转markdown | [htmlToMarkDown](https://domchristie.github.io/to-markdown/) |
| shell优化命令工具 | [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) |
<!-- more -->

## 二. 入门篇
OSX 采用的Unix文件系统，所有文件都挂在跟目录 / 下面，所以不在要有Windows 下的盘符概念。 
你在桌面上看到的硬盘都挂在 /Volumes 下。  
如接上个叫做 USBHD的移动硬盘，桌面上会显示出一个硬盘图标，它实际在哪里呢？ 在终端里执行 ls /Volumes/USBHD, 看看显示出的是不是这个移动硬盘的内容。 

| 作用 | 名称 |
| --- | --- |
| 根目录位置 | /核心 Mach_kernel 就在这里 |
| 驱动所在位置 | /Systme/Library/Extensions | 
| 用户文件夹位置 | /Users/用户名 | 
| 桌面的位置 | /Users/用户名/Desktop |

***
** 文件通配符为星号* **  
根目录标志 / 不是可有可无，cd /System 表示转到根目录下的System中，而cd System 表示转到当前目录下的 System中 
***  
**获得权限**  
为了防止误操作破坏系统，再用户状态下时没有权限操作系统重要文件的，所以先要取得root权限 
sudo －s 
然后输入密码，输入密码时没有任何回显，连星号都没有，只管输完回车就行了。 
***
**强行关闭xcode**
 
```objc
ps -A|grep 'Xcode'
kill -9 xxx
```

***
**获取本地ip**	 

```objc
ifconfig | grep "inet"
```

**建议服务器**  
```objc
python -m http.server [<portNo>]   
```

## 三. 操作大全篇

### 3.1 目录操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| mkdir | 创建一个目录 | mkdir dirname |
| rmdir | 删除一个目录 | rmdir dirname |
| mvdir | 移动或重命名一个目录 | mvdir dir1 dir2 |
| cd | 改变当前目录 | cd dirname |
| pwd | 显示当前目录的路径名 | pwd |
| ls | 显示当前目录的内容 | ls -la  **-w 显示中文，-l 详细信息， -a 包括隐藏文件** |
| dircmp | 比较两个目录的内容 | dircmp dir1 dir2 |

### 3.2 文件操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| cat | 显示或连接文件 | cat filename |
| pg | 分页格式化显示文件内容 | pg filename |
| more | 分屏显示文件内容 | more filename |
| od | 显示非文本文件的内容 | od -c filename |
| cp | 复制文件或目录 | cp file1 file2 ,**cp -R :递归复制，文件夹里的内容全部复制** |
| rm | 删除文件或目录 | rm filename。-f, --force 强制删除,忽略不存在的文件,不提示确认。-i在删除前需要确认|
| mv | 改变文件名或所在目录 | mv file1 file2 |
| ln | 联接文件 | ln -s file1 file2 |
| find | 使用匹配表达式查找文件 | find . -name "*.c" -print |
| file | 显示文件类型 | file filename |
| open | 使用默认的程序打开文件 | open filename |

### 3.3 选择操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| head | 显示文件的最初几行 | head -20 filename |
| tail | 显示文件的最后几行 | tail -15 filename |
| cut | 显示文件每行中的某些域 | cut -f1,7 -d: /etc/passwd |
| colrm | 从标准输入中删除若干列 | colrm 8 20 file2 |
| paste | 横向连接文件 | paste file1 file2 |
| diff | 比较并显示两个文件的差异 | diff file1 file2 |
| sed | 非交互方式流编辑器 | sed "s/red/green/g" filename |
| grep | 在文件中按模式查找 | grep "^[a-zA-Z]" filename |
| awk | 在文件中查找并处理模式 | awk '{print $1 $1}' filename |
| sort | 排序或归并文件 | sort -d -f -u file1 |
| uniq | 去掉文件中的重复行 | uniq file1 file2 |
| comm | 显示两有序文件的公共和非公共行 | comm file1 file2 |
| wc | 统计文件的字符数、词数和行数 | wc filename |
| nl | 给文件加上行号 | nl file1 >file2 |

### 3.4 安全操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| passwd | 修改用户密码 | passwd |
| chmod | 改变文件或目录的权限 | chmod ug+x filename |
| umask | 定义创建文件的权限掩码 | umask 027 |
| chown | 改变文件或目录的属主 | chown newowner filename |
| chgrp | 改变文件或目录的所属组 | chgrp staff filename |
| xlock | 给终端上锁 | xlock -remote |

### 3.5 编程操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| make | 维护可执行程序的最新版本 | make |
| touch | 更新文件的访问和修改时间 | touch -m 05202400 filename |
| dbx | 命令行界面调试工具 | dbx a.out |
| xde | 图形用户界面调试工具 | xde a.out |

### 3.6 进程操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| ps | 显示进程当前状态 | ps u |
| kill | 终止进程 | kill -9 30142 |
| nice | 改变待执行命令的优先级 | nice cc -c *.c |
| renice | 改变已运行进程的优先级 | renice +20 32768 |

### 3.7 时间操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| date | 显示系统的当前日期和时间 | date |
| cal | 显示日历 | cal 8 1996 |
| time | 统计程序的执行时间 | time a.out |

### 3.8 网络与通信操作

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| telnet | 远程登录 | telnet hpc.sp.net.edu.cn |
| rlogin | 远程登录 | rlogin hostname -l username |
| rsh | 在远程主机执行指定命令 | rsh f01n03 date |
| ftp | 在本地主机与远程主机之间传输文件 | ftp ftp.sp.net.edu.cn |
| rcp | 在本地主机与远程主机 之间复制文件 | rcp file1 host1:file2 |
| ping | 给一个网络主机发送 回应请求 | ping hpc.sp.net.edu.cn |
| mail | 阅读和发送电子邮件 | mail |
| write | 给另一用户发送报文 | write username pts/1 |
| mesg | 允许或拒绝接收报文 | mesg n |

### 3.9 Korn Shell 命令

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| history | 列出最近执行过的 几条命令及编号 | history |
| r | 重复执行最近执行过的 某条命令 | r -2 |
| alias | 给某个命令定义别名 | alias del=rm -i |
| unalias | 取消对某个别名的定义 | unalias del |

### 3.x 其它命令

| 命令名 | 功能描述 | 使用举例 |
| --- | --- | --- |
| uname | 显示操作系统的有关信息 | uname -a |
| clear | 清除屏幕或窗口内容 | clear |
| env | 显示当前所有设置过的环境变量 | env |
| who | 列出当前登录的所有用户 | who |
| whoami | 显示当前正进行操作的用户名 | whoami |
| tty | 显示终端或伪终端的名称 | tty |
| stty | 显示或重置控制键定义 | stty -a |
| du | 查询磁盘使用情况 | du -k subdir |
| df | 显示文件系统的总空间和可用空间 | df /tmp |
| w | 显示当前系统活动的总信息 | w |

## 四. Vim入门  
| 作用 | 名称 |
| --- | --- |
| i   |在默认的"指令模式"下按 i 进入编辑模式 |
| ESC |在非指令模式下按 ESC 返回指令模式 |
| a  | 进行插入操作 |
| :q | 在未作修改的情况下退出 |
| :w | 保存当前编辑的文件需要用 |
| :q! | 放弃所有修改，退出编辑程序 | 
| :wq | 保存并退出 |  
| h	| 在"指令模式"下移动: 左 |
| j	| 在"指令模式"下移动: 下 | 
| k	| 在"指令模式"下移动: 上 | 
| l	| 在"指令模式"下移动: 右 | 



## 参考目录
1. [HTMLToMarkDown](https://domchristie.github.io/to-markdown/)  
2. [Mac终端命令大全](http://www.jianshu.com/p/3291de46f3ff)   
3. [mac终端命令大全介绍](http://www.douban.com/note/75797151/)  
4. [Vim入门基础](http://www.jianshu.com/p/bcbe916f97e1)