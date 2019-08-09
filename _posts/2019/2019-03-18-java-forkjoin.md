---
layout: article
title:	Java - 多线程之Fork/Join
date:	2019-03-18 09:20:00
categories:
    - article
tags:
    - Java
    - Thread
---

`ForkJoinPool` 是JAVA1.7的新特性，是将一个大任务拆分成若干子任务（拆分到不能拆分）交给不同的**线程**去处理，再将一个个小任务的结果进行join汇总，有点像大数据的`MapReduce`的感觉

它提供基本的线程池功能，支持设置最大并发线程数，支持任务排队，支持线程池停止，支持线程池使用情况监控，也是AbstractExecutorService的子类，主要引入了“工作窃取”机制，在多CPU计算机上处理性能更佳

### 工作窃取模式

新任务被拆分成若干小任务加到**线程队列**中，并将小任务交给不同的线程去执行。当某个线程执行完任务的之后，会随机从其他线程的队列的尾部，偷一个任务放进自己的队列中

![image](https://user-images.githubusercontent.com/29170657/54501401-0caf1680-4960-11e9-8992-c2a17c2288ee.png)

### ForkJoinPool基本操作

~~~java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

/**
 * 求[x1,x2]范围的和
 */
public class MyRecursiveTask extends RecursiveTask<Integer> {

    /**
     * 每个线程处理的任务的最大值
     */
    private final static int MAX_TASK = 10;

    private int start;
    private int end;

    public MyRecursiveTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    /**
     * 任务拆分逻辑
     *
     * @return 结果集
     */
    @Override
    protected Integer compute() {
        if ((end - start) < MAX_TASK) {
            int sum = 0;
            System.out.println(Thread.currentThread().getName() + ": execute " + start + " - " + end + " task");
            for (int i = start; i < end; i++) {
                sum += i;
            }
            return sum;
        } else {
            System.out.println(Thread.currentThread().getName() + ": split " + start + "-" + end + " task");
            int middle = (start + end) / 2;
            MyRecursiveTask left = new MyRecursiveTask(start, middle);
            MyRecursiveTask right = new MyRecursiveTask(middle, end);
            left.fork();
            right.fork();
            return left.join() + right.join();
        }
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        Integer result = forkJoinPool.invoke(new MyRecursiveTask(0, 1000000));
        System.out.println(result);
        forkJoinPool.shutdown();
    }
}
~~~
