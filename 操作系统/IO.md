# 目录

1. 文件描述符
2. IO模型

## 一 文件描述符fd

Linux下一切皆文件。普通文件/字符设备文件（鼠标，键盘）/块设备文件（硬盘，光驱）/套接字/目录文件，所有的一切都抽象成文件，提供统一接口，方便程序调用。

文件描述符：一个索引值，用于描述一个进程中的打开的文件。操作系统通常为一个进程维护一个文件描述符表，该表包含一个指针数组，每一个指针都是一个文件指针，指向系统级文件表的文件偏移量（可用于获得inode指针，指向inode表，获得真实文件），文件描述符则是指针数组的下标。因此，通过进程文件描述符可以找到对应文件。通常0，表示标准输入文件（stdin），为键盘对应的设备文件，1，表示标准输出文件（stdout），为显示器对应的设备文件，2，表示标准错误（stderr），对应的设备也是显示器，从3开始，开始用于记录新打开或创建的一个文件。通常将没有被分配的最小的文件描述符分配给新打开的文件。

![image-20210121171016552](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210121171016552.png)

*“Linux下的系统调用接口open,close,read,write,lseek”：open打开或创建一个文件，成功返回文件描述符，出错返回-1；close（fd）关闭一个打开文件，成功返回0，出错返回-1；read从打开文件中读数据，需要参数fd，要读取文件的文件指针，要读取的字节数，返回读取到的字节数或0（达到尾端）；*

## 二 IO

IO有内存IO，网络IO，磁盘IO，通常我们说的IO指的是后两者。进程需要通过系统调用请求kernel协助完成IO，内核会为每个IO设备维护一个缓冲区。对于一个输入操作。进程需要系统调用进行IO时，首先内核查看缓冲区中有没有相应的缓存数据，没有再到设备中读取，一般来说设备IO速度较慢，需要等待。

一个read操作的两个阶段：

1.等待数据准备

2.将数据从内核拷贝到进程中。（有内核缓存区与进程缓存区）

## 二 IO模型

### 1 Blocking IO 阻塞IO

进程一直处于阻塞状态，直到内核中所有数据准备好并从内核空间拷贝到用户空间。

### 2 NonBlocking IO 非阻塞IO

通过设置socket使其变为非阻塞的socket，对一个非阻塞socket执行read，进程每隔一段时间对内核中的数据进行询问，如果没有准备好则返回进程一个EWOUDBLOCK，立即得到一个结果。一段时间后再进行询问，若准备好，则将数据从内核拷贝到进程。在内核进行数据准备过程中一直进行轮询（polling）。



### 3 IO multiplexing IO多路复用

用