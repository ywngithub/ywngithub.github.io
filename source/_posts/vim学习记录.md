---
title: vim学习记录
date: 2021-03-25 18:08:09
tags: vim
categories: vim
---
&emsp;&emsp;vim是一个字符终端编辑器，熟练使用可提高工作效率。
 <!-- more -->


单个单词命令
============
>
x          删除  
i           插入  
a          增加  
0          回到行首  
h          光标左移  
j           光标下移  
k          光标上移  
l           光标右移  
%         跳括号（用于查找括号是否匹配）  
y          复制命令  
v          可视模式  
A          调到行尾输入  
I           跳到行首输入  
=          缩进  


组合命令
================
>
助记：d(delete)      w(word)       $(行末)  
dd[p]      删除一行[粘贴]  
de          删除到行末（保留最后字符）  
dw         删除单词  
d$          删除到行末（包含最后字符）  

>
助记：c(change)      w(word)       $(行末)  
cw          替换一个单词  
c$           替换光标至行末  
ce           替换单词（待比较与cw的区别）  
  
>
u              撤销  
U             撤销一行  
Ctrl + r     前进  

>
:行号        跳转到指定行  

>
/  + n/N     搜索字符，向下/向上  
? + n/N     搜索字符，向上/向下  

>
Ctrl + o      跳向旧位置  
Ctrl +  i      跳向新位置  

替换相关
===========
>
:%s/old/new/g  


其它辅助命令
===============
>
grep -nr keyword *  
vim +linenumbers filepath     例如：vim +8 src/comm_handler.h  
:sp   	上下分割窗口   
:vs    	左右分割窗口  
ctrl+w	切换窗口  
:only         关闭其它窗口保留主窗口  
shirt + *     向下查找相同的单词  
shirt + #     向上查找相同的单词  
:! cmd       在vim中执行命令  

自动补全设置
========
>
sudo apt-get install vim-addon-manager   
sudo apt-get install vim-youcompleteme  
vim-addons install youcompleteme  
