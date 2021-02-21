---
title: linux基础知识-启动
date: 2021-02-21 10:53:50
categories: Linux基础知识
---
初始化相关
=====

命令  | 描述
------------- | -------------
 vi etc/rc.local | 开机会自动执行的脚本 |
 /etc/init.d/S90** | imx6ull |
<!-- more -->


启动级别
=====
/etc/rc0~rc6 共有7级启动

命令  | 描述
------------- | -------------
rc0 |停机（不能使用
rc1 |单用户模式
rc2 |多用户模式（没有NFS）
rc3 |完全多用户模式（所以上次Ubuntu图形界面损坏，重启进入命令行模式的时候修改为3是这个原因）
rc4 |没有使用，系统预留
rc5 |图像界面模式
rc6 |重启模式（不能使用）

存放不同的启动脚本，分为两类，**K是指Kill，停止的意思，S是指Start，启动的意思。**
后面紧跟数字，数字越大，优先级越低。
这些脚本都是从/etc/init.d软链接过来的，配置文件通常在/etc/init目录下。
 
rcS 单用户模式启动脚本
runlevel 查看当前运行级别
/etc/inittab 修改默认运行级别

 
inittab文件
===== 
想要从串口登录，可以配置/etc/inittab文件

增加**ttyGS0::respawn:/sbin/getty -L ttyGS0 115200 vt100**即可

ttyGS0为具体的串口号，根据具体的串口更改。

**id:runlevels:action:process**

>id 用来定义在inittab文件唯一的条目编号，长度为 1-4个字符
>
>runlevels 列出来运行的级别 为空则代表所有级别
>
>action 要执行的动作
>
>process 要执行的程序
>
>respawn 意思就是当后面的要执行的程序（/sbin/mingetty tty1） 终止了，init进程会自动重启该进程