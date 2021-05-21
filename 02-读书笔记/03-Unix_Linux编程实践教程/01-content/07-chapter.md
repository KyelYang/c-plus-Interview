# 第7章事件驱动编程：编写一个视频游戏
## 本章主要思路
### 如何做一个视频游戏（视频游戏其实和操作系统类似）
一个视频游戏综合了一些基本的概念和原则。  

#### 1. 空间
视频游戏的空间管理，主要是屏幕的管理。  

屏幕编程主要用到curses库，其中介绍并讲述了curses库的原理。  

#### 2. 时间
时间主要是时钟编程，时钟编程最典型的函数，即为sleep()。  

介绍sleep()的内部原理，主要包含以下三步：
- 为SIGALRM设置一个处理函数  
- 调用 alarm(num_seconds)  
- 调用pause  

时钟功能的升级版：间隔计时器。  
- 精度更高  
- 三种情形下的计时器：真实（一切情况下）、虚拟（用户态）、实用（用户态、核心态）
- 两种计时功能：两种间隔：初始和重复

#### 3. 中断（即信号处理）
早期的信号处理：  
- 默认操作(一般是终止进程)，比如，signal(SIGALRM，SIG_DFL)
- 忽略信号，比如，signal( SIGALRM，SIG_IGN)
- 调用一个函数，比如，signaKSIGALRM，handler)

早期的信号处理的问题：
- 不足以应对多信号同时产生的情况  
- 不知道信号被发送的原因  
- 处理函数不能安全的阻塞其他消息

信号处理2:sigaction。本章将只学习POSIX模型和相关的系统调用。  

防止数据被损毁：增加临界区——阻塞信号：sigprocmask和sigsetops

#### 4. 同时做几件事：与用户交互，并异步I/O  
- 方法1:使用O_ASYNC
- 方法2:使用aio_read

## 小结主要内容
- 有些程序的控制流很简单。而另外一些则要响应外部的事件。一个视频游戏要响应时钟和用户输入。操作系统也要响应时钟和外设。
- curses库有一些可以管理屏幕显示字符的函数。
- 一个进程通过设置计时器来安排事件。每个进程有3个独立的计时器。计时器通过发送信号来通知进程。每个计时器都可以被设置为只发送一次信号，或者按固定的间隙发送信号。
- 处理一个信号很简单。同时处理多个信号就复杂些了。进程能决定是忽略信号还是阻塞信号。进程能够告知内核哪些信号在什么时候阻塞或忽略。
- 有些函数执行一些复杂的任务是不能被打断的。程序可以通过小心地使用信号掩码来保护这些临界区代码。

## 屏幕编程：curses库
curses库是一组函数，程序员可以用它们来设置光标的位置和终端屏幕上显示的字符样式。大部分控制终端屏幕的程序使用curses。  
### 介绍curses
curses将终端屏幕看成是由字符单元组成的网格，每一个单元由(行、列)坐标对标示。坐标系的原点是屏幕的左上角，行坐标自上而下递增，列坐标自左向右递增。  

curses具有的函数包括可以将光标移动到屏幕上任何行、列单元，添加字符到屏幕或者从屏幕上删除字符，设置字符的可视属性(如颜色、亮度)，建立和控制窗口以及其他文本区域。用户手册有curses的所有函数的详细描述。这里将介绍其中的9个。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/58.jpg" width = 60% height = 60% /></div> 

### curses内部：虚拟和实际屏幕
curses设计成为能够在不阻塞通信线路的情况下更新文本屏幕。curses通过虚拟屏幕来最小化数据流量。  

真实屏幕是眼前的一个字符数组。curses保留了屏幕的两个内部版本。一个内部屏幕是真实屏幕的复制。另一个是工作屏幕，其上记录了对屏幕的改动。每个函数，比如move、addstr等都只在工作屏幕上进行修改。工作屏幕就像磁盘缓存，curses中的大部分的函数都只对它进行修改。  

refresh函数比较工作屏幕和真实屏幕的差异。然后refresh通过终端驱动送出那些能使真实屏幕与工作屏幕一致的字符和控制码。例如，如果真实屏幕的左上角是Smith、James,然后用addstr把Smith、Jane放在相同的位置，调用refresh也许只是用n和空格替换了James中的m和s。这种只传输改变的内容而不是影像本身的技术被用在视频流中。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/59.jpg" width = 60% height = 60% /></div> 

## 时钟编程: sleep
为了写一个视频游戏，需要把影像在特定的时间置于特定的位置，这样可以增加动态效果。  

用curses把影像置于特定的位置。然后在程序中添加时间响应。  

比如：先在一个地方画字符串，睡眠1秒钟，然后在原来的地方画空字符串以删除原有影像，最后将输出位置推进，这样便能产生移动效果。  

注意在两次请求之后通过调用refresh来保证每次循环后旧的影像消失，新的影像显示。  

### sleep()是如何工作的：使用Unix中的Alarms
系统中的每个进程都有一个私有的闹钟(alarm clock)。这个闹钟很像一个计时器，可以设置在一定秒数后闹铃。时间一到，时钟就发送一个信号SIGALRM到进程。除非进程为SIGALRM设置了处理函数(handler),否则信号将杀死这个进程。  

sleep函数由3个步骤组成：  
- 为SIGALRM设置一个处理函数  
- 调用alarm(num_seconds)  
- 调用pause  

系统调用pause挂起进程直到信号到达。任何信号都可以唤醒进程，而非仅仅等待SIGALRM。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/60.jpg" width = 60% height = 60% /></div> 

下面是alarm和pause的细节:  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/61.jpg" width = 60% height = 60% /></div> 

alarm设置本进程的计时器到seconds秒后激发信号。当设定的时间过去之后，内核发送SIGALRM到这个进程。如果计时器已经被设置，alarm返回剩余秒数(注意：调用alarm(0)意味着关掉闹钟)。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/62.jpg" width = 60% height = 60% /></div>

pause挂起调用进程直到一个信号到达。如果调用进程被这个信号终止，pause没有返回。如果调用进程用一个处理函数捕获，在控制从处理函数处返回后pause返回。这种情况下errno被设置为EINTR。  

异步处理思维：调度一个将要发生的动作很简单，通过调用alarm来设置计时器，然后继续做别的事情。当计时器计时到0，信号发送，处理函数被调用。  

## 时钟编程2: 间隔计时器
### 添加精度更高的时延：usleep
usleep(n)将当前进程挂起n微秒或者直到有一个不能被忽略的信号到达。  

### 三种计时器：真实、进程和实用
#### ITIMER_REAL
这个计时器计量真实时间，如同手表记录时间。也就是说不管程序在用户态还是核心态用了多少处理器时间它都记录。当这个计时器用尽，发送SIGALRM消息。  
#### ITIMER_VIRTUAL
这个计时器就像美式橄榄球中用的计时方法，只有进程在用户态运行时才计时。虚拟计时器(virtual timer)的30s比实际计时器(real timer)的30s要长。当虚拟计时器用尽，发送 SIGVTALRM消息。  
#### ITIMER_PROF
这个计时器在进程运行于用户态或由该进程调用而陷入核心态时计时。当这个计时器用尽，发送SIGPROF消息。  

### 两种间隔：初始和重复
每个间隔计时器的设置都有这样两个参数：初始时间和重复间隔。  

在间隔计时器用的结构体中初始时间是it_value，重复间隔是it_interval。如果不想要重复这一特征，将it_interval设置为0。要把两个时钟都关掉，设it_value为0。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/63.jpg" width = 60% height = 60% /></div>


间隔计时器的设置是通过struct itimerval来完成的。这个结构类型包括初始间隔和重复间隔，两者存储在struct timeval。以下是标准用法：  

```C
int set_ticker(int n_msecs){
  struct itimerval new_timeset;
  long n_sec,n_usecs;

  n_sec = n_msecs / 1000;
  n_usecs = (n_msecs % 1000) * 1000L;

  new_timeset.it_interval.tv_sec = n_sec;
  new_timeset.it_interval.tv_usec = n_usecs;

  new_timeset.it_value.tv_sec = n_sec;
  new_timeset.it_value.tv_usec = n_usecs;

  return setitimer(ITIMER_REAL,&new_timeset,NULL);
  }
```

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/64.jpg" width = 60% height = 60% /></div>  

getitimer将某个特定计时器的当前设置读到val指向的结构中。setitimer将计时器设置为newval指向的结构的值。如果oldval不指向null,之前计时器的设定将被复制到oldval指向的结构中。  

which的值指定将要被读或更新的计时器。计时器的编码分别是ITIMER_REAL、ITIMER_VIRTUAL 和ITIMER_PROF。  

### 计算机有几个时钟
一个系统只需要一个时钟来设置节拍。  

每个进程设置自己的计数时间，操作系统在每过一个时间片后为所有的计数器的数值做递减。每当内核收到系统时钟脉冲，它遍历所有的间隔计时器，使每个计数器减一个时钟单位。通过这个简单的机制，每个进程就可以设置自己的计时器。这个计时器在进程睡眠的时候也在倒计时。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/65.jpg" width = 60% height = 60% /></div> 

### 计时器小结
一个Unix程序用计时器来挂起执行和调度将要采取的动作。一个计时器是内核的一种机制。通过这种机制，内核在一定的时间之后向进程发送SIGALRM。alarm系统调用在特定的实际秒数之后发送SIGALRM给进程。setitimer系统调用以更髙的精度控制计时器，同时能够以固定的时间间隔发送信号。  

## 信号处理1:使用signal
中断处理是操作系统和系统软件的关键部分。Unix中的软件中断被称为信号 (signals)。  

### 早期的信号处理机制
各种事件促使内核向进程发送信号。这些事件包括用户的击键、进程的非法操作和计时器到时。一个进程调用signal在以下3种处理信号的方法之中选择：  
- 默认操作(一般是终止进程)，比如，signal(SIGALRM，SIG_DFL)
- 忽略信号，比如，signal( SIGALRM，SIG_IGN)
- 调用一个函数，比如，signaKSIGALRM，handler)

### 处理多信号的问题
#### 捕鼠器问题
在早期的版本中，信号处理函数在另一个方面也很像捕鼠器：在每次捕获之后，都必须重新设置它们。比如：一个信号处理函数可能如下：  
```C
void handler( int s)
{ 
/*process is vulnerable here*/
signal(SIGINT, handler);  /*reset handler*/
…                        /*do work here*/
}
```
就算设置的速度非常快，它还是需要时间的，在弹簧被触发和设置完成之间，就有可能有老鼠溜走了。这一脆弱的间隙使得原有的信号处理不可靠。有些人称此为“不可靠的信号”。  

#### 处理多信号的问题
多信号的情形非常多，不同版本的Unix的答案各不相同。写一个在各个系统下都能正常工作的程序很不容易。  

就好像消息队列的处理模式，遇到不同优先级，是放下立马出处理高优先级，还是先阻塞高优先级进程，等当前进程处理完后再处理等等各种场景...  

### 信号机制其他的弱点
#### 不知道信号被发送的原因  
#### 处理函数不能安全的阻塞其他消息

## 信号处理2: sigaction
### 处理多个信号：sigaction
在POSIX中用sigaction替代signal。参数非常相似。指定什么信号将被如何处理。如果愿意，还能得到这个信号上一次被处理时的设置。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/66.jpg" width = 60% height = 60% /></div> 

第一个参数Signum指明想要处理的消息。第二个参数action指向描述如何响应信号的结构体。第三个参数prevaction如果不是null的话，就是指向描述被替换的处理设置的结构体。如果新的操作设置成功则返回0，否则返回-1。  

#### 定制信号处理:struct sigaction
在过去，面对信号的处理只有简单的几种选择：SIG_DFL、SIG_IGN或者函数处理。这些选项在新的系统中作为结构体sigaction的部分定义依然提供。结构体sigaction定义了如何处理一个信号。以下是这个结构体的完整定义：  

```C
struct sigaction{
/* se only one of these two * /
void (*sa_handler)();  /*SIG_DFL, SIG_IGN,or function */
void (*sa_sigaction)(int,siginfo_t *,void *);  /* new handler */

sigset_t sa_mask; /* signals to block while handling */
int sa_flags; /* enable various behaviors */
}
```
- 选择sa_handler还是sa_sigaction?
首先，要在老的信号处理方式和新的更强大的信号处理方式之间作出选择。如果老的 处理方式(即SIG_DFL、SIG_IGN或者处理函数)就够用了，那么可以设置sa.handler为其中之一。当然，如果指定为旧的信号处理方式，那么只能得到信号编号。否则，如果设定sa_sigaction为一个处理函数，那么那个处理函数被调用的时候，不但可以得到信号编号而且可以获悉被调用的原因以及产生问题的上下文的相关信息。  

如何吿诉内核你使用的是新的信号处理方式？很简单，只需设置sa_flags的SA_SIGINFO位。  

- sa_flags
sa_flags是用一些位来控制处理函数如何应对上述多信号问题的。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/67.jpg" width = 60% height = 60% /></div>   

- sa_mask
最后，决定在处理一个消息时是否要阻塞其他信号。sa_mask中的位指定哪些信号要被阻塞。使用sa_mask，可以在逃离火灾现场时阻塞电话呼叫和来访者。sa_mask的值包括要被阻塞的信号集。阻塞信号是防止数据损毁的重要技术。  

### 信号小结
一个进程可能被各种来源的信号中断。信号可能在任何时候以任何顺序到达。signal提供了一种简单但是不完整的信号处理机制。POSIX接口，即sigaction提供了复杂的、明确定义的方法来控制进程如何对各种信号组合做出反应。  

现在已经知道如何在程序中管理时间和中断。视频游戏还需要最后一项技术：防止混乱。  

## 防止数据损毁
在一些情况下一个操作不应该被其他操作打断。  

在对一个数据结构(这里是列表)改动结束之前，其他函数不能读或写这个数据结构。  

当然，处理像火警一类的信号是安全的，因为这类处理并不读或修改数据。  

### 临界区
一段修改一个数据结构的代码如果在运行时被打断将导致数据的不完整或损毁，则称这段代码为临界区。当程序处理信号时，必须决定哪一段代码为临界区，然后设法保护这段代码。临界区不一定就在信号处理函数中，很多出现在常规的程序流中。保护临界区的最简单的办法就是阻塞或忽略那些处理函数将要使用或修改特定数据的信号。  

### 阻塞信号：sigprocmask和sigsetops
#### 在信号处理者一级阻塞信号
为了在处理一个信号的时候阻塞另一个信号，要设置struct sigaction结构中的sa_mask成员位，它在设置处理函数时被传递给sigaction。sa_mask是sigset_t类型，它定义了一个信号集。  

#### 一个进程的阻塞信号
在任何时候一个进程都有一些信号被阻塞。注意，是阻塞而不是忽略。这个信号集就称为信号挡板(signal mask)。通过sigprocmask可以修改这个被阻塞的信号集，sigprocmask作为一个原子操作根据所给的信号集来修改当前被阻塞的信号集：  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/68.jpg" width = 60% height = 60% /></div>  

sigprocmask修改当前的信号挡板设置。当how的值分别为SIG_BLOCK、SIG_UNBLOCK或SIG_SET时，* sigs所指定的信号将被添加、删除或替换。如果prev不是null，那么之前的信号挡板设置将被复制到* prev中。  

#### 用sigsetops构造信号集
一个sigset_t是一个抽象的信号集，可以通过一些函数来添加或删除信号。基本的函数如下：  
```C
sigemptyset(sigset_t *setp);  //清除由setp指向的列表中的所有信号

sigfillset(sigset_t *setp); //添加所有的信号到setp指向的列表

sigaddset(sigset_t *setpr,int signum);  //添加signum到setp指向的列表

sigdelset(sigset_t *setp, int signum); //从setp指向的列表中删除signum所标识的信号
```

#### 例子：暂时地阻塞用户信号
程序可以用以下代码来暂时地阻塞SIGINT和SIGQUIT信号:  
```C
sigset_t sigs,prevsigs; /* define two signal sets */
sigemptyset(&sigs); /* turn off all bits */
sigaddset(&sigs,SIGINT);  /* turn on SIGINT bit */
sigaddset(&sigs,SIGQUIT); /* turn on SIGQUIT bit */
sigprocmask(SIG_BLOCK,&sigs, &prevsigs);  /* add that to proc mask */

// .. modify data structure here..

sigprocmask(SIG_SET, *prevsigs, NULL); /* restore previous mask */
```

### 重入代码(Reentrant Code):递归调用的危睑
打断别人登记，而将自己的名字和地址插在别人的记录中间的例子引入另一个与数据损毁有关的概念：可重入函数。  

一个信号处理者或者一个函数，如果在激活状态下能被调用而不引起任何问题就称之为可重入的。  

在通过sigaction设置时，可以通过设置SA_NODEFER位来允许处理函数的递归调用。反之，可以通过清除此位来阻塞信号。如何选择呢？  
如果处理者不是可重人的，必须阻塞信号。但是如果阻塞信号，就有可能丢失信号。信号不像电话便条那样贴在那里等你回来处理。有些信号是非常重要的：丢掉一些是安全的吗？  

是丢掉信号还是弄乱数据？哪个更糟些？有没有办法同时避免这两个问题？设计使用号的程序时，这些是必须考虑的问题。信号处理上的错误现象不很有规律，尤其在系统处于高负载的情况下或者在精确的性能计量的时候。排错需要理解信号处理的工作机理，还要知道哪里可能会有问题。  

## 另一个信号的来源：其他进程——进程通信
信号来自间隔计时器、终端驱动、内核或者进程。一个进程可以通过kill系统调用向另一个进程发送信号： 

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/69.jpg" width = 60% height = 60% /></div> 

kill向一个进程发送一个信号。发送信号的进程的用户ID必须和目标进程的用户ID相同，或者发送信号的进程的拥有者是一个超级用户。一个进程可以向自己发送信号。  

一个进程可以向其他进程发送任何信息，包括一般来自键盘、间隔计时器或者内核的信号。比如一个进程可以向另一个进程发送SIGSEGV信号，就好像目标进程执行了非法内存读取。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/70.jpg" width = 60% height = 60% /></div> 

### 进程间通信的含义
接受信号的进程几乎可以设置任何信号的处理者。  

实际上，一组进程可以像橄榄球运动员传递橄榄球那样传递信号。  

### IPC信号设计：SIGUSR1、SIGUSR2
Unix有两个信号可以被用户程序使用。它们是SIGUSR1和SIGUSR2。这两个信号没有预定义任务。可以使用它们以避免使用已经有预定义语义的信号。  

将在后面几章学习进程间通信。编程时可以有很多方法组合使用kill和sigaction。  

## 输入信号：异步I/O
本章的动画和游戏等待两类事件：计时器信号和键盘输人。设置间隔计时器的处理函数来控制动画，通过调用getch阻塞程序以等待键盘输入。除了阻塞，还能像得到计时器信号那样通过信号来得到用户的输入吗？  

可以的。程序可以要求内核在得到输人时发送信号。这有点像要求邮递员在投递邮件时按门铃。这样你就不用坐在大门前整天盯着邮件箱而干些其他什么事情或者睡觉。任何时候只要一有信到你就能知道。  

Unix有两个异步输入(asynchronous input)系统。一种方法是当输入就绪时发送信号，另一个系统当输入被读人时发送信号。UCB中通过设置文件描述块(file descriptor)的O_ASYNC位来实现第一种方法。第二种方法是POSIX标准，它调用aio_read。  

### 方法1:使用O_ASYNC  
使用O_ASYNC需要对原有的弹球程序做4处改动:  
- 要建立和设置在键盘输人时被调用的处理函数
- 使用fcntl的F_SETOWN命令来告诉内核发送输入通知信号给进程。其他进程可能也连接到键盘。这里不想让这些进程发送信号
- 通过调用fcntl来设置文件描述符0中的0_ASYNC位来打开输入信号
- 循环调用pause等待来自计时器或键盘的信号

当有一个从键盘来的字符到达，内核向进程发送SIGIO信号。SIGIO的处理函数使用标准的curses函数getch来读人这个字符。当计时器间隔超时，内核发送以前已经处理的SIGALRM信号。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/71.jpg" width = 60% height = 60% /></div> 

### 方法2: 使用aio_read
相比设置文件描述符的O_ASYNC位，使用aio_read更加灵活，当然也复杂些。对原来的弹球程序做4处改动:  
- 设置输人被读入时所调用的处理函数on_input
- 设置struct kbcbuf中的变量来指明等待什么类型的输人，当输人发生时产生什么信号。在这个简单的程序中，需要从文件描述符0中读人一个字符，当字符被读人时希望收到SIGIO信号。实际上能指定任何信号，甚至是SIGARLM或SIGINT  
- 通过将以上定义的结构体传给aio.read来递交读人请求。和调用一般的read不同，aio_read不会阻塞进程。相反aio_read会在完成时发送信号  
- 实现处理函数，函数通过调用aio_return来得到输入的字符。然后处理这个字符  

### 弹球程序中需要异步读入吗
不。用户输人阻塞程序，间隔计时器驱动球移动的模式工作的很好。探求程序没有其他多余的工作要做，所以采用阻塞也可以。  

异步读入的优势在于程序不用被输入阻塞而可以做些其他事。 
