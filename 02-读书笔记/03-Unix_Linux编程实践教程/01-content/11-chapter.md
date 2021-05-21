# 第11章 连接到近端或远端的进程: 服务器与Socket(套接字)

## 本章主要内容
本章将关注于进程间的数据通信，这也是客户/服务器编程的基础知识。  

### 数据来源的三种方式：Unix提供一个接口来处理可能来自不同数据源的数据
### 介绍一个典型的服务端/客户端案例——bc：Unix中使用的计算器
这是典型的本地服务端/客户端案例，并使用了双向管道进行通讯。  

### 管道的缺陷
管道在一个进程中被创建，通过fork来实现共享。因此，管道只能连接相关的进程，也只能连接同一台主机上的进程。   

### 引出socket
socket允许在不相关的进程间创建类似管道的连接，甚至可以通过socket连接其他主机上的进程。  

socket建立连接后，返回的是一个文件描述符，代替了复杂的管道。  

### 为了更加便利的进行远程进程通讯，提供了fdopen和popen

#### fdopen——让文件描述符像文件一样使用

#### popen——在fdopen之上增加了shell程序的功能
popen库函数可以将任何shell程序嵌入服务器程序并且让对服务器的访问就像访问缓存文件一样。

### 客户端/服务器的典型demo  

- 基于电话的时间服务服务端涉及6个步骤。每个步骤对应于一个系统调用  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/117.jpg" width = 60% height = 60% /></div>

- 基于电话的时间服务客户端涉及4个步骤，每个步骤对应于一个系统调用  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/118.jpg" width = 60% height = 60% /></div>

## 小结主要内容
- 一些程序被作为单独的进程建立起来来接收和发送数据。在客户/服务器模型中，服务器进程为客户进程提供处理或数据服务
- 客户/服务器系统包含通信系统和协议。客户和服务器通过管道或socket进行通信。协议是会话过程中一系列规则的集合
- popen库函数可以将任何shell程序嵌入服务器程序并且让对服务器的访问就像访问缓存文件一样
- 管道是一对相连接的文件描述符。socket是一个未连接的通信端点，也是一个潜在的文件描述符。客户进程通过把自己的socket和服务器端的socket相连来创建一个通信连接
- sockets之间的连接可以扩展到另一台机器上。每个socket以机器地址和端口来标识
- 到管道和socket的连接使用文件描述符。文件描述符为程序提供了与文件、设备和其他的进程通信的统一编程接口  

## Unix提供一个接口来处理可能来自不同数据源的数据

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/106.jpg" width = 60% height = 60% /></div>

### (1,2)磁盘/设备文件
用open命令连接，用read和write传递数据。  

### (3)管道
用pipe命令创建，用fork共享，用read和write传递数据。  

### (4)Sockets
用socket Jisten和connect连接，用read和write传递数据。  



## bc：Unix中使用的计算器
### bc并不是一个计算器
大部分版本的be程序都只分析输人，并不执行操作其实，bc在内部启动了dc计算器程序，并通过管道与其进行通信。  

dc是一个基于栈的计算器，它需要用户在指定具体的操作符之前，先输人所要操作的数据。例如，用户输入2 2 +来代表2加2的操作。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/107.jpg" width = 60% height = 60% /></div>

就连GNU版本的bc也是把用户的输入转换成基于栈的后缀表达式。  

### 从bc方法中得到的思想
#### 客户/服务器模型
bc/dc程序对是客户/服务器模型程序设计的一个实例。  

dc提供服务：计算。dc所识别的语言是众所周知的逆波兰表示法。bc和dc之间通过标准输入stdin和标准输出stdout进 
行通信。be提供用户界面，并使用de提供的服务。这里be被称为de的客户。   

这两个部分是根本上独立的程序。可以使用不同版本的dc，这并不影响bc正常工作。类似地，可以编写一个图形界面的bc，而仍用dc作为计算引擎。甚至可以用这样一个程序来替换dc，该程序先分析dc所识别的语言，然后把它所要做的工作传给可能位于另一台更髙速计算机上的程序。  

#### 双向通信
客户/服务器模型不同于生产线的数据处理模型，它要求一个进程既跟另一个进程的标准输人也要和它的标准输出进行通信。传统的Unix的管道只是单方向地传送数据。  

#### 永久性服务
bc只是让单一的dc进程处于运行状态，这就不同于shell程序，这种程序中的每个用户命令都创建一个新的进程。bc程序持续不断地和dc的同一个实例进行通信，把用户的输入转换成命令传给dc。他们之间的关系并不同于标准函数中所使用的调用返回机制。  

bc/dc对被称之为协同进程（coroutines）以用来区别于子程序（subroutines）。两个程序都持续运行，当其中的一个程序完成自己的工作后将把控制权传给另一个程序。bc的任务是分析输入及打印，而dc则负责计算。  

### 编写 bc: pipe、fork、dup、exec
- 创建两个管道
- 创建一个进程来运行dc
- 在新创建的进程中，重定向标准输入和标准输出到管道，然后运行exec dc
- 在父进程中，读取并分析用户的输入，将命令传给dc，dc读取响应，并把响应传给用户

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/108.jpg" width = 60% height = 60% /></div>

### 对协同进程的讨论
一个客户/服务器模型程序要成为协同系统必须有明确指明消息结束的方法，并且程序必须使用简单并可预测的请求和应答。  

## 和进程通讯最简单常用的方法——fdopen:让文件描述符像文件一样使用（传入进程的文件描述符，以此跟进程通讯）
在tinybc.c中使用了库函数fdopen。fdopen与fopen类似，返回一个FILE \*类型的值，不同的是此函数以文件描述符而非文件作为参数。  

使用fopen的时候，将文件名作为参数传给它。fopen可以打开设备文件也可以打开常规的磁盘文件。  


**如只知道文件描述符而不清楚文件名的时候可以使用fdopen命令。**  

例如在管道的例子中，把一个通向管道的连接转换成FILE \*类型值之后，就可以使用标准缓存的I/O操作来对其进行操作了。注意tiny bc.c是如何使用fprintf和fgets来通过管道和dc进行通信的。  

**使用fdopen使得对远端的进程的处理就如同处理常规文件一样。**  

下一节将剖析popen函数，此函数通过封装pipe、fork、dup和exec等系统调用使得对程序和文件的操作变成了一回事。  

## 在fdopen上更进一步，与本地具有shell命令功能进程的通讯方法——popen：让进程看似文件
### popen的功能

fopen打开一个指向文件的带缓存的连接：  
```CPP
FILE * fp; 
fp = fopen( "filel", "r");
c = getc(fp);
fgets(buf, len,fp);
fscanf (fp,"%d %d %s", &x,&y,x);
fclose(fp);
```
fopen需要两个字符串变量作为参数：文件名和连接类型(例如:"r"、"w"、"a"、...)。  

popen看上去跟fopen很类似。popen打开一个指向进程的带缓冲的连接：  
```CPP
FILE* fp;
fp = popen("ls","r");
fgets(buf,len,fp);
pclose(fp);
```
图显示了popen和fopen之间的相似性。两者使用相同的语法格式，并具有相同的返回值类型。popen的第一个参数是要打开的命令的名称；它可以是任意的shell命令。第二个参数可以是"r"或"w"，但决不会是"a"。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/109.jpg" width = 60% height = 60% /></div>

下面的程序将 who | sort 作为数据源，通过popen来获得当前用户排序列表:  
```C
#include<stdio.h>
#include<stdlib.h>

int main(){
    FILE *fp;
    char buf[100];
    int i = 0;

    fp = popen("ls|sort","r");
    while(fgets(buf,100,fp) != NULL) printf("%3d %s",i++,buf);
    
    pclose(fp);
    return 0;
}
```
pclose命令是必须的。  

当完成对popen所打开连接的读写后，必须使用pclose关闭连接，而不能用fclose。进程在产生之后必须等待退出运行，否则它将成为僵尸进程(zombie)。而pclose中调用了wait函数来等待进程的结束。

### 实现popen——使用fdopen命令
这里需要一个新的进程来运行程序，所以要用到fork命令。需要一个指向该进程的连接，因此需要使用管道。并且使用fdopen命令将一个文件描述符定向到缓冲流中。最后，在该进程中要能够运行任何shell命令，所以要用到exec。但是会运行什么程序呢？惟一能够 
运行任意shell命令的程序是shell本身即/bin/sh。为了使程序员可以方倮的使用，sh支持-c选项，用以告诉shell执行某命令然后退出。例如：
```
sh -c "who | sort"
```

这里合并了pipe、fork、dup2和exec等系统调用，其流程如下：
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/110.jpg" width = 60% height = 60% /></div>

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/111.jpg" width = 60% height = 60% /></div>

具体代码见P335  

### 访问数据：文件、应用程序接口(API)和服务器
#### 方法1：从文件获取数据
可以通过读取文件来获取数据。第2章所写的who程序就是从utmp文件中读取数据的。基于文件的信息服务并不是很完美。客户端程序依赖于特定的文件格式和结构体中的特定成员名称。  

#### 方法2：从函数获取数据
可以通过调用函数来得到数据。  

一个库函数用标准的函数接口来封装数据的格式和位置。Unix提供了读取utmp文件的函数接口。getutent的帮助信息描述了读取utmp数 
据库函数的细节。这样的话，就算底层的存储结构变化了，使用这个接口的程序仍能正常工作。  

使用基于应用程序接口（API）的信息服务也并不一定是最好的方法。  

有两种方法可以使用库函数：  
- 一个程序可以使用静态连接来包含实际的函数代码。但是这些函数有可能包含的并不是正确的文件名或文件格式
- 一个程序可以调用共享库中的函数，但是这些共享库也并不是安装在所有的系统上，或者其版本并不是程序所要使用的版本  

#### 方法3：从进程获取数据
第三种方法是从进程中读取数据。  

bc/dc和popen例子显示了如何创建一个进程到另外一个进程的连接。一个要得到用户列表的程序可以使用popen来建立与who程序的连接。由who命令来负责使用正确的文件名和文件格式以及正确的库函数，而不是你的程序。  

调用独立的程序获得数据还有其他的好处。服务器程序可以使用任何程序设计语言编写：shell脚本、C、Java或是Perl都可以。以独立程序的方式实现系统服务的最大好处是客户端程序和服务器端程序可以运行在不同的机器上。所有要做的只是和不同机器上的一个进程相连接。  


## socket：与远端进程相连
### 管道的缺点  
管道使得进程向其他进程发送数据就像向文件发送数据一样容易，但是管道具有两个重大的缺陷。管道在一个进程中被创建，通过fork来实现共享。  

**因此，管道只能连接相关的进程，也只能连接同一台主机上的进程。**  

### socket
socket允许在不相关的进程间创建类似管道的连接，甚至可以通过socket连接其他主机上的进程。  

本节将学习socket的基础知识，理解如何用socket连接不同主机上的客户端和服务器端。其思想就跟打电话查询当地时间一样简单。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/112.jpg" width = 60% height = 60% /></div>  

### 类比：“电话中传来声音：现在时间是……”

#### 服务端：建立服务及与服务相关的操作

基于电话的时间服务涉及6个步骤。每个步骤与一个系统调用相对应  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/117.jpg" width = 60% height = 60% /></div>

**建立服务**  

##### 获取一根电话线：向内核申请一个socket  

socket是一个通信端点。就像位于墙上的电话插座一样socket是产生呼叫和接收呼叫的地方。系统调用socket创建一个socket。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/113.jpg" width = 60% height = 60% /></div>

socket调用创建一个通信端点并返回一个标识符。有很多种类型的通信系统，每个被称为一个通信域。Internt本身就是一个域。在后面会看到Unix内核是另一个域。Linux支持好几个其他域的通信。  

socket的类型指出了程序将要使用的数据流类型。SOCK_STRAEAM类型跟双向的管道类似。数据作为连续的字节流从一端写入，再从另一端读出。后面的章节中将介绍SOCK_DGRAM 类型。  

函数中最后的参数protocol指的是内核中网络代码所使用的协议，并不是客户端和服务器之间的协议。一个为0的值代表选择标准的协议。 

##### 为电话线申请号码：绑定地址到socket上，地址包括主机、端口

下一个步骤是把一个网络地址分配给socket。在Internet域中，地址由主机和端口构成。这里不能使用端口13，因为该端口已经为时间服务器保留。这里可使用端口13000来代替。  

可以为你的服务器端口选择任意的号码，只是该号码不要太小且不能已经被占用。低端口号可能已经被系统服务所占用，而不能再被普通用户使用。请检查系统中端口的限制范围。端口号是一个16位的数值，所以有很多端口号可用。  

系统调用bind如下所示

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/114.jpg" width = 60% height = 60% /></div>

bind调用把一个地址分配给socket。该地址的分配就类似于把一个电话号码分配给墙上的一个插座；当进程要与服务器连接的时候，它们就使用该地址。  

每个地址族都有自己的格式。因特网地址族(AF_INET)使用主机和端口来标志。地址就是一个以主机和端口为成员的结构体。自己写的程序应首先初始化该结构的成员，然后再填充具体的主机地址和端口号，最后填充地址族。关于建立上述数据的具体函数，可参阅帮助手册。  

当所有的部分被填充了之后，地址已经被绑定到该socket上。其他类型的socket会使用包含不同成员的地址。  

##### 允许接入电话：在socket上，允许接人呼叫并设置队列长度为1  

服务器接收接人的呼叫，所以这里的程序必须使用listen  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/115.jpg" width = 60% height = 60% /></div>  

listen请求内核允许指定的socket接收接人呼叫。并不是所用类型的socket都能接收接入呼叫。但SOCK_STREAM类型是可以的。第二个参数指出接收队列的长度。  

在本章的程序中请求的是一个长度为1的队列。队列最大长度则取决于具体socket的实现。  

**提供服务** 

##### 等待呼叫：等待/接收呼叫  

一旦socket被建立并被分配一个地址，而且准备等待接收呼叫，程序即将开始工作。服务器等待直到呼叫到来。它使用系统调用accept来接收调用。  

accept阻塞当前进程，一直到指定socket上的接人连接被建立起来，然后accept将返回文件描述符，并用该文件描述符来进行读写操作。此文件描述符实际上是连到呼叫进程的某个文件描述符的一个连接。  

accept支持一种类型的呼叫者的ID。在呼叫发起者一边socket有自己的地址，例如对于因特网连接，地址就是主机地址和端口号。  

如果callerid和addrlenp指针不为空的话，内核将把呼叫者地址填充到callerid所指向的结构中，并把该结构的长度填充到addrlenp所指向的内存单元中。  

就像人们使用来电显示的信息来决定如何处理打人的电话一样，一个网络程序可以使用呼叫进程的地址来决定如何处理该连接。  

##### 提供服务：传输数据  

accept调用所返回的文件描述符是一个**普通文件的描述符**。对它的操作从第2章中学习完open调用之后一直在使用。  

程序timeserv.c用fdopen将文件描述符定向到缓存的数据流，以便于使用fprintf调用来进行输出。在以前只能使用write来完成这项工作。  

因此使用open、fopen都可以，但fopen在进程进行通讯时，更加通用。  

##### 挂断：关闭连接  

accept所返回的文件描述符可以由标准的系统凋用close关闭。当一端的进程关闭了该端的socket，若另一端的进程在试图读数据的话，它将得到文件结束标记。这跟管道的工作原理类似。  


#### 客户端：使用服务

基于电话的时间服务客户端的实现包含4个步骤，每个步骤对应于一个系统调用  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/118.jpg" width = 60% height = 60% /></div>

##### 获取一根电话线：向内核请求建立socket  

客户端需要一个socket跟网络相连，就像时间服务中的客户端需要一条电话线跟电话网络相连一样。客户端必须建立Internet域(AF_INET)socket,并且它还必须是流socket(SOCK_STREAM)。  

##### 连接到具体号码：与服务器相连  

客户端需要连接到时间服务器。connect系统调用的作用实际上与打电话类似。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/116.jpg" width = 60% height = 60% /></div>

connect调用试图把由sockid所标识的socket和由serv_addrp所指向的socket地址相连接。  

如果连接成功的话conne ct返回0。而此时，sockid是一个合法的**文件描述符**，可以用来进行读写操作。  

写入该文件描述符的数据被发送到连接的另一端的socket，而从另一端写入的数据将从该文件描述符读取。

##### 使用服务：传送数据  

在成功连接之后，进程可以从该文件描述符读写数据，就像与普通的文件或管道相连接一样。  

##### 挂断电话：关闭连接  

读取时间之后，客户端关闭文件描述符然后退出。若客户端退出而不关闭描述符，内核将完成关闭文件描述符的任务。  

#### 如果想要调用远程服务器的shell程序，可以使用popen。具体见例子2

### 重要概念
#### 客户和服务器
服务器是提供服务的程序。  

在Unix中，服务器是一个程序不是一台计算机。服务器进程等待请求，处理请求，然后循环回去等待下一个请求。  

客户端进程则不需要循环，它只需建立一个连接，与服务器交换数据，然后继续自己的工作。  

#### 主机名和端口
运行于因特网上的服务器其实是某台计算机上运行的一个进程。  

这里计算机被称为主机。机器通常被指定一个名字如sales.xyzcorp.com，这被称为该主机的名字。服务器在该主机上拥有一个端口。    

主机和端口的组合才标识了一个服务器。

#### 地址族

#### 协议
协议是服务器和客户之间交互的规则。  

在时间服务器的例子中，协议很简单：客户呼叫，服务器回答，给出时间信息后挂断。  

### 端口号查询
如何知道端口13是时间服务端口而79是目录辅助服务呢？  

在文件/etc/services中定义了众所周知服务端口号的列表。  

从列表中可以看出时间服务是端口13。仔细研究该文件可以看到因特网主机上的标准服务，如ftp、telnet、finger和 http的端口。  

## 软件精灵
很多服务器程序都是以d结尾，如httpd、inetd、syslogd和atd。这里的d表示精灵(daemon)的意思，因而如名叫syslogd的服务器程序实际上是系统日志精灵(system log daemon)。  

精灵就是一个为他人提供服务的帮助者，它随时等待去帮助别人。在你的系统中，输人命令ps -el或ps -ax就可以看到以字符d结尾的进程。然后，可以去阅读这些命令的帮助信息，从而可以更加深入地理解Unix中用客户/服务器模型是如何来处理一些基础操作的。  

大部分精灵进程都是在系统启动后就处于运行状态了。位于类似于/etc/rc.d目录中的shell脚本在后台启动了这些服务，它们的运行与终端相分离，时刻准备提供数据或服务。  
