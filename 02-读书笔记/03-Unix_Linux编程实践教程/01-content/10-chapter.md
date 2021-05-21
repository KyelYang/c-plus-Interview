# 第10章 I/O重定向和管道

## 本章主要思路
本章将关注进程间通信的一种特殊形式：输人/输出重定向和管道(I/O redirection andPipes)。首先将介绍在编写shell脚本时I/O重定向和管道所起的作用。然后，本章将介绍操作系统中对I/O重定向的支持。最后，写一个程序来改变进程的输人和输出流。  

### shell脚本的特性和功能
### 标准I/O与重定向的若干概念（了解原理）
### 将stdin定向到文件的具体方法
这些例子显示了程序如何将标准输入重定向到文件。实际上，如果程序希望读取文件，它直接打开文件就可以了，根本不需要将标准输人重定向到文件。  

这些例子的真正意义在于说明一个程序如何将标准输入重定向到别的程序。  

### 重点：shell为其他程序重定向stdin
### 管道编程
#### 创建管道
#### 使用管道进行进程通讯：使用 pipe、fork 以及 exec
#### 管道的问题和进程通讯的后续思路


## 小结主要内容
- 输人/输出重定向允许完成特定功能的程序通过交换数据来进行相互协作
- Unix默认规定程序从文件描述符0读取数据，写数据到文件描述符1，将错误信息输出到文件描述符2。这三个文件描述符称为标准输人、输出及标准错误输出
- 当登录到Unix系统中，登录程序设置文件描述符0、1、2。所有的连接、文件描述符都会从父进程传递给子进程。它们也会在调用exec时被传递
- 创建文件描述符的系统调用总是使用最低可用文件描述符号
- 重定向标准输入、输出以及错误输出意味着改变文件描述符0、1、2的连接。有很多种技术来重定向标准I/O
- 管道是内核中的一个数据队列，其每一端连接一个文件描述符。程序通过使用pipe系统调用创建管道
- 当父进程调用fork的时候，管道的两端都被复制到子进程中
- 只有有共同父进程的进程之间才可以用管道连接

## shell脚本的特性
### shell脚本的三个特性  
- shell脚本的功能——与C相比简单易用
- 软件工具的灵活性——每一个工具完成一项特定的、通用的功能
- I/O重定向和管道的使用和作用

### 类似于C语言，但比C语言更简单
```C
//某人用C写了如下的调用：

x = func_a(func_b(y));  //将func_b的结果作为func_a的输入

//用shell写，就是：

prog_b | prog_a > x  //将prog_b的结果作为prog_a的输入并将最终结果放入x

```
**comm**  

使用Unix的工具comm，可以找出两个文件中共有的行。比较两个文件可以得到三个子集：仅文件1有的行，仅文件2有的行，两者共有的行。  

## 标准I/O与重定向的若干概念
所有的Unix I/O重定向都基于标准数据流的原理。  

考虑一下sort工具是如何工作的。sort从一个数据流中读取字节，再将结果输出到另一个流中，同时若有错误发生，则将错误报告给第三个流。如果忽略这些标准流的去向问题，sort工具的基本原型就如图所示。  

三个数据流分别如下：  
- 标准输入——需要处理的数据流
- 标准输出——结果数据流
- 标准错误输出一错误消息流

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/88.jpg" width = 60% height = 60% /></div>


### 3个标准文件描述符
所有的Unix工具都使用上图中所示的三种流的模型。此模型通过一个简单的规则来实现。这三种流的每一种都是一个特别的文件描述符  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/89.jpg" width = 60% height = 60% /></div>

概念：所有的Unix工具都使用文件描述符0、1和2。  

标准输入文件的描述符是0,标准输出的文件描述符是1，而标准错误输出的文件描述符则是2。Unix假设文件描述符0、1、2已经被打开，可以分别进行读、写和写的操作了。  


### 默认的连接：tty
通常通过shell命令行运行Unix系统工具时, stdin、stdout和sderr连接在终端上。  

大部分的Unix工具处理从文件或标准输人读人的数据。如果在命令行上给出了文件名，工具将从文件读取数据。若无文件名，程序则从标准输入读取数据。  

### 程序都输出到stdout
从另一方面说，大多数程序并不接收输出文件名；它们总是将结果写到文件描述符1，并将错误消息写到文件描述符2。如果希望将进程的输出写到文件或另一个进程的输入去，就必须重定向相应的文件描述符。  

### 重定向I/O的是shell而不是程序
通过使用输出重定向标志，命令 cmd > filename告诉shell将文件描述符1定位到文件。于是shell就将文件描述符与指定的文件连接起来。程序则持续不断地将数据写到文件描述符1中，根本没有意识到数据的目的地已经改变了。  

shell并不将重定向标记和文件名传递给程序。  

第二个概念是重定向可以出现在命令行中的任何地方，并且在重定向标识符周围并不需要空格来区分。甚至一个像 > listing ls这样的命令也是可以接受的。这样，“>”符号并不能终止命令和参数，它只不过是一个附加的请求而已。  

### “最低可用文件描述符(Lowest-Available-fd)”原则
那么什么是文件描述符呢？  

文件描述符的概念非常简单：它是一个数组的索引号。每个进程都有其打开的一组文件。这些打开的文件被保持在一个数组中。文件描述符即为某文件在此数组中的索引。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/90.jpg" width = 60% height = 60% /></div>

概念：当打开文件时，为此文件安排的描述符总是此数组中最低可用位置的索引。  

通过文件描述符建立一个新的连接就像在一条多路电话上接收一个连接一样。每当有用户拨一个电话号码，内部电话系统为这个拨号请求分配一条内部的线路号。在许多这样的系统上，下一个打进来的电话就被分配给最小可用的线路号。  

### 概念小结
首先，Unix进程使用文件描述符0、1、2作为标准输入、输出和错误的通道。其次，当进程请求一个新的文件描述符的时候，系统内核将最低可用的文件描述符赋给它。将这两个概念结合在一起，大家就可以理解I/O重定向是如何工作的了，也就可以自己写出程序来完成I/O的重定向。  

## 如何将stdin定向到文件

下面将详细地考察，程序如何将标准输入重定向以至可以从文件中读取数据。更加精确一点说，进程并不是从文件读数据，而是从文件描述符读数据。如果将文件描述符0定位到一个文件，那么此文件就成为标准输入的源。  

### 方法1: close then open

第一步是close(0)，即将标准输人的连接挂断。这里调用close(0)将标准输入与终端设备的连接切断。图中显示了当前文件描述符数组中的第一个元素现在处在空闲状态。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/91.jpg" width = 60% height = 60% /></div>

最后，使用open(filename,O_RDONLY)打开一个想连接到stdin上的文件。当前的最低可用文件描述符是0，因此所打开的文件将被连接到标准输人上去。如图所示，任何从标准输入读取数据的函数都将从此文件中读入。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/92.jpg" width = 60% height = 60% /></div>

```C
close(O);
fd = open(filename, ORDONLY);
if ( fd I = 0 ){
  fprintf(stderr, ”Could not open data as fd 0\n");
  exit(l);
}

..  /* read and print */
```
此程序并没有什么特别的地方，它仅仅挂断电话又拨了一个新的号码而已。当连接建立起来后，就可以从标准输入的一个新的源接收数据了。  

### 方法 2: open.. close.. dup.. close

- open(file)
第一步是打开stdin将要重定向的文件。这个调用返回一个文件描述符，这个描述符并不是0，因为0在当前已经被打开了。  

- close(0)
下一步是将文件描述符0关闭。文件描述符0现在已经空闲了。  

- dup(fd)
系统调用dup(fd)将文件描述符fd做了一个复制。此次复制使用最低可用文件描述符号。因此，获得的文件描述符是0。这样，就将磁盘文件与文件描述符0连接在一起了。  

- close(fd)
最后，使用close(fd)来关闭文件的原始连接，只留下文件描述符0的连接。将这种方法与把电话从一个分机转移到另一个分机的技术做一个比较。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/93.jpg" width = 60% height = 60% /></div>

```C
fd = open(filename, O_RDONLY);
close(0);
newfd = dup(fd);
if (newfd != 0) {
  fprintf(stderr, "Could not duplicate fd to 0\n");
  exit(l);
}
close(fd);

.. /* read and print */
```

### 方法 3: open.. dup2.. close
一个比上面更简单一点的方案是将close(0)和dup(fd)结合在一起作为一个单独的系统调用dup2。  

dup2(orig,new)将文件描述符old复制到文件描述符new，在此之前它先将文件描述符new上已经存在的连接关闭。  

```C
fd = open(filename, O_RDONLY);
newfd = dup2(fd,0);
if (newfd != 0) {
  fprintf(stderr, "Could not duplicate fd to 0\n");
  exit(l);
}
close(fd);

.. /* read and print */
```

### 系统调用dup小结

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/94.jpg" width = 60% height = 60% /></div>

系统调用dup复制了文件描述符oldfd。而dup2将oldfd文件描述符复制给newfd。两个文件描述符都指向同一个打开的文件。这两个调用都返回新的文件描述符，若发生错误，则返回-1。  

### 重点：shell为其他程序重定向stdin
这些例子显示了程序如何将标准输入重定向到文件。实际上，如果程序希望读取文件，它直接打开文件就可以了，根本不需要将标准输人重定向到文件。  

**这些例子的真正意义在于说明一个程序如何将标准输入重定向到别的程序**  

## 为其他程序重定向I/O: who > userlist

当某用户输入who〉userlist，shell运行who程序，并将who的标准输出重定向到名为userlist的文件上。这是如何完成的呢？  

关键之处就在于fork和exec之间的时间间隙。在fork执行之后，子进程仍然在运行shell程序，并准备执行exec。exec将替换进程中运行的程序，但它不会改变进程的属性和进程中所有的连接。也就是说，在运行过exec之后，进程的用户ID不会改变，其优先级不会改变，并且其文件描述符也和运行exec之前一样。注意，程序得到的是载入它的进程所打开的文件。下图展示了子进程的输出重定向。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/95.jpg" width = 60% height = 60% /></div>  

### 使用以上原则来重定向标准输出的具体过程 

#### 初始情况
如图所示，进程运行在用户空间中。文件描述符1连接在打开的文件f上。为了使这幅图清楚易理解，其他打开的文件并未画出来。 

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/96.jpg" width = 60% height = 60% /></div> 

#### 父进程调用fork之后
如图所示，新的进程出现了。此进程与原始进程运行相同的代码，但它知道自己是子进程。此进程包含了与父进程相同的代码、数据和打开文件的文件描述符。因此文件描述符1依然指向的是文件f。然后子进程调用了closed(1)。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/97.jpg" width = 60% height = 60% /></div>  

#### 在子进程调用closed(1)之后
如图所示，父进程并没有调用closed(1)，因此父进程中的文件描述符1仍然指向f。子进程调用closed(1)之后，文件描述符1变成了最低未用文件描述符。子进程现在试着打开文件g。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/98.jpg" width = 60% height = 60% /></div>  

#### 在子进程调用creat("g",m)之后  

如图所示，文件描述符1被连接到文件g。子进程的标准输出被重定向到g。子进程然后调用exec来运行who。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/99.jpg" width = 60% height = 60% /></div>   

#### 在子进程使用exec执行新程序之后
如图所示，子进程执行了 who程序。于是子进程中的代码和数据都被who程序的代码和数据所替代了，然而文件描述符被保留下来。打开的文件并非是程序的代码也不是数据，它们属于进程的属性，因此exec调用并不改变它们。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/100.jpg" width = 60% height = 60% /></div>   

who命令将当前用户列表送至文件描述符1。其实这组字节已经被写到文件g中去了，而who命令却毫不知晓。  

下面的程序whotofile. c展示了上面所说的这种方法：  
```C
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/wait.h>
#include<fcntl.h>

int main(){
    int pid;
    int fd;

    if((pid = fork()) == -1){
        perror("fork");
        exit(1);
    }

    if(pid == 0){
        close(1);
        fd = creat("userlist",0644);
        execlp("who","who",NULL);
        perror("execlp");
        exit(1);
    }

    if(pid != 0){
        wait(NULL);
        printf("Done running who.results in userlist\n");
    }
    return 0;
}
```

### 重定向到文件的小结
共有三个基本的概念，利用它们使得Unix下的程序可以轻易地将标准输人、输出和错误信息输出连接到文件：  

- 标准输人、输出以及错误输出分别对应于文件描述符0、1、2
- 内核总是使用最低可用文件描述符
- 文件描述符集合通过exec调用传递，且不会被改变

shell使用进程通过fork产生子进程与子进程调用exec之间的时间间隔来重定向标准输入、输出到文件。  

## 管道编程
管道是内核中的一个单向的数据通道。管道有一个读取端和一个写人端。实现who | sort这样的操作，需要两种技巧：如何创建管道，以及如何将标准输入和输出通过管道连接起来。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/101.jpg" width = 60% height = 60% /></div>   

### 创建管道
调用pipe来创建管道并将其两端连接到两个文件描述符。array\[0]为读数据端的文件描述符，而array\[1]则为写数据端的文件描述符。像一个打开的文件的内部情况一样，管道的内部实现隐藏在内核中，进程只能看见两个文件描述符。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/102.jpg" width = 60% height = 60% /></div>  

下图显示了进程创建一个管道前后的状况。前一张图（调用pipe之前）显示了标准文件描述符集。后一张图（调用pipe之后）显示了内核中新创建的管道，以及进程到管道的两个连接。注意，类似于open调用，pipe调用也使用最低可用文件描述符。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/103.jpg" width = 60% height = 60% /></div>  

### 使用fork来共享管道
当进程创建一个管道之后，该进程就有了连向管道两端的连接。当这个进程调用fork的时候，它的子进程也得到了这两个连向管道的连接，如图所示。父进程和子进程都可以将数据写到管道的写数据端口，并从读数据端口将数据读出。两个进程都可以读写管道，但是当一个进程读，另一个进程写的时候，管道的使用效率是最髙的。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/104.jpg" width = 60% height = 60% /></div>  

### 使用 pipe、fork 以及 exec
本章已经介绍了各种技巧和思路来编写将who的输出连接到sort的输人的程序。大家应该已经了解了如何去创建管道，如何在进程间共享管道，如何改变进程的标准输人以及如何改变进程的标准输出。  

在这里可以将这所有的技巧结合在一起，编写一个通用的程序pipe。它使用两个程序的名字作参数，例如：  
```
pipe who sort
pipe Is head
```

程序的内在逻辑如下所示:  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/105.jpg" width = 60% height = 60% /></div>

```C
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<fcntl.h>

#define oops(m,x) {perror(m);exit(x);}

int main(int ac,char **av){
    int thepip[2],newfd,pid;

    if(ac != 3){
        fprintf(stderr,"usage:pipe cmd1 cmd2\n");
        exit(1);
    }
    if(pipe(thepip) == -1) oops("Cannot get a pipe",1);

    if((pid = fork()) == -1) oops("Cannot fork",2);

    if(pid > 0){
        close(thepip[1]);
        if(dup2(thepip[0],0) == -1) oops("could not redirect stdin",3);

        close(thepip[0]);
        execlp(av[2],av[2],NULL);
        oops(av[2],4);
    }

    close(thepip[0]);
    if(dup2(thepip[1],1) == -1) oops("could not redirect stdout",4);
    close(thepip[1]);
    execlp(av[1],av[1],NULL);
    oops(av[1],5);

    return 0;
}
```

### 技术细节：管道并非文件
管道在许多方面都类似于普通文件。进程使用write将数据写人管道，又通过read把数据读出来。像文件一样，管道是不带有任何结构的字节序列。另一方面，管道又与文件不同，例如文件的结尾是否也适用于管道呢？下列技术细节清楚地阐述了文件与管道的相同点与不同点。  

#### 从管道中读数据
- 管道读取阻塞
当进程试图从管道中读数据时，进程被挂起直到数据被写进管道。  

- 管道的读取结束标志
当所有的写者关闭了管道的写数据端时，试图从管道读取数据的调用返回0，这意味着文件的结束。  

- 多个读者可能会引起麻烦
管道是一个队列。当进程从管道中读取数据之后，数据已经不存在了。如果两个进程都试图对同一个管道进行读操作，在一个进程读取一些之后，另一个进程读到的将是后面的内容。它们读到的数据必然是不完整的，除非两个进程使用某种方法来协调它们对管道的访问。  

#### 向管道中写数据
- 写入数据阻塞直到管道有空间去容纳新的数据
管道容纳数据的能力要比磁盘文件差的多。当进程试图对管道进行写操作的时候，此调用将挂起进程直到管道中有足够的空间去容纳新的数据。  

- 写入必须保证一个最小的块大小  
POSIX标准规定内核不会拆分小于512字节的块。而Linux则保证管道中可以存在4096字节的连续缓存。如果两个进程向管道写数据，并且每一个进程都限制其消息不大于512字节，那么这些消息都不会被内核拆分。  

- 若无读者在读取数据，则写操作执行失败
如果所有的读者都已将管道的读取端关闭，那么对管道的写人调用将会执行失败。  

为了避免数据丢失，内核采用了两种方法来通知进程：“此时的写操作是无意义的”。首先，内核发送SIGPIPE消息给进程。若进程被终止， 则无任何事情发生。否则write调用返回-1，并且将errno置为EPIPE。  

## 下一步做什么
传统的Unix管道在进程间进行数据的单向传输。若两个进程希望来回传递数据又该如何？两个没有关联的进程或两个进程在不同的机器上，那该如何实现呢？在后面几章中，将更加细致地研究一下管道以及网络编程，使用管道的思路可以轻易地扩展到套接字(socket)上去。  


