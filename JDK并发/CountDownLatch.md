# 什么是CountDownLatch

CountDownLatch允许一个或多个线程等待其他线程完成操作

简单地来说,假设在一次多线程的计算中,main线程需要等待其他线程执行完毕后,才可以继续往下执行,可以使用join方法来使调用该方法的线程一直等待下去,也可以使用CountDownLatch

# 使用方法

```java
public static void main(String[] args) throws InterruptedException {
        CountDownLatch c = new CountDownLatch(2);
        new Thread(() -> {
            System.out.println("1");
            c.countDown();
            System.out.println("2");
            c.countDown();
        }).start();
        c.await();
        System.out.println("3");
    }
```

先声明了一个CountDownLatch,其计数器值为2,调用await方法的线程,只有当其计数器值为0的时候,才会继续执行,否则就会阻塞.countDown()方法可以有多种应用,可以是多个线程,也可以是一个线程中的几个步骤

需要注意的是,CountDownLatch的计数器是不可重置的.