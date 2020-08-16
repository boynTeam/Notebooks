# Lock接口

Lock是JUC中几乎所有并发类都会实现的接口,它定义了一组用于控制多个线程访问的资源的方式.它提供了与synchronized关键字类似的功能,但是比它有更多的功能.

它不需要以方法级或者大括号级进行锁定,使得跨方法锁定变得简单,同时降低了锁的粒度. 并且增加了锁操作的灵活性.

其方法如下

![](http://imageblog.boyn.top/202001311503_12.png)

其中,lock和unlock是组合方法用于加锁与释放.

`tryLock()`和`tryLock(long,TimeUnit)`可以非阻塞地获得锁和超时获得锁

而`newCondition()`方法可以用于替代wait/notiry机制

## 使用Lock与Condition实现生产者消费者

我们知道,可以使用`synchronized`和`wait/notify`方法实现生产者和消费者模型,同样地,我们也可以使用这两个接口来实现同样的功能.

在这里,我们使用[可重入锁](./ReentrantLock.md)作为锁的实现

```java
/**
 * 使用可重入锁实现简单地生产者与消费者模型
 * @author Boyn
 * @date 2020/1/31
 */
public class ProducerConsumer {
    private static final Integer maxValue = 10;
    private Integer nowValue = 0;
    //默认是公平锁
    private Lock lock = new ReentrantLock();
    //未满
    private Condition notFull = lock.newCondition();
    //未空
    private Condition notEmpty = lock.newCondition();
    public void put(){
        lock.lock();
        try{
            while(nowValue.equals(maxValue)){
                System.out.println("It is full");
                notFull.await();
            }
            nowValue++;
            System.out.println("Put -- value:"+nowValue);
            notEmpty.signalAll();
        }catch (InterruptedException e){

        }
        finally {
            lock.unlock();
        }
    }

    public void take(){
        lock.lock();
        try{
            while (nowValue.equals(0)){
                System.out.println("It is empty");
                notEmpty.await();
            }
            nowValue--;
            System.out.println("Take -- value:"+nowValue);
            notFull.signalAll();
        }catch (InterruptedException e){

        }
        finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException{
        final ProducerConsumer ps = new ProducerConsumer();
        Runnable put = ()->{
            while (true){
                ps.put();
            }
        };
        Runnable take = ()->{
            while (true){
                ps.take();
            }
        };
        for (int i = 0; i < 5; i++) {
            new Thread(put).start();
        }
        for (int i = 0; i < 6; i++) {
            new Thread(take).start();
        }
    }
}
```

可以看到,我们使用`lock.newCondition`创建了两个条件变量,并在货物满或者货物空的时候,分别调用他们的`await()`方法,这会使线程在调用的地方阻塞,并且释放锁,而当我们放入货物时,会唤醒**空**这个条件,反之亦然,他们会尝试获得锁并继续恢复执行

上面的Lock与Condition的配合就与`synchronzied`与`wait/notify`相似,在JUC中,他们的工作是由AQS来完成的

# AQS队列同步器

AQS是用于构建锁和其他同步组件的基础工具,它主要的组件是

- 一个int变量用于表示同步状态
- 一个FIFO队列来完成资源获取线程的排队工作

AQS是JUC包中大部分同步容器的基础组件

## 同步的状态

在对锁的管理中,同步状态是必不可少会被更改的,就好像`synchronized`中规定,锁的状态是存放于对象头中的,而AQS则是使用一个int变量state来对状态进行表示,并提供了几个成员方法用于修改状态.这些状态的方法都是抽象的,我们通过实现AQS接口并实现这些方法来达到管理

![](http://imageblog.boyn.top/202001311649_814.png)

## 接口实现

AQS主要基于模板方式来继承,使用者在编写自己的同步组件的时候通过继承AQS,并实现其抽象方法,而我们在使用的时候,主要是调用AQS的模板方法,从而使其调用组件重写的方法来达到不同的目的.

以下是我们可以重写的方法

![](http://imageblog.boyn.top/202001311659_54.png)

以下是AQS为我们实现好的模板方法

![](http://imageblog.boyn.top/202001311700_499.png)

这些模板方法基本上可以分为3类

- 独占锁
- 共享锁
- 查询等待队列的状态

## 实现同步器接口

AQS涉及到理论的东西比较多,接下来我们可以自己实现一个互斥锁,来看看AQS是怎么来作为一个基础组件提供服务的

在这个互斥锁中,我们只设置了两种状态,0为未上锁,1为已上锁

```java
public class MuteX {
    private static class Sync extends AbstractQueuedSynchronizer{
        /**
         * 用于查看锁是否被占用
         * 由于我们只定义了两种状态,所以如果状态为1,则为上锁,状态为0,则为未上锁
         * @return
         */
        @Override
        protected boolean isHeldExclusively() {
            return this.getState()==1;
        }

        @Override
        protected boolean tryAcquire(int arg) {
            if(compareAndSetState(0,1)){
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            if (getState()==0){
                throw new IllegalMonitorStateException();
            }
            //执行下面方法的时候是没有竞争的
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
    }

    private final Sync sync = new Sync();

    public void lock(){
        sync.acquire(1);
    }

    public void release(){
        sync.release(1);
    }

    public boolean tryLock(){
        return sync.tryAcquire(1);
    }

    public boolean tryRelease(){
        return sync.tryRelease(1);
    }

    public boolean hasWaitingQueue(){
        return sync.hasQueuedThreads();
    }

}
```

在Mutex中,实现了一个Sync内部类用于继承AQS并实现其方法.有一点值得说的就是,在`lock`方法中,调用了sync的acquire方法,这个acquire方法是AQS中的模板方法

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

会先尝试获取锁,如果获取失败,则会将其放入等待队列中,而release则会将等待队列中的线程取出,并恢复其运行.

## AQS的同步队列

在刚刚说到了,AQS主要有两个部件,一个是同步状态,这个状态的值是可以由用户自定义的,还有一个就是同步队列,AQS已经为我们实现了一个FIFO的同步队列.如果当前线程获取锁失败的话,AQS会将当前线程和等待信息形成一个节点(Node),放入等待队列中

节点中的详细信息如图所示

![](http://imageblog.boyn.top/202001311740_816.png)

节点是构成等待队列的基础,AQS中使用了一个Deque用于记录这些节点.

在队列中,新加入的线程是采用CAS线程安全的方法进入队列的

![](http://imageblog.boyn.top/202001311741_222.png)

### 同步队列的节点操作

以上面说到的acquire方法为例,如果尝试获取锁但是失败了的话,会执行以下方法

`acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`

这句话先创建了一个节点状态为互斥的节点,并将其用CAS放入队列中

```java
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }

private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

其中,为什么需要两次插入呢?第一个插入是在addWaiter中进行的,如果这个成功插入了,说明此时没有线程的竞争问题,所以可以安全插入并返回这个新的节点,如果有竞争并插入失败了,就将其使用死循环的方法来设置,直到成功为止.

在成功插入尾部并返回了这个节点之后,就进入到了一个自旋等待的过程,当条件满足了,就会退出自旋状态继续下面的操作,否则依旧会自旋(并阻塞)

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

在代码中,我们可以看到,线程会陷入自旋,并判断自己的节点是否为等待队列的头结点,如果是,那么尝试获取锁并返回(如果获得了),如果条件都失败了,会将线程park掉,即阻塞.

独占式锁获取的流程代码如上所示,可以画出其流程图如下

![](http://imageblog.boyn.top/202001311801_519.png)
