# 第8章 进程和程序：编写命令解释器sh
## 本章主要思路
由浅入深，了解程序与进程，以及进程的相关操作。  
### 先了解进程的概念：进程=运行中的程序
### 通过命令ps了解进程的具体属性
### 从硬件层面了解进程（文件和进程、内存和程序）
### 通过实现shell来从系统调用角度更深入了解进程和程序

了解实现shell的基本过程：为了实现一个shell，需要学会：  
- 运行一个程序
- 建立一个进程
- 等待 exit()

#### 一个程序如何运行另一个程序：调用execvp
#### 如何建立新的进程：调用fork
#### 父进程如何等待子进程的退出（如何实现进程同步）：调用wait/exit
#### 子线程注销的过程：调用_exit

## 小结主要内容
- Unix通过将可执行代码载人进程并执行它来运行一个程序。进程是运行一个程序所需的内存空间和其他资源的集合。
- 每个运行中的程序在自己的进程中运行。每个进程都有一个惟一的进程ID、所有者、大小及其他属性。
- 系统调用fork通过复制进程来建立一个几乎和原来进程完全相同的副本进程。这个新建的进程被称为子进程。
- 一个程序通过调用exec函数族在当前进程中执行一个新的程序。
- 一个程序能通过调用wait来等待子进程结束。
- 调用程序能将一个字符串列表传给新程序的main函数。新的程序能通过调用exit来回传一个8位长的值。
- Unix shell通过调用fork、exec和wait来运行程序。

## 进程概念：进程=运行中的程序
在Unix术语中，一个可执行程序是一个机器指令及其数据的序列。  

一个进程是程序运行时的内存空间和设置。  

数据和程序存储在磁盘文件中，程序在进程中运行。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/72.jpg" width = 60% height = 60% /></div>

## 了解进程的具体属性：通过命令ps学习进程
进程存在于用户空间。用户空间是存放运行的程序和它们的数的一部分内存空间。可以通过使用ps(process status进程状态的简写)命令来查看用户空间的内容。这个命令会列出当前的进程。  
```
$ ps
PID TTY TIME CMD
1775 pts/l 00：00：17 bash
1981 pts/1 00：00：00 ps
```
这里有两个进程在运行：bash(shell)和ps命令。每个进程都有一个可以惟一标识它的数字，被称为进程ID。一般简称为PID。每个进程都与一个终端相连，这里是/dev/pts/1。每个进程都有一个已运行的时间。注意ps对已运行时间统计并不是非常的精确，从ps只用0秒就可以看出。  

```
$ ps —a
PID TTY TIME CMD
1779 pts/0 00：00：13 gv
1780 pts/0 00：00：07 gs
1781 pts/0 00：00：01 vi
2013 pts/2 00：00：23 xpaint
2017 pts/2 00：00：02 mail
2018 pts/1 00；00：00 ps
```
-a选项列出所有进程，包括在其他终端由其他用户运行的程序。但是带选项-a的输出并不包括shell。  

```
$ ps - la
F S UID PID PPID C PRI NI  ADDR SZ WCHAN TTY TIME CMD
000 S 504 1779 1731 0 69 0  - 1086 do_sel pts/0 00：00：13 gv
000 S 504 1780 1779 0 69 0  - 2309 do_sel pts/0 00：00：07 gs
000 S 504 1781 1731 0 72 0  - 1320 do_sel pts/0 00：00：01 vi
000 S 519 2013 1993 0 69 19 - 1300 do_sel pts/2 00：00：23 xpain
000 S 519 2017 1993 0 69 0  -  363 read_c pts/2 00：00：02 mail
000 R 500 2023 1755 0 79 0  -  750 -      pts/1 00：00：00 ps
```
名为S的一列表示各个进程的状态。S列的值为R说明ps对应的进程正在运行。其他进程的S列值都是S，说明它们都处于睡眠状态。每个进程都属于相应的由UID列指明的用户ID。每个进程都有一个进程ID(PID)，同时也有一个父进程ID(PPID)。  

标记为PR1和NI的列分别是进程的优先级和ni c ene ss级别。内核根据这些值来决定什么时候运行进程。
一个进程可以增加niceness级别，这就像在超市里在排队付账的时候让其他客户排到自己的前面。超级用户可以减少他的ni c ene ss级别，这就像排队的时候插队。  

一个进程有大小，这由SZ列表示。这列的数据表示这个进程占用的内存大小。在例子中mail程序比xpaint占用少得多的内存。因为后者耗费大量的内存来存储影像。程序在运行的时候占用的内存数量可能会动态的改变。如果程序在运行时分配内存，那么它占用的内存就会增加。  

WCHAN列显示进程睡眠的原因。上面的例子中所有睡眠的进程都是等待输入。read_c或do_sel代表内核的地址。  

### 系统进程
除了用户运行的进程外，其他一些是Unix系统用来完成系统任务的进程：  
```
$ ps - ax | head -25
PIDTTY STAT TIME COMMAND
1 ? S 0：05 in it
2 ? SW 3：54 Qkflushd]
3 ? sw 0：38 Lkupdate」
4 ? SW 0：00 [kpiod]
5 ? sw 2；13 [kswapd]
35 ? sw 0：00 [uhci - control]
36 ? sw 0：00 [khubd]
420 ? s 0：25 syslogd
423 ? s 0：36 klogd - k/boot/System, map - 2. 2.14
437 ? sw 0：00 匸inetd]
449 ? s 0：02 amd - f /etc/am. d/conf
461 ? sw 0：00 [rpciod]
466 ? s 0：00 cron
471 ? s 0：00 atd
476 ? s 0：00 sendmail: accepting connections on port 25
484 ? sw 0：00 [rpc.rstatd]
500 ? s 0：46 sshd
504 ? sw 0：00 [calserver]
506 ? sw 0：00 匸keyserver]
512 ? sw 0：00 [portsentry]
514 ? sw 0：00 [portsentry]
561 ttyl sw 0：00 [getty〕
562 tty2 sw 0：00 [getty]
563 tty3 sw 0：00 [getty]

$ ps - as wc - 1
  82
```
上面的例子显示了当前系统中运行的82个进程中的前24个。其中部分是系统进程。系统进程中的很大一部分是没有终端与之相连的。它们在系统启动时启动，而不是由用户在命令行输入。  

## 从硬件层面了解进程
### 进程管理和文件管理
文件包含数据，进程包含可执行代码。文件有一些属性，进程也有一些属性。内核建立和销毁文件，进程类似。就像管理磁盘的多个文件，内核管理内存中的多个进程，为它们分配空间，并记录内存分配情况。  

### 内存和程序
进程这个概念有些抽象，但是它代表了一些非常实际的实体：内存中的一些字节。下图演示了计算机内存的3种模式  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/73.jpg" width = 60% height = 60% /></div>

Unix系统中的内存分为系统空间和用户空间。进程存在于用户空间。内存实际上就是一个字节序列，或者一个很大的数组。如果机器有64 MB的内存，那意味着这个数组有大约6700万个内存位置。其中的一些用来存放组成内核的机器指令和数据。  

还有一些存放组成进程的机器指令和数据。一个进程不一定必须要占一段连续的内存。就像文件在磁盘上被分成小块，进程在内存也被分成小块。同样和文件有记录分配了的磁盘块的列表相似，进程也有保存分配到的内存页面(memory pages)的数据结构。因此，将进程表示为用户空间内的一个小方块只是某种程度的抽象。  

将内存表示为连续的字节数组也是一种抽象。现在的内存一般情况下是由小电路板上的一些芯片组成。  

建立一个进程有点像建立一个磁盘文件。内核要找到一些用来存放程序指令和数据的空闲内存页。内核还要建立数据结构来存放相应的内存分配情况和进程属性。  

使操作系统变得神奇的不仅是它的文件系统把一堆旋转圆盘上连续的簇变成有序组织的树状目录结构，而且以相似的机制，它的进程系统将硅片上的一些位组织成一个进程社会——成长、相互影响、合作、出生、工作和死亡。这有点像蚂蚁农庄。  



## 通过shell来从系统调用角度更深入了解进程和程序
### shell：进程控制和程序控制的一个工具
shell是一个管理进程和运行程序的程序。就像存在很多编程语言,Unix系统有很多种可用的shell，每种都有各自的风格和优势。所有常用的shell都有三个主要功能：  
- 运行程序
> grep、date、ls、echo和mail都是一些普通的程序，用C编写，并被编译成机器语言。shell将它们载人内存并运行它们。很多人把shell看成是一个程序启动器  
- 管理输入和输出
> shell不仅仅是运行程序。使用<、>和|符号可以将输入、输出重定向。这样就可以告诉shell将进程的输人和输出连接到一个文件或是其他的进程  
- 可编程
> shell同时也是带有变量和流程控制的编程语言  

### shell是如何运行程序的
一个shell的主循环执行下面的4步:  
- 用户键入a.out
- shell建立一个新的进程来运行这个程序
- shell将程序从磁盘载入
- 程序在它的进程中运行直到结束

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/74.jpg" width = 60% height = 60% /></div>

#### shell的主循环
shell由下面的循环组成： 
```C
while(!end_of_input)
  get_command
  execute_command
  wait for command to finish
```

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/75.jpg" width = 60% height = 60% /></div>
  
为了要写一个shell，需要学会：  
- 运行一个程序
- 建立一个进程
- 等待 exit()

#### 问题1: 一个程序如何运行另一个程序
答案：程序调用execvp。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/76.jpg" width = 60% height = 60% /></div>

调用过程：  
- 程序调用execvp
- 内核从磁盘将程序载入
- 内核将arglist复制到进程
- 内核调用 main(argc，argv)

下面是运行ls -l的完整程序:  
```C
#include<stdio.h>
#include<unistd.h>

int main(){
    char *arglist[3];

    arglist[0] = "ls";
    arglist[1] = "-l";
    arglist[2] = 0;

    printf("*** About to exec ls -l\n");
    execvp("ls",arglist);
    printf("*** ls is done. by\n");
}
```

execvp有两个参数：要运行的程序名和那个程序的命令行参数数组。令行参数以argv\[]传给程序。注意，将数组的第一个元素置为程序的名称一个元素必须是null。  

**第二条打印的消息哪里去了？**  

一个程序在一个进程中运行——也就是一些内存和内核中相应的数据结构。这样，execvp将程序从磁盘载人进程以便它可以被运行。但是载入到哪个进程呢？这就是问题之所在：内核将新程序载入到当前进程，替代当前进程的代码和数据。即相当于把当前进程的代码和数据覆盖掉了。  

**execvp就像换脑**  

exec系统调用从当前进程中把当前程序的机器指令清除，然后在空的进程中载人调用时指定的程序代码，最后运行这个新的程序。exec调整进程的内存分配使之适应新的程序对内存的要求。相同的进程，不同的内容。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/77.jpg" width = 60% height = 60% /></div>

execvp载入由file指定的程序到当前进程，然后试图运行它。execvp将以NULL结尾的字符串列表传给程序。execvp在环境变量PATH所指定的路径中查找file文件。如果执行成功，execvp没有返回值。当前程序从进程中清除，新的程序在当前进程中运行。  

execvp是一组基于execve系统调用函数中的一个，它们统称为exec。  

**此时shell的问题**  

前面所学已经足够写第一个版本的shell了。这里已经知道如何运行一个程序，还知道如何将命令行参数传给它。第一个shell提示用户输入程序名和参数，然后运行指定的程序。  

程序运行正常，但是就像设想的那样，execvp用命令指定的程序代码覆盖了 shell的程序代码，然后在命令指定的程序结束之后退出。这样shell就不能再次接受新的命令。为了运行新的命令，用户不得不再次运行shell。  

shell如何能做到在运行程序的同时还能等待下一个命令呢？方法之一就是启动一个新的进程，由这个进程来执行命令程序。  

#### 问题2:如何建立新的进程
答案：一个进程调用fork来复制自己。  

**解释fork**  

解决的方法之一就是复制一个自己，采用三位影像复制技术，逐个原子的复制一个完全等价的自己。当你建立了自己的复制品，将爱因斯坦的大脑放到它的脑壳里，这样你就可以继续你原来的计划和思考了。在你生命中的某个阶段，世界上只有一个你。当你按下复制机的复制按钮，世界上有了两个你。对自己的复制有点像马路上的岔口。开始只有一条路，然后有了两条。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/78.jpg" width = 60% height = 60% /></div>  

进程拥有程序和当前运行到的位置。进程调用fork，当控制转移到内核中的fork代码后，内核做：  
- 分配新的内存块和内核数据结构
- 复制原来的进程到新的进程
- 向运行进程集添加新的进程
- 将控制返回给两个进程

当一个进程调用fork之后，就有两个二进制代码相同的进程。而且它们都运行到相同的地方。但是每个进程都将可以开始它们自己的旅程。  

子进程不是从main函数的开始，而是从fork返回的地方开始它的生命之旅。  

**分辨父进程和子进程**  

这两个进程不是完全相同的。不同的进程，fork的返回值是不同的。在子进程中fork返回0,在父进程中fork返回4171（子进程的pid）。根据fork的返回值进程可以很容易的判断自己是子进程还是父进程。  

**fork小结**  

系统调用fork正是解决shell只能运行一条命令这个问题所需要的。使用fork，不但能够创建新的进程，而且能够分辨原来的进程和新创建的进程。新的进程能调用execvp来执行任何用户指明的程序。  

这里明确建立一个shell所需三项技术中的两项。知道了如何建立进程(fork)和如何运行一个程序(execvp)。最后需要知道如何让父进程等待子进程结束。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/79.jpg" width = 60% height = 60% /></div> 

#### 问题3:父进程如何等待子进程的退出（进程同步）
答案：进程调用wait等待子进程结束。  
```C
pid = wait(&status);
```

**解释wait()**  

系统调用wait做两件事。首先，wait暂停调用它的进程直到子进程结束。然后，wait取得子进程结束时传给exit的值。  

最终子进程会结束任务并调用exit(n)。n是0到255的一个数字。  

当子进程调用exit，内核唤醒父进程同时将子进程传给exit的参数。唤醒和传递退出(exit)值的动作由从exit的括号到父进程的箭头表示。这样wait执行两个操作：通知和通信。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/80.jpg" width = 60% height = 60% /></div>

**wait的两个重要特性**  

- wait阻塞调用它的程序直到子进程结束
这一特征使两个进程能够同步它们的行为。比如，父进程用fork创建一个子进程来对一个文件排序。父进程必须等排序结束后才能继续处理这个文件。系统调用exit和wait是一种协调这些任务的方法。  

- wait返回结束进程的pid
一个进程可以创建多个子进程。如果父进程需要等待其中某个子进程结束，则只需要判断返回的pid是否和该子进程的pid相同。  

**使用wait进行进程间的通信**

wait的目的之一是通知父进程子进程结束运行了。它的第二个目的是吿诉父进程子进程是如何结束的。  

一个进程以3种方式(成功、失败或死亡)之一结束。  

其一，一个进程可能顺利完成它的任务。按照Unix惯例，成功的程序调用exit(0)或者从main函数中return 0。  

其二，进程可能失败。比如进程可能由于内存耗尽而提前退出程序。按Unix惯例，程序遇到问题而要退出调用exit时传给它一个非零的值。程序员可以对不同的错误分配不同的值，手册中有详细的描述。  

最后，程序可能被一个信号杀死(见第6和第7章)。信号可能来自键盘、间隔计时器、内核或者其他进程。通常的情况下，一个既没有被忽略又没有被捕获信号会杀死进程的。  

父进程如何知道子进程是以何种方式退出的呢？  

答案在传给wait的参数之中。父进程调用wait时传一个整型变量地址给函数。内核将子进程的退出状态保存在这个变量中。如果子进程调用exit退出，那么内核把exit的返回值存放到这个整数变量中；如果进程是被杀死的，那么内核将信号序号存放在这个变量中。这个整数由3部分组成——8个bit是记录退出值，7个bit是记录信号序号，另一个bit用来指明发生错误并产生了内核映像(core dump)。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/81.jpg" width = 60% height = 60% /></div>

**wait小结**  
wait系统函数挂起调用它的进程直到得到这个进程的子进程的一个结束状态。结束状态是退出值或者是信号序号。如果有一个子进程已经退出或被杀死，对wait的调用立即返回。wait返回结束进程的pid。如果statusptr不是NULL，wait将退出状态或者信号序号复制到statusptr指向的整数中。这个值可以用<sys/wait.h>中的宏来检测。 

如果调用的进程没有子进程也没有得到终止状态值，则wait返回-1。  

#### 小结：shell如何运行程序

shell用fork建立新进程，用exec在新进程中运行用户指定的程序，最后shell用wait等待新进程结束。 
wait系统调用同时从内核取得退出状态或者信号序号以告知子进程是如何结束的。  

每个Unix shell都是使用下图所示的模型。现在将这3个系统调用组合在一起实现一个真正的shell。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/82.jpg" width = 60% height = 60% /></div>

### 思考：用进程编程
考虑一下函数和进程之间的相似性。  

#### execvp/exit就像call/return

**call/return**  

一个C程序由很多函数组成。一个函数可以调用另一个函数，同时传给它一些参数。被调用的函数执行一定的操作，然后返回一个值。每个函数都有它的局部变量，不同的函数通过call/return系统进行通信。  

这种通过参数和返回值在拥有私有数据的函数间通信的模式是结构化程序设计的基础。Unix鼓励将这种应用于程序之内的模式扩展到程序之间。  

**exec/exit**   

一个C程序可以fork/exec另一个程序，并传给它一些参数。这个被调用的程序执行一定的操作，然后通过exit(n)来返回值。调用它的进程可以通过wait(Suresult)来获取exit的返回值。子程序的exit返回值可以在result的8~15位之间找到。  

函数调用所用到的堆栈几乎是没有限制的。一个被调用的程序还可以调用其他程序，一个通过fork/exec调用起来的程序可以通过fork/exec调用别的程序。Unix使创建一个新进程方便而且快捷。用fork/exit和exit/wait来调用程序和返回结果不仅适用于shell，Unix程序经常被设计成一组子程序，而不是一个带有很多函数的大程序。  

由exec传递的参数必须是字符串。由于进程间通信的参数类型为字符串，这样就强迫了子程序的通信也必须使用文本作为参数类型。几乎是偶然的，这种基于文本的程序接口支持跨平台的交互，而这一点非常重要。  

**全局变量和fork/exec**  

Unix提供方法来建立全局变量。环境（environment）是一些传递给进程的字符串型变量集合。不会有副作用，它对fork/exec和exit/wait机制是一个有用的补充。下一章将看到它如何工作和如何使用它。  

### exit和exec的其他细节
#### 进程死亡：exit 和i_exit
exit是fork的逆操作，进程通过调用exit来停止运行。fork创建一个进程，exit删除进程。基本上是这样。  

exit刷新所有的流，调用由atexit和on_exit注册的函数，执行当前系统定义的其他与exit相关的操作。然后调用_exit。系统函数_exit是一个内核操作，这个操作处理所有分配给这个进程的内存，关闭所有这个进程打开的文件，释放所有内核用来管理和维护这个进程的数据结构。  

子进程传给exit的参数被如何处理了？那个进程的弥留之言被存放在内核直到这个进程的父进程通过wait系统调用取回这个值。如果父进程没有在等这个值，那么它将被保存在内核直到父进程调用wait，那时内核将通告这个父进程子进程的结束，并转达子进程的弥留之言。  

那些已经死亡但是还没有给exit赋值的进程被称之为**幽灵（zombie）进程**。很多比较新的版本的ps列出这些进程并标记为defunct。

**\_exit小结**  

系统调用_exit终止当前进程并执行所有必须的清理工作。这些工作在各个不同版本的Unix中有些不同，但都包括以下一些操作：  
- 关闭所有文件描述符和目录描述符
- 将该进程的PID置为init进程的PID
- 如果父进程调用wait或waitpid来等待子进程结束，则通知父进程
- 向父进程发送SIGCHLD

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/83.jpg" width = 60% height = 60% /></div>

这样，如果父进程在子进程之前退出，那么子进程将能继续运行，而不会成为“孤儿”，它们将是init进程的“子女”，这有点像孤儿由国家监护。注意，就算父进程没有调用wait,内核也会向它发送SIGCHLD消息。尽管对SIGCHLD消息的默认处理方法是忽略的。如果想响应这个消息，可以设置一个处理函数。  

#### exec家族
在已经实现的shell中和例程中用execvp来演示进程是如何运行一个程序的。execvp不是一个系统调用，它是一个库函数，这个函数通过系统调用execve来调用内核服务。execve中的e代表环境(environment)，所以将推迟到下一章来讨论它。  

具体函数有execlp、execl、execv等，详见P256  

