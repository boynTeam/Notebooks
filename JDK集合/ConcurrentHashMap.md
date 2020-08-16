ConcurrentHashMap是JUC工具类中,提供了并发安全的[HashMap](./HashMap.md)

## 底层结构

ConcurrentHashMap在JDK7,8也有不同的实现,我们先来看看JDK7中的CHM

它的底层结构如下

![](http://imageblog.boyn.top/201912281153_363.png)

由Segment数组,Entry数组组成,和HashMap一样是数组加链表的设置

```java
/**
 * Segment 数组，存放数据时首先需要定位到具体的 Segment 中。
 */
final Segment<K,V>[] segments;
transient Set<K> keySet;
transient Set<Map.Entry<K,V>> entrySet;

  static final class Segment<K,V> extends ReentrantLock implements Serializable {
       private static final long serialVersionUID = 2249069246763182397L;
       
       // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
       transient volatile HashEntry<K,V>[] table;
       transient int count;
       transient int modCount;
       transient int threshold;
       final float loadFactor;
       
}
```

在原理上,它使用了分段锁的技术,Segment是可重入锁的子类,理论上来说,ConcurrentHashMap支持的并发量最大为Segment数组的数量,每当一个线程占用锁的时候,只会锁定一部分的数据,而不会影响其他数据,而不是像HashTable一样只是简单地在get和put上面加上`synchronized`关键字造成效率低下.

而在JDK8中,做了一些结构上的调整,并且放弃了原有的Segment分段锁,采用[CAS](../JDK并发/CAS.md)+[synchronized](../JDK并发/synchronized关键字)来保证并发的安全性

![](http://imageblog.boyn.top/201912281552_994.png)

## put

主要步骤是:

- 先通过key定位对应的Segment
- 在Segment中进行具体的put
- put中先尝试获取锁
- 如果没有获取到锁,则自旋等待锁的释放
- 如果自旋的次数达到自旋上限,也更改为阻塞锁
- 获取到了锁之后,后面put的步骤与HashMap的相似,当然了Segment不允许key为null

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}

final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}
```

在JDK8中,由于取消了分段锁的设计,所以put的步骤也整个被重写了

```java
/** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh; K fk; V fv;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else if (onlyIfAbsent // check first node without acquiring lock
                     && fh == hash
                     && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                     && (fv = f.val) != null)
                return fv;
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key, value);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                        else if (f instanceof ReservationNode)
                            throw new IllegalStateException("Recursive update");
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```



- 根据key计算出hashcode
- 判断是否需要进行初始化
- Node f为当前key对应的Node,如果为空,则表明当前位置是可以写入数据的,利用CAS尝试向其写入数据,但是如果失败,说明有其他线程正在调用这个结点上的值,用自旋来保证成功
- 如果当前位置的hashcode- == MOVED == -1 则表示需要扩容
- 如果都不满足,则使用synchronized来写入数据
- 如果数量大于...,则将结点转换成红黑树

## get

get方法十分简单,我们只需要将key通过hash定位到某一个segment后,再通过hash定位到具体的元素上面,由于HashMap上面的元素通过[volatile](../JDK并发/volatile关键字.md)关键词进行修饰,保证了内存的可见性,所以每一次获取的时候,都可以确保是最新值

 ConcurrentHashMap 的 get 方法是非常高效的，**因为整个过程都不需要加锁**。 

而JDK8中,对其改动也不大

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。

