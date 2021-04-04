---
title: linux学习笔记(4)—标准I/O
date: 2018-09-21 21:58:07
categories: Linux学习笔记
---

一、引言
=======
&emsp;&emsp;本文学习标准I/O库，不仅是Unix，很多其它操作系统都实现了标准I/O，标准I/O库处理了很多细节，如缓冲区分配，已优化的块长度执行I/O等，用户无需担心使用正确的块长度，方便用户使用。<!-- more -->

二、流和FILE对象
============
&emsp;&emsp;①对于标准I/O库而言，所有操作都围绕"流"进行，当用标准I/O库打开一个文件时，那么一个流将与一个文件关联。  
&emsp;&emsp;②对于ASCII字符集，一个字符用一个字节表示，对于国际字符集（宽字符集），一个字符可用多个字节表示，标准I/O文件可用于单字符集或宽字符集，流的定向决定了所读、写的字符类型，当一个流初次创建时并没有被定向，只有两个函数可以改变流的定向：

>freopen函数清除一个流的定向。  
>fwide函数设置流的定向，不改变已定向的流。

函数原型：

```
#include <stdio.h>
#include <wchar.h>
int fwide(FILE *fp, int mode);
```
>mode<0试图将流指定为字节定向。

>mode=0不试图设置。   

>mode>0试图将流指定为宽定向。     

&emsp;&emsp;③打开一个流时，标准I/O函数fopen返回一个FILE对象的指针，该对象通常是一个结构，包含了标准I/O库为管理该流需要的所有信息，包括用于实际I/O的文件描述符、指向用于该流缓冲的指针、缓冲区长度、当前在缓冲区中的字符数以及出错标志。  
&emsp;&emsp;④为了引用一个流，需要将FILE指针作为参数传递给每个标准I/O函数，我们称指向FILE对象的指针（类型为FILE*）为文件指针。


三、标准输入、标准输出和标准错误
=============
&emsp;&emsp;对一个进程预定义了三个流，分别是标准输入、标准输出和标准错误，这些流引用的文件与文件描述符STDIN_FILENO、STDOUT_FILENO和STDERR_FILENO所引用的相同。  
&emsp;&emsp;这三个标准I/O流通过预定义文件指针stdin、stdout和stderr加以引用，定义在<stdio.h>中。

四、缓冲
=========
&emsp;&emsp;①标准I/O库提供缓冲的目的是尽可能的减少read和write调用的次数；还可以对每个I/O流自动的进行缓冲管理，从而避免应用程序考虑开辟缓冲区大小带来的麻烦。  
&emsp;&emsp;②标准I/O库有三种缓冲：  
>全缓冲：填满标准I/O缓冲区才进行实际I/O操作，对于驻留在磁盘上的文件通常是由标准I/O库实施全缓冲的，术语冲洗（flush）说明标准I/O缓冲区的写操作，可由标准I/O例程自动冲洗或者调用fflush函数冲洗一个流。  
>行缓冲：当在输入输出中遇到换行符时，标准I/O库执行I/O操作，只有写了一行之后才进行实际I/O操作，当流涉及一个终端（如标准输入和标准输出）时，通常使用行缓冲。  
>不带缓冲：标准错误stderr通常是不带缓冲的，可使得错误信息可以尽快的显示出来。  

&emsp;&emsp;③函数setbuf和setvbuf  
函数作用：更改缓冲类型/强制冲洗流。  
函数原型：

```
#include <stdio.h>
void setbuf(FILE *restrict fp, char *restrict buf);
void setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
int fflush(FILE *fp);
```

函数说明：
setbuf函数打开或关闭缓冲机制，参数buf必须指向一个长度为BUFSIZ的缓冲区，一般而言该流被设置为全缓冲，为了关闭缓冲，将buf设置为NULL。使用setvbuf，可以精确地说明所需的缓冲类型，由mode参数指定，_IOFBF(全缓冲)，_IOLBF(行缓冲) ，_IONBF(无缓冲)，size参数指定长度。fflush函数用于强制冲洗一个流（所有未写的数据都被传送至内核）。

五、打开/关闭流
============
&emsp;&emsp;①用下列3个函数打开一个标准I/O流：
函数原型：

```
#include <stdio.h>
FILE *fopen(const char *restrict pathname, const char * restrict type);
FILE *freopen(const char *restrict pathname, const char * restrict type, FILE * restrict fp);
FILE *fdopen(int fd, const char *type);
int fclose(FILE *fp);
```
函数说明：  
	fopen函数打开路径名为pathname的一个指定文件。  
 	freopen函数在一个指定的流上打开指定的文件，若该流已打开，则先关闭该流，若该流已定向，则使用freopen清除该定向，此函数一般用于将一个指定的文件打开为一个预定义的流：标准输入、标准输出或标准错误。  
 	fdopen取一个已有的文件描述符（可能从open、dup、dup2、fcntl、pipe、socket、socketpair或accept函数得到此文件描述符），并使一个标准I/O流与该描述符相结合，此函数一般用于创建管道和网络通信函数返回的描述符，因为这些特殊文件不能用fopen打开。  
 	type参数如下图：  
 ![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/5.1.PNG)  
使用字符b作为type的一部分，使得标准I/O系统可以区分文本文件和二进制文件，因为Unix系统并不对这两种文件进行区分，所以Unix系统环境指定b实际无作用。

六、读/写流
==========
&emsp;&emsp;①每次一个字符的I/O

```
int getc(FILE *fp);
int fget(FILE *fp);
int getchar(void); (=getc(stdin))

int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c); (=putc(int c, stdout))
```

getc和fgetc的区别是前者可被实现为宏，后者不能，意味着fgetc可得到其函数地址，这就允许将fgetc的地址作为一个参数传送给另一个函数，调用fgetc时间比getc长，因为调用函数时间通常长于调用宏。注意不管出错还是到达文件尾端，这些函数都返回同样的值，为了区分这两种不同的情况，必须调用ferror和feof。

```
int ferror(FILE *fp);
int feof(FILE *fp);
void clearer(FILE *fp);
```
在大多数实现中，为每个流在FILE对象中维护了两个标志：
>出错标志  
>文件结束标志

&emsp;&emsp;②每次一行的I/O

```
char *fgets(char *restrict buf, int n, FILE *restrict fp);
char *gets(char *buf);

int fputs(const char *restrict str, FILE *restrict fp);
int puts(const char *str);
```

gets从标准输入读，而fgets从指定的流读，fgets必须指定缓冲的长度n，此函数一直读到下一个换行符为止，读入的字符被送入缓冲区，该缓冲区以null字节结尾。gets不推荐使用，因为不能指定缓冲区长度，这样就可能造成缓冲区溢出，gets和fgets的另一个区别是前者并不将换行符存入缓冲区中。

七、二进制I/O
============
&emsp;&emsp;上面函数以一次一个字符或一行的方式进行读写操作，如果进行二进制I/O操作，那么更希望一次读/写一个完整的结构，如果使用getc或putc读写一个结构，那么必须循环通过整个结构，每次循环处理一个字节，会很麻烦而且效率低，如果使用fputs和fgets，那么因为fputs遇到null字节就停止，而在结构中可能含有null字节，所以不能使用它实现读结构的要求，如果输入数据中含有null字节或换行符，则fgets也不能正常工作，因此提供下列函数执行二进制I/O操作：

```
size_t fread(void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
size_t fwrite(const void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
```

八、定位流
=======
&emsp;&emsp;①方法1：ftell和fseek函数，他们都假设文件的位置可以存放在一个长整型中。

```
long ftell(FILE *fp);
int fseek(FILE *fp, long offset, int whence);
void rewind(FILE *fp);
```
&emsp;&emsp;对于一个二进制文件，其文件位置指示器从文件起始位置开始度量，并以字节为度量单位，ftell用于二进制文件时，返回的就是这种字节位置，为了用fseek定位一个二进制文件，必须指定一个字节offset，以及解释这种偏移量的方式。whence取值为：SEEK_SET表示从文件的起始位置开始，SEEK_CUR表示从当前文件位置开始，SEEK_END表示从文件尾端开始。
对于一个文本文件，文件位置不能用简单的字节偏移量来度量，因为在非Unix系统中，它们可能以不同的格式存放文本文件，为了定位一个文本文件，whence一定等于SEEK_SET，而且offset只有两种取值：0（后退至文件的起始位置），或是对该文件的ftell所返回的值。使用rewind函数也可以将一个流设置到文件的起始位置。  
&emsp;&emsp;②方法2：除了偏移量类型是off_t而非long以外，ftello函数与ftell相同，fseeko函数与fseek相同。

```
off_t ftello(FILE *fp);
int fseeko(FILE *fp, off_t offset, int whence);
```
可将off_set类型定义为长于32位。  
&emsp;&emsp;③方法3：fgetpos和fsetpos两个函数是ISO C标准引入的。

```
int fgetpos(FILE *restrict fp, fpos_t *restrict pos);
int fsetpos(FILE *fp, const fpos_t *pos);
```
fgetpos将文件位置指示器的当前值存入由pos指向的对象中，在以后调用fsetpos时，可以使用此值将流重新定义到该位置。

九、格式化I/O
=============
&emsp;&emsp;①格式化输出由5个printf函数处理。

```
int printf(const char *restrict format, …);
int fprintf(FILE *restrict fp, const char *restrict format, …);
int sprintf(char *restrict buf, const char *restrict format, …);
int dprintf(int fd, const char *restrict format, …);
int snprintf(char *restrict buf, size_t n, const char *restrict format, …);
```

&emsp;&emsp;printf将格式化的数据写到标准输出，fprintf写至指定的流，dprintf写至指定的文件描述符，sprintf写至数组buf中，且在该数组的尾端自动添加一个null字节，该字符不包含在返回值中，为了防止buf缓冲区溢出，引入了snprintf函数，可指定长度n。  

&emsp;&emsp;format参数中，转换说明以%开始，除转换说明外，格式字符串中其它字符将按原样输出，一个转换说明有4个可选择的部分，如下面方框中所示：  

>[flags][fldwidth][precision][lenmodifier]convtype

>【标志】【宽度】【精度】【长度】转化类型

flag参数取值如下：  
 ![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/5.2.PNG) 

&emsp;&emsp;fldwidth参数说明了最小字段宽度，转换后参数字符数若小于宽度，则用空格填充，字段宽度是一个非负十进制数，或者是一个（*）号。  
&emsp;&emsp;precision参数说明整形转换后最少输出数字位数，浮点数转化后小数点后的最少位数，字符串转换后最大字节数。精度是一个（.），其后跟随一个非负十进制数或一个（*）。  
&emsp;&emsp;lenmodified参数说明参数长度，取值如下：  

 ![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/5.3.PNG) 

&emsp;&emsp;convtype参数不是可选的，它控制如何解释参数，取值如下：

 ![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/5.4.PNG) 


&emsp;&emsp;②格式化输入由3个scanf函数处理。

```
int scanf(const char *restrict format, …);
int fscanf(FILE *restrict fp, const char *restrict format, …);
int sscanf(const char *restrict buf, const char *restrict format, …);
```

&emsp;&emsp;scanf族用于分析输入字符串，并将字符序列转换成指定类型的变量，在格式之后的各参数包含了变量的地址，用转换结果对这些变量赋值。  
&emsp;&emsp;format参数中，转换说明以%字符开始，有3个可选择的部分，如下所示：
>[*][fldwidth][m][lenmodifier]convtype

*参数用于抑制转化，按照说明的其余部分对输入进行转换，但转换结果并不存放在参数中（也就是跳过此数据）。  
fldwidth参数说明最大宽度（即最大字符数）。  
lenmodified参数说明要用转换结果赋值的参数大小，如同printf函数族中一样。  
convtype参数不是可选的，类似于printf函数族的转换类型字段，但两者有差别，一个差别是作为一种选项，输入中带符号的可以赋予无符号类型，例如，输入流中的-1被转换成4294967295。支持的转换类型如下：  
 
![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/5.5.PNG) 

m参数是赋值分配符，它可以%c、%s以及%[转化符，迫使内存缓冲区分配空间以接纳转换字符串，在这种情况下，相关的参数必须是指针地址，分配的缓冲区地址必须复制给该指针，如果调用成功，该缓冲区不在使用时，由调用者负责通过调用free函数来释放该缓冲区。

十、实现细节
==========
&emsp;&emsp;在Unix中，标准I/O库最终都要调用内核I/O接口函数，每个标准I/O流都有一个与其相关联的文件描述符，可以对一个流调用fileno函数以获得其描述符。

```
#include <stdio.h>
int fileno(FILE * fp);
```
如果需要调用dup或fcntl等函数，则需要此函数转化。




十一、临时文件
==============
&emsp;&emsp;ISO C标准I/O库提供了两个函数以帮助创建临时文件。

```
#include <stdio.h>
char *tmpnam(char *ptr);
FILE *tmpfile(void);
```
&emsp;&emsp;tmpnam函数产生一个与现有文件名不同的一个有效文件名字符串，每次调用它时，都产生一个不同的文件名，一般在/tmp下，最多调用次数是TMP_MAX。
tmpfile创建一个临时二进制文件（类型wb+），在关闭该文件或程序结束时将自动删除这种文件，注意，Unix对二进制文件不进行特殊区分。
程序示例如下：

```
#include "apue.h"
int main(void)
{
        char name[L_tmpnam], line[MAXLINE];
        FILE *fp;
        printf("%s\n", tmpnam(NULL));
        tmpnam(name);
        printf("%s\n", name);
        if((fp = tmpfile()) == NULL)
                err_sys("tmpfile error");
        fputs("one line of output\n", fp);
        rewind(fp);
        if(fgets(line, sizeof(line), fp) == NULL)
                err_sys("fgets error");
        fputs(line, stdout);
        exit(0);
}
```
>执行：./a.out  
>/tmp/fileUMleJc  
>/tmp/fileSX9RnR  
>one line of output  

&emsp;&emsp;tmpfile函数经常使用的标准Unix技术是先调用tmpnam函数产生一个唯一的路径名，然后，用该路径名创建一个文件，并立即unlink它，对一个文件解除链接并不删除其内容，关闭文件时才删除其内容，而关闭文件可以是显式的，也可以在程序终止时自动进行。


十二、内存流
============
&emsp;&emsp;可以看到，标准I/O库把数据缓存在内存中，因此每次一个字符和一行字符的I/O更有效，内存流就是标准I/O流，虽然使用FILE指针进行访问，但其实没有底层文件，所有的I/O都是通过在缓冲区与主存来回传送字节来完成的，即便这些流看起来更像文件流，其实它们的某些特征更适用于字符串操作。有3个函数用于创建内存流：

```
#include <stdio.h>
FILE *fmemopen(void *restrict buf, size_t size, const char *restrict type);
```
fmemopen函数允许调用者提供缓冲区用于内存流，buf参数指向缓冲区的开始位置，size参数指定了缓冲区字节的大小，若果buf为空，fmemopen函数分配size字节数的缓冲区，在这种情况下，当流关闭时缓冲区会被释放。
type参数控制如何使用流：

![图片](https://raw.githubusercontent.com/ywngithub/MyPostImage/master/5.6.PNG)

``` 
#include <stdio.h>
FILE *open_memstream(char **bufp, size_t *sizep);
#include <wchar.h>
FILE *open_wmemstream(wchar_t **bufp, size_t *sizep);
```
open_memstream函数创建的流是面向字节的，open_wmemstream函数创建的流是面向宽字节的。

十三、	小结
=============
&emsp;&emsp;大多数Unix应用程序都是用标准I/O库，它使用缓冲技术，而他正是产生许多问题，引起许多混淆的地方。














