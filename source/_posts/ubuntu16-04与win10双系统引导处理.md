---
title: ubuntu16.04与win10双系统引导处理
date: 2021-03-20 16:08:09
tags: 双系统引导
categories: 系统问题
---
&emsp;&emsp;系统环境：只有一块硬盘，且win10系统装在硬盘的前面，ubuntu16.04在后面。
 <!-- more -->

&emsp;&emsp;win系统重装后系统引导会被破坏，导致开机只能进入win系统，此时可以用u盘制作一个Ubuntu安装盘，从U盘启动后进入Ubuntu试用，启动系统，打开终端，输入命令:
>sudo add-apt-repository ppa:yannubuntu/boot-repair  
>sudo apt-get update  
>sudo apt-get install -y boot-repair && boot-repair

安装boot-repair工具，软件截图如下：  
![](1.png)  
选择第一项根据提示修复，修复成功后大概率win系统不在grub引导中，此时进入Ubuntu系统，打开终端，输入命令：  
>sudo update-grub   
>sudo grub-install /dev/sda  

修复win系统引导即可。