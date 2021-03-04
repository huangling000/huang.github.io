epoll:一个IO事件通知工具

epollAPI执行与poll（2）类似的任务：监视多个文件描述符以查看是否可以对其中任何一个进行I/O。epollAPI既可以用作边缘触发 Level-triggered 接口，也可以用作级别触发Level-triggered接口，可以很好地扩展到大量关注的文件描述符。提供以下系统调用来创建和管理epoll实例。

epoll_create()：创建一个epoll实例并返回一个引用该实例的文件描述符。epoll_create1()是对epoll_create()的扩展。

epoll_ctl()：如果对某个特定文件描述符感兴趣，通过这个函数进行注册。当前在一个epoll实例上注册的文件描述符集称为epoll集。

epoll_wait()：等待IO事件，如果当前没有可用的事件，那么将阻塞调用线程。

epoll事件分发接口可以同时作为边缘触发（ET）和级别触发（LT）。这两种机制的区别可以描述如下。假设发生这种情况：

## epoll_create



       #include <sys/epoll.h>
       
       int epoll_create(int size);
       int epoll_create1(int flags);
 epoll_create()  creates  an  epoll(7) instance.  Since Linux 2.6.8, the size argument is ignored, but must be  greater  than  zero;  see  NOTES below.