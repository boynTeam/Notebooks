CAS 的全称是 CompareAndSwap，是一条 CPU 并发原语。它的功能是判断内存某个位置的值是否为预期值，如果是则更新为新的值，这个过程是**原子的**。

CAS 的作用是比较当前工作内存中的值和主内存中的值，如果相同则执行操作，否则继续比较直到主内存和工作内存中的值一致为止。主内存值为V，工作内存中的预期值为A，要修改的更新值为B，当且仅当A和V相同，将V修改为B，否则什么都不做。

## CAS的底层原理

在Java中,CAS都是通过Unsafe来操作的,而Unsafe里面都是native方法

Unsafe类正如其名,是不安全的类,其不安全的地方在于,假如说我们调用了allocateMemory方法申请一段内存,这段内存并不受JVM管理,而是直接向操作系统申请的,所以我们在完成操作之后要对其进行释放.

在CAS操作中,我们可以调用Unsafe类中的比如 **getAndAddInt** 函数,它会实现CAS的汇编指令,依赖于硬件达到原子读写的操作

我们可以看一下原子类AtomicInteger中,是如何递增的

```java
public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        return v;
    }
```

在其中,o为this指针,offset为偏移量,delta为更改量,在类中,offset是这样获取的

```java
private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
```

即获取原子类的Class对象中,value域在Class对象中的偏移量

函数被这样调用

```java
U.getAndAddInt(this, VALUE, 1) + 1;
```

在Unsafe的方法中,我们可以看到,有一个do-while的循环,这个是用于自旋来保证改变值成功

在循环中,首先通过地址偏移量获取对象的值,然后通过CAS操作,尝试给这个值+1,当失败后,进入自旋来保证成功,也称CAS为自旋操作

## CAS的优缺点

优点:CAS是自旋来保证更改成功的,而不是阻塞线程,这样可以提高并发量,同时使原子更改更加灵活

缺点:

1. 如果CAS失败，会一直尝试。如果CAS长时间不成功，会给CPU带来很大的开销。
2. CAS 只能用来保证**单个**共享变量的原子操作，对于多个共享变量操作，CAS无法保证，需要使用锁。
3. 存在 ABA 问题。

## ABA问题

回顾刚刚CAS的操作,我们可以看到,它是从内存中取出值,然后更改的.但是这样也会带来问题,假设当前值是A,而有另外的线程将其改为B之后,再改回为A,随后本线程进行CAS操作,这样子在本线程看来这个值并没有被改变,但是事实上已经被改变过了.

这种类似于数据库中的幻读问题,我们可以使用增加版本号的方式来解决,在修改XX值成功的同时,也要修改其版本值,最常见的就是一个自增计数器.

