---
layout: article
title:	Java - 多线程之基础
date:	2019-03-16 11:02:00
categories:
    - article
tags:
    - Java
    - Thread
---

多线程是Java编程中的核心

为什么用使用多线程呢!!~其实多线不一定能提升程序的性能，可能反而降低性能。CPU的运行时间是很宝贵，一但线程处于阻塞状态的话（例如请求的后台数据库的时候），CPU就处于的空
转，很是浪费CPU的的时间。所以把任务交给多个线程去执行，当某个线程阻塞之后，其他的线程继续执行任务，最大利用的CPU的效率。

但多线程下也会导致很多问题，比如共享全局变量导致线程不安全，死锁等问题。

### 基本概念

1. **程序**
    
    是指令和数据的有序集合，其本身没有任何运行的含义，是一个静态的概念。

2. **进程**
    
    是程序被加内存中执行过程。每一个进程至少要有一个线程（main）。

3. **线程**

    是进程的一个实例,是操作系统能够进行运算调度的最小单位,也是进程中的实际运作单位。在单线程的模型中，一旦线程的挂掉，进程也会崩溃。

4. **多线程**
    
    进程像一列火车一样，而线程就是火车中的车厢。火车可以由多节车厢组成，也可以是只有一节车厢组成的火车。

进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，因为线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间。

### 线程状态

1. **新建状态(New)** 线程对象(`new Thread()`)创建 
2. **就绪状态(Runnable)** 该对象的`start()`执行之后，处于随时被CPU调度
3. **运行状态(Running)** 线程获取CPU权限进行执行
4. **阻塞状态(Blocked)** 线程放弃CPU使用权
    
    4.1 等待堵塞：执行的线程执行wait()方法，JVM会把该线程放入等待池中。

    4.2 同步堵塞：执行的线程在获取对象的同步锁时，若该同步锁被别的线程占用。则JVM会把该线程放入锁池中。

    4.3 其它堵塞：执行的线程执行sleep()或join()方法，或者发出了I/O请求时。JVM会把该线程置为堵塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完成时。线程又一次转入就绪状态

    > 线程执行`yieid()`、`sleep()`、`join()`或者`wait()`都可以让线程放弃CPU的使用权处于阻塞状态。
    > `notify()`,`notifyAll()`可以唤醒线程处于就绪状态，等待CPU再一次调用
    >`wait()`和`notify()`,`notifyAll()`是Object类的方法，`sleep()`和`yield()`是Thread类的方法
    >`wait()`会释放对象的同步锁，而`sleep()`和`yield()`则不会释放锁。
5. **死亡状态(Dead)** 线程运行完了或者因异常退出了run()方法，该线程结束生命周期。

![677054-20170401135927524-613651161](https://user-images.githubusercontent.com/29170657/54470153-1e68b080-47de-11e9-977c-19a55c88f302.jpg)

### 创建线程的几种方式

匿名调用

~~~java
public class MyTread {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("我是新建一个子线程");
            }
        }).start();
    }
}
~~~

`Runnable`是一个函数式接口，也可以使用`Lambda`表达式

~~~java
public class MyTread {
    public static void main(String[] args) {
        new Thread(()->System.out.println("我是新建一个子线程")).start();
    }
}
~~~

继承`Thread`

~~~java
public class MyTread extends Thread{
    @Override
    public void run() {
        System.out.println("我是新建一个子线程");
    }

    public static void main(String[] args) {
        new MyTread().start();
    }
}
~~~

现实`Runnable`

~~~java
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        System.out.println("我是新建一个子线程");
    }

    public static void main(String[] args) {
        new Thread(new MyRunnable()).start();
    }
}
~~~

#### 线程池

~~~java
public interface Executor {
    /**
    * 提交线程
    */
    void execute(Runnable command);
}
~~~

~~~java
public interface ExecutorService extends Executor {
    /**
    * 启动有序关闭，其中先前提交的任务将被执行，但不会接受任何新任务
    */
    void shutdown();
    /*
    * 尝试停止所有主动执行的任务，停止等待任务的处理，并返回正在等待执行的任务列表
    */
    List<Runnable> shutdownNow();
    /*
    * 如果此执行者已关闭，则返回 true
    */
    boolean isShutdown();
    /*
    * 如果所有任务在关闭后完成，则返回 true
    */
    boolean isTerminated();
    /*
    * 阻止所有任务在关闭请求完成后执行，或发生超时，或当前线程中断，以先到者为准
    */
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    /*
    * 提交一个可运行的任务执行，并返回一个表示该任务的`Future<?>`
    */
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    /*
    * 执行给定的任务，返回持有他们的状态和结果的所有完成的`Future<?>`列表
    */
    List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
~~~

~~~java
/**
* @param corePoolSize 池中要保留的线程数
* @param maximumPoolSize 类中允许的最大线程数池
* @param keepAliveTime 超出核心线程数量以外的线程空余存活时间
* @param unit 存活时间的单位
* @param workQueue 保存待执行任务的队列
* @param threadFactory 存放任务的队列
* @param handler 超出线程范围和队列容量的任务的处理程序
*/
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
~~~

Executors提供四种线程池

1. newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

    ~~~java
    class MyCachedTreadPool {
        public static void main(String[] args) {
            ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
            for (int i = 0; i < 100; i++) {
                int currentI = i;
                cachedThreadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + ":执行任务" + currentI);
                });
            }
        }
    }
    ~~~
2. newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

    ~~~java
    class MyFixedTreadPool {
        public static void main(String[] args) {
            ExecutorService fixedThreadPool = Executors.newFixedThreadPool(2);
            for (int i = 0; i < 100; i++) {
                int currentI = i;
                fixedThreadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + ":执行任务" + currentI);
                });
            }
        }
    }
    ~~~

3. newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。

    ~~~java
    class MyScheduledThreadPool {
        public static void main(String[] args) {
            ExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(2);
            ((ScheduledExecutorService) scheduledThreadPool).scheduleAtFixedRate(() -> {
                System.out.println(Thread.currentThread().getName() + ":延迟1秒后每3秒执行一次任务");
            }, 1, 3, TimeUnit.SECONDS);
        }
    }
    ~~~

4. newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

    ~~~java
    class MySingleThreadPool {
        public static void main(String[] args) {
            ExecutorService singleThreadPool = Executors.newSingleThreadExecutor();
            for (int i = 0; i < 100; i++) {
                int currentI = i;
                singleThreadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + ":执行任务" + currentI);
                });
            }
        }
    }
    ~~~