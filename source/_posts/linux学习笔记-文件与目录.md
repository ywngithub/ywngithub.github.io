---
title: linux学习笔记(2)—文件与目录
date: 2018-09-05 12:00:00
categories: Linux编程
---
一、引言
=====
&emsp;&emsp;本文将学习文件系统的其它性质和文件的性质，将从stat函数开始，逐个学习stat结构和的每一个成员以了解文件的所有属性，学习修改这些属性的各个函数（更改所有者，更改权限等）和Unix文件系统的结构以及符号链接。<!-- more -->

二、函数stat、fstat、fstatat和lstat
=====
&emsp;&emsp;①函数原型：  

```
#include <sys/stat.h>  
int stat(const char *restrict pathname, struct stat *restrict buf);  
int fstat(int fd, struct stat *buf);  
int lstat(const char *restrict pathname, struct stat *restrict buf;  
int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);    
```
&emsp;&emsp;所有4个函数的返回值：成功返回0，出错返回-1。  
&emsp;&emsp; **小知识：restrict是C99中一种类型限定符，作用是告诉编译器，对象已经被指针所引用，不能通过除该指针外所有其它直接或间接的方式修改该对象的内容。**  
&emsp;&emsp;②函数说明  
>1、 **stat函数** 将返回与此命名文件有关的信息结构。  
>2、 **fstat函数** 获得已在描述符fd上打开文件的有关信息。  
>3、 **lstat函数** 类似于stat,但当命名文件是一个符号链接时，lstat返回该符号链接的有关信息，而不是由该符号链接引用的文件信息。  
>4、 **fstatat函数** 为一个相对于当前打开目录(由fd参数指向)的路径名返回文件统计信息。  
>5、 **buf是一个stat指针** ，它指向一个已经定义的结构，函数将buf指向的结构进行填充，stat结构如下： 

``` 
struct stat{
    mode_t              st_mode;  /* file type & mode (permissions) */
    ino_t               st_ino;   /* i-node number (serial number) */
    dev_t               st_dev;   /* device number (file system) */
    nlink_t             st_rdev;  /* device number for special files */
    uid_t               st_uid;   /* user ID of owner */
    gid_t               st_gid;   /* group ID of owner */
    off_t               st_size;  /* size in bytes, for regular files */
    struct timespec     st_atime; /* time of last access */
    struct timespec     st_mtime; /* time of last modification */
    struct timespec     st_ctime; /* time of last file status change */
    blksize_t           st_blksize;/* best I/O block size */
    blkcnt_t            st_blocks; /* number of disk blocks allocated */
};
```
&emsp;&emsp; **timespec结构** 按照 **秒** 和 **纳秒** 定义了时间，包含以下两个成员：
> **time_t tv_sec;**  
> **long tv_nsec;**  

&emsp;&emsp;使用stat最多的地方应该是ls -l，可以获得一个文件的所有信息。

三、文件类型
=====
&emsp;&emsp;①文件类型
> 1、 **普通文件（regular file）** ：最常用的文件类型，这种文件包含某种格式的数据，至于是文本或二进制数据，对于Unix内核而言并无区别，对普通文件内容的解析由处理该文件的应用程序进行（例外是二进制可执行文件，为了执行程序，内核必须解析二进制文件，因此所有二进制可执行文件都有一个标准格式，才能保证内核找到程序和数据的加载位置）。  
> 2、 **目录文件（directory file）** ：目录其实也是一种文件，它包含了其它文件的名字以及指向这些文件有关信息的指针，任何具有读权限的进程都可以读取目录文件，但只有内核才可以直接写目录文件，进程必须使用特定函数才更改目录。  
> 3、 **块特殊文件（block special file）** ：这种类型的文件提供对设备（如磁盘）带缓冲的访问，每次以固定的长度访问。  
> 4、 **字符特殊文件（block special file）** ：这种类型的文件提供对设备不带缓冲的访问，每访问长度可变，系统中所有设备是块特殊文件或字符特殊文件。  
> 5、 **FIFO**：这种类型的文件用于进程间通信，也被为命令管道。  
> 6、 **套接字（socket）**：这种类型的文件用于进程间的网络通信。  
> 7、 **符号链接（symbolic link）** ：这种类型的文件指向另一个文件。  

&emsp;&emsp;②文件类型信息
&emsp;&emsp;包含在stat结构中的st_mode成员中，用下图中的宏来确定文件类型。

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.1.PNG)

&emsp;&emsp;③实例程序：读取命令行，对每一个命令行参数打印其文件类型。

```
#include "apue.h"
int main(int argc, char* argv[])
{   
    int i; 
    struct stat buf;
    char *ptr;
    for(i=1; i<argc; i++)
    {
         printf("%s:",argv[i]);
        if(lstat(argv[i], &buf) < 0)
        {
            err_ret("lstat error");
            continue;
        }
        if(S_ISREG(buf.st_mode))
            ptr = "regular";
         else if(S_ISDIR(buf.st_mode))
            ptr = "directory";
        else if(S_ISCHR(buf.st_mode))
            ptr = "character special";
        else if(S_ISBLK(buf.st_mode))
            ptr = "block special";
        else if(S_ISFIFO(buf.st_mode))
            ptr = "fifo";
        else if(S_ISLNK(buf.st_mode))
            ptr = "symbolic link";
         else if(S_ISSOCK(buf.st_mode))
            ptr = "socket";
        else
            ptr = "unknow mode";
         printf("%s\n",ptr);
    }
}
```

> **注释**:    
>行10：使用stat函数获取文件信息，而不是stat的原因是以便检测符号链接。  
>行15-30：使用宏S_ISxxx检测是哪种类型的文件。  
>
> **执行命令**：  
/a.out /etc/passwd /etc /dev/log /dev/tty /var/lib/oprofile/opd_pipe /dev/sr0 /dev/cdrom   
>
> **结果**：  
>/etc/passwd:regular  
>/etc:directory  
>/dev/log:symbolic link  
>/dev/tty:character special  
>/var/lib/oprofile/opd_pipe:lstat error: No such file or directory  
>/dev/sr0:block special  
>/dev/cdrom:symbolic link  
> **小知识：在命令行末端加入"\"可在第二行继续输入当前命令**。  


&emsp;&emsp;④不同类型文件的一个统计值

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.2.PNG)

&emsp;&emsp;可见 **普通文件** 是最主要的类型。

四、设置用户ID和组ID
=====
&emsp;&emsp;与一个进程相关联的ID有6个或更多（对于进程而言）：

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.3.PNG)


>1、 **实际用户ID和实际组ID** ：取值为口令文件中的登录项，通常在一个登录会话期间这些值并不改变，但是超级用户进程可以改变（进程最初执行时所用ID）。  
>2、 **有效用户ID、有效组ID和附属组ID** ：进程执行时对文件的访问权限（进程实际执行中所用ID）。  
>3、 **保存的设置用户ID和保存的设置组ID** ：这个概念涉及到可执行程序文件的设置用户ID位，当可执行程序文件passwd的设置用户ID位(s)已经设置时，非root用户进程(exec)启动passwd程序，则该进程的有效用户ID和保存的设置用户ID都将被设置为这个可执行程序文件的所有者(即root)，也就是暂时可以用root的权限来访问文件了。
>4、 **通常有效用户ID等于实际用户ID，有效组ID等于实际组ID** 。  
>5、 **每个文件有一个所有者和组所有者，分别由st_uid和st_gid指定** 。  



五、文件访问权限
=====
&emsp;&emsp;①	st_mode值（文件模式）包含了对文件的访问权限位（对于文件而言）：

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.4.PNG)

> 	1、当用路径访问一个文件时，每个目录都应该都具有执行权限。  
> 	2、目录的执行权限与读权限有区别，读权限允许读目录获取该目录的所有文件列表，执行权限是要执行一个文件时，该文件所在目录需要的权限。  
> 	3、对于一个文件的读权限决定了我们是否能够打开现有文件进行读操作，这与open函数的O_RDONLY和O_RDWR标志有关。  
> 	4、对于一个文件的写权限决定了我们是否能够打开现有文件进行写操作，这与open函数的O_WRONLY和O_RDWR标志有关。  
> 	5、为了在open函数中对一个文件指定O_TRUNC标志，必须对该文件具有写权限。  
> 	6、为了在一个目录中创建一个新文件，必须对该目录具有写权限和执行权限。  
> 	7、问了在一个目录中删除一个文件，必须对包含该文件的目录具有写权限和执行权限，对该文件本身的权限并不关心。   
> 	8、如果用7个exec函数中任何一个执行某文件，都必须对该文件具有执行权限，该文件必须是一个普通文件。  

&emsp;&emsp;②	进程每次打开、新建或删除一个文件时，内核将对该文件进行访问权限测试，文件的所有者（st_uid和st_gid）是文件的性质；有效用户ID、有效组ID和附属组ID则是进程的性质。
> 	1、若进程的有效用户ID是0（即超级用户），则允许访问。  
> 	2、若进程的有效用户ID等于文件所有者ID（即进程拥有此文件），那么如果所有者适当的访问权限位被设置，则允许访问，否则禁止访问。  
> 	3、若进程的有效组ID或附属组ID之一等于文件的组ID，那么如果所有者适当的访问权限位被设置，则允许访问，否则禁止访问。  
> 	4、若其它用户适当的访问权限位被设置，则允许访问，否则禁止访问。  

&emsp;&emsp;以上4个步骤顺序执行。  


六、新文件和目录的所有权
=====
&emsp;&emsp;使用open和create函数创建新文件时，并没有设置文件的用户ID和组ID，以后学习mkdir函数时就会了解到如何创建一个新目录，并设置所有权规则。 **新文件的用户ID将设置为进程的有效用户ID，而组ID将设置为进程的有效组ID或者它所在目录的组ID**。

七、函数access和faccessat
=====
&emsp;&emsp;当用open函数打开一个文件时，内核以进程的有效用户ID和有效组ID为基础进行访问测试，有时进程也希望按实际用户ID和实际组ID进行访问测试  
&emsp;&emsp;①	函数作用：按实际用户ID和实际组ID进行访问测试。  
&emsp;&emsp;②	函数原型：  

```
#include <unistd.h>
int access(const char *pathname, int mode);
int faccessat(int fd, const char *pathname, int mode, int flag);
```
&emsp;&emsp;返回值：0-成功，-1-失败。  
> **mode参数**：R_OK-测试读权限，W_OK-测试写权限，X_OK-测试执行权限。  
> **flag参数**：改变faccessat的行为，取值为AT_EACCESS时，访问检查用的是调用进程的有效用户ID和有效组ID，而不是实际用户ID和实际组ID。  

八、函数umask
====
&emsp;&emsp;①	函数作用：每个文件都有9个访问权限位，为进程设置st_mode创建屏蔽字，并返回之前的值。  
&emsp;&emsp;②	函数原型：  

```
#include <sys/stat.h>
mode_t umask(mode_t, cmask);
```
>cmask参数：由五①图中9个常量相"或"组成。

&emsp;&emsp;③	程序示例：

```
#include "apue.h"
#include "fcntl.h"

#define RWRWRW (S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH)

int main(void)
{
    umask(0);
    if(creat("foo", RWRWRW) < 0)
        err_sys("creat error for foo");

    umask(S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH);
    if(creat("bar", RWRWRW) < 0)
        err_sys("creat error for foo");

    exit(0);
}
```
> **执行(%表示shell响应)：**:  
>umask  
>%002  
>
>./a.out  
>ls -l foo bar  
>%-rw------- 1 sar 0 dec 7 21:20 bar  
>%-rw-rw-rw- 1 sar 0 dec 7 21:20 foo  
>
>umask  
>%002  

&emsp;&emsp;④	umask文件访问权限位  

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.5.PNG)

&emsp;&emsp;⑤	umask命令  
>umask--打印当前屏蔽字  
>umask -S--打印符号格式的屏蔽字  
>umask xxxx--修改屏蔽字  

九、函数chmod、fchmod和fchmodat
=====
&emsp;&emsp;①	函数作用：更改现有文件的访问权限。  
&emsp;&emsp;②	函数原型：  

```
#include <sys/stat.h>
int chmod(const char *pathname, mode_t mode);
int fchmod(int fd, mode_t mode);
int fchmodat(int fd, const char *pathname, mode_t mode, int flag);
```
&emsp;&emsp;返回值：0-成功，-1-失败。  
> **参数mode**：下图常量按位或

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.6.PNG)


十、粘着位
=====
&emsp;&emsp;如果可执行程序的这一位被设置，那么该程序第一次被执行，在其终止时， **程序正文（机器指令）的一个副本仍保存在交换区，使得下一次执行改程序时可以很快的载入内存** ，因此称为"粘着"，也就有了 **常量S_ISVTX** ，但当今Unix系统大多数都配置了 **虚拟存储系统** 以及 **快速文件系统** ，因此不再需要使用这项技术。

十一、函数chown、fchown、fchownat和lchown
=====
&emsp;&emsp;①	函数作用：更改文件的用户ID和组ID，如果参数owner和group有一个为-1，则对应的ID不变。  
&emsp;&emsp;②	函数原型：  

```
#include<unistd.h>
int chown(const char *path, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int lchown(const char *path, uid_t owner, gid_t group);
```
&emsp;&emsp;返回值：0-成功，-1-失败。

十二、文件长度
=====
&emsp;&emsp;①	stat结构中的 **成员st_size** 表示单位为字节的文件长度，此字段只对 **普通文件、目录文件和符号链接** 有意义。
>1、 **普通文件**：文件长度可以是0，开始读这种文件时，将得到文件结束标志。  
>2、 **目录文件**：文件长度是16/512的整倍数。  
>3、 **符号链接**：文件长度等于路径名的字符长度。  

&emsp;&emsp;②	 大多数现代Unix系统提供字段 **st_blksize和st_blocks** ，前一个是指 **对文件I/O较合适的块长度** ，后一个是指 **分配的实际512字节块数**。

十三、文件截断
=====
&emsp;&emsp;有时需要在文件尾端处截去一些数据以缩短文件，将文件长度截断为0是一个特例，在打开文件时使用O_TRUNC可以实现这一点，为了截断文件可以使用函数truncate和fturncoat。  
&emsp;&emsp;函数原型：

```
#include <unistd.h>
int truncate(const char *pathname, off_t length);
int ftruncate(int fd, off_t length);
```
&emsp;&emsp;函数作用：将一个现有文件长度截断为length，如果该文件以前长度大于length，则超过以外的数据就不能再访问，如果小于length，文件长度将增加，新增长度的数据为0。

十四、文件系统
=====
&emsp;&emsp;①	可以把一个磁盘分为一个或多个分区，每个分区可包含一个文件系统，i节点是固定长度的记录项，它包含有关文件的大部分信息，此磁盘、分区和文件系统以及具体的柱面组的i节点和数据块示意图如下：

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.7.PNG)
![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.8.PNG)
 
&emsp;&emsp;②	**i节点** 包含了文件的所有信息，文件类型、文件访问权限位、文件长度和指向文件数据块的指针等，有两项数据存放在目录项中， **文件名和i节点编号（数据类型为ino_t）**。

&emsp;&emsp;③	对于Linux系统，链接分为两种，分别是硬链接和符号链接，默认ln命令产生硬链接。
> **硬链接**：硬连接指通过索引节点来进行连接。在Linux的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号(Inode Index)。在Linux中，多个文件名指向同一索引节点是存在的。一般这种连接就是硬连接。硬连接的作用是 **允许一个文件拥有多个有效路径名（在stat结构中，链接计数包含在st_nlink成员中，基本系统数据类型为nlink_t，LINK_MAX限制了链接数的最大值）** ，这样用户就可以建立硬连接到重要文件，有防止 **"误删"** 的功能。其原因如上所述，因为对应该目录的索引节点有一个以上的连接。 **只删除一个连接并不影响索引节点本身和其它的连接，只有当最后一个连接被删除后，文件的数据块及目录的连接才会被释放。也就是说，文件真正删除的条件是与之相关的所有硬连接文件均被删除** 。

> **符号链接**：另外一种连接称之为符号连接（Symbolic Link），也叫软连接。软链接文件有 **类似于Windows的快捷方式** 。它实际上是一个特殊的文件。在符号连接中，文件实际上是一个文本文件，其中包含的有另一文件的位置信息。

***具体未搞懂***
---

十五、函数link、linkat、unlink、unlinkat和remove
=====
&emsp;&emsp;①link、linkat函数作用：创建一个指向现有文件的链接。  
&emsp;&emsp;函数原型：

```
#include <unistd.h>
int link(const char *existingpath, const char *newpath);
int linkat(int efd, const char *existingpath, const char *newpath, int flag);
```

&emsp;&emsp;②unlink、unlinkat函数作用：删除一个现有的目录项。  
&emsp;&emsp;函数原型：

```
#include <unistd.h>
int unlink(const char *pathname);
int unlinkat(int fd, const char *pathname, int flag);
```

&emsp;&emsp;③remove函数作用：解除对一个文件或目录的链接。  
&emsp;&emsp;函数原型：

```
#include <stdio.h>
int remove(const char *pathname);
```
>对于文件--remove与unlink功能相同。  
>对于目录--remove与rmdir功能相同。

&emsp;&emsp;④实例程序如下：打开一个文件，然后解除它的链接，睡眠15s后终止。

```
#include "apue.h"
#include <fcntl.h>
int main(void)
{
    if(open("tempfile", O_RDWR) < 0)
        err_sys("open error");
    if(unlink("tempfile") < 0)
        err_sys("unlink error");
    printf("unlink ok\n");
    sleep(15);
    printf("done\n");
    exit(0);
}
```
> **执行(%表示shell响应)：**  
>ls -l tempfile    查看文件大小  
>%-rw-r--r-- 1 yanwn yanwn 10974 9月   4 06:30 tempfile  
>
>df /home    检查可用空间  
>%文件系统          1K-块    已用     可用 已用% 挂载点  
>%/dev/sda1      20509264 8004032 11440376   42% /  
>
>./a.out &    后台运行  
>%3088    进程号  
>% unlink ok  
>
>ls -l tempfile    观察文件是否还存在  
>% ls: 无法访问'tempfile': 没有那个文件或目录    目录项已经被删除  
>
>df /home    再次检查可用空间  
>%文件系统          1K-块    已用     可用 已用% 挂载点  
>%/dev/sda1      20509264 8004032 11440376   42% /  
>%done    15s过后，进程结束  
>
>df /home    最后检查可用空间  
>%文件系统          1K-块    已用     可用 已用% 挂载点  
>%/dev/sda1      20509264 8004024 11440384   42% /  

&emsp;&emsp; **分析：unlink这种特点经常用来确保程序崩溃时，它所创建的临时文件也不会被保存下来，进程用open或create创建一个文件，然后调用unlink，因为该文件是打开的，所以不会被删除，只有当进程关闭该文件或终止时（内核关闭该进程打开的所有文件），该文件的内容将会被清除。如果pathname是符号链接，那么unlink删除的是本身，而不是该链接指向的文件，任何函数都不能删除链接所指向的文件。**

十六、函数rename和renameat
=====
&emsp;&emsp;函数作用：对文件或目录重命名。  
&emsp;&emsp;函数原型：  

```
#include <stdio.h>
int rename(const char *oldname, const char *newname)
int renameat(int oldfd, const char *oldname, int newfd, const char *newname);
```

十七、符号链接
=====
&emsp;&emsp;①符号链接是一个文件的间接指针，以下为各个函数对符号链接的处理情况示意图：  
![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/4.9.PNG)

&emsp;&emsp;②函数symlink和symlinkat用于创建一个符号链接，readlink和readlinkat用于打开链接本身，并读取链接地址。  
&emsp;&emsp;函数原型：

```
#include<unistd.h>
int symlink(const char*actualpath,const char *sympath);
int symlinkat(const char *actualpath,int fd,const char *sympath);
ssize_t readlink(const char* restrict pathname,char *restrict buf,size_t bufsize);
ssize_t readlinkat(int fd,const char* restrict pathname,char *restrict buf,size_t bufsize);
```
> 	1、symlink函数创建了一个指向actualpath的新目录项sympath。在创建此符号链接时，并不要求actualpath已经存在。并且actualpath和sympath并不需要位于同一文件系统中。  
> 	2、symlinkat函数与symlink函数类似，但sympath参数根据相对于打开文件描述符引用的目录(由fd指定)进行计算。如果sympath参数指定的是绝对路径或者fd参数设置了AT_FDCWD值，那么symlinkat就等同于symlink函数。  
> 	3、readlink和readlinkat函数组合了open、read和close的所有操作。如果函数成功执行，则返回读入buf的字节数。在buf中返回的符号链接的内容不以null字符终止。  
> 	4、当pathname参数指定的是绝对路径名或者fd参数的值为AT_FDCWD，readlinkat函数的行为与readlink相同。但是，如果fd参数是一个打开目录的有效文件描述符并且pathname参数是相对路径名，则readlinkat计算相对于由fd代表的打开目录的路径。  




十八、	文件时间及相关函数
=====
***暂时忽略***
---



十九、	函数mkdir、mkdirat和rmdir
=====
&emsp;&emsp;函数作用：mkdir、mkdirat用于创建目录，rmdir用于删除空目录。  
&emsp;&emsp;函数原型：  

```
#include <sys/stat.h>
int mkdir(const char *pathname, mode_t mode);
int mkdirat(int fd, const char *pathname, mode_t mode);
int rmdir(const char *pathname);
```


二十、函数chdir、fchdir
=====
&emsp;&emsp;每个进程都有一个当前工作目录，当用户登录Unix系统时，其当前工作目录通常是口令文件（/etc/passwd）中登录项的第6个字段--用户的起始目录。 **当前工作目录是进程的一个属性，起始目录是登录名的一个属性** 。  
&emsp;&emsp;函数作用：更改当前工作目录 **（只影响调用进程的本身）** 。  
&emsp;&emsp;函数原型：

```
#include<unistd.h>
int chdir(constchar*pathname);
int fchdir(int fd);
```


二十一、设备特殊文件
=====
&emsp;&emsp;st_dev和st_rdev这两个字段容易混淆：
>1、每个文件系统所在的存储设备都由 **主、次设备号** 表示，设备号所用数据类型为dev_t, **主设备号标识设备驱动程序，有时编码为与其通信的外设板，次设备标识特定的子设备** ，一个磁盘驱动器通常包含多个文件系统，在同一磁盘上各文件系统的主设备号通常相同，但次设备号不同。  
>2、用 **宏major和minor** 来访问主、次设备号。  
>3、 **系统中每个文件名关联的st_dev值是文件系统的设备号** 。  
>4、 **只有字符文件和块文件才有st_rdev值，此值包含实际设备的设备号** 。  

二十二、小结
=====
&emsp;&emsp;本文围绕stat函数，学习了stat结构中的每一个成员，了解了Unix文件和目录的各个属性，学习了有关于文件和目录的大部分函数。

