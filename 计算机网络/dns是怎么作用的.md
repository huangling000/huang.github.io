用我的语言给你描述一下dns是怎么作用的

在我们访问一个服务器时，为了方便记忆，通常使用一个域名作为输入内容进行访问。但浏览器和服务端通信是需要通过主机名与端口号进行通信的。dns则是用来解析域名，得到相对应的主机名。dns实质上是一些分级管理的数据库，里面存储了域名到主机的映射。当解析一个域名的时候，首先看浏览器缓存内是否能够查询到对应主机名，如果没有，则到本地dns服务器上进行搜寻，如果本地dns服务器上有，则直接返回，如果没有，本地域名服务器通过迭代查询的方式从根域名服务器开始向下查询，最终将查询结果返回给浏览器（客户端）。

dns服务器通常在一个区域内有主dns服务器与辅助dns服务器，辅助dns服务器会定时向主dns服务发送请求进行检查更新自己内部的数据。这个过程传输的数据量大，且需要保证数据的可靠性，传输层使用tcp协议。当客户端向dns服务发送域名查询请求时，传输数据量小，且希望时间可以比较快，因此使用udp协议。