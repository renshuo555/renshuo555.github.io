---
layout: post
title: Java线程池学习总结
date: 2021-08-29 
categories: Java
tags: 线程池 并发
author: renshuo
mathjax: true
---

* content
{:toc}

线程池 (Thread Pool) 是一种基于池化的思想管理线程的工具。本文是基于JDK 1.8源码的Java线程池学习笔记

<!--more-->

## 为什么要使用线程池

- **降低资源消耗** 通过利用`已经创建的线程`，降低`创建和销毁线程`造成的消耗，提高系统资源利用率。
- **提高相应速度** 任务需要执行时，减少创建线程的时间消耗。
- **提高线程的可管理性** 线程是稀缺资源，如果无限制创建，不仅会消耗系统资源，还会因为线程的不合理分布导致资源调度失衡，降低系统的稳定性。使用线程池可以进行统一的`分配、调优和监控`。

## 如何使用线程池

### 创建ThreadPoolExecutor对象

在Java中通过创建 `ThreadPoolExecutor` 对象使用线程池。`Executors` 中也提供了几种创建线程池的静态方法，针对特定场景预置了构造参数。

``` java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

#### int corePoolSize

要保留在线程池中的线程数量，如果没有设置 `allowCoreThreadTimeOut` ，即使线程是空闲的，也不会被销毁。

如果提交一个任务，并且当前池中的线程数量小于 `corePoolSize`，即使池中有空闲线程可用，也会创建新的线程。

#### int maximumPoolSize

线程池中允许的最大线程数。

当阻塞队列已满，并且线程数小于 `maximumPoolSize`，会创建新的线程执行任务。

#### long keepAliveTime

当线程数量超过 `corePoolSize`，多余的空闲线程在终止前，等待新任务的最长时间。

#### TimeUnit unit

`keepAliveTime` 参数的时间单位，可选单位有DAYS、HOURS、MICROSECONDS、MILLISECONDS、MINUTES、NANOSECONDS、SECONDS。

#### BlockingQueue\<Runnable\> workQueue

当前线程数大于`corePoolSize` 并且小于 `maximumPoolSize`时，任务执行前会被保存在该阻塞队列中。

| 名称                  | 容量     | 描述                                                         |
| --------------------- | -------- | ------------------------------------------------------------ |
| ArrayBlockingQueue    | 有界     | 基于数组实现，按照先入先出(FIFO)原则对元素进行排序。支持公平锁和非公平锁。 |
| LinkedBlockingQueue   | 可选有界 | 由链表结构组成，按照先入先出(FIFO)原则对元素进行排序，默认长度为`Integer.MAX_VALUE`。 |
| PriorityBlockingQueue | 无界     | 支持线程优先级排序，默认自然序，不能保证同优先级元素的顺序。 |
| DelayQueue            | 无界     | 在 PriorityBlockingQueue 的基础上实现延迟获取，只有延时期满后的元素才能出队列。 |
| SynchronousQueue      | 0        | 不存储元素，每一个`put`操作必须等待另一个线程的`take`操作，否则添加操作一直处于阻塞状态。<br/>支持公平锁和非公平锁。 |
| LinkedTransferQueue   | 无界     | 由链表结构组成，相对于其它队列，多了 `transfer` 和 `tryTransfer` 方法 |
| LinkedBlockingDeque   | 可选有界 | 由链表结构组成的双向阻塞队列，默认长度为`Integer.MAX_VALUE`。<br/>队列头部和尾部都可以添加和移除元素，多线程并发时，可将锁竞争最多降到一半。 |

#### ThreadFactory threadFactory

创建线程时使用的工厂类。

#### RejectedExecutionHandler handler

当线程数量大于`maximumPoolSize`，或者线程池不在运行状态时，对提交的任务执行的拒绝策略。

JDK中预置的四种：

- **ThreadPoolExecutor.CallerRunsPolicy** 在调用者的线程中执行任务，如果调用者线程已经关闭，则直接丢弃任务。
- **ThreadPoolExecutor.AbortPolicy** 直接抛出`RejectedExecutionException`异常，**默认策略**。
- **ThreadPoolExecutor.DiscardPolicy** 直接丢弃任务。
- **ThreadPoolExecutor.DiscardOldestPolicy** 丢弃线程池将要执行的下一个任务，并执行当前任务。

### 任务的提交

使用`void execute(Runnable command)` 方法提交任务。

其余的带有返回值的 `submit()` 方法以及批量提交任务的方法，也是通过 `FutureTask` 完成的对 `execute()` 方法的封装。

### 线程池的关闭

可以调用以下两种方法关闭线程池：

- `void shutdown()` 已提交的任务会继续执行，停止接受新的任务。
- `List<Runnable> shutdownNow()` 尝试停止所有正在执行的任务，停止对等待队列中任务的分配执行，并返回等待执行的任务列表。<br>不能保证正在执行的线程一定会停止。例如，典型的实现将通过 `Thread.interrupt()` 取消，因此任何未能响应中断的任务可能永远不会终止。

当调用上述两种方法关闭线程池后，都会使 `boolean isShutdown()` 返回为 `true`。当线程池中所有的任务都关闭后，`boolean isTerminated()` 才会返回 `true`.

## 线程池的原理解析

先看一下ThreadPoolExecutor的类图：

![ThreadPoolExecutor-uml](thread-pool/ThreadPoolExecutor-uml.png)

- `Executor` 将任务提交和任务执行解耦。用户只需要提交实现了`Runnable`接口的任务运行逻辑，由执行器(`Executor`)负责线程的调配和任务的执行。
- `ExecutorService` 增加了一些能力
  1. 执行一个或者一批带有返回值 `Future` 的异步任务的能力。
  2. 管理线程池的关闭能力。
- `AbstractExecutorService` 抽象类，将执行任务的流程串联起来，其中带返回值 `Future` 的 `submit` 方法是通过 `FutureTask` 完成的对 `execute` 方法的封装，保证下层实现只需要关注一个执行任务方法。

### 生命周期管理

线程池的生命周期是内部维护的，并不对外暴漏。使用一个变量同时维护了运行状态 `runState` 和线程数量 `workerCount` 两个值。

``` java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;        // 高3位保存runState, 低29位保存workerCount
private static final int COUNT_MASK = (1 << COUNT_BITS) - 1;   // 0001 1111 1111 1111 1111 1111 1111 1111 用于位运算操作

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~COUNT_MASK; }  // 获取当前运行状态
private static int workerCountOf(int c)  { return c & COUNT_MASK; }   // 获取当前线程数量
private static int ctlOf(int rs, int wc) { return rs | wc; }          // 通过运行状态和线程数量生成 ctl
```

| 运行状态 runState | 状态描述                                                     |
| :---------------- | :----------------------------------------------------------- |
| RUNNING           | 能接受新提交的任务，并且也能处理阻塞队列中的任务。           |
| SHUTDOWN          | 关闭状态，不再接受新提交的任务，**但可以继续处理阻塞队列中已保存的任务**。 |
| STOP              | 不能接受新任务，也不处理队列中的任务，会中断正在处理任务的线程。 |
| TIDYING           | 当所有任务已经终止，有效线程数量 workerCount 为0，会变为TIDYING状态。 |
| TERMINATED        | 在`terminated()`方法执行后，线程池彻底终止，会变为TERMINATED状态。 |

![lifecycle](thread-pool/lifecycle.png)

### 任务的管理

#### 提交任务

当使用 `execute()` 提交一个任务到线程池中时，会有一下三种情况：

- 直接申请线程执行该任务。
- 缓冲到阻塞队列中等待线程执行。
- 拒绝该任务。

> **接下来，看一下 JDK 1.8 源码中是怎么处理的**

1. 如果 `workerCount < corePoolSize` (当前线程数小于核心线程数)，执行`addWorker()`创建并启动一个线程来执行新提交的任务。
2. 如果 `workerCount >= corePoolSize` (当前线程数大于等于核心线程数)，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
3. 如果`workerCount >= corePoolSize && workerCount < maximumPoolSize` (当前线程数大于等于核心线程数但是小于设置的最大线程数)，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
4. 如果`workerCount >= maximumPoolSize` (当前线程数大于等于设置的最大线程数)，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

**所以，大体上的流程是：**

- 先创建核心线程执行任务；
- 然后放到阻塞队列中；
- 再然后允许创建临时的一些线程执行任务，但是不能超过设置的最大线程数；
- 最后，按照拒绝策略处理任务。

> `void execute(Runnable command)` 源码阅读

``` java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    int c = ctl.get();      // 获取ctl，高3位保存runState, 低29位保存workerCount
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

**先来看主要逻辑的第一个`if`分支**

``` java
if (workerCountOf(c) < corePoolSize) {
    if (addWorker(command, true))
        return;
    c = ctl.get();
}
```

- 当前存在的线程数小于核心线程数时，会执行`addWorker()`方法。
- `boolean addWorker(Runnable firstTask, boolean core)`主要是创建一个用于执行任务的线程，后文有详细讲解。如果`addWorker`返回`true`代表任务提交成功，执行`return`。
- 如果`addWorker()`返回`false`，可能有以下两种情况，会重新计算`ctl`的值
  - 多个线程同时提交了任务，实际上已不满足`workerCount < corePoolSize`
  - 另外的线程修改了线程池的状态，导致线程池已经不是`RUNNING`状态

**继续看第二个`if`分支**

``` java
if (isRunning(c) && workQueue.offer(command)) {    // 注释1
    int recheck = ctl.get();
    if (! isRunning(recheck) && remove(command))   // 注释2
        reject(command);
    else if (workerCountOf(recheck) == 0)          // 注释3
        addWorker(null, false);
}
else if (!addWorker(command, false))               // 注释4
    reject(command);
```

强调一下，当执行到这个分支时，处于以下两种情况之一：

- `workerCount >= corePoolSize`
- `addWorker()`返回`false`，线程池不是`RUNNING`状态

那么这个分支主要的执行逻辑如下，对应了代码中的注释序号：

1. 判断当前线程状态

   - 如果是`RUNNING`状态，则提交任务到阻塞队列中
   - 如果当前不是`RUNNING`状态，或者**提交到阻塞队列失败**，则执行`注释4`位置代码

2. 当任务提交到阻塞队列成功后，再次校验当前线程池状态，这是为了避免其它线程修改了线程池状态，比如调用了`shutdown()`或者`shutdownNow()`，那么就需要将刚提交的任务撤回

   - `remove()`成功，执行拒绝策略
   - `remove()`失败，说明已经被消费掉，执行`注释3`位置代码

3. 执行 `else if (workerCountOf(recheck) == 0)` 判定成立，说明当前存在的线程数为0，执行`addWorker()`方法创建一个新的线程

   - 如果当前是`RUNNING`状态，有任务提交到了阻塞队列，但是没有线程干活了
   - 如果是`remove()`失败，说明已经被消费掉，但是也有可能没有线程干活了，进行兜底操作

   至于为什么线程数会变为0，是因为如果设置了`allowCoreThreadTimeOut = true`，会允许回收核心线程。

4. 如果当前状态不是`RUNNING`状态，或者**提交到阻塞队列失败**，则执行 `else if (!addWorker(command, false))`

   - 如果当前不是`RUNNING`状态，尝试`addWorker()`创建一个线程，如果失败就执行拒绝策略
   - 如果是提交到阻塞队列失败的情况，代表阻塞队列满了。此时`addWorker()`失败代表存在的线程数达到了`maximumPoolSize`，执行拒绝策略

#### 执行任务

从上文的[提交任务](####提交任务)部分可知，任务执行有两种可能：

- 第一种是直接由新创建的线程执行提交的任务。
- 第二种是先提交任务到队列中，线程从队列中获取并执行任务。

第一种将在后文线程创建部分进行讲解，本节内容主要讨论线程如何从具备缓存功能的队列中不断获取并执行任务的。

> `private Runnable getTask()` 源码阅读

``` java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {    // 注释1
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;   // 注释2

        if ((wc > maximumPoolSize || (timed && timedOut))              // 注释3
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            Runnable r = timed ?                                       // 注释4
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
            workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

先阅读一下API文档对该方法的描述，大体内容如下：

> 当前的`worker`(可先理解成线程)根据当前的线程池设置，阻塞地或者定时地从队列中获取任务。当发生**以下4种情况之一**时，当前`worker`必须退出(可理解为线程销毁)，返回`null`。
>
> 1. 当前线程数达到`maximumPoolSize`限制
> 2. 线程池已经停止
> 3. 线程池关闭并且缓冲队列为空
> 4. 当前缓冲队列是非空的，并且当前线程不是池中的最后一个线程，那么在当前`worker`从缓冲队列**超时等待**获取任务`-->`超时后，`getTask()`会返回`null`

在正常的流程中，会从缓冲队列中获取一个任务并返回。主要需要注意的是前文阐述的返回null的4种情况。

- 情况1，对应代码中`注释3`位置，`wc > maximumPoolSize`成立，那么`wc > 1`也会成立，通过`CAS`操作减少线程数量，并`return null`

- 情况2、情况3，对应代码中`注释1`位置

  - `rs >= SHUTDOWN && rs >= STOP` 代表线程池处于`TIDYING`或者`TERMINATED`状态，线程池已停止
  - `rs >= SHUTDOWN && workQueue.isEmpty()` 代表线程池已经关闭，并且队列为空，当前的`worker`失业了

- 情况4，稍微有一点复杂

  - 首先看代码中`注释2`位置，`boolean timed = allowCoreThreadTimeOut || wc > corePoolSize`

    (1) `allowCoreThreadTimeOut = true`代表允许回收核心线程

    (2) `wc > corePoolSize` 代表当前线程不是核心线程，只是一个临时工

    **综合来说，`timed = true`代表当前线程是允许回收的。**

  - 继续来看代码`注释3`位置`if`条件中的`(wc > 1 || workQueue.isEmpty())`，要么当前池中线程不只一个，要么缓冲队列中没有待执行任务了

  - 还是看`注释3`位置`if`条件，去除情况1的代码`wc > maximumPoolSize`，并且认为`timed = true`，`(wc > 1 || workQueue.isEmpty()) = true`那么判断条件只剩下了`if(timedOut) {...}`

  - `timedOut = true`只有在`timed = true`并且`poll`方法等待超时成立。

  综合上述，情况4成立时，会通过`CAS`操作减少线程数量，并`return null`，代码中的`continue`只是为了多试几次。

### 内部的线程的管理

线程池为了掌握线程的状态并维护线程的生命周期，设计了线程池的工作线程`Worker`(**内部类**)，以下是部分代码：

``` java
public class ThreadPoolExecutor extends AbstractExecutorService {

    private final ReentrantLock mainLock = new ReentrantLock();

    // 使用HashSet记录所有的Worker，使用mainLock管理并发
    private final HashSet<Worker> workers = new HashSet<Worker>();

    private final class Worker extends AbstractQueuedSynchronizer implements Runnable
    {
        /** Thread this worker is running in.  Null if factory fails. */
        // Worker 持有的线程，线程工程创建失败时为null
        final Thread thread;  
        /** Initial task to run.  Possibly null. */
        // 初始化任务，可以为null
        Runnable firstTask;

        /**
         * Creates with given first task and thread from ThreadFactory.
         * @param firstTask the first task (null if none)
         */
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
        // 直接调用外部类ThreadPoolExecutor的 final void runWorker(Worker w) 方法
        public void run() {
            runWorker(this);
        }
    }
}
```

#### 线程的创建

在上文的[提交任务](####提交任务)部分中，提到过`boolean addWorker(Runnable firstTask, boolean core)`，这个方法是用来创建并执行一个线程的，`firstTask`代表`Worker`的初始化任务，`core`代表是否是核心线程。

> `boolean addWorker(Runnable firstTask, boolean core)` 源码阅读

``` java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

#### 线程的运行

> `final void runWorker(Worker w)` 源码阅读

``` java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
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
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

#### 线程的回收

> `void processWorkerExit(Worker w, boolean completedAbruptly)` 代码阅读

``` java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        completedTaskCount += w.completedTasks;
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }

    tryTerminate();

    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {
        if (!completedAbruptly) {
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        addWorker(null, false);
    }
}
```

## 进一步学习线程池

关于一些框架中是如何使用线程池的，可以学习以点击以下两个链接

[面试官：线程池如何按照core、max、queue的执行循序去执行？（内附详细解析）](https://www.cnblogs.com/wang-meng/p/13128890.html)

[Spring 中的线程池](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html)

线程池中还预留了一些钩子函数作为扩展点，以及一些动态调整参数的方法。还有一些`getter`方法，可以读取到当前线程池的运行状态以及参数。

建议阅读源码以及官方文档进一步学习。

## 参考文章:

> [Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
>
> [【万字图文-原创】 | 学会Java中的线程池，这一篇也许就够了！](https://www.cnblogs.com/wang-meng/p/12945703.html)
>
> [面试官：线程池如何按照core、max、queue的执行循序去执行？（内附详细解析）](https://www.cnblogs.com/wang-meng/p/13128890.html)
