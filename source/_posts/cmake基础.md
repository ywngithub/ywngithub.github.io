---
title: cmake基础
date: 2021-07-15 17:08:09
tags: cmake
categories: cmake
---

&emsp;&emsp;Make是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。他能够输出各种各样的makefile或者project文件，能测试编译器所支持的C++特性,类似UNIX下的automake。 <!-- more -->

find_package
=========
>模块模式：  
>（1）先到CMAKE_MODULE_PATH目录中查找Find<name>.cmake文件。  
>（2）再到自己的模块目录<CMAKE_ROOT>/share/cmake-x.y/Modules/查找。  
配置模式：  
>（1）如果模块模式找不到，则会查找 <Name>Config.cmake 或者<lower-case-name>-config.cmake文件。  

不管使用哪一种模式，只要找到包，就会定义下面这些变量：  
>NAME_FOUND  
>	&emsp;&emsp;NAME_INCLUDE_DIRS or NAME_INCLUDES  
>	&emsp;&emsp;NAME_LIBRARIES or NAME_LIBRARIES or NAME_LIBS  
>	&emsp;&emsp;NAME_DEFINITIONS  
这些都在 Findname.cmake 文件中。  

&emsp;&emsp;现在，在你的代码的顶层目录中的 CMakeLists.txt 文件中，我们检查变量<NAME>_FOUND 来确定包是否被找到 。如果找到这个包，我们用 NAME_INCLUDE_DIRS 调用 include_directories() 命令，用 NAME_LIBRARIES 调用 target_link_libraries() 命令。  
