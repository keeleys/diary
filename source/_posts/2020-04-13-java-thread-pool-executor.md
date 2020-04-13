---
title: 通过源码一点点了解java线程池
date: 2020-04-13 11:41:13
tags: java
---

## 前言

也许网上很多线程池的知识，也会给你一些各种定论，线程池的优势是xxx哇，线程池参数xxx哇等等定论。
但知识要真正掌握在自己手上，必须经过自己的剖析，源码的理解，才能明白为什么是这样的。



## 线程池对象`ThreadPoolExecutor`

不管程序里是通过`Executors.newXXXXX`的方法来构建线程池，还是主动新建，都是用的`ThreadPoolExecutor`的类构造方法进行实例化一个线程池的。这个类也是我们主线程调用线程池的入口。

{% asset_img total2.png 线程流程 %}

### ThreadPoolExecutor的参数

可以让我们在构造的时候传入的全部完成的参数有七个,我先介绍下他们，这里不用记这些，后面会有用具体工作流程来讲解这些是怎么用到我们的程序中的。

* corePoolSize: 
    ```
    即使线程闲置，线程池也会一直会持有的线程数量。当然这里官方说了，这里的前置条件是没有设置`unlessallowCoreThreadTimeOut`参数，这个参数不在构造器里面，需要手动set，默认是false的,后面会讲。（the number of threads to keep in the pool, even if they are idle, `unlessallowCoreThreadTimeOut` is set）
    ```

* maximumPoolSize 
    ```
    线程池允许同时运行的最大线程数
    （the maximum number of threads to allow in the pool）
    ```

* keepAliveTime 
    ```
    有多余的空闲的线程大于corePollSize的时候，多余线程的最大保留时间，这个时间会等超过部分的线程先把当前线程里的任务先执行完毕之后再计算。
    (when the number of threads is greater than the core, this is the maximum time that excess idle threads  will wait for new tasks before terminating.)
    ```
* unit
    ```
    keepAliveTime的单位，枚举参数。最后程序会根据时间和单位这两个参数，转化成纳秒。
    (the time unit for the {@code keepAliveTime} argument)
    ```

* workQueue 
    ```
    仅用于存放execute方法传入的Runnable对象的队列，用来在线程池执行这些任务之前，阻塞在队列中等待运行。
    （the queue to use for holding tasks before they are 
    executed.  This queue will hold only the {@code Runnable}
    tasks submitted by the {@code execute} method.）
    ```
* threadFactory
    ```
    线程池里面新建线程的时候用到的线程池工厂类，不传的话，默认是用Executors.defaultThreadFactory)
    （the factory to use when the executor creates a new thread)
    ```
* handler
    ```
    达到了最大线程值并且队列的容量也达到了上限的时候，再运行新任务会调用的异常处理类。
    the handler to use when execution is blocked
    because the thread bounds and queue capacities are reached
    ```

{% asset_img threadPoolExecutorParams.png 线程参数 %}

### 主要属性

```java
/**
这个字段存储了线程池的状态和工作中的线程数量。
高3位存放状态，剩下29位存线程数量
**/
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

/** 五个状态 **/
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

/** 操作线程池对象的大部分公共方法的时候 会用这个加锁 **/
private final ReentrantLock mainLock = new ReentrantLock();

/** 存放所以的Worker对象 */
private final HashSet<Worker> workers = new HashSet<Worker>();

/** 给awaitTermination方法做支持的阻塞条件锁 */
private final Condition termination = mainLock.newCondition();
```


### 线程池状态流转

```java

/** The runState provides the main lifecycle control, taking on values:

*   ctl属性初始化的时候就是running了。
*   RUNNING:  Accept new tasks and process queued tasks

    调用shutdown()方法的时候改成SHUTDOWN,不接受新任务了，但是队列如果还有任务会继续执行。
*   SHUTDOWN: Don't accept new tasks, but process queued tasks

    调用 shutdownNow() 的时候，改STOP,并且队列里面的任务也不读了，发送中断指令给正在工作的线程。
*   STOP:     Don't accept new tasks, don't process queued tasks,
*             and interrupt in-progress tasks

    终止前的过度状态，为了终止前调用钩子方法。
*   TIDYING:  All tasks have terminated, workerCount is zero,
*             the thread transitioning to state TIDYING
*             will run the terminated() hook method

    线程终止
*   TERMINATED: terminated() has completed
**/
```

## 线程池方法
 
### execute方法执行的时候是怎么到子线程的呢，task的添加顺序是什么呢

挑关键的说一下,这里说下task就是用户传递的Runnable对象，worker是线程里面实际工作的线程对象，它的firstTask属性保存了task对象。

调用execute(task)的时候，如果运行数量小于核心线程数量，会先把核心线程数(corePoolSize)建满，然后再把多余的task加到队列。

线程池预热起来之后，除了前面几次execute的时候worker运行的是传入的参数的task，后面执行的task都会放到阻塞队列里面去执行。
拿到task之后，worker执行线程的start方法，等待系统调用。

调用addWorker里面会去构建一个新的Worker对象，可以看到worker持有thread对象，runnable参数是自己本身

```java
Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this);
}
```

### 线程池运行task是否是线程安全的呢

worker对象是实现了AbstractQueuedSynchronizer，又实现了Runnable接口，相当于既实现了同步器，又是可运行的线程。

worker线程获取到task之后，在调用task.run()的方法的业务中会加锁。
这个锁我理解的是是为了判断worker是否正在执行任务，因为执行Runnable方法的过程中worker线程是持有锁的，然后shutdown方法会试图tryLock，如果拿不到worker的锁会放弃停止这个线程。

但是对于同task多个worker运行来说，其实task不是线程安全的，因为如果有多个worker同时运行同样的task对象，但是每个worker线程都是自己持有自己的锁，不同worker不会等待。

### worker是怎么一直保持线程数量的呢，超时时间是在哪里用的呢

我们来看runWorker方法的部分源码解答
```java
 try {
    // getTask代码取任务这个是关键。这里不停的从getTask获取task，然后执行，除非getTask返回null才会跳出循环
    while (task != null || (task = getTask()) != null) {
        // worker对象锁上
        w.lock();
        // ctl是线程池状态，wt是当前运行的线程，这里是判断如果线程池停止了，让当前线程中断。
        if ((runStateAtLeast(ctl.get(), STOP) ||
                (Thread.interrupted() &&
                runStateAtLeast(ctl.get(), STOP))) &&
            !wt.isInterrupted())
            wt.interrupt();
        try {
            beforeExecute(wt, task);
            Throwable thrown = null;
            try {
                // 手动运行用户传递的task
                task.run();
            } catch (RuntimeException x) {
                thrown = x; throw x;
            } catch (Error x) {
                thrown = x; throw x;
            } catch (Throwable x) {
                thrown = x; throw new Error(x);
            } finally {
                afterExecute(task, thrown);
            }
        } finally {
            task = null;
            // 不管运行成功还是失败，都只运行一次，外面的循环也不会停止。
            w.completedTasks++;
            w.unlock();
        }
    }
    // 能执行到这里 说明worker被中断了。
    completedAbruptly = false;
} finally {
    // 清理当前退出的worker线程，善后。
    processWorkerExit(w, completedAbruptly);
}
```

getTask()源码
```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 判断线程池状态和工作线程数量
        // 大体意思是线程池是shutdown，工作队列是空的时候，不用继续获取队列了(这时候要停线程了)。
        // 线程池是shutdown后面的状态(stop,TIDYING,TERMINATED)，也不用继续获取任务了。
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        // 线程池工作线程数量
        int wc = workerCountOf(c);

        // 当前worker线程是否需要判断超时。
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // 如果 线程需要判断时间，并且也超时了
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
                // 线程池工作线程数 -1，线程的删除会在外面的processWorkerExit方法执行
            if (compareAndDecrementWorkerCount(c))
                // ======返回null=====
                return null;
            continue;
        }

        try {
            // 这里就是core线程数一直持有的原因了，因为线程会一直阻塞去获取workQueue队列里面的task.有了就直接返回给上一个方法运行，没有就一直等待，不可能返回null的。
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            // 运行到这里，说明确实返回null了，只有一种情况返回null，poll方法超时。
            // take会一直阻塞获取的，poll到时间队列还没有数据，会返回null。
            timedOut = true;
        } catch (InterruptedException retry) {
            // 运行到这里 说明worker线程被中断了，而不是worker超时了。
            timedOut = false;
        }
    }
}
```

### 怎么让线程池里面的线程全部都执行完毕之后，主线程再继续执行.
```
这里可以搭配线程池的 shutdown() + awaitTermination()来实现
因为shutdown会一直等到所有任务执行完毕之后才把线程设置为TERMINATED，
awaitTermination持有一个条件锁(termination)会一直自旋的awaitNanos锁。如果线程设置为TERMINATED之后，
termination锁会被释放，然后awaitTermination就会返回结果给主线程了。
这时候可以得知，所以线程肯定执行完了。
```