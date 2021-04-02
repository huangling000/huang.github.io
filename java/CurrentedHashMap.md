相比HashMap而言，是多线程的。底层数据与HashMap数据结构相同。

HashMap的数据结构（数组+链表+红黑树），桶中的结构可能是链表，也可能是红黑树，红黑树是为了提高查找效率。HashMap在多线程下会形成环形链表。

ConcurrentHashMap继承了AbstractMap抽象类，该抽象类定义了一些基本操作，同时，也实现了ConcurrentMap接口，ConcurrentMap接口也定义了一系列操作，实现了Serializable接口表示ConcurrentHashMap可以被序列化。

### CAS原理分析（Compare and Swap）

CAS加volatile关键字是实现并发包的基石，没有CAS就不会有并发包。**sychronized**是一种独占锁，悲观锁，java.util.concurrent中借助了CAS指令实现了一种区别于sychronized的一种乐观锁。

**乐观锁**：每次拿数据时都认为别的线程不会修改数据，所以不会上锁。但是在更新的时候会判断一下在此期间别的线程有没有修改过数据。乐观锁适用于读操作多的场景，可以提高程序的吞吐量。在Java中java.util.concurrent.atomic包下面的原子变量就是使用了乐观锁的一种实现方式CAS实现。

**悲观锁**：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁。这样下一个线程想拿这个数据时，需要等待上一个线程使用完毕释放锁，否则会一直阻塞。java中的synchronized的实现也是一种悲观锁。

### 乐观锁的实现方式-CAS

**jdk1.5之前锁存在的问题**：

1.5之前都是靠sychronized关键字保证同步，sychronized保证了无论哪个线程持有共享变量的锁，都会采用独占的方式来访问这些变量。这种情况下：

1 在多线程竞争下，加锁，释放锁会导致较多的上下文切换和调度延时，引起性能问题

2 如果一个线程持有锁，其他的线程就都会挂起，等待持有锁的线程释放锁。

3 如果一个优先级高的线程等待一个优先级低的线程释放锁，会导致优先级倒置，引起性能风险

乐观锁：每次不加锁而是假设没有并发冲突去操作同一变量，如果有并发冲突导致失败，则重试直至成功。

**乐观锁**

冲突检测+数据更新

典型的实现机制（CAS):

CAS操作包括三个操作数：**需要读写的内存位置（V），预期原值（A），新值（B），**如果内存位置与预期原值相匹配，那么将内存位置的值更新为B，不匹配则不做任何操作。

**乐观锁是一种思想，CAS只是这种思想的一种实现方式。**

在jdk1.5新增的java.util.concurrent(JUC java并发工具包）就是建立在CAS之上的。相比synchronized这种堵塞算法，CAS是非堵塞算法的一种常见实现。

JUC下的原子操作类AtomicInteger.getAndIncrement()方法：

```
public class AtomicInteger extends Number implements java.io.Serializable {  
    private volatile int value; // volatile修饰value
 
    public final int get() {  
        return value;  
    }  
 
    public final int getAndIncrement() {  
        for (;;) {  
            int current = get();  
            int next = current + 1;  
            //compareAndSet方法利用JNI完成CPU的操作
            if (compareAndSet(current, next))  
                return current;  
        }  
    }  
 
    public final boolean compareAndSet(int expect, int update) {  
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  
    }  
}
```

JNI中compareAndSwapInt类似逻辑如下：

```
if (this == expect) {
     this = update
     return true;
 } else {
     return false;
 }
```

this == except和this == update如何保证原子性？

java实现CAS的原理：

compareAndSwapInt是借助调用底层CPU指令来实现的。程序会根据当前处理器类型来决定是否为cmpxchg指令添加lock前缀。如果程序是在多线程上运行，就会为cmpschg指令加上lock前缀，如果是在单处理器上运行，就省略lock前缀。

**CAS缺陷：**

1 ABA问题

一个线程从内存V中取出A，这时候另一个线程也从内存A中取出A，并进行一些操作变成了B，然后又将位置变成A，这时线程one进行CAS操作发现内存中仍然是A。然后one操作成功。尽管one操作成功，但仍然存在潜在问题。

如一个单向链表实现的栈，栈顶为A，此时线程T1已经知道A.next为B，然后希望用CAS将栈顶替换为B：　head.compareAndSet(A,B); 在T1执行上面这条指令之前，线程T2介入，将A、B出栈，再pushD、C、A，此时堆栈结构如下图，而对象B此时处于游离状态：

![](https://img-blog.csdn.net/20180813094930693?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTEzNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

此时轮到线程T1执行CAS操作，检测发现栈顶仍为A，所以CAS成功，栈顶变为B，但实际上B.next为null，所以此时的情况变为：

![](https://img-blog.csdn.net/20180813094950294?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTEzNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

其中堆栈中只有B一个元素，C和D组成的链表不再存在于堆栈中，平白无故就把C、D丢掉了。

解决方法：（AtomicStampedReference 带有时间戳的对象引用）

从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

```java
public boolean compareAndSet(
               V      expectedReference,//预期引用
               V      newReference,//更新后的引用
              int    expectedStamp, //预期标志
              int    newStamp //更新后的标志
)
```

2 循环时间长，开销大

自旋CAS（不成功，就一直循环执行，直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

3 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是**对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁**，或者有一个取巧的办法，就是**把多个共享变量合并成一个共享变量来操作**。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

**CAS与synchronized使用情景：**

1 对于竞争资源较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户内核态间切换操作额外浪费cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少。可以获得较好的性能

2 对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

jdk1.6之后，synchronized的底层实现主要依靠Lock-Free的队列，基本思路是自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。在线程冲突较少的情况下，可以获得和CAS类似的性能；但冲突严重的情况下，性能远高于CAS

**concurrent包**

由于java的CAS同时具有 volatile 读和volatile写的内存语义，因此Java线程之间的通信现在有了下面四种方式：

1. A线程写volatile变量，随后B线程读这个volatile变量。
2. A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
3. A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
4. A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。
   

Java的CAS会使用现代处理器上提供的高效机器级别原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键（从本质上来说，能够支持原子性读-改-写指令的计算机器，是顺序计算图灵机的异步等价机器，因此任何现代的多处理器都会去支持某种能对内存执行原子性读-改-写操作的原子指令）。同时，volatile变量的读/写和CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现的基石。如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式：

　1. 首先，声明共享变量为volatile；　
 　2.  然后，使用CAS的原子条件更新来实现线程之间的同步；
 　3. 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

AbstractQueuedSynchronizer（AQS抽象的队列同步器），atomic类，非堵塞数据结构这些concurrent包中的基础类都是使用这种模式实现的，而concurrent包中的高层类又是依赖于这些基础类实现的。concurrent包的整体示意图如下：

![](https://img-blog.csdn.net/20180813101953553?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTEzNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### CurrentedHashMap实现

1.8版本之前：采用segment分段锁的形式对table进行管理。segment中的每个元素是一一段hashtable表（一个数组），继承了ReentrantLock类，调用该类的接口可以是获得对象（这个segment对象，即segement中的hahsmap）的锁以及释放操作。若遇到hash冲突，节点插在当前位置链表的链表头。Segment块默认16个，并发度是16，可以初始化时显式指定，后期不能修改。

1.8版本之后：采用HashMap+CAS+synchronized+红黑树对currentedHasgMap实现。不再采用分段锁，而是直接对表的node进行处理，每个node都有自己的并发度。

### 1.8中原理和实现

**数据结构**

摒弃Segment概念，直接用Node数组+链表+红黑树的数据结构实现。并发控制使用Synchronized和CAS来操作。

一些产量设计和数据结构

个人认为最大容量和默认初始值，负载因子，链表转红黑树阈值，红黑树转链表阈值，控制标识符比较重要

```
// node数组最大容量：2^30=1073741824
private static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认初始值，必须是2的幕数
private static final int DEFAULT_CAPACITY = 16;
//数组可能最大值，需要与toArray（）相关方法关联
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
//并发级别，遗留下来的，为兼容以前的版本
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
// 负载因子
private static final float LOAD_FACTOR = 0.75f;
// 链表转红黑树阀值,> 8 链表转换为红黑树
static final int TREEIFY_THRESHOLD = 8;
//树转链表阀值，小于等于6（tranfer时，lc、hc=0两个计数器分别++记录原bin、新binTreeNode数量，<=UNTREEIFY_THRESHOLD 则untreeify(lo)）
static final int UNTREEIFY_THRESHOLD = 6;
static final int MIN_TREEIFY_CAPACITY = 64;
private static final int MIN_TRANSFER_STRIDE = 16;
private static int RESIZE_STAMP_BITS = 16;
// 2^15-1，help resize的最大线程数
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
// 32-16=16，sizeCtl中记录size大小的偏移量
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
// forwarding nodes的hash值
static final int MOVED     = -1; 
// 树根节点的hash值
static final int TREEBIN   = -2; 
// ReservationNode的hash值
static final int RESERVED  = -3; 
// 可用处理器数量
static final int NCPU = Runtime.getRuntime().availableProcessors();
//存放node的数组
transient volatile Node<K,V>[] table;
/*控制标识符，用来控制table的初始化和扩容的操作，不同的值有不同的含义
 *当为负数时：-1代表正在初始化，-N代表有N-1个线程正在 进行扩容
 *当为0时：代表当时的table还没有被初始化
 *当为正数时：表示初始化或者下一次进行扩容的大小*/
private transient volatile int sizeCtl;
```

put操作

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode()); //两次hash，减少hash冲突，可以均匀分布
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) { //对这个table进行迭代
        Node<K,V> f; int n, i, fh;
        //这里就是上面构造方法没有进行初始化，在这里进行判断，为null就调用initTable进行初始化，属于懒汉模式初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {//如果i位置没有数据，就直接无锁插入
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)//如果在进行扩容，则先进行扩容操作
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //如果以上条件都不满足，那就要进行加锁操作，也就是存在hash冲突，锁住链表或者红黑树的头结点
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) { //表示该节点是链表结构
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //这里涉及到相同的key进行put就会覆盖原先的value
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {  //插入链表尾部
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {//红黑树结构
                        Node<K,V> p;
                        binCount = 2;
                        //红黑树结构旋转插入
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) { //如果链表的长度大于8时就会进行红黑树的转换
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);//统计size，并且检查是否需要扩容
    return null;
}
```

put过程大概有以下几个步骤（对当前的table无条件自循环直到put成功）：

1 如果table还没有初始化先进行初始化；

2 如果没有hash冲突直接CAS插入

3 如果还在进行扩容就先进行扩容

4 如果存在hash冲突，就加锁来保证线程安全，一种是链表形式就直接遍历到尾端插入，一种是红黑树就按照红黑树结构插入

5 如果该链表的数量大于阈值8，就要先转换成红黑树的结构，break再一次进入循环

6 添加成功调用addCount（）方法统计size，检查是否需要扩容。

initTable方法：

```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {//空的table才能进入初始化操作
        if ((sc = sizeCtl) < 0) //sizeCtl<0表示其他线程已经在初始化了或者扩容了，挂起当前线程 
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {//CAS操作SIZECTL为-1，表示初始化状态
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];//初始化
                    table = tab = nt;
                    sc = n - (n >>> 2);//记录下次扩容的大小
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

如果相应位置Node还未初始化，则通过CAS插入相应的数据

```java
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
/********************************/
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
```

如果容器正在扩容，则会调用helpertransfer（）方法帮助扩容

```java
/**
 * Helps transfer if a resize is in progress.
 * 帮助从旧的table的元素复制到新的table中
 */
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    if (tab != null && (f instanceof ForwardingNode) &&
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) { //新的table nextTba已经存在前提下才能帮助扩容
        int rs = resizeStamp(tab.length);
        while (nextTab == nextTable && table == tab &&
               (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                transfer(tab, nextTab);//调用扩容方法
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

