strace -ff -o out java 类名

一些linux命令：jps，netstat -natp

tail -f 文件名：打印文件倒数十行

nc localhost 8090：相当于一个链接TCP的程序

查看一些系统调用

man 2 socket

man 2 bind



通信建立的过程

socket->bind->listen

一个程序如何执行？

server端，一定进行三个系统调用 new server socket：socket（返回文件描述符，如3）->bind（3，8090）（绑定文件描述符与端口号）->listen（3）（监听端口）

accept（3，     ：accept系统调用会阻塞 =5

recv（5  阻塞，如果一直阻塞，那么while （true）会导致阻塞以致无法接收下一个客户端

所以使用采用多线程，基于BIO的每线程每连接。

优势：可以接收很多连接

问题：1 线程内存浪费

2 CPU调度的消耗，消耗CPU时间片。

根源： blocking 阻塞： accept与recv都是阻塞的。

解决问题： NonBlocking 非阻塞



NIO:

1 操作系统的NIO（NonBlocking）

2 java的NIO（New IO）接口，可以设置socket非阻塞

非阻塞：方法一定给出返回，accept返回-1或文件描述符。java中返回null或者一个客户端的socket。

整个java程序中只有一个主线程，主线程中有一个名为clients的linkedlist存放接收的客户端socket。

准备serversocket，绑定监听并设置成非阻塞，随后在一个while死循环：1 接收客户端，如果客户端接收，将客户端设置成非阻塞，并将客户端加入clients中。2 for循环遍历clients中的socket，但这里的recv不会阻塞，所以遍历完后会重新进入while死循环。所以一个主线程就能够做到接收客户端与遍历客户端，不会开辟多个线程。

