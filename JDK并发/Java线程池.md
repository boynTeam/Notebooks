

在Java编程中,多线程往往是不可避免的,然而我们如果只是简单的创建多个线程,然后执行完了之后对线程进行销毁,那么线程创建和销毁的开销无疑太大了,所以我们可以想一个办法,来让线程预先创建好,被统一管理,需要的时候直接取出执行任务.这个就是我们要说的线程池


# ThreadPoolExecutor

在线程池中,最核心的一个类是`ThreadPoolExecutor`,我们可以来看看这个类是怎么样来实现的.

我们先来看看其构造方法

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }

public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

可以看到,其实前三个构造器都是间接地调用第四个构造器,其参数意义如下:

- corePoolSize,核心池的大小,在创建了线程池并初始化后,线程池中没有任何线程,只有当有任务来才会去创建任务并执行线程.但是线程池也可以预创建线程,在没有任务来之前就创建好`corePoolSize`个线程.如果不预创建,那么就来一个任务创建一个线程,并放入线程池,达到`corePoolSize`这个数目后,就把到达的任务先放入缓存队列
- maximumPoolSize,表示在线程池中最多可以创建多少个线程.
- keepAliveTime,表示线程没有任务执行的时候多长时间会终止.
- unit:keepAliveTime的时间单位,可以通过TimeUnit类来表示时间
- wordQueue,用户指定的一个阻塞队列,用于存储等待执行的任务,一般来说,阻塞队列有`ArrayBlockingQueue,LinkedBlockingQueue,SynchronousQueue`
- threadFactory,线程工厂,用于创建线程,
- handler,用来表示当拒绝处理任务时候的策略.有以下四种:1.AbortPolicy 丢弃任务并抛出异常 2. DiscardPolicy 丢弃任务但不抛出异常 3. DiscardOldestPolicy 丢弃任务队列最前面的任务,然后重试执行任务 4. CallerRunsPolicy 由调用的线程来处理该任务.

ThreadPoolExecutor的继承关系如图所示

![](http://imageblog.boyn.top/201912072239_661.png)

其中,最顶层的Executor只有一个方法

```java
public interface Executor {

    void execute(Runnable command);
}
```

就是用来执行传送进去的任务的,明白了顶层接口的含义,我们就对下面的线程池的实现有了一个大概的方向,那就是无论如何变化,核心就是将传送进去的任务执行.

在ThreadPoolExecutor中,有几个非常重要的方法

- `execute()`这个就是我们上面说到的顶层接口中定义的方法,在ThreadPoolExecutor中进行了具体的实现,通过这个方法可以向线程池提交一个任务,让线程池来执行
- `submit()`与execute()的语义基本相同,但它可以返回任务执行的结果,本质上,是用Future来进行结果的接收.
- `shutdown()`关闭线程池
- `shutdownNow()`关闭线程池

# 线程池具体实现

## 线程池状态

在ThreadPoolExecutor中,用一个原子整形变量来表示运行状态,并围绕这个变量,定义了一系列的状态值与操作状态值的函数

![](http://imageblog.boyn.top/201912072252_112.png)

在初始的时候,线程池处于RUNNING的状态,如果调用了shutdown方法,则处于SHUTDOWN状态,此时不会接收新任务,但会等待所有任务执行完毕

```java
public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(SHUTDOWN);
            interruptIdleWorkers();
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }
```

如果调用shutdownNow方法,则处于STOP状态,此时不会接收新的任务,且尝试结束正在执行的任务

```java
public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(STOP);
            interruptWorkers();
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
    }
```

当所有工作线程已经被销毁,队列清空后,变为TERMINATED状态.

## 任务的执行

在线程池中,任务执行是了解其原理的重点.我们先来看看其重要的成员变量

```java
private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock();   //线程池的主要状态锁，对线程池状态（比如线程池大小
                                                              //、runState等）的改变都要使用这个锁
private final HashSet<Worker> workers = new HashSet<Worker>();  //用来存放工作集
 
private volatile long  keepAliveTime;    //线程存活时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
 
private volatile int   poolSize;       //线程池中当前的线程数
 
private volatile RejectedExecutionHandler handler; //任务拒绝策略
 
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
 
private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
 
private long completedTaskCount;   //用来记录已经执行完毕的任务个数
```

假如有一个工厂，工厂里面有10个工人，每个工人同时只能做一件任务。

　　因此只要当10个工人中有工人是空闲的，来了任务就分配给空闲的工人做；

　　当10个工人都有任务在做时，如果还来了任务，就把任务进行排队等待；

　　如果说新任务数目增长的速度远远大于工人做任务的速度，那么此时工厂主管可能会想补救措施，比如重新招4个临时工人进来；

　　然后就将任务也分配给这4个临时工人做；

　　如果说着14个工人做任务的速度还是不够，此时工厂主管可能就要考虑不再接收新的任务或者抛弃前面的一些任务了。

　　当这14个工人当中有人空闲时，而新任务增长的速度又比较缓慢，工厂主管可能就考虑辞掉4个临时工了，只保持原来的10个工人，毕竟请额外的工人是要花钱的。

在这个例子中,10就是corePoolSize,14就是maximumPoolSize.

我们来看看execute方法的具体实现

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

首先判断提交的任务是否为空

在这里,对于线程池状态的判断使用了一系列的位运算,我们尽可能地忽略位运算的复杂性,而以通俗的语言来解释.对线程池的状态判断,有3种方式决定一个任务的提交有什么行为

1. 如果正在运行的线程少于corePoolSize,那么就创建一个新线程,并将传入的任务指派给这个线程.
2. 如果addWorker这个方法执行成功了,那么就等于创建一个新线程成功,后面的函数不会再执行,但如果执行不成功,那么首先执行双重检查以防线程池停止或其他状态发生,如果线程池仍然处于运行状态,那么就将该任务放入任务缓存队列中,`(isRunning(c) && workQueue.offer(command))`用得巧妙的点在于`&&`的短路机制,使得代码更加简洁了.
3. 如果线程数目大于`maxPoolSize`,那么就执行拒绝策略

在这里,我们接触到了一个叫做Worker的类,在前面所举的例子中,我们描述了一个工厂中,工人在工作的情景,同样的在这个线程池中,我们也通过worker来进行任务的分配

```java
private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
```

Worker同样是一个线程,称之为工作线程,其中run方法的实现如下

```java
public void run() {
            runWorker(this);
        }
```

其中只有一个runworker方法,这个是什么呢,这个runworker方法可以说是线程池中最核心的实现了,我们来看一下

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
                    try {
                        task.run();
                        afterExecute(task, null);
                    } catch (Throwable ex) {
                        afterExecute(task, ex);
                        throw ex;
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

忽略其中的安全检查和异常的抛出,我们抽取出其中的逻辑:首先将创建worker时指派的任务作为第一个任务开始执行,执行完毕后,从任务缓冲队列中取出任务持续执行.

 这里有一个非常巧妙的设计方式，假如我们来设计线程池，可能会有一个任务分派线程，当发现有线程空闲时，就从任务缓存队列中取一个任务交给空闲线程执行。但是在这里，并没有采用这样的方式，因为这样会要额外地对任务分派线程进行管理，无形地会增加难度和复杂度，这里直接让执行完任务的线程去任务缓存队列里面取任务来执行。 

## 线程池中的线程初始化

默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线程。

　　在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到：

- prestartCoreThread()：初始化一个核心线程；
- prestartAllCoreThreads()：初始化所有核心线程

　　下面是这2个方法的实现：

```java
public boolean prestartCoreThread() {
        return workerCountOf(ctl.get()) < corePoolSize &&
            addWorker(null, true);
    }

public int prestartAllCoreThreads() {
        int n = 0;
        while (addWorker(null, true))
            ++n;
        return n;
    }
```

在这个预创建线程中,我们把传入任务设为null,让worker线程阻塞在`while (task != null || (task = getTask()) != null) `

## 任务缓存队列及排队策略

　　在前面我们多次提到了任务缓存队列，即workQueue，它用来存放等待执行的任务。

　　workQueue的类型为BlockingQueue<Runnable>，通常可以取下面三种类型：

　　1）ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

　　2）LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；

　　3）synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。

## 任务拒绝策略

　　当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略：

```
`ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。``ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。``ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）``ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务`
```

通常来说,如果我们选择BlockingQueue,并且没有手动指定容量上限,那么是不会采取拒绝策略的,因为此时多余的任务都会放入WorkingQueue中,但是有时候我们需要控制总的任务数量,可以手动指定Queue的容量,从而达到执行拒绝策略的目的

## 线程池的关闭

　　ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()，其中：

- shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

## 线程池容量的动态调整

　　ThreadPoolExecutor提供了动态调整线程池容量大小的方法：setCorePoolSize()和setMaximumPoolSize()，

- setCorePoolSize：设置核心池大小
- setMaximumPoolSize：设置线程池最大能创建的线程数目大小

　　当上述参数从小变大时，ThreadPoolExecutor进行线程赋值，还可能立即创建新的线程来执行任务。