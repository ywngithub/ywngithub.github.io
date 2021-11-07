---
title: makefile基础
date: 2021-06-14 19:20:09
tags: makefile
categories: makefile
---

&emsp;&emsp;一个工程中的源文件不计其数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 makefile就像一个Shell脚本一样，也可以执行操作系统的命令。<!-- more -->  

patsubst：替换通配符
=======
>obj=$(patsubst %.c,%.o,$(dir) )  
>在$(patsubst %.c,%.o,$(dir) )中，patsubst把$(dir)中的变量符合后缀是.c的全部替换成.o，

wildcard：扩展通配符
=======
>src=$(wildcard *.c ./sub/*.c)  
>wildcard把 指定目录 ./ 和 ./sub/ 下的所有后缀是c的文件全部展开。

格式
=======
>target : prerequisties  
>目标：需要依赖的条件  
>简单示例：  
>hello:hello.cc  
>&emsp;&emsp;gcc  hello.cc -o hello  
>但如果文件多了呢？按部就班写会显得很长，所有这时候makefile中的常用命令就产生了，如下：  
>$@ 表示目标文件  
>$^ 表示所有的依赖文件  
>$< 表示第一个依赖文件  
>$? 表示比目标还要新的依赖文件列表  
>-c  只编译不链接  
