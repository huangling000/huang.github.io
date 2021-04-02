## 什么是Executor框架

一个用于创建与管理线程地接口，即线程池。

Eexecutor作为灵活且强大的异步执行框架，其支持多种不同类型的任务执行策略，提供了一种标准的方法将任务的提交过程和执行过程解耦开发，基于生产者-消费者模式，其提交任务的线程相当于生产者，执行任务的线程相当于消费者，并用Runnable来表示任务，Executor的实现还提供了对生命周期的支持，以及统计信息收集，应用程序管理机制和性能监视等机制。

### 结构

**1 任务**：包括被执行任务需要实现的接口Runnable，Callable

**2 任务的框架**：包括任务执行机制的核心接口Executor，以及继承自Executor的ExecutorService接口。Executor框架有两个类实现了Executor接口，ThreadPoolExcutor和ScheduledThreadPoolExcutor。

**3 返回的结果**：包括Future接口与实现Future接口的FutureTask类。

## Executor与ExecutorService的区别

* ExecutorService接口继承了Executor接口
* Executor接口定义了一个execute方法接收一个runnable接口对象，且不返回任何结果，ExecutorService接口中的submit方法可以接收Runnable对象与Callable对象，并且通过一个Future对象返回运算结果
* ExecutorService还允许客户调用shutdown方法平滑的中止线程池。

**Executor**：一个接口，其定义了一个接收Runnable对象的方法executor，其方法签名为**executor(Runnable command)**,该方法接收一个Runable实例，它用来执行一个任务，任务即一个实现了Runnable接口的类，一般来说，Runnable任务开辟在新线程中的使用方法为：new Thread(new RunnableTask())).start()，但在Executor中，可以使用Executor而不用显示地创建线程：executor.execute(new RunnableTask()); // 异步执行

**ExecutorService**

是一个比Executor使用更广泛的子类接口，其提供了生命周期管理的方法，**返回 Future 对象**，以及可跟踪一个或多个异步任务执行状况返回Future的方法；可以调用ExecutorService的**shutdown（）方法**来平滑地关闭 ExecutorService，调用该方法后，将导致**ExecutorService停止接受任何新的任务且等待已经提交的任务执行完成**(已经提交的任务会分两类：一类是已经在执行的，另一类是还没有开始执行的)，当所有已经提交的任务执行完毕后将会关闭ExecutorService。因此我们一般用该接口来实现和管理多线程。

通过 **ExecutorService.submit() 方法返回的 Future 对象**，可以调用**isDone（）方法**查询Future是否已经完成。当任务完成时，它具有一个结果，你可以调用**get()方法**来获取该结果。你也可以不用isDone（）进行检查就直接调用get()获取结果，在这种情况下，get()将阻塞，直至结果准备就绪，还可以取消任务的执行。Future 提供了 cancel() 方法用来取消执行 pending 中的任务。

## Executors类

提供了一系列工厂方法用于创建线程池，返回的线程池都实现了ExecutorService接口。

**newFixedThreadPool**：创建固定数目的线程池

初始化

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
    }
 public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>(),threadFactory);
    }
```

本质：创建了可以设置核心线程数和线程工厂的ThreadPoolExecutor对象（线程池），线程池缓冲队列采用LinkedBlockingQueue，大小为Integer.MAX_VALUE，线程不执行任务的存活时间为0，但设置的核心线程会一直执行，不会超时退出。

**newCacheThreadPool**:创建一个核心线程数为0，最大线程数为整型最大值的线程池。

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());
    }
```

本质，创建了一个核心线程数为0，最大线程数为整型最大值，线程空闲时间为60s，缓冲队列为SynchronousQueue的线程池。放入线程池内的任务会尽可能地复用已经开辟的线程来执行，如果已经开辟的线程不够，则启动新的线程。直到线程达到整型最大数后抛出异常。

**newSingleThreadExecutor**：创建了一个大小为1的固定线程池

```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>()));
    }
```



使用此线程时，同时执行的任务只有1个，核心线程数与最大线程数都设置为1，其他任务在LinkedBlockingQueue中等待。线程空闲时间为0。

**newScheduledThreadPool**:创建一个corePoolSize为传入参数，最大线程数为整型的最大数的线程池，此线程池支持定时以及周期性执行任务。

```java
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```

本质是创建了一个设定了核心线程数的ScheduledThreadPoolExecutor。

### ScheduledThreadPoolExecutor类的构造

```java
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,new DelayedWorkQueue());
    }
```

线程的最大线程数为整型的最大数，线程空闲时间为0，缓冲队列为DelayedWorkQueue。

ScheduledExecutorService接口继承了ExecutorService，在ExecutorService的基础上新增了以下几个方法：

1. public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);

command：执行的任务 Callable或Runnable接口实现类

delay：延时执行任务的时间

unit：延迟时间单位

2. public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay,long period,TimeUnit unit);

command：执行的任务 Callable或Runnable接口实现类

initialDelay：第一次执行任务延迟时间

period：连续执行任务之间的周期，从上一个任务开始执行时计算延迟多少开始执行下一个任务，但是还会等上一个任务结束之后。

unit：initialDelay和period时间单位

## 自定义线程池

**ThreadPoolExcutor**

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

**corePoolSize**：线程池中有几个线程在运行

**maximumPoolSize**：最多可以有几个线程在运行

**keepAliveTime**：超出corePoolSize大小的那些线程的生存时间,这些线程如果长时间没有执行任务并且超过了keepAliveTime设定的时间，就会消亡。

unit：

**workQueue**： 存放任务的队列

**threadFactory**：创建线程的工厂

**handler**： 当workQueue已经满了，并且线程池线程数已经达到maximumPoolSize，将执行拒绝策略。

**ThreadPoolExecutor的关键属性**

```java
//这个属性是用来存放 当前运行的worker数量以及线程池状态的
//int是32位的，这里把int的高3位拿来充当线程池状态的标志位,后29位拿来充当当前运行worker的数量
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
//存放任务的阻塞队列
private final BlockingQueue<Runnable> workQueue;
//worker的集合,用set来存放
private final HashSet<Worker> workers = new HashSet<Worker>();
//历史达到的worker数最大值
private int largestPoolSize;
//当队列满了并且worker的数量达到maxSize的时候,执行具体的拒绝策略
private volatile RejectedExecutionHandler handler;
//超出coreSize的worker的生存时间
private volatile long keepAliveTime;
//常驻worker的数量
private volatile int corePoolSize;
//最大worker的数量,一般当workQueue满了才会用到这个参数
private volatile int maximumPoolSize;

```

**用户通过submit提交一个任务。线程池会执行如下流程:**

(1) 判断当前运行的worker数量是否超过corePoolSize,如果不超过corePoolSize。就创建一个worker直接执行该任务。—— 线程池最开始是没有worker在运行的

(2) 如果正在运行的worker数量超过或者等于corePoolSize,那么就将该任务加入到workQueue队列中去，按照FIFO的原则依次等待执行（线程池中有线程空闲出来后依次将缓冲队列中的任务交付给空闲的线程执行）。

(3) 如果workQueue队列满了,也就是offer方法返回false的话，就检查当前运行的worker数量是否小于maximumPoolSize,如果小于就创建一个worker直接执行该任务。

(4) 如果当前运行的worker数量是否大于等于maximumPoolSize，那么就执行RejectedExecutionHandler来拒绝这个任务的提交。

**总结起来，也即是说，当有新的任务要处理时，先看线程池中的线程数量是否大于corePoolSize，再看缓冲队列workQueue是否满，最后看线程池中的线程数量是否大于maximumPoolSize。**

另外，当线程池中的线程数量大于corePoolSize时，如果里面有线程的空闲时间超过了keepAliveTime，就将其移除线程池，这样，可以动态地调整线程池中线程的数量。

## WorkQueue的排队策略

**1 直接提交。**缓冲队列采用**SynchronousQueue**，它将任务直接交给线程处理而不保存它们。如果不存在可用于立即运行任务的线程（即线程池中的线程都在工作），则试图把任务加入缓冲队列将会失败，因此会构造一个新的线程来处理新添加的任务，并将其加入到线程池中。直接提交通常要求无界 maximumPoolSizes（Integer.MAX_VALUE） 以避免拒绝新提交的任务。newCachedThreadPool采用的便是这种策略。

**2 无界队列。**   **LinkedBlockingQueue**，理论上是可以对无限多的任务排队，当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列。newFixedThreadPool采用的便是这种策略。

**3 有界队列。**当使用有限的 *maximumPoolSizes* 时，有界队列（一般缓冲队列使用**ArrayBlockingQueue**，并制定队列的最大长度）有助于防止资源耗尽，但是可能较难调整和控制，队列大小和最大池大小需要相互折衷，需要设定合理的参数。

## 创建线程池的正确方法

1. 线程资源必须通过线程池创造，不要显示地创造一个线程。因为线程池可以灵活管理创建的线程，尽量做到线程复用，减少系统在创建与销毁线程上做的开销，并且管理线程的数量与执行，解决创造大量同类线程导致消耗完内存或“过度切换”的问题。

2. 线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式进行创建，这样可以更加明确线程池的运行规则，避免资源耗尽的风险。

   FixedThreadPool和SingleThreadPool会导致堆积大量的请求（因为使用了LinkedBlockingQueue），从而导致OOM

   CacheThreadPool创建线程最大数量为Integer.MAX_VALUE，可能会创建大量线程，导致OOM。

**正确方法**

1. 直接调用ThreadPoolExecutor的构造函数，自己指定最大线程数与BlockingQueue容量。

```java
private static ExecutorService executor = new ThreadPoolExecutor(10, 10,
        60L, TimeUnit.SECONDS,
        new ArrayBlockingQueue(10));
```

这种情况下，提交任务时，发现核心线程数已达到，缓冲队列已满，最大线程数也以达到，那么直接抛出异常java.util.concurrent.RejectedExecutionException。

2. 使用开源类库：如apache和guava等。

```java
//使用guava提供的ThreadFacterBuilder创建线程池
public class ExecutorsDemo {

    private static ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
        .setNameFormat("demo-pool-%d").build();

    private static ExecutorService pool = new ThreadPoolExecutor(5, 200,
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

    public static void main(String[] args) {

        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            pool.execute(new SubThread());
        }
    }
}
```

