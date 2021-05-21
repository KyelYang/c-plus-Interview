# 第三章 目录与文件属性：编写ls

### ls常用的命令
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/10.jpg" width = 60% height = 60% /></div> 

如果对Unix不是很熟悉，那么可能需要解释一下-a这个选项，在Unix中，ls—般不会列出以.开始的文件，所以可以把这样的文件看作是隐藏文件。ls加上了-a以后，遇到这样的文件也必须把它列出来。对操作系统（例如内核）而言，文件名前面的.没有任何特殊的含义，它只对ls的使用者有意义。  

某些应用程序的配置文件是位于用户的主目录下以开始的某个文件，这是由习惯形成的，因为在大多数情况下可以将它们隐藏。但是需要时可以直接被打开编辑，不需要任何特殊的操作。  

### ls命令工作流程  
- 打开目录——opendir
- 读取目录内容——readdir
- 打印目录中的文件名
- 关闭目录——closedir

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/11.jpg" width = 40% height = 40% /></div> 

#### 关于目录  
**什么目录**  

目录是一种特殊的文件，它的内容是文件和目录的名字。从某种程度上说，目录文件与上一章讲的utmp文件很类似。它们都包含很多记录，每个记录的格式由统一的标准定义。每条记录的内容代表一个文件或目录。  

与普通文件不同的是，目录文件永远不会空，每个目录都至少包含两个特殊的项——"."和".."，其中"."表示当前目录，".."表示上一级目录。  

**如何读目录的内容**  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/12.jpg" width = 60% height = 60% /></div> 

通过联机帮助可以知道，从目录读数据与从文件读数据是类似的,opendir打开一个目录，readdir返回目录中的当前项，closedir关闭一个目录，seekdir、telldir、rewinddir与lseek的功能类似。  

目录是文件的列表，更确切地说，是记录的序列，每条记录对应一个文件或子目录。通过readdir来读取目录中的记录,readdir返回一个指向目录的当前记录的指针，记录的类型是struct dirent,这个结构定义在/usr/include/dirent.h中，联机帮助中也可以査到。  
```C
struct dirent
{ 
ino_t d_ino;  
off_t d_off;
unsigned short d_reclen;  //文件大小
char d_name[l];  //文件名
}
```

### ls -l命令 
#### ls -l命令工作流程
- 打开目录——opendir
- 读取目录内容——readdir
- 打印目录中各文件属性——stat
  - 打印模式信息——使用stat中的掩码 
  - 打印链接数
  - 根据uid搜索用户名并打印——getpwuid
  - 根据gid搜索用户名并打印名——getgrgid
  - 打印文件大小
  - 打印最后修改时间——ctime
  - 打印文件名
- 关闭目录——closedir

#### ls -l显示各字段说明
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/13.jpg" width = 50% height = 50% /></div>

每行都包含7个字段：
- 模式(mode)
> 每行的第一个字符表示文件类型。"-"代表普通文件，"d"代表目录等  
> 接下来的9个字符表示文件访问权限，分为读权限、写权限和执行权限，又分别针对3种对象：用户、同组用户和其他用户，所以一共 需要9位来表示  

- 链接数(links)
> 链接数指的是该文件被引用的次数  
- 文件所有者(owner)的用户名
- 文件所有者所在的组(group)
- 文件大小（字节）
> 所有的目录大小相等，都是1024字节，因为目录所占空间的分配是以块(block)为单位的，每个块512字节，所以目录的大小经常是相等的。如果是一般的文件，size列则显示了文件中数据的实际字节数  
- 最后修改时间
- 文件名(name)

#### 用stat得到文件信息

磁盘上的文件有很多属性，如文件大小、文件所有者的ID等。如果需要得到文件属性，进程可以定义一个结构struct stat，然后调用stat，告诉内核把文件属性存放到这个结构中。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/15.jpg" width = 40% height = 40% /></div>

stat把文件fname的信息复制到指针bufp所指的结构中。  

- stat的联机帮助和头文件/usr/include/sys/stat.h描述了 struct stat的成员变量

> stat结构中其他未被ls -l用到的成员变量未在这里列出。  

#### 文件类型和许可权限  
strode是一个16位的二进制数，文件类型和权限被编码在这个数中。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/16.jpg" width = 60% height = 60% /></div>

其中前4位用作文件类型，最多可以标识16种类型，目前已经使用了其中的7个。  

接下来的3位是文件的特殊属性，1代表具有某个属性,0代表没有，这3位分别是set-user-ID位、set-group-ID位和sticky位。  
- set-user-ID位用来用户密码
- set-group-ID位是用来设置程序运行时所属组的
- sticky位告诉内核即使没有人在使用程序，也要把它放在交换空间中  


最后的9位是许可权限，分为3组，对应3种用户，它们是文件所有者、同组用户和其他用户。其他用户指与用户不在同一个组的人。每组3位，分别是读、写和执行的权限。相应的地方如果是1，就说明该用户拥有对应的权限，0代表没有。  

在sys/stat.h中每一位都有相应的掩码  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/17.jpg" width = 50% height = 50% /></div>

#### 将用户/组ID转换成字符串  
**通过getpwuid来得到完整的用户列表**  

可以通过库函数getpwuid来访问用户信息，如果用户信息保存在/etc/passwd中，那么getpwuid会査找/etc/passwd的内容，如果用户信息在NIS中，getpwuid会从NIS中获取信息，所以用getpwuid使程序有很好的可移植性。

**通过getgrgid来访问组列表**   

在网络计算系统中，组信息也被保存在NIS中。Unix系统提供getgrgid函数屏蔽掉实现的差异。用这个函数，用户可以得到组名而不用操心实现的细节。getgrgid的用户手册对这个函数及相关函数做了详细解释。在ls -l中，可以这样得到组名  


### 设置和修改文件的属性
#### 文件类型  

文件的类型有普通文件、目录文件、设备文件、socket文件、符号链接文件、命名管道（named pipe）文件等。

#### 文件类型的创建  

文件类型是在创建文件的时候建立的，如用系统调用creat建立一个普通文件。其他类 型的文件如目录、设备等，可使用不同的函数创建。

#### 修改文件类型  

文件一经创建，类型就无法修改。  

#### 创建文件的模式
creat的第二个参数指定了要创建文件的许可位：  
```C
fc( = creat("new_file", 0744 ); //指定新创建文件的许可位为rwxr-r--
```
#### 改变文件的模式
程序可以通过系统调用chmod来改变文件的模式，如：
```C
chmod("new_file",04764); 
chmod("new_file",S_ISUID | S_IRWXU | S_IRGRP | S_IWGRP | S_IROTH);
```

上述两条指令的作用相同，第一条是八进制来表示，第二条是用sys/stat.h中定义 的符号来表示。后者有明显的优点，当系统定义的许可位的值改变时，无需修改程序。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/18.jpg" width = 60% height = 60% /></div>  

#### 用命令来修改文件的许可权限和特殊属性
Shell命令chmod也可以用来完成上述操作。它可以通过两种模式指定权限和属性，八进制模式和符号模式。

```
chmod 777 new_file //让所有人都拥有new_file的rwx权限
chmod u=rws,g=rw,o=r new_file  //切记等号两边一定不能加空格
```

### 文件的链接数
关于链接数的详细讨论在下一章。简而言之，链接数就是文件被引用的次数(别名的数量)。如果一个文件在目录树中一共有3个别名，那么这个文件的链接数就是3。增加文件的别名(使用link)会使链接数增加，减少别名(使用unlink)会使链接数减少。  

### 修改文件所有者和组
通过系统调用chown来修改文件所有者和组：
```C
chown( "file", 200, 40);
```
将文件file 的用户id改为200, 组id改为40, 如果后两个参数的值都是-1 ，那么文件所有者和组都不会改变。一般用户不大会修改文件的文件所有者和组，但root经常出于管理上的目的要修改这些内容。文件所有者可以把文件的组改成任何一个他所属的组。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/19.jpg" width = 60% height = 60% /></div> 

Shell命令chown和chgrp可以用来修改文件所有者和组。  

### 文件时间
每个文件都有3个时间：最后修改(modification)时间、最后访问(access)时间和属性最后修改时间，当文件被操作时，内核会自动地修改这些时间，也可以编程来修改最后修改时间和最后访问时间。  

#### 修改最后修改时间和最后访问时间
utime系统调用可以用来设置最后修改时间和最后访问时间，使用一个包含两个time_t结构的变量，一个用来存放更新的最后修改时间，另一个是最后访问时间。  

<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/20.jpg" width = 60% height = 60% /></div> 

#### 什么时候需要修改文件时间
举个例子，在文件备份的时候，文件的最后修改时间和访问时间会被记录下来，当文件被恢复的时候，希望文件的这些时间与原来的相同，这时候就可以用utime。恢复时做了两件事，一是把备份的文件复制回去，二是把最后修改时间和访问时间改成备份时候的情况，这样被恢复的文件就与备份时的完全一样。  

Shell命令touch可以修改文件的最后访问时间和最后修改时间。

### 文件名

#### 创建文件——creat
#### 移动文件——mv
#### 修改文件名——rename
系统调用rename可以修改文件/目录的名字，还可以移动文件的位置，它有两个参数， 原文件名和新文件名。  
<div align=center><img src="https://github.com/KyelYang/c-plus-Interview-data/blob/master/02-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/03-Unix_Linux%E7%BC%96%E7%A8%8B%E5%AE%9E%E8%B7%B5%E6%95%99%E7%A8%8B/02-image/21.jpg" width = 60% height = 60% /></div>
