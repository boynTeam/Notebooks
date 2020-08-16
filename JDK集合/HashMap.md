# 哈希表原理

哈希表是一个高效查找元素的数据结构.它的底层数据结构是一个数组,当某个元素要插入哈希表的时候,先计算它的哈希值h,然后根据它的哈希值,跟数组长度进行取余`h%length-1`(要减一只是因为数组下标从0开始算),就可以根据取余结果放入对应的数组下标之中了.如果要取这个元素,那么就根据这个`key`值,查找对应的数组下标,来获取相应的元素

**Hash冲突**

在进行哈希值的过程中,必不可少的会发生哈希冲突,也就是两个元素并不相同,却有着同样的哈希值.发生了这样的情况,一般来说哈希表有两种解决方法,一种是链地址法(拉链法),一种是开放地址法(也称探测法,可以分为线性探测等).在这里着重说一下链地址法,因为这个方法是用的最多的,`HashMap`解决冲突的办法就是链地址法.

链地址法将哈希表的数组中的所有元素换成了链表的结点,当发生哈希冲突的时候,就可以将这个元素照样放入对应的数组下标,然后通过创建链表新结点的方式,来将冲突的元素都放入一个链表中

在查找的时候,我们通过`key`的哈希值查找到了对应的数组下标之后,获得的是对应链表的头节点,并可以遍历这个链表,获取与key相同的元素.

# HashMap

jdk的HashMap的原理与这个相似.下面从底层结构,put,get,rehash的几个角度来看一下HashMap做了什么优化

## 底层结构

在**JDK7**中,HashMap的底层结构是一个Entry,其表示一个键值对,Entry就是上面所说的一个链表的结点

```java
/**
* 内部类
* hashmap中每一个键值对都是存在Entry对象中，entry还存储了自己的hash值等信息
* Entry被储存在hashmap的内部数组中。
* @param <K> 键值名key
* @param <V> 键值value
*/
static class Entry<K,V> implements Map.Entry<K,V> {
        //键对象，final修饰，不可变
        final K key;
        //值对象
        V value;
        //下一个键值对Entry对象
        Entry<K,V> next;
        //键对象的哈希值
        int hash;

        //提供了一个有参构造方法
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
       ...
    }
```

在HashMap中,首层的数组就是由Entry类所形成的数组` Entry[] table; `



而在JDK8中,为了防止在某个hash值碰撞程度过大,从而令链表的查询时长过长,所有就使用链表+[红黑树](../数据结构/红黑树)的方式来进行冲突的处理方式,当存储的个数大于等于8的时候,不再使用链表进行存储,而使用红黑树作为存储结构,而为了配合红黑树,底层的存储结构改名为Node,但是结构与Entry基本一致

## put

我们要知道,HashMap是支持null值作为key的,一般来说,null的key作为数组的第一位放入,put方法中,首先检验key是否为null,如果是,则将其放入数组第一位.

如果key不是null,那么就可以根据Key所属类调用hashCode方法,并根据这个hashCode再次计算一次HashMap自己实现的hash(为什么要这样,请看下文),得到最终的hash值,然后再根据数组的长度取下标(数组的长度要求为2的幂是有说法的,请看下文).拿到了下标之后,我们首先要遍历对应的链表,寻找看看有没有对应的key已经在链表中,就是有没有重复的`key`元素,如果有的话,那么就替换其`value`,如果没有,就放入链表的尾结点中,作为新的结点

```java
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value); //null总是放在数组的第一个链表中
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        //遍历链表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            //如果key在链表中已存在，则替换为新value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }

 

void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e); //参数e, 是Entry.next
    //如果size超过threshold，则扩充table大小。再散列
    if (size++ >= threshold)
            resize(2 * table.length);
}

private V putForNullKey(V value) {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
```

 **JDK7:**发生hash冲突时，新元素插入到链表头中，即新元素总是添加到数组中，就元素移动到链表中。 **JDK8：**发生hash冲突后，会优先判断该节点的数据结构式是红黑树还是链表，如果是红黑树，则在红黑树中插入数据；如果是链表，则将数据插入到链表的尾部并判断链表长度是否大于8，如果大于8要转成红黑树。 

## get

获取元素的过程同样需要计算哈希值

首先,判断元素是否为null,如果不是,则计算其hash,然后取下标,找到链表的头结点,遍历这个链表,找到对应key的value并返回,如果没有这个元素,就返回null

```java
public V get(Object key) {
        if (key == null)
            return getForNullKey();
        int hash = hash(key.hashCode());
        //先定位到数组元素，再遍历该元素处的链表
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                return e.value;
        }
        return null;
}

private V getForNullKey() {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        return null;
    }
```

## rehash

如果HashMap的大小超过了负载因子(默认0.75),就会触发reshash机制,会创建相当于原来两倍大小的数组,重新调整map的大小,并将原来的对象一个个地用最新的大小重新计算下标放入新的数组中

**JDK7:**在扩容resize（）过程中，采用单链表的头插入方式，在将旧数组上的数据 转移到 新数组上时，转移操作 = 按旧链表的正序遍历链表、在新链表的头部依次插入，即在转移数据、扩容后，容易出现链表逆序的情况 。 多线程下resize()容易出现死循环。此时若（多线程）并发执行 put（）操作，一旦出现扩容情况，则 容易出现 环形链表，从而在获取数据、遍历链表时 形成死循环（Infinite Loop），即 死锁的状态 。

**JDK8:**由于 JDK 1.8 转移数据操作 = 按旧链表的正序遍历链表、在新链表的尾部依次插入，所以不会出现链表 逆序、倒置的情况，故不容易出现环形链表的情况 ，但jdk1.8仍是线程不安全的，因为没有加同步锁保护。

```java
void transfer(Entry[] newTable)
{
    Entry[] src = table;
    int newCapacity = newTable.length;
    //下面这段代码的意思是：
    //  从OldTable里摘一个元素出来，然后放到NewTable中
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}
```

## HashMap并发下的不安全性

HashMap是一个非线程安全的类,在多线程的环境下,会造成一些复杂的死循环问题,使得程序出错.在多线程下,如果同时插入或者修改,那么会抛出一个错误.但是如果多个线程进行rehash,那么就会产生死循环



