# 什么是ReentrantLock

ReentrantLock,也称可重入锁,是一种可重入的互斥锁,与内置的监视器锁synchronized功能类似,不过具有更多功能.

可重入锁将会记录最后一次成功锁定但还没有解锁的线程.它允许同一个线程多次进入并加锁,同时它还支持公平性与非公平性的加锁

# 公平锁与非公平锁

对于锁来说,会出现多个线程尝试获取锁,但是只能有一个线程成功获取,其他线程则被阻塞的情况.当锁被释放后,如何决定下一个获取锁的线程呢?

对于公平锁来说,其遵循先来后到的规则,预先进入阻塞等待队列的线程,也会优先出队获取资源.反之,如果获取锁是非顺序的,则称为非公平锁.一般来说,非公平锁的效率要比公平锁高,但是非公平锁会发生某个线程的饥饿问题.

# ReentrantLock的可重入性

可重入的意思是当同一个线程再次加锁时,能够获取锁,而不被阻塞.为了实现可重入的特性,需要解决两个问题

- 锁需要识别尝试获取锁的线程是否为拥有锁的线程
- 锁的最终释放.如何保证在线程可能重入多次的情况下,确保该线程每次重入都释放了

我们以acquire的过程来看看是如何保证的,由于可重入锁的默认实现是非公平锁,所以我们以其为例,来看看它的具体实现

```java
final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

在可重入锁中,将同步状态作为一个计数器,记录重入线程的次数,当同步状态为0的时候,就表明未上锁,通过CAS将状态转换,然后设置互斥线程为当前的线程.而如果同步状态不为0的时候,就表明当前锁已经被获取,而如果尝试重新加锁的是当前获得锁的线程,那么就会允许其进入,并将同步状态的计数器+1.

那么同样的,释放锁的时候也会将同步状态-1

```java
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

而对于公平锁的实现来说,它在获取锁的时候,先要判断等待队列是否为空,如果不为空,则要等待队列前面的结点先出队获取锁,再让公平锁的线程获取锁

```java
protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

