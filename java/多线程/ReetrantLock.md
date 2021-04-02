## 同步互斥

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

### 1 synchronized

**1 同步一个代码块**

```
public void func() {
    synchronized (this) {
        // ...
    }
}
```

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

**2 同步一个方法**

```
public synchronized void func () {
    // ...
}
```

**3 同步一个类**

```
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

**作用于整个类**，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

**4 同步一个静态方法**

```
public synchronized static void fun() {
    // ...
}
```

### ReenTrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```java
import java.util.concurrent.locks.*;
public class LockExample{
    //创建一个lock对象
    private Lock lock = new ReentrantLock();
    public void lock(){
        //用lock对象对方法进行上锁
        lock.lock();
        try{
            for(int i = 0; i < 10;i++){
                System.out.println(i);
            }
        }finally{
            //确保锁被释放
            lock.unlock();
        }
        
    }
}

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class Test{
    public static void main(String[] args){
        
        LockExample lockExample = new LockExample();
        ExecutorService executor = Executors.newCachedThreadPool();
        executor.execute(()->lockExample.lock());
        executor.execute(()->lockExample.lock());
    }
}
```

### 比较

1 锁的实现：synchronized是JVM实现的，ReenTrantLock是jdk实现的。

2 性能：优化后的synchronized性能与ReenTrantLock差不多。

3 等待可中断：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

4 公平锁：

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

5 绑定多个条件

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

