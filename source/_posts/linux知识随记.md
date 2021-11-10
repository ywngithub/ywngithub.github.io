---
title: linux知识随机
date: 2021-09-07 18:08:09
tags: linux
categories: linux
---
&emsp;&emsp;一些linux相关知识，持续更新...
 <!-- more -->

Linux命令行参数前加双杠--,单杠-和不加杠-的区别
=============
①　参数前单杠的表明后面的参数是字符形式；  
②　参数前双杠的则表明后面的参数是单词形式。  
①　参数前有杠的是System V风格。  
②　参数前没有杠的是BSD风格。  
System V和BSD两种风格的区别主要是：  
系统启动过程中 kernel 最后一步调用的是 init 程序，init 程序的执行有两种风格，即 System V 和 BSD。  
System V 风格中 init 调用 /etc/inittab，BSD 风格调用 /etc/rc，它们的目的相同，都是根据 runlevel 执行一系列的程序。  
 
Linux 系统把 /lib 和 /usr/lib 两个目录作为默认的库搜索路径
===========
设置库文件的搜索路径有两种方法：  
01.在环境变量 LD_LIBRARY_PATH 中指明库的搜索路径。  
02.在 /etc/ld.so.conf 文件中添加库的搜索路径。然后运行 ldconfig 命令更新 /etc/ld.so.cache 文件之后才可以将自己可能存放库文件的路径都加入到/etc/ld.so.conf中是明智的选择，  
添加方法也极其简单，将库文件的绝对路径直接写进去就OK了，一行一个。  
 
socat是一个netcat(nc)的替代产品，可以称得上nc++。
===============
socat的地址类型很多，有ip, tcp, udp, ipv6,pipe,exec,system,open,proxy,openssl等  
 
1、socat串口转发  
socat udp4-listen:11161,reuseaddr,fork UDP:[监控服务器IP]:161  
 
udp4-listen：在本地建立的是一个udp ipv4协议的监听端口；  
reuseaddr，绑定本地一个端口；  
fork，设定多链接模式，即当一个链接被建立后，自动复制一个同样的端口再进行监听  
 
AT串口/dev/ttyUSB1映射到5555端口：  
sudo socat -d -d /dev/ttyUSB0,raw,nonblock,ignoreeof,cr,echo=0 TCP4-LISTEN:5555,reuseaddr(实测可用，创建了一个TCP服务器。端口5555)  
sudo socat file:/dev/ttyS0,b115200,inlcr=1  tcp-listen:5555  
转发到minicom终端： socat /dev/ttyUSB1,raw,nonblock,ignoreeof,cr,echo=0 /dev/ttyS1,raw  
转发到终端(电脑端)： socat /dev/ttyUSB1,raw,nonblock,ignoreeof,cr,echo=0 /dev/tty,raw  
  
2、网络转发  
socat TCP4-LISTEN:188,reuseaddr,fork TCP4:192.168.1.22:123 &  
 
（在本地监听188端口，并将请求转发至192.168.1.22的123端口）  
 
TCP4-LISTEN：在本地建立的是一个TCP ipv4协议的监听端口；  
reuseaddr：绑定本地一个端口；  
fork：设定多链接模式，即当一个链接被建立后，自动复制一个同样的端口再进行监听  
 
socat启动监听模式会在前端占用一个shell，因此需使其在后台执行。  
 
3、终端绑定  
Setting up two serial lines  
now on a terminal window run socat  
socat -d -d PTY PTY:  
The output should look like the following one:    
2013/09/20 14:07:10 socat[6871] N PTY is /dev/pts/5   
2013/09/20 14:07:10 socat[6871] N PTY is /dev/pts/6  
2013/09/20 14:07:10 socat[6871] N starting data transfer loop with FDs [3,3] and [5,5]  
Now you have two "serial" ports connected back to back  
Testing the ports  
open a new terminal and issue the following command  
sudo cat /dev/pts/5  
open a new terminal and issue the following command  
sudo echo "Hello World" > /dev/pts/6  
On the first terminal you should see the string "Hello World".  
 
终端
=============
一般ttyS0对应com1，ttyS1对应com2  
查看串口驱动：cat /proc/tty/drivers/serial  
查看串口设备：dmesg | grep ttyS*  
查看USB信息：dmesg | tail  
 
sh首次链接出现yes/no提示
================
首次进行ssh链接时，出现以下提示：  
The authenticity of host '58.221.186.137 (58.221.186.137)' can't be established.  RSA key fingerprint is   a0:00:d3:33:54:96:40:03:ff:ad:15:a9:59:22:f4:2a.    
Are you sure you want to continue connecting (yes/no)?    
修改文件： /etc/ssh/ssh_config   
在文件中添加如下信息：StrictHostKeyChecking no  
 
段错误位置寻找
===================
ulimit -c unlimited  
echo "core-%e-%p-%t" > /proc/sys/kernel/core_pattern  
/usr/gdbserver/bin/arm-linux-gnueabihf-gdb DNYS-4G_APP core        // 复制到Ubuntu执行  
where/bt         // gdb窗口输入  
 
.gdbinit文件设置gdb配置  
set auto-load safe-path /  
set solib-search-path /home/ywn/code/dnys-2d_app/DNYS-2D_APP_Qt/DNYS-4G_APP/Expand/lib/ARM  
set solib-search-path /usr/local/Qt5.12_arm/lib/   
 
去除可执行文件中的符号信号  
=================
strip  a.out  