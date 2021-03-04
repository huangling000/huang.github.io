# 多路复用器

同步IO多路复用器：select（），poll（）

select() and  pselect()  允许一个程序监听多个文件描述符， 直到一个或者几个的文件描述为某类IO操作准备就绪。

如果可以在不阻塞的情况下执行相应的I/O操作（例如，read(2)），则认为文件描述符已准备就绪。

select（）和pselect（）的操作是相同的，但有以下三个区别：

（i） select（）使用的超时值是struct timeval（秒和微秒），而pselect（）使用struct timespec（秒和纳秒）。

（ii）select（）可以更新timeout参数以指示还剩多少时间。pselect（）不会更改此参数。

（iii）select（）没有sigmask参数，其行为与使用NULL sigmask调用的pselect（）相同。


```
 int select(int nfds, fd_set *readfds, fd_set *writefds,fd_set *exceptfds, struct timeval *timeout);

```

**多路复用器只是可以得到文件可以读可写的状态**，读数据需要程序自己去使用recv等函数去读取数据。

如果是程序自己读取IO，那么这个IO模型，无论是BIO，NIO，多路复用器，都是同步IO模型。

windows：IOCP 内核有线程把程序注册的方法执行，把程序的内容执行后将数据拷贝到程序内存空间，程序无需调用方法主动去读取数据。

socket->3

bind(3, 8090)

listen(3)

如果有一万个连接，1个可读

while（true）

select（fds） O（1） 返回状态（poll（fds）与select差不多，但select有1024文件的限制，可以通过编译select源代码进行改变）

accept（fd）

recv（fd）

优势：通过一次系统调用，把fds传递给内核，内核进行遍历，这种遍历减少了系统调用次数。

弊端：

1 重复传递fd  解决方案，内核开辟空间保留fd

2 每次select，poll，都要重新遍历所有的fd