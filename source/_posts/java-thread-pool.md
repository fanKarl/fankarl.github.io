---
title: Java 线程池
date: 2020-06-05 17:32:18
categories:
    - Java
tags:
    - 线程池
---

### 1. 线程池

线程池就是一个池子容器，用于存储我们创建的线程，方便管理和复用线程。

线程池适用场景是：1.任务执行时间短 2. 任务数量大，执行频繁 的情况

相对于直接创建线程，它具有以下优势：
1. 减少线程的创建于销毁带来的开销
    1. 减少时间成本开销，提高响应速度
    2. 减少系统资源开销，避免过度消耗资源，过早耗尽资源
2. 把线程统一放在池子里，方便管理和监控
3. 线程的创建和任务的提交是分开进行的，解耦合，提高了扩展性，可以自由定义一些功能，比如定时、延时、周期等

了解了使用线程池的场景和优势，下面我们看一下线程池的使用

<!--more-->

### 2. 线程池的使用

线程池相关的类在JDK提供的JUC中，不论是通过 Executors 还是自定义 ThreadPoolExecutor 创建，实际上都是围绕下图中的类展开的。

![Executor](http://coderfan.codeagles.com/java-executor.png)

- Executor 类提供了任务执行的 execute(runnable) 抽象方法
- ExecutorService 在 Executor 基础上新增了线程管理相关函数，同时也是通过 ThreadPoolExecutor 实现的对象实例
- ThreadPoolExecutor 是使用线程池的关键类，我们可以通过它配置所需参数，下面会进行详细讲解。


了解了类的继承关系，我们来看一下使用 Executor 所需要掌握的知识点，如思维导图所示：

![java 线程池的使用](http://coderfan.codeagles.com/java-thread-pool-basic.png)

#### 2.1 创建

线程池的创建有两种方式，Executors 创建和自定义 ThreadPoolExecutor,而 Executors所提供的默认实现实际也是对创建 ThreadPoolExecutor的封装，所以我们直接来看如何直接创建线程池

##### ThreadPoolExecutor

```jav
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {....}
```
ThreadPoolExecutor 构造函数中就是针对线程池的配置

- corePoolSize，核心线程池数，可以根据自己需要设置，一般根据CPU数进行合理配置，比如 CPU*2
- maximumPoolSize，线程池最大线程数
- keepAliveTime & unit 线程池工作线程 空闲之后最长等待时间，超时之后要被回收
- workQueue，线程池阻塞队列，当线程池核心线程满了，需要把等待线程放入阻塞队列中。阻塞队列是线程安全的，有三种阻塞队列
    - 直接切换队列，synchronousQueue，该队了每次只允许进入一个任务，该任务被取出以后允许下一个任务入队
    - 有界队列，比如 ArrayBlockingQueue，有界队列意思就是初始化的时候队列容量就已经确定，不可改变
    - 无界队列，比如LinkedBlockingQueue，关于链表阻塞队列网上有说它可以是有界队列也可以是无界队列。但是我个人倾向于认定它就是无界队列。首先无界队列的意思就是没有边界，计算机中到达上限MAX_VALUE。但是如果只是设定MAX_VALUE不能说是无界，更重要的是该队列可以手动扩容的，这是它不同于数组阻塞队列的地方，数组队列一旦设定一个值就不可变了。
- ThreadFactory，该方法是可以自行创建线程的工厂抽象类，方便我们在创建线程的同时做一些额外的操作。
- handler：RejectedExecutionHandler，任务拒绝策略，当线程池不再接收新任务的时候执行的策略，主要有以下几种策略选择，我们可以根据需要选择策略模式
    - AbortPolicy, 该策略直接抛出异常，属于默认策略
    - CallerRunsPolicy，在调用者所在线程执行任务
    - DiscardOldestPolicy，直接丢弃队列头部的最旧的任务
    - DiscardPolicy，直接丢弃该任务

##### Executors.newXXXThreadPool

了解了 ThreadPoolExecutor 的构造函数，我们可以根据 Executors 提供的newXXXThreadPool()系列方法来了解 ExecutorService 的创建

Executors 一共提供了 5 种线程池
![常用线程池](http://coderfan.codeagles.com/java-thread-pool-categories.png)

###### newCachedThreadPool()
这种线程池 corePoolSize =0，maximumPoolSize = MAX_VALUE， 也就是说它没有核心线程，全都是工作线程，线程空闲时间是 60s，也就是说如果没有空闲线程就创建新线程执行任务，如果有空闲线程，超过60s就进行回收。这种线程使用的阻塞队列就是第一种直接切换队列

###### newFixedThreadPool()

这种 线程池 corePoolSize 和 maximumPoolSize 都是固定的，可以在创建的时候指定，这也就意味着没有工作线程，新任务要么交给核心线程执行，要么在阻塞队列进行等待，或者直接被拒绝。没有工作线程自然空闲等待时间设置0即可，它使用的是无界队列 LinkedBlockingQueue

###### newSingleThreadPool()

newSingleThreadPool相当于newFixedThreadPool的极端，它只允许一个核心线程，没有工作线程，空闲等待时间为0，使用无界队列


###### newScheduledThreadPool()

newScheduledThreadPool 创建需要借助 ThreadPoolExecutor 的子类 ScheduledThreadPoolExecutor，它的核心线程数也是需要指定，工作线程数是MAX_VALUE，工作线程空闲等待时长10ms，另外它所用队列是 DelayedWorkQueue，这个队列我还没研究，不多介绍了。

#### 2.2 执行

执行线程池有一个唯一的方法，就是 Executor.execute(runnable),线程池的任务管理、线程管理、状态维护都从这里开始。在后面会详细介绍

#### 2.3 关闭

ExecutorService提供了两种关闭线程池的方法，shutdown() 和 shuatdownNow(),他们都能关闭线程池，但是是有区别的
- 调用shutdown()，阻塞队列不再接收新的任务，但是会继续执行正在进行和阻塞队列中缓存的任务
- 调用 shuatdownNow()，不仅不再接收新任务，也会立即终止所有正在执行的任务，队列中的任务也不再执行。

#### 2.4 监控

线程池还提供了一系列监控方法，都在思维导图中有说明，比较简单，这里不再赘述。

### 3. ThreadPoolExecutor

看完了线程池的使用，我们来看一下 ThreadPoolExecutor，线程池设计比较好的地方就是对任务和线程进行了解耦，因此 ThreadPoolExecutor也可以分为三部分来看，任务管理、线程管理、线程池状态维护,如图所示

![ThreadPoolExecutor](http://coderfan.codeagles.com/java-thread-pool-executor.png)

#### 3.1 状态维护

线程池通过 runstate（运行状态） 和 workerCount（线程数量）来记录线程池状态，但是为了方便管理，使用32位二进制数字进行记录，其中高三位用于记录runState，低29位用来记录workerCount

线程状态一共有五种：RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED

他们执行流程我从参考文献借一张图来展示

![runState](http://coderfan.codeagles.com/java-thread-pool-state.png)

- 任务的执行都是在 RUNNING 状态下执行的
- 当调用shutdown()，会进入SHUTDOWN 状态，当阻塞队列为空，工作线程为0，进入下一状态
- 调用 shutdownNow()，进入STOP状态，当工作线程为0 进入下一状态
- SHUTDOWN 和 STOP 结束后都会进入 TIDYING 状态，此时有效线程数为0
- 最后状态是TERMINATED，此时所有任务都已经结束，也可以通过调用terminated()方法进入该状态

#### 3.2 任务管理

##### 任务调度
任务调度 execute(runable)是线程池的主要入口

```java
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
       
        int c = ctl.get();
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

大致执行流程是：
1. 检查状态，判断是否是正在运行状态，如果是继续提交新任务,否则执行拒绝策略
2. 检查核心线程数
    1. workerCount < corePoolSize, 新建一个核心线程执行任务
    2. workerCount >= corePoolSize, 执行下一步
3. 检查队列
    1. 队列未满，加入阻塞队列，等待线程提取
    2. 队列已满，检查最大线程数，符合corePoolSize <= workerCount < maximumPoolSize，创建工作线程，否则执行拒绝策略

在代码中可以看出来，每次要新建线程都会执行addWorker方法，这里先记下来。

##### 任务缓冲

线程池的核心功能就是管理任务，可以对任务进行缓存。它需要借助BlockingQueue来实现。BlockingQueue相比普通 队列主要多了两个操作：
1. 队列为空的时候，线程获取任务需要等待，直到队列非空
2. 队列满的时候，线程需要等待队列可用再提交任务

##### 任务申请

每次提交新任务有两种方式执行：
1. 直接创建新线程执行，一般线程初始创建的时候采用该方法。创建线程借助 addWorker方法，我们稍后介绍。
2. 放入缓存队列，等待线程取出后执行。线程执行完成以后会继续从队列取任务，也就实现了线程复用。


##### 任务拒绝

任务拒绝是在线程池拒绝接受新任务的时候执行，一般是核心线程已满、阻塞队列已满、最大线程已满情况下触发。具体可使用策略前面已经说过，不再赘述

#### 3.3 线程管理-Worker

前面说到线程增加需要用到addWorker().这里我们看一下它的源码:
```java
    private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            /**
             * 1. 检查运行状态,只要不是 RUNNING 状态就直接 return
             * 2. 检查队列是否为空、firstTask是否为空
             **/
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

            //检查线程数量
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
            //下面是新建Worker，Worker中保存线程
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                //...省略代码...
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

addWorker中先是进行一系列检查，检查runState、检查线程数量、检查阻塞队列，之后才真正开始创建线程，创建线程就用到了线程管理的核心类WorkerWorker是线程池内的工作线程，实际是一个封装类，它内部维护了一个对应的Thread，方便线程池掌握线程的状态。

```java
 private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
        final Thread thread;
        Runnable firstTask;
        volatile long completedTasks;

        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }

        public void run() {
            runWorker(this);
        }
}
```

从 worker的部分代码可以发现，他实现了Runnable类，内部持有Thread的引用。
- 构造函数显示 Thread的引用是通过 getThreadFactory().newThread(this) 实现的ThreadFactory 可以用默认提供的也可以我们自定义实现。
- 关于firstTask，允许为空。如果创建worker的时候，firstTask不为空，则线程创建后可以直接执行它，也就对应核心线程创建时的情况。如果firstTask为空，则线程创建后去阻塞队列取任务执行，对应的就是非核心线程的创建。

然后我们来一下 run()方法的具体实现 runWorker(this)

```java
    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) {
                w.lock();
                
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
                    } //...异常处理...
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

代码逻辑很清晰：
1. while循环判断 firstTask不为空或者，getTask 不为空进入循环。
2. 循环中 先判断线程状态，判断是否需要interrupt
3. task.run() 执行任务
3. while循环结束 getTask为null，执行processWorkerExit(w, completedAbruptly)结束线程


以上关于线程池的任务管理、线程管理和运行状态就介绍完了，中间省略了很多细节，比如线程锁、运行状态的切换等等，这个以后有时间再补充

### 参考文献

1. [Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
2. [深入理解 Java 线程池：ThreadPoolExecutor](https://juejin.im/entry/58fada5d570c350058d3aaad)
3. [如何优雅的使用和理解线程池](https://crossoverjie.top/2018/07/29/java-senior/ThreadPool/)
