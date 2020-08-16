# 什么是Semaphore

Semaphore,又称信号量,是用于控制同时访问特定资源的线程数量的工具,对信号量最形象的比喻就是银行柜台了,假设有10个线程,将其当作10位客户,然后信号量的值为3,表示开设了3个柜台,那么同一时间只可能有3位客户在柜台中,其他客户只能排队等待,一位客户完成业务后,下一位才能开始办理.

# 使用方法

```java
public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 9; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();
                    System.out.println("run :" +new Date().getTime());
                    Thread.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            }).start();
        }
    }
```

我们创建了一个资源量为3的信号量,然后创建了9个线程,首先需要执行semaphore.acquire()来获取资源,如果资源为空,则阻塞,然后打印时间并等候5秒,最后使用semaphore.release()释放资源.

运行结果为

```
run :1579694197670
run :1579694197670
run :1579694197670
run :1579694197675
run :1579694197675
run :1579694197675
run :1579694197681
run :1579694197681
run :1579694197681
```

可以看到,在同一时间里面,确实只有3个线程在运行