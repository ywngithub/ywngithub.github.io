---
title: conan基础
date: 2021-08-07 17:08:09
tags: conan
categories: conan构建
---

&emsp;&emsp;Conan是一个分散的包管理器，具有客户端 - 服务器架构。这意味着客户端可以从不同的服务器（“远程”）获取软件包以及上传软件包，类似于git远程控制器的“git”推拉模型。
<!-- more -->  
1.安装
=====
>pip install conan

2.获取远程列表
======
>conan remote list

3.增加远程库
========
>conan remote add conan-transit https://conan-transit.bintray.com


4.管理依赖包
=========
包的命令规则：包名/版本@用户/渠道  
>conan install glog/0.4.0@bincrafters/stable -r conan-center  
>conan remove glog/0.4.0@bincrafters/stable  

自动拉取依赖  
>conan install ..  

>-pr = profile  
>--build=missing 服务器上没有的话就编译，一般有源码  

在库这里改完代码验证  
>conan create -s build_type=Release -o clients=True . "mosquitto/2.0.10@screenshare/stable" --profile=t31-mips-linux  
