---
title: FreeNOS学习笔记(2)-FreeNOS安装和使用
date: 2021-04-04 11:10:55
tags: FreeNOS笔记
categories: FreeNOS
---

一、测试平台
=======
虚拟机： VMware14.0.0  
操作系统：Ubuntu16.04  
FreeNOS：v1.0.3
<!-- more -->

二、环境设置
=======
Ubuntu中执行以下命令：
>$ sudo apt-get update  
>$ sudo apt-get install build-essential scons genisoimage xorriso qemu-system binutils-multiarch u-boot-tools liblz4-tool  

如果Ubuntu是64-bit系统，还需要安装GCC multilib软件包以针对32位架构进行交叉编译：
>$ sudo apt-get install gcc-multilib g ++-multilib  

也可以安装LLVM / Clang编译器：
>$ sudo apt-get install clang  

三、构建FreeNOS
=======
运行以下命令下载源代码，用FreeNOS的版本替换“ xxx”：
>$ wget http://www.FreeNOS.org/pub/FreeNOS/source/FreeNOS-xxxtar.gz   
>$ tar zxf FreeNOS-xxxtar.gz  

也可以使用git命令下载最新源代码：
>$ git clone https://github.com/nieklinnenbank/FreeNOS

要使用默认设置（Intel，在启用调试的情况下使用GCC）构建FreeNOS，执行以下命令：
>$ scons

**什么是scons？**  
&emsp;&emsp;scons是一个Python写的自动化构建工具，从构建这个角度说，它跟GNU make是同一类的工具，是一种改进，并跨平台的gnu make替代工具，其集成功能类似于autoconf/automake 。scons是一个更简便，更可靠，更高效的编译软件。


要清理您的构建目录，请使用：
>$ scons -c

三、运行FreeNOS
=======
在宿主机上自动测试FreeNOS，执行：
>$ scons test

在qemu虚拟机上自动测试FreeNOS，执行：
>$ scons qemu_test

开始在qemu虚拟机上通过串行终端运行FreeNOS，执行：
>$ scons qemu

&emsp;&emsp;此时进入FreeNOS命令行终端，如下图：  
![](1.png)  
输入任意用户名，密码为空即可进入。

**注意：** 在虚拟机中运行的Ubuntu默认是不开启KVM中，此时qemu无法运行，会提示：找不到或者不支持KVM，此时将vmware处理器选项中的虚拟化技术打开即可，如图所示：  
![](2.png)  