### 重做日志

用来崩溃恢复。一条记录需要更新时，InnoDB引擎会先把记录写到redolog里，并更新内存。更新完成。mysql崩溃时，从记录的redolog中找到提交的更改内容所以不会担心Mysql异常重启后，数据的丢失。

它由两部分组成：**redolog buffer**，**redolog file**

先写日志，再写磁盘，即预写日志的功能。当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log里面，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做

它们在磁盘上会落实成固定个数固定大小的日志文件。一般有2个或4个，这可以在初始化MySQL实例的时候，可以进行配置。

mysql的这种崩溃恢复功能就是cache-safe，而实现这个cache-safe功能的主要组件就是redolog。只有InnoDB才有这个日志，所以它代替myisam成为主流存储引擎原因之一。

redolog属于物理日志，记录每一个page页中具体存储的值，在数据页上做什么修改。与物理日志对应的是逻辑日志，逻辑日志是记录的每一个page页面中具体数据是怎么变动的。它会记录一个变动的过程。

### binlog(归档日志)

1 redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。 

2 redo log 是物理日志，记录的是“在某个数据页上做了什么修改”,虽然没有保存整页数据，但是可以记录页面级别的变更。binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。 

3 redo log 是循环写的，空间固定会用完;binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

### 两段提交

![](https://img-blog.csdnimg.cn/20210113222656788.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2phdmFhbmRkb25ldA==,size_16,color_FFFFFF,t_70)

一条更新语句大致执行流程：

1 server服务层的执行器，调用存储引擎层的API接口，去查询数据

2 innoDB存储引擎层查询buffer pool缓存中的数据

3 如果查询缓存中包含待查询的数据，则直接返回给server服务层的执行端

4 如果缓存中没有结果则从磁盘中去读取数据，读数据后，再返回给server服务层，同时把查询到的数据更新到buffer pool中的数据内容。

5 server层的执行器收到查询后的数据后，执行更新操作。

6 server层调用innoDB存储引擎层的API接口更新数据。

7 innoDB存储引擎层更新数据到change buffer缓存池中

8 innoDB存储引擎层记录redolog，并把其状态设置为prepare状态。

9 innoDB存储引擎层通知server的执行器，change buffer已经更新，redolog进入prepare状态待提交，可以记录binlog日志

10 server层的执行器记录binlog到binlog的缓存池中。

11 server层的执行器在记录完binlog之后，通知innoDB存储引擎层，binlog已经记录完成

12 innoDB存储引擎层收到server记录完binlog的通知后，更新redolog buffer中的redolog为commit状态。

至此，更新语句执行完成。可以看到在记录redolog的过程中，有两个过程，第一个状态提交为prepare状态，然后等待server层的binlog日志记录完成后，更新为commit状态。

通常我们说 MySQL 的“双 1”配置，指的就是 sync_binlog 和 innodb_flush_log_at_trx_commit 都设置成 1。也就是说，一个事务完整提交前，需要等待两次刷盘，一次是 redo log(prepare 阶段)，一次是 binlog。

### redolog对事务的支持
事物的四大特性之一：持久性 ，在MySQL的innodb存储引擎底层来说就是靠redolog来实现的。具体来说就是只要事务提交成功，那么对数据库做的修改就被永久保存下来了，不可能因为任何原因再回到原来的状态，通过前面的介绍，我们可以知道在崩溃恢复的时候，对已经提交成功事物，就是从redolog日志文件中恢复的。所以redolog是支持事务的innodb存储引擎实现事务的持久性的方式。

MySQL在异常重启做恢复的时候，会去比较redolog中的内容和binlog中的内容。两阶段提交是实现crash-safe功能核心。如果不使用“两阶段提交”，数据库的状态就有可能和用它的日志恢复出来的库的状态不一致。

如果redolog改为prepare状态，binlog记录完成之后，redolog还没来得及改为commit状态的时候，比较两log内容一致，mysql异常重启也能从redolog中恢复数据。如果binlog没有完成记录，redolog已经改为prepare状态，通过两个log内容比较发现不一致，会认为这条记录是无效的，回滚事务。

**发生的时机**

单独写一个update时

begin。。。commit语句序列，执行到commit语句发生

### binlog写入机制

事务执行过程中，先把日志写到 binlog cache，事 务提交的时候，再把 binlog cache 写到 binlog 文件中。

write 和 fsync 的时机，是由参数 **sync_binlog** 控制的:

sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync;

sync_binlog=1 的时候，表示每次提交事务都会执行 fsync;

**sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才fsync。**

因此，**在出现 IO 瓶颈的场景里，将 sync_binlog 设置成一个比较大的值，可以提升性 能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0， 比较常见的是将其设置为 100~1000 中的某个数值。**

但是，将 sync_binlog 设置为 N，对应的风险是:如果主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。

### redolog日志的刷盘（写入机制）

为了提高写数据的效率，并不是直接把数据写在磁盘上，而是先写在一个缓存池buffer中，然后再从buffer中再刷新到磁盘上。redolog也是采用类似的方式，他有一个redolog buffer的缓冲池用于存储数据，然后再从这个缓冲池总刷新数据到磁盘中。具体什么时候把数据刷新到磁盘，有参数进行控制。

redolog buffer里的内容不需要每次生成后都要直接持久化到磁盘。如果事务执行期间，Mysql发生异常重启，那这部分日志就丢了。由于事务并没有提交，所以这时日志丢了也不会有损失。

但事务还没提交，redo log buffer中的部分日志也有可能被持久化到磁盘。

**写入状态**

![](http://eternityz.gitee.io/image/image_hosting/mysql-image/23.redolog%E5%92%8Cbinlog%E7%9A%84%E5%86%99%E5%85%A5%E6%B5%81%E7%A8%8B/%E5%9B%BE2redolog%E5%AD%98%E5%82%A8%E7%8A%B6%E6%80%81.png)

1 redo log buffer中，物理上是在mysql进程内存中

2 write到文件系统的page cache里

3 持久化到磁盘，对应的是hard disk。

12速度都比较快，3的速度较慢

**写入策略**

为了控制redo log的写入策略，innoDB提供了innodb_flush_log_at_trx_commit 参 数，它有三种可能取值:

1. 设置为 0 的时候，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中 ;
2. 设置为 1 的时候，表示每次事务提交时都将 redo log 直接持久化到磁盘;
3. 设置为 2 的时候，表示每次事务提交时都只是把 redo log 写到 page cache。

InnoDB 有一个后台线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写 到文件系统的 page cache，然后调用 fsync 持久化到磁盘。

注意，事务执行中间过程的 redo log 也是直接写在 redo log buffer 中的，这些 redo log 也会被后台线程一起持久化到磁盘。也就是说，一个没有提交的事务的 redo log，也 是可能已经持久化到磁盘的。

实际上，除了后台线程每秒一次的轮询操作外，还有两种场景会让一个没有提交的事务的 redo log 写入到磁盘中。

**一种是，redo log buffer 占用的空间即将达到 innodb_log_buffer_size 一半的时 候，后台线程会主动写盘。**

**另一种是，并行的事务提交的时候，顺带将这个事务的 redo log buffer 持久化到磁 盘。**

通过设置两个log参数，可以设置规定它们写入磁盘的方式，从而提升一定的IO性能。如innodb_flush_log_at_trx_commit 设置为1，那么就能在提交时让redolog暂时先保存在page cache中国，风险是主机断电时会丢失数据。将sync_binlog设置为大于1的值，风险是主机掉电会丢binlog日志。

