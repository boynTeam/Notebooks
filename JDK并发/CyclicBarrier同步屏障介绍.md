# 什么是同步屏障

CyclicBarrier,又称为同步屏障,是一种高级的并发工具,字面的意思是可循环使用的屏障.具体做的事就是让一组线程到达了这个屏障被阻塞,等待数量足够之后,令所有线程继续运行.

# 使用方法

我们先来看一个简单的例子

```java
public static void main(String[] args) throws BrokenBarrierException, InterruptedException {
        CyclicBarrier barrier = new CyclicBarrier(2);
        new Thread(() -> {
            try {
                barrier.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("Sub Thread Output");
        }).start();
        barrier.await();
        System.out.println("Main Thread Output");
    }
```

在main函数中,我们首先声明了一个CyclicBarrier,并指定其拦截数为2,然后新建了一个线程,调用了屏障的await方法,同时主线程也调用了await方法

在程序运行时,会输出

```
Sub Thread Output
Main Thread Output
```

这两行的顺序是不一定的,主要是看谁先运行到了await函数,这个与该屏障实现的原理有关,在待会会说到

# 应用场景

我们可以将我们的线程理解为水流,而将这个屏障理解为大坝,只有当水达到一定的高度或者其他参数时,大坝才会开闸.同样地我们在进行多线程的计算时,如果每个结果之间不互相依赖,并且他们需要最终汇总成一个总结果,可以试着使用这个同步屏障.

# 原理

其核心的方法就是await方法,我们来看一下其源码(await方法调用了wait方法)

```java
private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            final Generation g = generation;

            if (g.broken)
                throw new BrokenBarrierException();

            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }

            int index = --count;
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();
                    ranAction = true;
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

                if (g != generation)
                    return index;

                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();
        }
    }
```

在内部定义了一个锁lock对象,当一个线程调用线程的await方法,先令计数器-1,如果为0了,那么就唤醒所有被阻塞的线程,运行他们.在dowait方法中,trip是lock对象的condition成员,调用trip.await会将线程直接放入lock的等待队列中并释放锁.如果被唤醒了之后,这些线程会依次地从等待队列中出列,并尝试获取锁,然后释放锁,从await方法中退出,返回主方法的逻辑中去,从而实现了屏障的作用