

# ReentrantLock
在JDK5.0版本之前，重入锁的性能远远好于synchronized关键字，JDK6.0版本之后synchronized 得到了大量的优化，二者性能也不分伯仲，但是重入锁是可以完全替代synchronized关键字的。除此之外，重入锁又自带一系列高逼格UBFF：可中断响应、锁申请等待限时、公平锁。另外可以结合Condition来使用，使其更是逼格满满。

```java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockTest implements Runnable{
    public static ReentrantLock lock = new ReentrantLock();
    public static int i = 0;

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
            lock.lock();  //lock.lock()  ①
            try {
                i++;
            } finally {
                lock.unlock(); // lock.unlock()②
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockTest test = new ReentrantLockTest();
        Thread t1 = new Thread(test);
        Thread t2 = new Thread(test);
        t1.start();t2.start();
        t1.join(); t2.join(); // main线程会等待t1和t2都运行完再执行以后的流程
        System.err.println(i);
    }
}
```
使用重入锁进行加锁是一种显式操作，通过何时加锁与释放锁使重入锁对逻辑控制的灵活性远远大于synchronized关键字。同时，需要注意，有加锁就必须有释放锁，而且加锁与释放锁的分数要相同，这里就引出了“重”字的概念，如上边代码演示，放开①、②处的注释，与原来效果一致。

## 响应中断
```java
public class KillDeadlock implements Runnable{
    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();
    int lock;

    public KillDeadlock(int lock) {
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            if (lock == 1) {
                lock1.lockInterruptibly();  // 以可以响应中断的方式加锁
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {}
                lock2.lockInterruptibly();
            } else {
                lock2.lockInterruptibly();  // 以可以响应中断的方式加锁
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {}
                lock1.lockInterruptibly();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (lock1.isHeldByCurrentThread()) lock1.unlock();  // 注意判断方式
            if (lock2.isHeldByCurrentThread()) lock2.unlock();
            System.err.println(Thread.currentThread().getId() + "退出！");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        KillDeadlock deadLock1 = new KillDeadlock(1);
        KillDeadlock deadLock2 = new KillDeadlock(2);
        Thread t1 = new Thread(deadLock1);
        Thread t2 = new Thread(deadLock2);
        t1.start();t2.start();
        Thread.sleep(1000);
        t2.interrupt(); // ③
    }
}

```
t1、t2线程开始运行时，会分别持有lock1和lock2而请求lock2和lock1，这样就发生了死锁。但是，在③处给t2线程状态标记为中断后，持有重入锁lock2的线程t2会响应中断，并不再继续等待lock1，同时释放了其原本持有的lock2，这样t1获取到了lock2，正常执行完成。t2也会退出，但只是释放了资源并没有完成工作。



## 锁申请等待限时
可以使用 tryLock()或者tryLock(long timeout, TimeUtil unit) 方法进行一次限时的锁等待。

前者不带参数，这时线程尝试获取锁，如果获取到锁则继续执行，如果锁被其他线程持有，则立即返回 false ，所以不会产生死锁。 
后者带有参数，表示在指定时长内获取到锁则继续执行，如果等待指定时长后还没有获取到锁则返回false。
```java
public class TryLockTest implements Runnable{
    public static ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        try {
            if (lock.tryLock(1, TimeUnit.SECONDS)) { // 等待1秒
                Thread.sleep(2000);  //休眠2秒
            } else {
                System.err.println(Thread.currentThread().getName() + "获取锁失败！");
            }
        } catch (Exception e) {
            if (lock.isHeldByCurrentThread()) lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        TryLockTest test = new TryLockTest();
        Thread t1 = new Thread(test); t1.setName("线程1");
        Thread t2 = new Thread(test); t1.setName("线程2");
        t1.start();t2.start();
    }
}
/**
 * 运行结果:
 * 线程2获取锁失败！
 */ 
```
上述示例中，t1先获取到锁，并休眠2秒，这时t2开始等待，等待1秒后依然没有获取到锁，就不再继续等待，符合预期结果。

## 公平锁
所谓公平锁，就是按照时间先后顺序，使先等待的线程先得到锁，而且，公平锁不会产生饥饿锁，也就是只要排队等待，最终能等待到获取锁的机会。使用重入锁（默认是非公平锁）创建公平锁：
```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}

public class FairLockTest implements Runnable{
    public static ReentrantLock lock = new ReentrantLock(true);

    @Override
    public void run() {
        while (true) {
            try {
                lock.lock();
                System.err.println(Thread.currentThread().getName() + "获取到了锁！");
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        FairLockTest test = new FairLockTest();
        Thread t1 = new Thread(test, "线程1");
        Thread t2 = new Thread(test, "线程2");
        t1.start();t2.start();
    }
}
/**
 * 运行结果:
 * 线程1获取到了锁！
 * 线程2获取到了锁！
 * 线程1获取到了锁！
 * 线程2获取到了锁！ 
 * ......（上边是截取的一段）
 */
```
1和t2交替获取到锁。如果是非公平锁，会发生t1运行了许多遍后t2才开始运行的情况。

## 配合 Conditon
配合关键字synchronized使用的方法如：await()、notify()、notifyAll()，同样配合ReentrantLock 使用Conditon
### Conditon
任何一个java对象都天然继承于Object类，在线程间实现通信的往往会应用到Object的几个方法，比如
```java
wait(),
wait(long timeout),
wait(long timeout, int nanos)
notify(),
notifyAll()
```
几个方法实现等待/通知机制，同样的， 在java Lock体系下依然会有同样的方法实现等待/通知机制。从整体上来看Object的wait和notify/notify是与对象监视器配合完成线程间的等待/通知机制，而Condition与Lock配合完成等待通知机制，前者是java底层级别的，后者是语言级别的，具有更高的可控制性和扩展性。两者除了在使用方式上不同外，在功能特性上还是有很多的不同：

- Condition能够支持不响应中断，而通过使用Object方式不支持；
- Condition能够支持多个等待队列（new 多个Condition对象），而Object方式只能支持一个；
- Condition能够支持超时时间的设置，而Object不支持


```java
public interface Condition {
    //当前线程进入等待状态，如果其他线程调用condition的signal或者signalAll方法并且当前线程获取Lock从await方法返回，如果在等待状态中被中断会抛出被中断异常；
    void await() throws InterruptedException; 

    void awaitUninterruptibly(); // 与await()相同，但不会再等待过程中响应中断

    //当前线程进入等待状态直到被通知，中断或者超时；
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    boolean awaitUntil(Date deadline) throws InterruptedException;

    //唤醒一个等待在condition上的线程，将该线程从等待队列中转移到同步队列中，如果在同步队列中能够竞争到Lock则可以从等待方法中返回。
    void signal(); // 类似于Obejct.notify()

    //唤醒所有等待在condition上的线程
    void signalAll();
}
```
调用condition的signal或者signalAll方法可以将等待队列中等待时间最长的节点移动到同步队列中，使得该节点能够有机会获得lock。按照等待队列是先进先出（FIFO）的，所以等待队列的头节点必然会是等待时间最长的节点，也就是每次调用condition的signal方法是将头节点移动到同步队列中。

## 例子

ReentrantLock 实现了Lock接口，可以通过该接口提供的newCondition()方法创建Condition对象：
```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();

    //可以多次调用lock.newCondition()方法创建多个condition对象，也就是一个lock可以持有多个等待队列。而在之前利用Object的方式实际上是指在对象Object对象监视器上只能拥有一个同步队列和一个等待队列，而并发包中的Lock拥有一个同步队列和多个等待队列
    Condition newCondition();
}


import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockWithConditon implements Runnable{
    public static ReentrantLock lock = new ReentrantLock(true);
    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
        
        try {
            lock.lock();
            System.err.println(Thread.currentThread().getName() + "-线程开始等待...");
            //调用condition.await方法释放锁,将当前线程加入到等待队列
            condition.await();

            //如果该线程能够从await()方法返回的话一定是该线程获取了与condition相关联的lock
            System.err.println(Thread.currentThread().getName() + "-线程继续进行了");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockWithConditon test = new ReentrantLockWithConditon();
        Thread t = new Thread(test, "线程ABC");
        t.start();
        Thread.sleep(1000);
        System.err.println("过了1秒后...");

        //获取锁
        lock.lock();
        //唤醒一个
        condition.signal(); // 调用该方法前需要获取到创建该对象的锁否则会产生java.lang.IllegalMonitorStateException异常
        //释放锁
        lock.unlock();
    }
}







