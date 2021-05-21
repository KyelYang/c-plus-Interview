# 第五章 连接控制：学习stty

连接控制对象主要包括：文件、终端、设备。  

## 主要学习内容
前面章节中已经讲述了一些与文件和目录相关的程序。  

计算机还有其他的数据来源，如调制解调器、打印机、扫描仪、鼠标、扬声器、照相机和终端等这样的外部设备。  

在本章中将学习这些设备与目录和文件的相似之处和不同之处，并了解如何将这些想法用于管理设备间的连接。  

本章的项目是编写命令stty的另外一个版本。stty用来让用户检测、修改控制键盘和示器连接属性的设置。  

## 小结
### 主要内容
- 内核在进程和外部世界间交换数据。外部世界包括磁盘文件、终端和外部设备（像打印机、磁带驱动器、声卡和鼠标）。到磁盘文件和终端的连接有相似之处但也有差异。  

- 磁盘文件和设备文件都有名字、属性和权限位。标准文件系统调用open、read、write、close和lseek可被用于任何文件或设备。文件权限位以同样的方式应用于控制设备文件和磁盘文件的访问。  

- 到磁盘文件的连接在处理和传输数据方面不同于到设备文件的连接。内核中管理与设备连接的代码被称为设备驱动程序。通过使用fcntl和ioctl,进程可以读取和改变设备驱动程序的设置。

- 到终端的连接是如此的重要，以至函数tcgetattr和tcsetattr专门用来提供对终端驱动器的控制。

- Unix命令stty使得用户能够访问tcgetattr和tcsetattr函数。

### 图示
进程使用write将数据写入文件描述符，用read从文件描述符读出数据。文件描述符可被连接到磁盘文件、终端和外部设备。文件描述符指向设备驱动程序时，设备驱动程序具有属性设置，如图所示

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/33.jpg" width = 60% height = 60% /></div> 


## 设备和文件的相同之处
对Unix来说，声卡、终端、鼠标和磁盘文件是同一种对象。  

在Unix系统中，每个设备都被当做一个文件。每个设备都有一个文件名、一个i-节点号、一个文件所有者、一个权限位的集合和最近修改时间。你所了解的和文件有关的所有内容都将被运用于终端和其他的设备。  

### 设备具有文件名
每个加载到Unix机器的设备（终端、打印机、鼠标、磁盘等）都通过文件名表示。  

通常，表示设备的文件存放在目录/dev中，但是可以在任何目录中创建设备文件。  

### 设备和系统调用
设备不仅具有文件名，而且支持与所有文件相关的系统调用：open、read、write、lseek、close 和 stat。 

和磁盘文件相关的系统调用同样可以为其他设备服务。实际上，Unix没有其他的方法用来和设备通信。  


### 例子：终端就像文件
Unix的很多用户输人来自终端。ttysd.ttyse等文件都代表终端。按传统定义终端是键盘和显示单元。  

终端最重要的功能是接受来自用户的字符输人和将输出信息显示给用户。显示输出单元甚至可以产生盲文打印或声音。  

命令tty用来告知用户所在终端的文件名。用终端文件做以下试验：
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/34.jpg" width = 60% height = 60% /></div> 

从以上输出可以知道，终端tty对应的设备描述文件名为/dev/pts/2。可以对该文件使用任何与文件相关的命令和进行任何文件操作，如cp、重定向符">"、mv、ln、rm、cat或ls等各种命令。  

### 设备文件的属性
设备文件具有磁盘文件的大部分属性。  

#### 设备文件和文件大小  
常用的磁盘文件由字节组成，磁盘文件中的字节数就是文件的大小。设备文件是链接，而不是容器。键盘和鼠标不存储击键数和点击数。设备文件的i-节点存储的是指向内核子程序的指针，而不是文件的大小和存储列表。内核中传输设备数据的子程序被称为设备驱 
动程序。  

在/dev/pts/2这个例子中，从终端进行数据传输的代码是在设备一进程表中编号为136的子程序。该子程序接受一个整型参数。在/dev/pts/2中，参数是2。136和2这两个数被称为设备的主设备号和从设备号。主设备号确定处理该设备实际的子程序，而从设备号被作为参数传输到该子程序。  

#### 设备文件和权限位
每个文件都有相应的读、写和执行的权限。  

当文件实际上表示设备时，权限位表示什么意思呢？向文件写人数据就是把数据发送到设备，因此，权限写意味着允许向设备发送数 
据。  

在这个例子中，文件所有者和组tty的成员拥有写设备的权限，但是只有文件的所有者有读取设备的权限。读取设备文件就像读取普通文件一样，从文件获得数据。如果除了文件所有者还有其他用户能够读取/dev/pts/2,那么其他人也能够读取在该键盘上输入的字符，读取其他人的终端输人会引起某些麻烦。  

另一方面，向其他人的终端写人字符是Unix中write命令的目标。  

### 编写write程序
在即时消息和聊天室出现之前，Unix用户通过使用命令write和在其他终端上的用户聊天。  

详细代码见P128  

这个例程和前面章节中的例子表明终端就像其他连接到Unix机器的设备一样，能够以磁盘文件的方式被处理。  

### 设备文件和i-节点
目录是文件名和i-节点号的列表。目录并不能区分哪些文件名代表磁盘文件，哪些文件名代表设备。文件类型的区别体现在i-节点上。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/35.jpg" width = 60% height = 60% /></div> 

每个i-节点编号指向i-节点表中的一个结构。i-节点可以是磁盘文件的，也可以是设备文件的。i-节点的类型被记录在结构stat的成员变量st.mode的类型区域中。  

磁盘文件的i-节点包含指向数据块的指针。设备文件的i-节点包含指向内核子程序表的指针。主设备号用于告知从设备读取数据的那部分代码的位置。  

考虑一下read是如何工作的。内核首先找到文件描述符的i-节点，该i-节点用于告诉内核文件的类型。如果文件是磁盘文件，那么内核通过访问块分配表来读取数据。如果文件是设备文件，那么内核通过调用该设备驱动程序的read部分来读取数据。其他的操作，例如open、write、lseek和close等都是类似的。  

## 设备与文件的不同之处
磁盘文件和设备文件都有文件名和属性，从表面上看很类似。系统调用open用于创建与文件和设备的连接。但是与磁盘文件的连接不同于与终端的连接。  

下图显示了带有两个文件描述符的进程，一个是到磁盘文件的连接，另一个是到终端用户的连接。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/36.jpg" width = 60% height = 60% /></div>

与磁盘文件的连接通常包含内核缓冲区。从进程到磁盘的字节先被缓冲，然后才从内核的缓冲区被发送出去。  

磁盘连接具有缓冲这样一个属性。到终端的连接则不同，进程需要尽快把到终端的数据传送出去。  

与终端或调制解调器的连接也具有属性。连接拥有波特率、奇偶位、暂停位的个数。一般情况下所输入的字符都会显示在屏幕上，但是有些时候，例如当输入密码时，字符并不回显在屏幕上。回显字符不是键盘任务的一部分，也不是程序应该做的；回显是连接的一个属性，到磁盘文件的连接没有这些属性。  

## 连接属性和控制
## 磁盘连接的属性
系统调用open用于在进程和磁盘文件之间创建一个连接。该连接含有若干个属性。  

### 属性1：缓冲
下图显示了当两个管道通过一个进程单元连接时文件描述符的情况。那个进程单元是用来进行缓冲和完成其他进程任务的。在方框内的是控制变量，用以决定文件描述符应该采取哪个进程步骤。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/37.jpg" width = 60% height = 60% /></div>

可以通过修改控制变量改变文件描述符的动作。例如，通过简单的3步操作关闭磁盘缓冲。  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/38.jpg" width = 60% height = 60% /></div>

首先，生成一个系统调用将控制变量从文件描述符复制到进程。然后，修改这个复制过来的控制变量。最后，将修改过的值送回内核。新的设置被安置在进程代码中，内核根据新的设置处理数据。下面是遵循上述3步的代码：
```CPP
#include<fcntl.h> 
int s;  //settings
s = fcntl(fd,F_GETFL);  //get flags
s|= O_SYNC; //set SYNC bit
result = fcntl(fd,F_SETFL,s); //set flags
if ( result = = 1)  //if error
  perror("setting SYNC"); //report
```

文件描述符的属性被编码在一个整数的位中。系统调用fcntl通过读写该整数位来控制文件描述符。  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/39.jpg" width = 60% height = 60% /></div>

这3个步骤(从内核中读取设置到变量，修改这些设置，将设置返回内核)是Unix中读取和修改连接属性的典型方法。  

设置O_SYNC会关闭内核的缓冲机制，如果没有很充分的理由，最好不要关闭缓冲。  


### 属性2：自动添加模式
文件描述符的另一个属性是自动添加模式(auto-append mode)。自动添加模式对于若干个进程在同一时间写入文件是很有用的。这是为了防止进程写入数据时造成覆盖的现象，导致信息丢失。这种情况被称为竞争。  

如何避免这种竞争？有很多方法避免竞争。竞争是系统编程所面临的重要问题，后面需要多次回到这个话题。在这个特定的情况中，内核提供一个简单的解决办法：自动添加模式。当文件描述符的O_APPEND位被开启后，每个对write的调用自动调用lseek将内容添加到文件的末尾。  

下面的代码启动自动添加模式，然后调用write：
```CPP
#include<fcntl.h> 
int s;  //settings
s = fcntl(fd,F_GETFL);  //get flags
s |= O_APPEND;  //set APPEND bit
result = fcntl(fd, F—SETFL,s);  //set flags
if(result == -1) perror ( "setting APPEND"); // report
else write(fd,&rec,1); //write record at end
```
术语竞争和原子操作(atomoc operation)密切相关。对lseek和write的调用是独立的系统调用，内核可以随时打断进程，从而使后面这两个操作被中断。当O_APPEND被置位，内核将lseek和write组合成一个原子操作，被连接成一个不可分割的单元。  

### 用open控制文件描述符
fcntl并不是仅有的用来设置文件描述符属性的方法。通常在打开一个文件时，应该知道需要怎样的设置。可以通过系统调用open的第二个参数的一部分来设置文件描述符的属性位。例如，调用：
```C
fd = open(WTMP_FILE,O_WRONLY|O_APPEND|O_SYNC);  //O-SYNC和O_APPEND是文件描述符的两个属性
```
以写方式打开文件wtmp并将O_APPEND和O_SYNC位开启。open的第二个参数不只是读、写或读/写的选择。

### 磁盘连接小结
内核在磁盘和进程间传输数据。内核中进行这些传输的代码有很多选项。程序可使用open和fcntl系统调用控制这些数据传输的内部运作。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/40.jpg" width = 60% height = 60% /></div>

## 终端连接的属性
### 终端的I/O并不如此简单
终端和进程之间的连接看起来简单。通过使用getchar和putchar就能够在设备和进程间传输字节。数据流的这种抽象使得键盘和屏幕看起来就像在进程中一样。  

一个简单的实验表明这个模型并不完整（具体案例见书P135）。  

以上例子说明了3种处理：  
- 进程在用户输入Return后才接收数据  
- 进程将用户输入的Return（ASCII码13）看作换行符（ASCII码10）  
- 进程发送换行符，终端接收回车换行符  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/41.jpg" width = 60% height = 60% /></div>

### 终端驱动程序
终端和进程之间的连接如图所示  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/42.jpg" width = 60% height = 60% /></div>

处理进程和外部设备间数据流的内核子程序的集合被称为终端驱动程序或tty驱动程序。  

驱动程序包含很多控制设备操作的设置。进程可以读、修改和重置这些驱动控制标志。  

### stty命令
stty命令让用户读取和修改终端驱动程序的设置。  

#### 可以使用stty显示驱动程序设置
#### 可以使用stty改变驱动程序设置  
这里是一些使用stty修改驱动程序属性的例子：  
```
$ stty erase X  # make 'X' the erase key：使用stty用来改变删除键
$ stty - echo # type invisibly：关闭按键回显。当输入密码时，字符并不回显在屏幕上
$ stty erase @ echo # multiple requests：使用stty一次性改变多种设置。同时将删除键改为@，并将回显模式开启
```

### 编写终端驱动程序：关于设置
tty驱动程序包含很多对传人的数据所进行的操作。这些操作被分为4种：  
- 输人：驱动程序如何处理从终端来的字符
- 输出：驱动程序如何处理流向终端的字符
- 控制：字符如何被表示——位的个数、位的奇偶性、停止位等
- 本地：驱动程序如何处理来自驱动程序内部的字符  

输人处理包括将小写字母转换为大写字母，去除最高位及将回车符转换为换行符。输出处理包括用若干个空格符代替制表符，将换行符转换为回车符及将小写字母转换为大写字母。控制设置包括奇偶性及停止位的个数。本地处理包括回显字符给用户及缓冲输入直到用户按回车键。  

除了开和关设置外，驱动程序维护了一张含有特殊意义键的列表。例如，用户可能按退格键来删除一个字符。终端驱动程序会注意并处理这个删除键。除此之外，终端驱动程序还负责对其他一些控制字符进行处理。  

### 编写终端驱动程序：关于函数
改变终端驱动程序的设置就像改变磁盘文件连接的设置一样：  
- 从驱动程序获得属性
- 修改所要修改的属性
- 将修改过的属性送回驱动程序

```CPP
#include<Ctermios.h>
struct termios settings;  /* struct to hold attributes */
tcgetattr(fd, &settings); /* get attribs from driver */
settings.c_lflag |= ECHO; /* turn on ECHO bit in flagset */
tcsetattr(fd,TCSANOW,&settings); /* send attribs back to driver */
```

库函数tcgetattr和tcsetattr提供对终端驱动程序的访问。  

tcgetattr从与文件fd相关的终端驱动程序中获取当前设置，并把它复制到info指针所指的结构中。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/43.jpg" width = 60% height = 60% /></div>

tcsetattr从info所指的结构中将驱动程序的设置复制到与文件fd相关的终端驱动程序中。  

when参数告诉tcsetattr在什么时候更新驱动程序设置。when的允许值如下所示。 
- TCSANOW：  立即更新驱动程序设置
- TCSADRAIN：等待直到驱动程序队列中的所有输出都被传送到终端。然后进行驱动程序的更新
- TCSAFLUSH：等待直到驱动程序队列中的所有输出都被传送出去。然后，释放所有队列中的输入数据，并进行一定的变化

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/44.jpg" width = 60% height = 60% /></div>

### 编写终端驱动程序：关于位
termios结构类型包括若干个标志集和一个控制字符的数组。所有的Unix版本包含以下结构：  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/45.jpg" width = 60% height = 60% /></div>  

每个标志集的独立位的含义如图所示  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/46.jpg" width = 60% height = 60% /></div>  

每个属性在标志集中都占有一位。属性的掩码定义在termios.h中。要测试一个属性，需要将标志集与那个位的掩码相与。要启动这个属性，将该位开启。要禁止这个属性，将该位关闭。上面的情况如下所示：  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/47.jpg" width = 60% height = 60% /></div>  

### 编写终端驱动程序：几个程序例子（具体见书P142）
#### 1.例子：echostate.c——显示回显位的状态
0是标准输入的文件描述符，该文件描述符通常附属在键盘上。  

#### 2.例子：setecho.c——改变回显位的状态
#### 3.例子：showtty.c——显示大量驱动程序属性  

### 终端连接小结
终端是人们用来和Unix进程进行通信的设备。终端拥有一个可以让进程读取字符的键盘和可让进程发送字符的显示器。终端是一个设备，所以它在目录树中表现为一个特殊的文件，通常在/dev这个目录中。  

进程和终端间的数据传输和数据处理由终端驱动程序负责，终端驱动程序是内核的一部分。该内核代码提供缓冲、编辑和数据转换。程序可通过调用tcgetattr和tcsetattr查看和修改该驱动程序的设置。  

## 其他设备编程：ioctl

到磁盘文件的连接有一个属性集，到终端的连接有另外一个属性集。其他类型的设备也有各自的属性集。程序员如何查看和控制一个设备的设置呢？  

每个设备文件都支持系统调用ioctl  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/48.jpg" width = 60% height = 60% /></div>  

系统调用ioctl提供对连接到fd的设备驱动程序的属性和操作的访问。每种类型的设备都有自己的属性集和ioctl操作集。  

阅读头文件是了解设备类型以及相关函数的好方法。联机帮助中有关设备的内容也包括属性和函数的列表。例如，Linux中有关st(4)的联机帮助讲述了使用ioctl控制SCSI磁带驱动器的细节。  

## 文件、设备和流

任何数据的源或目的地都被Unix视为文件。  

基本的系统调用既适用于磁盘文件也同样适用于设备文件，它们的区别体现在对连接的操作上。  

磁盘文件的文件描述符包含对缓冲属性和扩展属性的定义代码。终端的文件描述符包含编辑、回显、字符转换和其他操作的属性定义代码。  

可以把每个处理步骤描述成连接的一个属性，但是反过来说，连接也可以看作是处理步骤的组合。  

这是数据流模型和连接属性的流模型的一个大概的想法。流模型的一个重要特征是处理的模块化。  

如果不满意仅能支持像大小写转换这样的终端驱动程序，可以设计并安装一个可将数字转换成罗马数字的模块。  

也就是，可以编写一个能完成从阿拉伯数字到罗马数字转换的处理模块。将它写到流模型规范，然后使用特殊的系统调用将该模块安装到系统上。流数据经过它的处理就从阿拉伯数字变成罗马数字了。  
