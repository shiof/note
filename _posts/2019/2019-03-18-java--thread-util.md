---
layout: article
title:	Java - 多线程之CountDownLatch、CyclicBarrier和Semaphore
date:	2019-03-18 14:48:00
categories:
    - article
tags:
    - Java
    - Thread
---

### CountDownLatch

`CountDownLatch` 是`java.util.concurrent`的一个同步辅助类，允许一个或多个线程等待，直到在其他线程中执行的一组操作完成。

`await()` 方法让当前的线程处于阻塞状态，直到`countDown()`被调用，锁存器的计数减一，如果锁存器的计数到零后，才释放所有处于等待的线程才释放。

但是`CountDownLatch` 是**一次性**的（构造器初始化时，要指定锁存器的计数），锁存器计数一旦归零后，无法被重置，之后的线程调用了`await()`也不会处于阻塞状态。

### 基础用法

~~~java
public class CountDownLatchDome {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(1);
        new Thread(()->{
            try {
                Thread.sleep(10000);
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        countDownLatch.await();
        System.out.println("等待10s后输出");
    }
}
~~~

### CyclicBarrier

`CyclicBarrier`和`CountDownLatch`的功能基本一致，但它与`CountDownLatch`不同是在释放线程之后还可以重用（`reset()重置计数`），所以称它为循环的屏障，同时还提供了一个可选的`Runnable`命令，在最后一个任务结束后执行。

### 基本使用

~~~java
public class CyclicBarrierDome {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(4, ()->{
            System.out.println("所有的线程已经执行完了");
        });

        for (int i = 0; i < 4; i++) {
            int finalI = i;
            new Thread(()->{
                try {
                    System.out.println(Thread.currentThread().getName()+ "： 执行任务 " + finalI);
                    Thread.sleep(1000);
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
~~~