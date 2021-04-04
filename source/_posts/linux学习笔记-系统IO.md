---
title: linux学习笔记(2)—系统I/O
date: 2018-09-02 12:00:00
categories: Linux学习笔记
---
一、引言
=====
&emsp;&emsp;①可用的 **文件I/O函数** ：打开文件、读文件、写文件。  
&emsp;&emsp;②常用5个函数： **open、read、write、lseek、close** （不同缓冲长度影响将不同）。  
&emsp;&emsp;③ **不带缓冲I/O** 是指每个read、write都调用内核中的一个系统函数，需要传入缓存区参数。  
&emsp;&emsp;④ **原子操作** 及 **dup、fcntl、sync、fsync、ioctl** 函数。 <!-- more -->

二、文件描述符 
===
&emsp;&emsp;①当打开一个现有文件或创建一个新文件时，内核向进程返回一个 **文件描述符** ，文件描述符（file descriptor）是内核为了高效管理已被打开的文件所创建的 **索引** ，其值是一个非负整数，用于指代被打开的文件，所有执行I/O操作的系统调用都通过文件描述符，程序刚刚启动的时候：
> **0是标准输入（STDIN_FILENO）  
>1是标准输出（STDOUT_FILENO）  
>2是标准错误（STDERR_FILENO）**  
>如果此时去打开一个新的文件，它的文件描述符会是3。  

&emsp;&emsp;②文件描述符变化范围： **0~OPEN_MAX-1** 。

三、函数open和openat
===
&emsp;&emsp;①open、openat打开/创建文件，成功则返回文件描述符，失败返回-1。  
&emsp;&emsp;②open、opena函数原型。  
&emsp;&emsp; **#include<fcntl.h>**  
&emsp;&emsp; **int open(const char *path, int oflags, …, mode_t mode);**  
&emsp;&emsp; **int openat(int fd, const char *path, int oflags, …, mode_t mode);**  
&emsp;&emsp;path-路径名（绝对路径）或者文件名（相对路径）。  
&emsp;&emsp;oflags-打开文件采取的动作
>以下是必选参数（只能一个）：  
>O_RDONLY(只读)  
>O_WRONLY（只写）  
>O_RDWR（可读可写）  

>以下是可选参数：  
>O_APPEND(每次写操作都写入文件的末尾)  
>O_CREAT(如果指定文件不存在，则创建这个文件)  
>O_EXCL(如果要创建的文件已存在，则返回-1，并且修改errno的值)  
>O_TRUNC(如果文件存在，并且以只写/读写方式打开，则清空文件全部内容)  
>O_NOCTTY(如果路径名指向终端设备，不要把这个设备用作控制终端)  
>O_NONBLOCK(如果路径名指向 FIFO/块文件/字符文件，则把文件的打开和后继I/O设置为非阻塞模式(nonblocking mode))  

&emsp;&emsp;fd-区别open、openat，有3种可能性:  
>path是绝对路径名，此时fd无用，open=openat。  
>path是相对路径名，fd指定相对路径起始。  
>path是相对路径名，fd=AT_FDCWD，相对路径起始=当前工作目录。  

&emsp;&emsp;③open、openat函数返回的文件描述符一定是最小未用的。  
&emsp;&emsp;④openat可以让线程使用相对路径名打开目录中的文件，而不受限于当前目录。  

四、函数creat
===
&emsp;&emsp;①creat函数用于创建新文件，成功则返回文件描述符，失败返回-1。  
&emsp;&emsp;②creat函数原型。  
&emsp;&emsp; **int creat (const char *path, mode_t mode);**  
&emsp;&emsp;③类似于open。  

五、函数close
===
&emsp;&emsp;①close函数用于关闭文件，成功则返回文件描述符，失败返回-1。  
&emsp;&emsp;②close函数原型。  
&emsp;&emsp; **int close(int fd);**  
&emsp;&emsp;③当一个进程终止时，内核自动关闭它打开的所有文件。  

六、函数lseek
===
&emsp;&emsp;①lseek用于使光标偏移，成功则返回新的偏移量（初始默认为0），失败返回-1。  
&emsp;&emsp;②lseek函数原型。  
&emsp;&emsp; **off_t lseek(int fd, off_t offset, int whence);**  
>whence=SEEK_SET，该文件偏移至文件开始处offset个字节。  
>whence=SEEK_CUR，该文件偏移至当前偏移量加offset。  
>whence=SEEK_END，该文件偏移至当文件长度加offset。  

&emsp;&emsp;③检测当前偏移量 **off_t currpos = lseek(fd, 0, SEEK_CUR);**  
&emsp;&emsp;④检测标准输入能否设置偏移量。

```
#include "apue.h"
int main(void)
{
    if(lseek(STDIN_FILENO, 0, SEEK_CUR) == -1)
        printf("cannot seek!");
    else
        printf("seek OK!");
    exit(0);
}
```
&emsp;&emsp;应该测试返回值是否为-1，而不是<0，因为某些设备允许负的偏移量，文件偏移量可以大于文件的当前长度。

七、函数read
===
&emsp;&emsp;①read函数用于打开文件读数据,成功返回字节数，到达文件尾返回0，失败返回-1。  
&emsp;&emsp;②read函数原型。  
&emsp;&emsp; **ssize_t read(int fd, void *buf, size_t nbytes);** 



八、函数write
===
&emsp;&emsp;①write函数用于打开文件写数据，成功返回写入字节数，失败返回-1。  
&emsp;&emsp;②write函数原型。  
&emsp;&emsp; **ssize_t write(int fd, const void *buf, size_t nbytes);**  
&emsp;&emsp;③write返回值通常等于nbytes，它的一个出错原因是磁盘被写满，或者超过了给定一个进程的文件长度限制。  
&emsp;&emsp;④对于普通文件，写操作从当前偏移量开始，如果打开文件指定了O_APPEND选项，则每次写操作前，将文件偏移量设置到文件尾端，一次写成功后，该文件偏移量增加实际写的字节数。  

九、I/O的效率
===
&emsp;&emsp;①只使用write和read函数将标准输入复制到标准输出。  

```
#include "apue.h"
#define BUFFSIZE 4096
int main(void)
{
    int n;
    char buf[BUFFSIZE];
    while((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0)
        if(write(STDIN_FILENO, buf, n) != n)
            err_sys("write error!");
    if(n<0)
        err_sys("read error!");
}
```
&emsp;&emsp;②磁盘块长度由 **st_blksize** 表示，缓冲区 **BUFFSIZE** 使用4096（等于st_blksize）时，进程使用系统CPU时间最短。



十、文件共享
===
&emsp;&emsp;①Unix系统支持不同进程间共享打开文件 **（多个进程同时读）**。  
&emsp;&emsp;②内核使用三种数据结构表示打开文件，分别是 **文件描述符表**、 **文件表**和 **V节点表**。  
&emsp;&emsp;每个 **进程** 在 **进程表**中都有一个 **记录项**，记录项中包含一张 **打开文件描述符表** ，每个描述符占用一项。与每个文件描述符相关联的是：
>(a)	文件描述符标志。  
>(b)	指向一个文件表项的指针。  

&emsp;&emsp;内核为所有打开文件维持一张 **文件表** ，每个文件表项包含:  
>(a)	文件状态标志(读、写、添写、同步和非阻塞等)。  
>(b)	当前文件偏移量。  
>(c)	指向该文件 V 节点表项的指针。  

&emsp;&emsp;每个打开文件都有一个v节点结构(v-node)，v节点包含了 **文件类型** 和对此文件进行各种操作的 **函数指针** 。v节点还包含了从磁盘读取的 **i节点(i-node)** 的信息，i节点信息包含了 **文件的所有者、文件长度、文件所在的设备、指向文件的实际数据块在磁盘上所在位置的指针** 等。  
&emsp;&emsp;图1表示了三张表之间的关系，该进程有两个不同的打开文件，分别是标准输入和标准输出。

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/3.1.PNG)

 **图1:一个进程打开两个文件的内核数据结构**

&emsp;&emsp;图2表示两个进程打开同一个文件的内核数据结构，假定第一个进程在文件描述符3上打开该文件，而另一个进程在文件描述符4上打开该文件，则打开该文件的 **每个进程都得到一个文件表项** ，但对一个给定的文件 **只有一个v节点表项**。

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/3.2.PNG)

 **图2:两个进程打开一个文件的内核数据结构**  

&emsp;&emsp;③对前面所述操作的进一步说明。
>(a)每个进程都有自己的对打开文件的 **当前偏移量**。  
>(b)在完成每个write后,在文件表项中的当前文件偏移量即增加所写的字节数。如果这使当前文件偏移量超过了当前文件长度，则在i节点表项中的当前文件长度被设置为当前文件偏移量。  
>(c)若一个文件用lseek定位到文件当前的尾端，则文件表项中的当前文件偏移量被设置为i节点表项中的当前文件长度。 **（这与O_APPEND标志打开文件是不同的，使用lseek定位到文件尾端处后，下次调用write写数据不一定是写在该文件的真正尾端，因为lseek和write两个不是原子操作，中间可以有另一个进程已使文件长度变长了。）**   
>(d)如果用O_APPEND 标志打开了一个文件，则相应标志被设置到文件表项的文件状态标志中， **每次对这种具有添写标志的文件执行写操作时，在文件表项中的当前文件偏移量首先被设置成i节点表项中的文件长度，这就使得每次写的数据都添加到文件的当前尾端处。**  
>(e)lseek 函数只修改文件表项中的当前文件偏移量，没有进行任何I/O操作。  

&emsp;&emsp;④以上过程可使得多个进程同时 **读取** 同一个文件，每个进程都有它自己的文件表项和当前偏移量。  



十一、原子操作
===
&emsp;&emsp;①任何多于一个函数调用的操作都不是原子操作，因为两个函数调用之间，内核有可能会临时挂起进程。  
&emsp;&emsp;②Unix系统提供了一种原子操作方法，即在打开文件时设定 **O_APPEND标志** ，这样做使得每次写操作之前，都将进程的当前偏移量设置为文件尾端，于是每次写操作的时候就不用lseek函数了。  
&emsp;&emsp;③函数pread和pwrite。  
&emsp;&emsp; 	调用pread相当于先调用lseek后再调用read，但调用pread时，此过程无法被打断，并且不更新当前文件偏移量。  
&emsp;&emsp; 	调用pwrite相当于先调用lseek后再调用write。  
&emsp;&emsp;④ **一般而言，原子操作是指由多步组成的一个操作，如果该操作原子的执行，则要么执行完所有步骤，要么一步也不执行，原子操作也就是能被打断的最小操作。**








十二、函数dup和dup2
===
&emsp;&emsp;①两个函数均用来复制一个现有的文件描述符，成功返回新的文件描述符，失败返回-1。  
&emsp;&emsp;②函数原型。  
&emsp;&emsp; **#include <unistd.h>**  
&emsp;&emsp; **int dup(int fd);**  
&emsp;&emsp; **int dup2(int fd, int fd2);**  
&emsp;&emsp;③dup函数返回的一定是当前可用数值最小的文件描述符，dup2函数可以指定新的文件描述符。  



十三、函数sync、fsync和fdatasync
===
&emsp;&emsp;①Unix系统实现设有高速缓冲区，大多数磁盘I/O都通过缓冲区进行，当我们向文件写入数据时，内核通常先将数据复制到缓冲区，然后排入队列，晚些时候再存入磁盘，这种方式成为延迟写，如果内核要重用缓冲区，它会把所有延迟写数据块写入磁盘。  
&emsp;&emsp;②为了保证磁盘与缓冲区数据一致，Unix系统提供了sync、fsync和fdatasync函数。  
>sync函数：只是将修改后的数据排入写队列，然后返回，不等待写入磁盘。  
>fsync函数：只对文件描述符fd指定的文件起作用，并等待写入磁盘。  
>fdatasync函数：类似于fsync，但它只影响文件的数据部分和同步更新文件的属性。  



十四、函数fcntl
===
&emsp;&emsp;①用于改变已经打开文件的属性，成功返回由cmd决定，失败返回-1。  
&emsp;&emsp;②函数原型。  
&emsp;&emsp; **#include <fcntl.h>**  
&emsp;&emsp; **int fcntl(int fd, int cmd, …/* int arg */);**  
&emsp;&emsp;cmd有11种取值，这里未列出。 



十五、函数ioctl
===
&emsp;&emsp;①ioctl是设备驱动程序中对设备的I/O通道进行管理的函数，即对设备的一些特性进行控制，例如串口的传输波特率、马达的转速等等，需要驱动程序提供对ioctl的支持。  
&emsp;&emsp;②函数原型。  
&emsp;&emsp; **#include <sys/ioctl.h>**  
&emsp;&emsp; **int ioctl(int d,int request,....);** 



十六、小结
===
&emsp;&emsp;本文学习了文件描述符的定义和基本作用，以及Unix系统提供的基本I/O函数。在多个进程读取文件时，了解了文件共享机制，为了防止多个进程同时写同一文件带来的干扰，学习了原子操作，最后还学习了fcntl和ioctl函数。



