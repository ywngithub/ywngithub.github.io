---
title: 初始LLVM
date: 2021-06-28 20:25:56
tags: LLVM
categories: LLVM
---

什么是LLVM？
=======
&emsp;&emsp;LLVM项目是模块化、可重用的编译器以及工具链技术的集合。<!-- more --> 
![](1.png)  

Frontend:前端  
&emsp;&emsp;词法分析、语法分析、语义分析、生成中间代码  
Optimizer:优化器  
&emsp;&emsp;中间代码优化  
Backend:后端  
&emsp;&emsp;生成机器码
![](2.png)  

什么是Clang？
=========
&emsp;&emsp;LLVM项目的一个子项目，基于LLVM架构的C/C++/Objective-C编译器前端。

相比于GCC，Clang具有如下优点  
>编译速度快:在某些平台上，Clang的编译速度显著的快过GCC(Debug模式下编译OC速度比GGC快3倍)  
>占用内存小:Clang生成的AST所占用的内存是GCC的五分之一左右  
>模块化设计:Clang采用基于库的模块化设计，易于 IDE 集成及其他用途的重用  
>诊断信息可读性强:在编译过程中，Clang 创建并保留了大量详细的元数据 (metadata)，有利于调试和错误报告  
>设计清晰简单，容易理解，易于扩展增强  
![](3.png)  
  
