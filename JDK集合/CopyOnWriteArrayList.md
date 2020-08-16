# 什么是CopyOnWriteArrayList

CopyOnWriteArrayList是一个线程安全的ArrayList类,并且比Vector更加地高效.它是通过读写分离来实现的.

具体来说,在读的时候,不需要上锁,可以直接对数组进行读取

而在写的时候,会将对方法进行上锁,然后将数组复制一份,然后在复制的数组上面进行操作,再将类中的数组引用更改为新的数组

## 实现

我们通过其源码来剖析其实现

首先,我们看看get

```java
public E get(int index) {
        return get(getArray(), index);
    }
    private E get(Object[] a, int index) {
        return (E) a[index];
    }
 final Object[] getArray() {
        return array;
    }
```

我们刚刚说过,数组的读取是不需要加锁的,我们可以直接对其进行读取

其次,我们来看看add代码

```java
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

add代码使用了可重入锁来保证同一时刻只有一个线程进行线程的复制修改

对于数组对象来说,它是用volatile修饰的,将旧的数组引用指向新的数组对于volatile的happens-before规则来说,写线程对于数组引用的修改是能够对读线程可见的

## 最终一致性

COW放开了锁的限制,但是换来的是限制了数据的实时性,只能保证数据的最终一致性

<!-- Markdeep: --><style class="fallback">body{visibility:hidden;white-space:pre;font-family:monospace}</style><script src="markdeep.min.js" charset="utf-8"></script><script src="https://casual-effects.com/markdeep/latest/markdeep.min.js" charset="utf-8"></script><script>window.alreadyProcessedMarkdeep||(document.body.style.visibility="visible")</script>


