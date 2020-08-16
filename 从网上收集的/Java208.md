# 一、Java基础

## 1.JDK 和 JRE 有什么区别？*

JDK指的是Java开发工具包,用于Java程序的开发

JRE指的是Java Runtime Enviorment,即Java运行时环境

JDK中会包含着JRE,而JRE则不包括JDK

一般来说,我们在写Java程序的时候,需要用到JDK来将我们写的程序从`.java`文件编译成`.class`文件,并放到JRE上面运行

对于服务器环境来说,如果我们不需要在上面开发Java程序,而只是部署运行的话,安装JRE一般来说是足够了,但是也有特例,即JSP文件,它会将JSP文件编译成Servlet文件后再运行,所以需要JDK



## *2.== 和 equals 的区别是什么？*

`==`比较两者的值是否相同,如果是基本类型,则直接表示比较两者的值,而对于引用类型来说,则是比较两者的地址是否相同

equals是`Object`类中的一个方法,我们自己写的类如果要实现比较效果,就要重写这个方法,用于比较两个对象是否相等(这个相等的性质可以由我们自己设定),equals这个函数在实现的时候,至少需要实现以下几个性质:

- 自反性（reflexive）。对于任意不为`null`的引用值x，`x.equals(x)`一定是`true`。
- 对称性（symmetric）。对于任意不为`null`的引用值`x`和`y`，当且仅当`x.equals(y)`是`true`时，`y.equals(x)`也是`true`。
- 传递性（transitive）。对于任意不为`null`的引用值`x`、`y`和`z`，如果`x.equals(y)`是`true`，同时`y.equals(z)`是`true`，那么`x.equals(z)`一定是`true`。
- 一致性（consistent）。对于任意不为`null`的引用值`x`和`y`，如果用于equals比较的对象信息没有被修改的话，多次调用时`x.equals(y)`要么一致地返回`true`要么一致地返回`false`。
- 对于任意不为`null`的引用值`x`，`x.equals(null)`返回`false`。



## *3.两个对象的 hashCode()相同，则 equals()也一定为 true，对吗？*

不一定.我们需要知道,hashCode()表示的是对象的一个哈希码,其计算方式可以有很多种,但是大部分方法都会产生哈希冲突(即不同的对象返回同一个哈希码),但是调用equals比较的时候,并不一定相同



## *4.final 在 java 中有什么作用？*

final表示'最终'的意思,它用在不同的地方可以表示不同的作用.比如如果它用在了类的声明中,则表示这个类是不可以被继承的,如`java.lang.String`中的方法签名就有`final`,表示它不可以被继承.如果它用在了方法中,则子类不可以将他重写,并且在早期的版本中,使用final修饰的java方法会转化为内嵌调用,增加效率,但是现在已经没有这个效果了.

如果它用于修饰变量,则其值在被初始化过后不能再被修改,简单来说,基本类型不能改值,引用类型不能改引用,当然,引用类型内部的对象只要不是final的,还是可以变的





## *5.java 中的 Math.round(-1.5) 等于多少？*

 Math.round(-1.5)的返回值是-1。四舍五入的原理是在参数上加0.5然后做向下取整。 

```java
public class test {
	public static void main(String[] args){
		System.out.println(Math.round(1.3));   //1
		System.out.println(Math.round(1.4));   //1
		System.out.println(Math.round(1.5));   //2
		System.out.println(Math.round(1.6));   //2
		System.out.println(Math.round(1.7));   //2
		System.out.println(Math.round(-1.3));  //-1
		System.out.println(Math.round(-1.4));  //-1
		System.out.println(Math.round(-1.5));  //-1
		System.out.println(Math.round(-1.6));  //-2
		System.out.println(Math.round(-1.7));  //-2
	}
}
```



## *6.String 属于基础的数据类型吗？*

不属于,它在`java.lang`包中,不需要import就可以使用,但是它仍然是一个引用类型





## *7.java 中操作字符串都有哪些类？它们之间有什么区别？*

`String`,`StringBuilder`,`StringBuffer`

在这三个类中,String类是不可变的,一旦被初始化,就无法再改变

而其余两个都是可变的,Builder,Buffer可以用于构建字符串,而区别在于,Builder是非线程安全的,而Buffer是线程安全的





## *8.String str="i"与 String str=new String("i")一样吗？*

str的值是一样的,但是指向的类型不一样,直接赋值的时候,其实是创建了一个值为"i"的String常量,放入常量池中,再将这个引用赋值给str,而通过new函数进行创建,则是在堆中创建了一个新对象,并赋值给str

当我们执行以下代码的时候

```java
String str = "i";
        String str2 = "i";
        System.out.println(str==str2); //true,因为是同一个在常量池中的引用
        str = new String("i");
        str2 = new String("i");
        System.out.println(str==str2); //false,因为是在堆中的不同对象

```





## *9.如何将字符串反转？*

最简单的方法就是将字符串赋值给StringBuilder,然后调用`reverse`方法,再将其换回来

也可以将字符串对象调用`toCharArray()`转换为字符数组,再按数组的方式倒置



## *10.String 类的常用方法都有那些？*

`length()` -- 字符串长度  `toCharArray()`转换为字符串数组 `substring(int,int)`子字符串 `compareTo`与其他字符串比较



## *11.抽象类必须要有抽象方法吗？*

不一定,抽象类可以没有抽象方法,但是不能被实例化





## *12.普通类和抽象类有哪些区别？*

抽象类可以有抽象方法,抽象类不能被实例化





## *13.抽象类能使用 final 修饰吗？*

不能,抽象类要被继承才能够被实例化,如果不能被实例化,则没有任何作用了





## *14.接口和抽象类有什么区别？*

从语义上面来说,接口是定义一个类的行为,而抽象类是定义一个类的抽象父类

而接口虽然可以有默认方法,但是其所有方法的作用域都是`public`,而抽象类可以有`private`,`protect`,`public`等等.





## *15.java 中 IO 流分为几种？*

IO流主要分为字节流和字符流,对应读取的方式就是字节和字符了,其详细信息可以看图

![](http://imageblog.boyn.top/202002071519_755.png)





## *16.BIO、NIO、AIO 有什么区别？*

BIO:阻塞型IO

NIO:非阻塞IO(多路复用IO)

AIO:异步IO

这三种IO,对应的就是最常用的几种IO模型





## *17.Files的常用方法都有哪些？*

Files类用于操作File对象,提供了许多诸如创建文件(`createFile`),复制文件(`copy`),删除文件(`delete`)和读写文件的操作





# **二、容器**

## *18.java 容器都有哪些？*

![](http://imageblog.boyn.top/202002071523_389.png)

其中,最常用的是`ArrayList`,`LinkedList`,`HashMap`和`PriorityQueue`,要理解他们的底层数据结构以及读写数据的实现原理





## *19.Collection 和 Collections 有什么区别？*

Collection是一个接口,基本所有Java的集合类都会实现这个接口中的方法,这个接口主要定义了添加,删除以及遍历元素的行为,Collections是一个工具类,提供了对实现了Colletion接口的类的常用操作,如排序,打乱等等操作





## *20.List、Set、Map 之间的区别是什么？*

List指列表,其中的元素可以重复,有顺序

Set指集合,其中的元素是不能重复的

Map指索引表,索引表可以包括红黑树,哈希索引等索引方式,其特点是查找元素的速度比较快





## *21.HashMap 和 Hashtable 有什么区别？*

HashMap与Hashtable在实现上都是基于线性表+链表+红黑树,但是Hashtable是线程安全的,而HashMap不是线程安全





## *22.如何决定使用 HashMap 还是 TreeMap？*

当我们在遍历的时候,需要按照键有序输出的时候,可以使用TreeMap,TreeMap的底层使用红黑树进行索引,所以在遍历的时候是前序遍历,键的顺序是有序的



## *23.说一下 HashMap 的实现原理？*

[HashMap](../JDK集合/HashMap)



## *24.说一下 HashSet 的实现原理？*

HashSet是基于HashMap进行实现的, 底层采用 HashMap 来保存元素 ,在插入和查询的时候,会使用HashMap的`put`和`get`,值指定为一个空的Object





## *25.ArrayList 和 LinkedList 的区别是什么？*

ArrayList和LinkedList的最大区别在于其底层实现的不同,ArrayList基于数组实现,而LinkedList基于双向链表实现,所以ArrayList可以实现随机读取,而LinkedList可以在头尾进行O(1)的插入而无需扩容





## *26.如何实现数组和 List 之间的转换？*

```java
        Integer[] nums = {1, 2, 3, 4, 5};
        List<Integer> list = new ArrayList<>();
        Collections.addAll(list, nums);
        Integer[] nums2 = new Integer[list.size()];
        list.toArray(nums2);
```





## *27.ArrayList 和 Vector 的区别是什么？*

他们的底层都是使用数组进行数据的存储的,区别在于Vector是线程安全的





## *28.数组和 ArrayList 有何区别？*

Array：它是数组，申明数组的时候就要初始化并确定长度，长度不可变，而且它只能存储同一类型的数据，比如申明为String类型的数组，那么它只能存储S听类型数据
ArrayList：它是一个集合，需要先申明，然后再添加数据，长度是根据内容的多少而改变的，ArrayList可以存放不同类型的数据，在存储基本类型数据的时候要使用基本数据类型的包装类

当能确定长度并且数据类型一致的时候就可以用数组，其他时候使用ArrayList





## *29.在 Queue 中 poll()和 remove()有什么区别？*

他们都会将队头元素出队并返回,但是如果当队列为空的时候,poll返回null,remove会报`NoSuchElement`异常





## *30.哪些集合类是线程安全的？*

JUC中的类,还有`Vector`,`HashTable`等





## *31.迭代器 Iterator 是什么？*

在很多Collection类中,都可以见到迭代器 Iterator以内部类的形式出现在其中,作为该集合的迭代器对象,可以调用`next`方法来遍历集合中的元素





## *32.Iterator 怎么使用？有什么特点？*

```java
ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
		Iterator it = list.iterator();
		while (it.hasNext()) {
			String next = (String) it.next();
			System.out.println(next);
		}
```

迭代器的特点在于不能在迭代中移除这个元素





## *33.Iterator 和 ListIterator 有什么区别？*

ListIterator支持在迭代的时候动态修改元素,插入和删除等操作,还可以倒序遍历





## *34.怎么确保一个集合不能被修改？*

 可以使用Collections.unmodifiableXXX来将一个集合类替换为不可修改的类



# **三、多线程**

## *35.并行和并发有什么区别？*

并发是指可以处理多个任务,但是不一定是同时.并行是可以同时处理多个任务





## *36.线程和进程的区别？*

线程是调度的最小单位,进程是获取资源的基本单位.一般来说,一个进程中可以有多个线程,而真正在执行任务的就是线程,多个线程共享进程中的资源



## *37.守护线程是什么？*

守护线程是一种特殊的线程类型,与我们平常使用的线程(称为用户线程)不同,守护线程不需要我们手动关闭,在JVM中,当只剩下守护线程时,JVM就会自动关闭.最常见的守护线程就是JVM的垃圾回收线程了



## *38.创建线程有哪几种方式？*

可以通过实现Runnable接口,继承Thread等形式,还可以直接用lambda表达式创建一个简单的线程表达式

```java
new Thread(()-> System.out.println("Hi")).start();
```





## *39.说一下 runnable 和 callable 有什么区别？*

他们都是表示线程的接口,但是Runnable是在没有返回值下调用的,而Callable是在有返回值下调用,可以使用`Future`类进行返回值的封装





## *40.线程有哪些状态？*

1. 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
2. 运行(RUNNABLE)：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
3. 阻塞(BLOCKED)：表示线程阻塞于锁。
4. 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
5. 超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。
6. 终止(TERMINATED)：表示该线程已经执行完毕。



## *41.sleep() 和 wait() 有什么区别？*

sleep()表示线程的睡眠,会在这里阻塞直到睡眠时间结束

wait()要在`synchronized`的作用域下使用,它也会导致线程阻塞,但是会释放锁使其他线程可以尝试获取这个锁





## *42.notify()和 notifyAll()有什么区别？*

在`synchronized`作用域下调用`wait`的线程都会被放入阻塞队列中,`notify`会唤醒队列中队头的线程,使其从阻塞状态转变为就绪状态,而`notifyAll`会唤醒所有线程,共同竞争这个锁





## *43.线程的 run()和 start()有什么区别？*

run()是定义在Runnable中的一个方法,它表示线程在运行时候的代码,start()是定义在Thread中,用于表示创建一个新的线程的方法.如果我们创建线程时,运行的是run方法,则不会创建一个新的线程,而是在原有线程中运行run中的代码.只有调用start的时候,才会真正的有线程被创建运行





## *44.线程池有哪几个重要属性*

- corePoolSize,核心池的大小,在创建了线程池并初始化后,线程池中没有任何线程,只有当有任务来才会去创建任务并执行线程.但是线程池也可以预创建线程,在没有任务来之前就创建好`corePoolSize`个线程.如果不预创建,那么就来一个任务创建一个线程,并放入线程池,达到`corePoolSize`这个数目后,就把到达的任务先放入缓存队列
- maximumPoolSize,表示在线程池中最多可以创建多少个线程.
- keepAliveTime,表示线程没有任务执行的时候多长时间会终止.
- unit:keepAliveTime的时间单位,可以通过TimeUnit类来表示时间
- wordQueue,用户指定的一个阻塞队列,用于存储等待执行的任务,一般来说,阻塞队列有`ArrayBlockingQueue,LinkedBlockingQueue,SynchronousQueue`
- threadFactory,线程工厂,用于创建线程,
- handler,用来表示当拒绝处理任务时候的策略.有以下四种:1.AbortPolicy 丢弃任务并抛出异常 2. DiscardPolicy 丢弃任务但不抛出异常 3. DiscardOldestPolicy 丢弃任务队列最前面的任务,然后重试执行任务 4. CallerRunsPolicy 由调用的线程来处理该任务.





## *45.线程池都有哪些状态？*

线程池有以下几种状态

![](http://imageblog.boyn.top/202002071756_724.png)

1、RUNNING

(1) 状态说明：线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理。
(02) 状态切换：线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0！

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

2、 SHUTDOWN

(1) 状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。
(2) 状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。

3、STOP

(1) 状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。
(2) 状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

4、TIDYING

(1) 状态说明：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
(2) 状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。
当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

5、 TERMINATED

(1) 状态说明：线程池彻底终止，就变成TERMINATED状态。
(2) 状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。





## *46.线程池中 submit()和 execute()方法有什么区别？*

submit()用于提交有返回值的任务,即`Callable`接口定义的任务,execute用于提交没有返回值的任务,即`Runnable`接口定义的任务





## *47.在 java 程序中怎么保证多线程的运行安全？*

对竞态资源加锁





## *48.多线程锁的升级原理是什么？*

锁的级别从低到高：

无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁

在大多数场景下,锁的竞争者通常只有一个线程,而如果少量线程为了获取释放锁进行频繁的阻塞操作,就会大大降低效率,所以在JDK6中,将`synchronized`改为了多状态的锁

无锁:没有对资源进行锁定

偏向锁: 对象的代码一直被同一线程执行，不存在多个线程竞争，该线程在后续的执行中自动获取锁，降低获取锁带来的性能开销。偏向锁，指的就是偏向第一个加锁线程，该线程是不会主动释放偏向锁的，只有当其他线程尝试竞争偏向锁才会被释放 

轻量级锁: 当锁是偏向锁的时候,如果有其他线程对这个锁进行尝试获取时,就会转为轻量级锁,通过自旋的方式获取锁,线程不会阻塞,从而提高性能

重量级锁:当自旋超过一定次数,或者当一个线程在自旋的时候还有另外一个线程尝试获取锁的时候,就会转化为重量级锁,重量级锁依靠对象内部的监视器进行实现,而监视器则是依靠操作系统的Mutex Lock实现,所以会从用户态陷入内核态,使得切换成本比较高





## *49.什么是死锁？*

在对多段资源的多个锁竞争的情况下,就有可能出现死锁的情况

假设我们有*资源A*与*资源B*,他们都需要获得*锁X*和*锁Y*之后才可以访问

*线程1*获得了*锁X*,*线程2*获得了*锁Y*,并且在没有获得另外锁的情况下,不会解除已经获得的锁,这个时候,就产生了死锁





## *50.怎么防止死锁？*

死锁形成的条件有以下几个,破坏任意一个即可

- 互斥：每个资源要么已经分配给了一个进程，要么就是可用的。
- 占有和等待：已经得到了某个资源的进程可以再请求新的资源。
- 不可抢占：已经分配给一个进程的资源不能强制性地被抢占，它只能被占有它的进程显式地释放。
- 环路等待：有两个或者两个以上的进程组成一条环路，该环路中的每个进程都在等待下一个进程所占有的资源。





## *51.ThreadLocal 是什么？有哪些使用场景？*

简单来说,ThreadLocal就是附带在每一个线程中,可以用get/set方法来进行数据存储的一个类.也就是说一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值





## *52.说一下 synchronized 底层实现原理？*

详见48题





## *53.synchronized 和 volatile 的区别是什么？*

`synchronized`和`volatile`区别在于,前者可以保证可见性和原子性,后者只能保证可见性而无法保证原子性

使用synchronized需要加锁,而volatile不需要,比synchronized要快





## *54.synchronized 和 Lock 有什么区别？*

`synchronized`和Lock的区别在于synchronized是Java中的关键字,Lock是接口.

Lock的锁粒度可以做的比synchronized更小,而且支持锁的中断,非阻塞获取等特性





## *55.synchronized 和 ReentrantLock 区别是什么？*

`ReentrantLock`显示获得、释放锁，`synchronized`隐式获得释放锁

 `ReentrantLock`可响应中断、可轮回，`synchronized`是不可以响应中断的，为处理锁的不可用性提供了更高的灵活性

 `ReentrantLock`是`API`级别的，`synchronized`是`JVM`级别的

 `ReentrantLock`可以实现公平锁

 `ReentrantLock`通过`Condition`可以绑定多个条件

底层实现不一样， `synchronized`是同步阻塞，使用的是悲观并发策略，`lock`是同步非阻塞，采用的是乐观并发策略



## *56.说一下 atomic 的原理？*

atomic通过CAS进行实现





# **四、反射**

## 57.*什么是反射*？

反射,简单来说就是在运行时获取类的元信息,可以通过反射更改类的私有变量,执行类的方法等



## *58.什么是 java 序列化？什么情况下需要序列化？*

序列化：将对象写入到IO流中**

反序列化：从IO流中恢复对象**

意义：序列化机制允许将实现序列化的Java对象转换位字节序列，这些字节序列可以保存在磁盘上，或通过网络传输，以达到以后恢复成原来的对象。序列化机制使得对象可以脱离程序的运行而独立存在。

使用场景：所有可在网络上传输的对象都必须是可序列化的，比如RMI（remote method invoke,即远程方法调用），传入的参数或返回的对象都是可序列化的，否则会出错；所有需要保存到磁盘的java对象都必须是可序列化的。通常建议：程序创建的每个JavaBean类都实现Serializeable接口。

## *59.动态代理是什么？有哪些应用？*

要讲清楚动态代理,我们首先要知道什么是代理模型,这个是一种结构化的设计模型,简单来说这个设计模式会使用一个代理对象把真实对象包装起来,并将我们对真实对象的调用转化为对代理对象的调用,这样就可以实现一些比如打印日志,强化对象功能的操作了.

动态代理广泛地运用在Java中,比如Spring AOP,Java注解获取对象等等





## *60.怎么实现动态代理？*

常见实现动态代理的方式,有基于接口的JDK的动态代理和通过继承类的CGLIB



# **五、对象拷贝**

## *61.为什么要使用克隆？*

克隆可以实现对象的复制

## *62.如何实现对象克隆？*

实现`clone`方法

## *63.深拷贝和浅拷贝区别是什么？*

浅拷贝指的是只是拷贝对象的引用,一旦对拷贝对象中某些域更改,原对象也会更改

而深拷贝是指从对象的最底层开始进行值的复制,深拷贝对象与原对象无关系,修改一个,另一个不会改变





# **六、Java Web**

## *64.jsp 和 servlet 有什么区别？*

jsp是基于文本的程序,可以与html代码共存,在java中,JSP本身就是Servlet,他在访问的时候会被编译为一个HttpJspPage类,作为Servlet被访问

Servlet是一种规范,服务器可以通过多个Servlet访问不同的地址,运行在服务器端



65.jsp 有哪些内置对象？作用分别是什么？(略)

66.说一下 jsp 的 4 种作用域？(略)

## *67.session 和 cookie 有什么区别？*

session保存在服务器上,而cookie保存在客户端(浏览器中),通常来说,session和cookie都是为了保存用户浏览的'状态'而存在的,但是session能够保存的长度比cookie要大





## *68.说一下 session 的工作原理？*

session在服务端中一般有一个session ID用于定位session,通常来说会存放在一个map中

## *69.如果客户端禁止 cookie 能实现 session 还能用吗？*

可以通过别的方式实现,只要在客户端能够保存一个标识码定位服务端中的session,就可以继续使用





70.spring mvc 和 struts 的区别是什么？(略)

## *71.如何避免 sql 注入？*

Java中,可以使用 **PreparedStatement** ,它会将语句预编译,从而防止sql注入.

使用Mybatis时,我们写xml中的sql语句时同样也使用了预编译的技术,并且,如果在mybatis的xml语句中 `$`是直接进行字符串替换的,安全性不高,一般使用`#`



## *72.什么是 XSS 攻击，如何避免？*

 XSS攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。 这些恶意网页程序通常是JavaScript，但实际上也可以包括Java、 VBScript、ActiveX、 Flash 或者甚至是普通的HTML。 

对用户输入的文件要进行过滤

## *73.什么是 CSRF 攻击，如何避免？*

 **跨站请求伪造**（英语：Cross-site request forgery），也被称为 **one-click attack** 或者 **session riding**，通常缩写为 **CSRF** 或者 **XSRF**， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。**XSS** 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。 

 跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去运行。这利用了web中用户身份验证的一个漏洞：**简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的**。 

避免措施可以检查HTTP头部的Reference字段,查看是否同源

# **七、异常**

## *74.throw 和 throws 的区别？*

throw用于方法的语句中,表示抛出错误

throws用于方法的签名中,表示在方法中有可能会抛出某个异常,要求调用它的方法捕捉或抛出



## *75.final、finally、finalize 有什么区别？*

final用于声明类,方法或者变量,finally用于指定`try-catch`语句中最后执行的语句块,finalize推荐不要用,是类似于C+中的析构函数,但是Java有垃圾回收机制,所以`finalize`方法不一定会在调用时就被执行



## *76.try-catch-finally 中哪个部分可以省略？*

finally



## *77.try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？*

会



## *78.常见的异常类有哪些？*

![](http://imageblog.boyn.top/202002080929_691.png)



# **八、网络**

## *79.http 响应码 301 和 302 代表的是什么？有什么区别*？

301:永久重定向

302:临时重定向





## *80.forward 和 redirect 的区别？*

**forward（转发）**：

是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,因为这个跳转过程实在服务器实现的，并不是在客户端实现的所以客户端并不知道这个跳转动作，所以它的地址栏还是原来的地址.

**redirect（重定向）**：

是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.

转发是服务器行为，重定向是客户端行为。





## *81.简述 tcp 和 udp的区别？*

- tcp面向流传输,udp面向包传输
- tcp面向连接,而udp无需连接
- tcp可以保证可靠传输,而udp不能



## *82.tcp 为什么要三次握手，两次不行吗？为什么？*

如果只是两次握手就可以建立连接,那么服务端就有可能耗费更多资源,因为服务端有可能接收到已经断开连接的包或者超时的包,但仍然为其建立连接





## *83.说一下 tcp 粘包是怎么产生的？*

如果一个TCP包写入的数据小于缓冲区大小,网卡可能会将多次写入的数据发送到网络上,会发生粘包

如果应用程序接收不及时,缓冲区溢出,也有可能粘包

所以我们在应用层中可以引入一些分隔符,如`\n`来进行包和包之间的划分





## *84.OSI 的七层模型都有哪些？*

自底向上:物理层,链路层,网络层,传输层,会话层,表示层,应用层





## *85.get 和 post 请求有哪些区别？*

### 作用

GET 用于获取资源，而 POST 用于传输实体主体。

### 参数

GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体中。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看。

因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 `中文` 会转换为 `%E4%B8%AD%E6%96%87`，而空格会转换为 `%20`。POST 参数支持标准字符集。

```
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1
POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
```

### 安全

安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的。

GET 方法是安全的，而 POST 却不是，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变。

安全的方法除了 GET 之外还有：HEAD、OPTIONS。

不安全的方法除了 POST 之外还有 PUT、DELETE。

### 幂等性

幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。

所有的安全方法也都是幂等的。

在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。

GET /pageX HTTP/1.1 是幂等的，连续调用多次，客户端接收到的结果都是一样的：

```
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
```

POST /add_row HTTP/1.1 不是幂等的，如果调用多次，就会增加多行记录：

```
POST /add_row HTTP/1.1   -> Adds a 1nd row
POST /add_row HTTP/1.1   -> Adds a 2nd row
POST /add_row HTTP/1.1   -> Adds a 3rd row
```

DELETE /idX/delete HTTP/1.1 是幂等的，即使不同的请求接收到的状态码不一样：

```
DELETE /idX/delete HTTP/1.1   -> Returns 200 if idX exists
DELETE /idX/delete HTTP/1.1   -> Returns 404 as it just got deleted
DELETE /idX/delete HTTP/1.1   -> Returns 404
```

### 可缓存

如果要对响应进行缓存，需要满足以下条件：

- 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的。
- 响应报文的状态码是可缓存的，包括：200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501。
- 响应报文的 Cache-Control 首部字段没有指定不进行缓存。

### XMLHttpRequest

为了阐述 POST 和 GET 的另一个区别，需要先了解 XMLHttpRequest：

> XMLHttpRequest 是一个 API，它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 AJAX 中被大量使用。

- 在使用 XMLHttpRequest 的 POST 方法时，浏览器会先发送 Header 再发送 Data。但并不是所有浏览器会这么做，例如火狐就不会。
- 而 GET 方法 Header 和 Data 会一起发送。





86.如何实现跨域？(略)

87.说一下 JSONP 实现原理？(略)



# **九、设计模式**

## *88.说一下你熟悉的设计模式？*

面试中常用的设计模式有 工厂(简单工厂与抽象工厂),策略模式,单例模式等等





## *89.简单工厂和抽象工厂有什么区别？*

抽象工厂可以称之为'工厂的工厂'...





**十、Spring/Spring MVC**

90.为什么要使用 spring？

91.解释一下什么是 aop？

92.解释一下什么是 ioc？

93.spring 有哪些主要模块？

94.spring 常用的注入方式有哪些？

95.spring 中的 bean 是线程安全的吗？

96.spring 支持几种 bean 的作用域？

97.spring 自动装配 bean 有哪些方式？

98.spring 事务实现方式有哪些？

99.说一下 spring 的事务隔离？

100.说一下 spring mvc 运行流程？

101.spring mvc 有哪些组件？

102.@RequestMapping 的作用是什么？

103.@Autowired 的作用是什么？

**十一、Spring Boot/Spring Cloud**

104.什么是 spring boot？

105.为什么要用 spring boot？

106.spring boot 核心配置文件是什么？

107.spring boot 配置文件有哪几种类型？它们有什么区别？

108.spring boot 有哪些方式可以实现热部署？

109.jpa 和 hibernate 有什么区别？

110.什么是 spring cloud？

111.spring cloud 断路器的作用是什么？

112.spring cloud 的核心组件有哪些？

**十二、Hibernate**

113.为什么要使用 hibernate？

114.什么是 ORM 框架？

115.hibernate 中如何在控制台查看打印的 sql 语句？

116.hibernate 有几种查询方式？

117.hibernate 实体类可以被定义为 final 吗？

118.在 hibernate 中使用 Integer 和 int 做映射有什么区别？

119.hibernate 是如何工作的？

120.get()和 load()的区别？

121.说一下 hibernate 的缓存机制？

122.hibernate 对象有哪些状态？

123.在 hibernate 中 getCurrentSession 和 openSession 的区别是什么？

124.hibernate 实体类必须要有无参构造函数吗？为什么？

**十三、Mybatis**

125.mybatis 中 #{}和 ${}的区别是什么？

126.mybatis 有几种分页方式？

127.RowBounds 是一次性查询全部结果吗？为什么？

128.mybatis 逻辑分页和物理分页的区别是什么？

129.mybatis 是否支持延迟加载？延迟加载的原理是什么？

130.说一下 mybatis 的一级缓存和二级缓存？

131.mybatis 和 hibernate 的区别有哪些？

132.mybatis 有哪些执行器（Executor）？

133.mybatis 分页插件的实现原理是什么？

134.mybatis 如何编写一个自定义插件？

**十四、RabbitMQ**

135.rabbitmq 的使用场景有哪些？

136.rabbitmq 有哪些重要的角色？

137.rabbitmq 有哪些重要的组件？

138.rabbitmq 中 vhost 的作用是什么？

139.rabbitmq 的消息是怎么发送的？

140.rabbitmq 怎么保证消息的稳定性？

141.rabbitmq 怎么避免消息丢失？

142.要保证消息持久化成功的条件有哪些？

143.rabbitmq 持久化有什么缺点？

144.rabbitmq 有几种广播类型？

145.rabbitmq 怎么实现延迟消息队列？

146.rabbitmq 集群有什么用？

147.rabbitmq 节点的类型有哪些？

148.rabbitmq 集群搭建需要注意哪些问题？

149.rabbitmq 每个节点是其他节点的完整拷贝吗？为什么？

150.rabbitmq 集群中唯一一个磁盘节点崩溃了会发生什么情况？

151.rabbitmq 对集群节点停止顺序有要求吗？

**十五、Kafka**

152.kafka 可以脱离 zookeeper 单独使用吗？为什么？

153.kafka 有几种数据保留的策略？

154.kafka 同时设置了 7 天和 10G 清除数据，到第五天的时候消息达到了 10G，这个时候 kafka 将如何处理？

155.什么情况会导致 kafka 运行变慢？

156.使用 kafka 集群需要注意什么？

**十六、Zookeeper**

157.zookeeper 是什么？

158.zookeeper 都有哪些功能？

159.zookeeper 有几种部署模式？

160.zookeeper 怎么保证主从节点的状态同步？

161.集群中为什么要有主节点？

162.集群中有 3 台服务器，其中一个节点宕机，这个时候 zookeeper 还可以使用吗？

163.说一下 zookeeper 的通知机制？

**十七、MySql**

164.数据库的三范式是什么？

165.一张自增表里面总共有 7 条数据，删除了最后 2 条数据，重启 mysql 数据库，又插入了一条数据，此时 id 是几？

166.如何获取当前数据库版本？

167.说一下 ACID 是什么？

168.char 和 varchar 的区别是什么？

169.float 和 double 的区别是什么？

170.mysql 的内连接、左连接、右连接有什么区别？

171.mysql 索引是怎么实现的？

172.怎么验证 mysql 的索引是否满足需求？

173.说一下数据库的事务隔离？

174.说一下 mysql 常用的引擎？

175.说一下 mysql 的行锁和表锁？

176.说一下乐观锁和悲观锁？

177.mysql 问题排查都有哪些手段？

178.如何做 mysql 的性能优化？

**十八、Redis**

179.redis 是什么？都有哪些使用场景？

180.redis 有哪些功能？

181.redis 和 memecache 有什么区别？

182.redis 为什么是单线程的？

183.什么是缓存穿透？怎么解决？

184.redis 支持的数据类型有哪些？

185.redis 支持的 java 客户端都有哪些？

186.jedis 和 redisson 有哪些区别？

187.怎么保证缓存和数据库数据的一致性？

188.redis 持久化有几种方式？

189.redis 怎么实现分布式锁？

190.redis 分布式锁有什么缺陷？

191.redis 如何做内存优化？

192.redis 淘汰策略有哪些？

193.redis 常见的性能问题有哪些？该如何解决？

**十九、JVM**

194.说一下 jvm 的主要组成部分？及其作用？

195.说一下 jvm 运行时数据区？

196.说一下堆栈的区别？

197.队列和栈是什么？有什么区别？

198.什么是双亲委派模型？

199.说一下类加载的执行过程？

200.怎么判断对象是否可以被回收？

201.java 中都有哪些引用类型？

202.说一下 jvm 有哪些垃圾回收算法？

203.说一下 jvm 有哪些垃圾回收器？

204.详细介绍一下 CMS 垃圾回收器？

205.新生代垃圾回收器和老生代垃圾回收器都有哪些？有什么区别？

206.简述分代垃圾回收器是怎么工作的？

207.说一下 jvm 调优的工具？

208.常用的 jvm 调优的参数都有哪些？



 https://www.nowcoder.com/discuss/186528?type=2&order=0&pos=33&page=2 

 https://www.nowcoder.com/discuss/146537 

 https://www.nowcoder.com/discuss/352030?type=2&order=0&pos=52&page=2 