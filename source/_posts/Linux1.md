---
title: Unix笔记1—基础知识
date: 2018-08-30 20:15:34
tags: 学习
categories: 技术
---
一、引言
=====
&emsp;&emsp;操作系统为所运行的程序提供服务，典型服务有：
>①执行新程序  
>②打开文件  
>③读取文件  
>④分配存储区  
>⑤获取系统时间  

二、Unix体系结构
=====
&emsp;&emsp;操作系统是一种软件，它控制计算机系统资源，提供程序运行环境，通常称其为 **内核（kernel）** ,因为它相对较小，而且处于环境的核心。内核接口称为 **系统调用，公用函数库** 建立于系统调用之上，应用程序可以使用 **系统调用** 和 **公用函数库** , **Shell** 是一个特殊的应用程序，为运行其它应用程序提供一个接口。

三、登录
=====
&emsp;&emsp;登录Unix时，必须键入 **用户名** 和 **密码** ，存在于 **/etc/passwd** 文件。

四、shell
=====
&emsp;&emsp;shell是一个 **命令行解释器** ，它读取用户输入，然后执行命令，输入来自终端或者文件（shell脚本）。

五、文件和目录
=====

&emsp;&emsp;① **文件系统** 是目录和文件的一种层次结构，起点是根目录（root），符号为"\"。  
&emsp;&emsp;② **目录** 是一个包含目录项的文件，目录项=文件名+文件属性，文件属性包含文件类型（普通文件/目录）、文件大小、文件所有者、文件权限及文件最后修改的时间。   
&emsp;&emsp;③ **文件名** 是目录项中的各个名字，"/"和" "不能出现在文件名中，创建新目录是会自动创建两个文件名，分别是"." **当前目录** 和".." **上一目录** ，处于根目录时，"."=".."。   
&emsp;&emsp;④ **路径名** 是由斜线分割的一个或多个文件名组成的序列，分别 **绝对路径** 和 **相对路径** 。   
&emsp;&emsp;⑤ **列出目录** 中所有文件名的代码如下： 

```
#include "apue.h"
#include "dirent.h"

int main(int argc, char *argv[]) 
{
  DIR *dp;
  struct dirent *dirp;
  if(argc != 2)
    err_quit("usage: ls directory_name");
  if((dp = opendir(argv[1])) == NULL)
    err_sys("can't open %s", argv[1]);
  while((dirp = readdir(dp)) != NULL)
    printf("%s\n",dirp->d_name);
  closedir(dp);
  exit(0);
}
```
> **注释** ：  
>行1：apue.h为作者自定义头文件，需先下载至系统。  
>行2：系统头文件，/user/include/dirent.h，可以使用 **dirent** 结构和 **opendir** 、 **readdir** 函数。  
>行4：ISO C风格的main函数声明， **argc参数** 是传入命令行的字符串数目， ***argv[]** 是具体的命令行字符。  
>行6：opendir将返回DIR结构指针。  
>行7：readdir将返回dirent结构指针。  
>行8、9：传入命令行字符串数目错误，输出提示信息。  
>行10、11：传入路径为空时，输出提示信息。  
>行12、13：读取每个目录项并打印。  
>行14：释放目录  
>行15：exit终止程序，参数0表示正常结束，1-255表示出错。  

> **执行** ：  
>使用 **cc *.c** 进行编译（cc=gcc），默认生成 **a.out** 文件，运行命令行 **./a.out /dir** ，即可列出dir目录下所有文件名，dir为所需目录名。

六、输入和输出
=====

&emsp;&emsp;① **文件描述符** 是一个小的非负整数，内核用来标定一个特定进程正在访问的文件。当内核打开一个文件或创建一个文件时，它都返回一个文件描述符，在读/写文件时使用这个文件描述符。  
&emsp;&emsp;② **标准输入、标准输出、标准错误**   
当运行一个新程序时，shell将打开以上三个文件描述符，一般这三个文件描述符都指向终端，shell可以将这些文件描述符重新定向到某个文件，如：ls>file.list。  
&emsp;&emsp;③ **不带缓冲的I/O**   
函数open、read、lseek、close提供了不带缓冲的I/O，这些函数都使用文件描述符。  
&emsp;&emsp;④ **复制任一Unix普通文件** 代码如下： 

```
#include "apue.h"
#define BUFFSIZE 4096
int main(void)
{
   int n;
   char buf[BUFFSIZE];
   while((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0)
      if(write(STDOUT_FILENO, buf, n) != n)
         err_sys("write error!");
   if(n<0)
      err_sys("read error!");
    exit(0);
}
```
> **注释：**  
>行7、8、9：常量STDIN_FILENO和STDOUT_FILENO是POSIX标准的一部分，定义在<unistd.h>中，它们指定了标准输入和输出的文件描述符，值分别为0和1，read函数返回读取的字节数，读到文件尾端时将返回0，如果读错误将返回-1。  

> **执行：**  
>./a.out > data ：输入是终端（ctrl+D结束），输出是data文件。  
>./a.out <infile >outfile：输入是infile文件，输出是outfile文件。  

&emsp;&emsp;⑤ **标准I/O**
标准I/O函数为那些不带缓冲的I/O函数提供了一个 **带缓冲的接口** ，使用标准I/O函数无需担心选取最佳缓冲区大小，标准I/O函数还简化了输入行的处理，例如：fgets函数读取一个完整的行，read函数读取指定字节数，printf是我们最熟悉的标准I/O函数之一，<stdio.h>包含了所有标准I/O函数的原型。  
&emsp;&emsp;⑥ **将标准输入复制到标准输出** 代码如下：  

```
#include "apue.h"
int main(void)
{
   int c;
   while((c = getc(stdin)) != EOF)
      if(putc(c, stdout) == EOF)
         err_sys("output error!");
      if(ferror(stdin))
         err_sys("input error!");
      exit(0);
}
```
> **注释：**  
>行5、6：函数getc一次读取一个字符，putc显示在屏幕上，常量EOF（end of file）定义在<stdio.h>中，表示文件结束标志（Ctrl+D），值为-1，stdin和stdout也定义在<stdio.h>，分别表示标准输入和标准输出。  

> **执行：**  
>./a.out  

七、程序和进程
=====

&emsp;&emsp;① **程序** 是存储在磁盘中的可执行文件，内核使用 **exec** 函数，将程序 **读入内存** ， **并执行程序** 。  
&emsp;&emsp;② **进程和进程ID**   
程序的执行实例被称为 **进程（process）** ，某些操作系统用 **任务（task）** 表示正在被执行的程序，Unix系统确保每个进程都有一个唯一的数字标识符，成称为进程ID，它总是一个非负的整数。  
&emsp;&emsp;③ **打印进程ID号代码如下：**  

```
#include "apue.h"
int main(void)
{
   printf("hello world form process ID %ld\n",(long)getpid());
   exit(0);
}
```
> **注释：**  
>getpid函数用于返回进程ID值，类型为pid_t类型。  

> **执行：**  
>./a.out  

&emsp;&emsp;④ **进程控制。**  
有3个用于进程控制的主要函数， **fork、exec、waitpid（exec有7种变体）**   
&emsp;&emsp;⑤ **从标准输入读取命令并执行** 代码如下：  

```
#include "apue.h"
#include "sys/wait.h"

int main(void)
{
   char buf[MAXLINE];
   pid_t pid;
   int status;
   printf("%% ");
   while(fgets(buf, MAXLINE, stdin) != NULL)
   {
      if(buf[strlen(buf) - 1] == '\n')
      {
        buf[strlen(buf) - 1] = 0;
      }
      if((pid = fork()) < 0)
      {
         err_sys("fork error!");
      }
      else if(pid == 0)
      {
         execlp(buf, buf, (char*)0);
         err_ret("couldn't execute: %s", buf);
         exit(127);
      }
      if((pid = waitpid(pid, &status, 0)) < 0)
        err_sys("waitpid error");
      printf("%% ");
   }
   exit(0);
```
> **注释：**  
>行12、13、14、15：因为fgets每一行都以换行符终止，而execlp函数要以null结尾，因此要替换一下。  
>行16：调用fork创建一个新进程，新进程是调用进程的一个副本，新进程称为 **子进程** ，调用进程称为 **父进程** ，fork对父进程返回子进程的ID，对子进程则返回0，fork调用一次，返回两次。  
>行22：子进程调用execlp（从PATH 环境变量中查找文件并执行， **执行成功函数将不会返回，失败返回-1** ）执行从标准输入的命令，用新的程序文件替换子进程原先执行的程序文件。  
>行27：父进程通过调用waitpid等待子进程执行完毕，其参数1指定要等待的进程，waitpid返回子进程的终止状态（status变量）。  

&emsp;&emsp;⑥ **线程和线程ID** 
一个进程内的所有线程共享同一个地址空间、文件描述符、栈及其进程相关的属性，所以各线程需要采集同步措施，以避免不一致性，线程也有ID，但只在本身进程内起作用。  

八、出错处理
=====
&emsp;&emsp;当Unix系统函数出错时，一般返回一个负值，而 **整形变量errno** 被设置为具有特定信息的值，文件<errno.h>定义了errno以及它可以被赋的值，POSIX和ISO C将errno定义为一个符号，它扩展为一个可修改的 **整形左值** ，它可以是一个包含出错编号的 **整数** ，也可以是一个返回出错编号指针的函数，使用应该注意两项：1、如果没有出错，其值不会被清除，因此当函数返回值表明错误时，才需要去检验其值。2、任何函数都不会将errno的值置0。  
&emsp;&emsp;C标准定义了两个函数，用于打印出错信息。  
 **#include <string.h>**  
 **char *strerror(int errnum);**   //返回错误字符串指针，通常errnum就是errno  
 **#include <stdio.h>**  
 **void perror(const char *msg);**   //首先输出msg指向的字符串，然后添加冒号和空格，接着输出errno值的出错消息，最后换行。  
实例代码如下：  

```
#include "apue.h"
#include <errno.h>
int main(int argc, char *argv[])
{
   fprintf(stderr, "EACCES: %s\n",strerror(EACCES));
   errno = ENOENT;
   perror(argv[0]);
   exit(0);
}
```
九、用户标识
=====
&emsp;&emsp;① **用户ID** 是一个数值，用户ID为0的用户是 **根用户** 和 **超级用户** 。  
&emsp;&emsp;② **组ID** 是一个数值，组用户用于将若干用户集合到项目或部门中去，这种机制允许同组的各个成员之间共享资源，组文件通常是/etc/group。  
&emsp;&emsp;③ **getuid** 和 **getgid** 用于获取用户ID和组ID。  
&emsp;&emsp;④ **附属组ID**。

十、信号
=====
&emsp;&emsp;信号用于通知进程发生了某种情况，进程有以下三种方式处理：
>忽略信号  
>按系统默认方式处理  
>提供一个函数  

很多情况都会产生信号，终端键盘有两种产生信号的办法，分别称为 **中断键（Delete或Ctrl+C）** 和 **退出键（Ctrl+\）** ，他们被用于终止当前运行的进程，另一种产生信号的办法是调用 **kill函数** ，在一个进程中调用此函数可向另一个进程发送信号，信号函数是 **signal** 。

十一、时间值
=====
&emsp;&emsp;Unix使用过两种不同的时间值：  
> **日历时间：** 该值来自UTC，1970-01-01 00:00:00，这个时间所经过的秒数值，系统基本数据类型 **time_t** 用于保存这种时间。    
> **进程时间：** 也称为CPU时间，用以度量进程使用的中央处理器资源，以时钟滴答计算，系统基本数据类型 **clock_t** 保存这种时间，使用sysconf可以得到每秒时钟滴答数。  

&emsp;&emsp;Unix系统为一个进程维护了三个进程时间值：  
> **时钟时间**---进程运行的时间总量。  
> **用户CPU时间**---执行用户指令所需时间。  
> **系统CPU时间**---该进程执行内核程序所经历的时间。  

十二、小结
===
&emsp;&emsp;本文快速浏览了unix系统的全局，对Unix系统有了一个基本印象。
