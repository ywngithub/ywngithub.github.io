---
title: Linux段错误调试跟踪记录
date: 2021-07-26 20:15:09
tags: 段错误
categories: 调试
---

&emsp;&emsp;有些时候我们在实现一段C/C++代码的时候，由于对一个非法内存进行了误操作，在程序运行的过程中，出现了"段错误"。这种问题很多人经常遇到。遇到这种问题是非常无语的，只是提示了"段错误"，接着什么都没有，如果我们一味的去看代码找太疼苦了，因为我们都相信自己写的代码没问题，现实就是现实。下面介绍一种方法，可以有效的定位出现"段错误的地方"。 <!-- more -->  
&emsp;&emsp;当我们的程序崩溃时，内核有可能把该程序当前内存映射到core文件里，方便程序员找到程序出现问题的地方。  
&emsp;&emsp;什么是core dump?  
&emsp;&emsp;core的意思是内存，dump的意思是扔出来，堆出来。  
&emsp;&emsp;使用Linux core文件调试段错误的命令如下：  
![](1.jpeg)  
其中参数含义：  
>ulimit -c unlimited 		设置core文件生成的大小，当前设置为“无穷大”，设置为0表示不生成  
>echo "core-%e-%p-%t" > /proc/sys/kernel/core_pattern  设置core文件生成的格式，%e表示进程/线程名称，%p表示进程号，%t表示生成时间（UTC格式）  

&emsp;&emsp;当程序出现段错误时，在本地目录会生成一个core开头的文件，如下：
![](2.jpeg)  

&emsp;&emsp;把这个文件拷贝到Ubuntu上，使用命令arm-linux-gnueabihf-gdb 【程序文件】 【core文件】
即可找到出错的位置。
![](3.jpeg)  
![](4.jpeg)  

&emsp;&emsp;注意：Debug版本才可以记录行的位置，Release版本只能记录函数名称。
