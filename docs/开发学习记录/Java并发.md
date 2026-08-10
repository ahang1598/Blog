

# 1. 底层原理

## 1.1 volatile

原则：
- Lock前缀指令会引起处理器缓存回写到内存
- 一个处理器的缓存回写到内存会导致其他处理器的缓存无效

内存模型
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/202205081718936.png)


特点：
- 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入
- 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性




## 1.2 synchronized
Java SE 1.6中为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁，以及锁的存储结构和升级过程

锁对象形式
- 对于普通同步方法，锁是当前实例对象。
- 对于静态同步方法，锁是当前类的Class对象。
- 对于同步方法块，锁是Synchonized括号里配置的对象

底层实现：
JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，但两者的实现细节不一样。
代码块同步是使用monitorenter和monitorexit指令实现的。
monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处。
线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor的所有权，即尝试获得对象的锁


Java SE 1.6中，锁一共有4种状态，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。
锁可以升级但不能降级。

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/202205081743988.png)


### 1.2.1 偏向锁
当一个线程访问同步块并获取锁时，会在<mark style="background: #FF5582A6;">对象头和栈帧中的锁记录里存储锁偏向的线程ID</mark> ，以后该线程再次进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。

获取锁：对象头的Mark Word里是否存储着指向当前线程的偏向锁
如果测试成功，表示线程已经获得了锁。
如果测试失败，则需要再测试一下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁）：
	- 如果没有设置，则使用CAS竞争锁；
	- 如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程

撤销偏向锁：
当Mark Word设置偏向锁置 1
此时线程B请求时，发现指向的是线程A
- 线程A存活时，撤销偏向锁，锁升级
- 线程A未存活，替换Mark Word里指向线程为B

### 1.2.2 轻量级锁

1. JVM会先在当前线程的栈桢中创建用于存储<mark style="background: #FF5582A6;">锁记录</mark> 的空间，并将对象头中的<mark style="background: #FF5582A6;">Mark Word复制到锁记录</mark> 中
2. 然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁
3. 如果失败，表示其他线程竞争锁，当前线程便尝试使用<mark style="background: #FF5582A6;">自旋</mark> 来获取锁。当自旋一定次数后锁膨胀升级为重量级锁

### 1.2.3 重量级锁

重量级锁时，当锁被其他线程获取时，新来的线程会被阻塞，直到当前线程执行完。

### 1.2.4 锁对比

| 锁       | 优点                                 | 缺点                            | 适用场景                       |
| -------- | ------------------------------------ | ------------------------------- | ------------------------------ |
| 偏向锁   | 加锁解锁无消耗                       | 存在锁竞争会产生额外消耗        | 单线程访问同步块               |
| 轻量级锁 | 竞争的线程不会阻塞，提高程序响应速度 | 得不到锁的线程会不断自旋消耗CPU | 追求响应速度，同步块执行速度快 |
| 重量级锁 | 没有自旋消耗CPU                      | 线程阻塞，响应缓慢              | 追求吞吐量，同步块执行时间长                               |


## 1.3 原子操作
定义：不可被中断的一个或一系列操作

### 1.3.1 处理器原子操作
方式一：总线控制
总线锁就是使用处理器提供的一个`LOCK＃`信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该
处理器可以独占共享内存

方式二：缓存锁定
“缓存锁定”是指内存区域如果被缓存在处理器的缓存行中，并且在Lock操作期间被锁定，那么当它执行锁操作回写到内存时，处理器不在总线上声言`LOCK＃`信号，而是修改内部的内存地址，并允许它的缓存一致性机制来保证操作的原子性。
因为缓存一致性机制会阻止同时修改由两个以上处理器缓存的内存区域数据，当其他处理器回写已被锁定的缓存行的数据时，会使缓存行无效。

### 1.3.2 Java原子操作CAS

方式一：循环CAS实现原子操作
定义：CompareAndSet，先读取一个变量，然后当修改该变量时，会再次读取该变量最新值，如果相同则直接设置，不同再具体处理。

**三大问题**
ABA问题：
		如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号：<mark style="background: #BBFABBA6;">A→B→A就会变成1A→2B→3A</mark> 
	
循环时间长开销大：
		<mark style="background: #BBFABBA6;">自旋CAS</mark> 如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令，那么效率会有一定的提升。pause指令有两个作用：第一，它可以延迟流水线执行指令（de-pipeline），使CPU不会消耗过多的执行资源

只能保证一个共享变量的原子操作：
		把多个共享变量合并成一个共享变量来操作。比如，有<mark style="background: #BBFABBA6;">两个共享变量i＝2，j=a，合并一下ij=2a</mark> ，然后用CAS来操作ij。
		JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

方式二：锁机制实现原子操作
锁机制保证了只有获得锁的线程才能够操作锁定的内存区域

有偏向锁、轻量级锁和互斥锁。
有意思的是除了偏向锁，JVM实现锁的方式都用了循环CAS，即当一个线程想进入同步块的时候使用循环CAS的方式来获取锁，当它退出同步块的时候使用循环CAS释放锁


## 1.4 内存模型
Java的并发采用的是<mark style="background: #FF5582A6;">共享内存模型</mark> ，Java线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。
`JMM`：Java内存模型

### 1.4.1 重排序
源代码到指令序列的重排序
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/202205082102635.png)

对于单处理器中，涉及到写操作时不会改变指令顺序而导致结果不一致。
- `a = 1; b = 1; c = a + b;` 这种对于`a,b`变量赋值顺序变化没有影响，但是`c`不能出现在`a,b`之前

但是对多处理器，数据依赖性就会被打破。`i++`在多线程下结果不一定正确。


### 1.4.2 concurrent包
基石：volatile和CAS
首先，声明共享变量为volatile。
然后，使用CAS的原子条件更新来实现线程之间的同步。
同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/202205082128308.png)



### 1.4.3 happens-before

happens-before规则如下。
- 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的
读。
- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
- start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
- join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回

注意　两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！<mark style="background: #BBFABBA6;">happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见</mark> ，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）

### 1.4.4 懒加载与双重锁检查

[4 1 2 单例模式的实现](设计模式.md#4%201%202%20单例模式的实现)
```java
/**
 * 双重检查方式
 */
public class Singleton {

    //私有构造方法
    private Singleton() {}
	
    // 添加volatile
    private static volatile Singleton instance;

   //对外提供静态方法获取该对象
    public static Singleton getInstance() {
		//第一次判断，如果instance不为null，不进入抢锁阶段，直接返回实际
        if(instance == null) {
            synchronized (Singleton.class) {
                //抢到锁之后再次判断是否为空
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

# 2. 线程

https://www.processon.com/view/link/61cbdbad1e08532c3ee65aee#map

## 2.1 创建

### 2.1.1 2种方式
[3 1 创建和运行线程](java多线程.md#3%201%20创建和运行线程)
方式一：继承Thread
```java
public class MyThread extends Thread {  
    @Override  
    public void run() {  
        System.out.println("extend Thread");  
        System.out.println(Thread.currentThread().getName());  
    }  
}
```

方式二：实现Runnable接口，然后配合Thead使用
```java
public class MyRunnable implements Runnable{  
    @Override  
    public void run() {  
        System.out.println("implements Runnable");  
        System.out.println(Thread.currentThread().getName());  
    }  
}
```

| 区别         | 继承Thread               | 实现Runnable     |
| ------------ | ------------------------ | ---------------- |
| 代码架构     |                          | 解耦了任务和线程 |
| 新建线程消耗 | 新的线程需要新建，消耗大 | 复用旧线程       |
| 双继承       | 无法再继承，扩展性差     | 可以再继承       |
| 底层源码     | 重写了run方法            | run方法调用了target方法，也就是调用了Runnable方法                 |

当同时继承Thread方法和实现Runnable接口时，继承会覆盖run方法，而无法调用Runnable方法。只会执行继承的方法。
```java
// 最后输出extend Thread
// 采用了匿名内部类方式
new Thread(new Runnable() {  
    @Override    public void run() {  
        System.out.println("implements Runnable");  
    }  
}) {  
    @Override    public void run() {  
        System.out.println("extend Thread");  
    }  
}.start();
```

lambda表达式：`new Thread( ()-> System.out.println("use lambda") ).start();`

### 2.1.2 start与run
首先start方法启动子线程，而run启动主线程
start调用了start0本地方法，但是run只是调用了普通的方法

```java
public synchronized void start() {  
	// 对每个子线程都先判断状态，如果已运行Running状态则失败
	if (threadStatus != 0)  
        throw new IllegalThreadStateException();  
	// 加组
	group.add(this);  
  
    boolean started = false;  
    try {  
    // 运行本地方法，启动子线程
        start0();  
        started = true;  
    } 
	......
}  
  // 本地方法start0
private native void start0();
```

由于子线程启动前会对检查当前状态，如果不符合就会抛出异常
那么以下代码启动两次线程就会报错。
```java
Thread thread = new Thread();  
thread.start();  
thread.start();
```

## 2.2 中断

`interrupt()`：会设置线程的打断标志为true，如果该线程调用了`wait,join,sleep`方法的话就会抛出异常。
`isInterrupted()`：返回线程打断标志true or false，并且不会清除打断标志，当前咋样就还是咋样
`interrupted`：返回线程打断标志true or false，但是会清除打断标志，不管之前咋样，后面要改成false

最佳实践一：对中断异常的方法异常抛出方法外
```java
 public class MyInterrupt implements Runnable {  
    @Override  
    public void run() {  
	    // 主动判断打断标志，如果出现true则中断线程
        while(true && !Thread.currentThread().isInterrupted()){  
            System.out.println("I am fine");  
            try {  
	            // 接收方法的错误
                doSomeThing();  
            } catch (InterruptedException e) {  
	            // 由于sleep时打断标志被清除，所以需要再设置打断标志true
                Thread.currentThread().interrupt();  
                System.out.println("save exception log");  
                e.printStackTrace();  
            }  
        }  
    }  
	// 错误往上抛
    private void doSomeThing() throws InterruptedException {  
        Thread.sleep(5000);  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        Thread thread = new Thread(new MyInterrupt());  
        thread.start();  
        Thread.sleep(1000);  
        thread.interrupt();  
    }  
}
```


最佳实践二：方法内处理异常并设置打断异常
```java
public class MyInterrupt2 implements Runnable {  
    @Override  
    public void run() {  
        while(true) {  
	        // 当线程被打断时设置如何处理
            if (Thread.currentThread().isInterrupted()) {  
                System.out.println("interrupted...");  
                break;            }  
            doSomeThing();  
        }  
    }  
  
    private void doSomeThing() {  
        try {  
            Thread.sleep(5000);  
        } catch (InterruptedException e) {  
        // 错误内部处理，并且设置已经被清除的打断标志为true
            Thread.currentThread().interrupt();  
            System.out.println("save execption log");  
            e.printStackTrace();  
        }  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        Thread thread = new Thread(new MyInterrupt2());  
        thread.start();  
        Thread.sleep(2000);  
        thread.interrupt();  
    }  
}
```


响应中断方法
```java
Object. wait()/ wait(long)/ wait( long, int)
Thread.sleep( long)/sleep( long, int)
Thread.join)/join( long)/ join( long, int)
java.util.concurrent. BlockingQueue. take()/put(E)
java.util. concurrent. locks. Lock. lockInterruptibly()

java.util concurrent. CountDownLatch.await()
java.util.concurrent. CyclicBarrier.await()
java.util. concurrent.Exchanger.exchange()
java.nio.channels.InterruptibleChannel相关方法
java.nio.channels.Selector的相关方法
```



面试题：
如何停止线程
原理：采用interrupt来设置打断标记true，然后在线程中判断打断标记，如果被打断采取一定设置，比如在sleep时会出现异常，可以日志采集。sleep异常后会重置打断标记为false。
错误的方法有：stop、suspend，以及通过volatile的boolean会出现无法处理长时间阻塞线程。

如何处理不可中断线程
针对不同情况，比如ReentryLock里面有对应的中断方法。

## 2.2 状态
Java线程状态变迁
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/202205082227544.png)
## 2.3 基本使用

### 2.3.1 wait/notify
[3 3 Thread的常见方法](java多线程.md#3%203%20Thread的常见方法)

`Object`：wait,notify

```java
// 声明一个对象
public static Object lock = new Object();  
public static void main(String[] args) {  
    new Thread(new Runnable() {  
        @Override        
        public void run() {  
	        // wait/notify需要在同步代码块中，保证代码块完整性。
            synchronized (lock) {  
                try {  
                    System.out.println("waiting.."); 
                    // Object方法wait，释放锁
                    lock.wait();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
                System.out.println("get lock");  
            }  
        }    }).start();  
  
    new Thread(new Runnable() {  
        @Override        
        public void run() {  
            synchronized(lock){  
            // 唤醒wait
                lock.notify();  
                System.out.println("notify");  
            }  
        }    }).start();  
}
```


notifyall唤醒所有wait线程
```java  
/**  
 * 描述： 3个线程，线程1和线程2首先被阻塞，线程3唤醒它们。notify, notifyAll。 start先执行不代表线程先启动。 
 * 结果： threadA和threadB都会被唤醒，然后都打印 waiting to end
 * 如果使用notify的话，只有一个线程被唤醒，还有一个WAITING状态
 */  
public class WaitNotifyAll implements Runnable {  
  
    private static final Object resourceA = new Object();  
  
    public static void main(String[] args) throws InterruptedException {  
        Runnable r = new WaitNotifyAll();  
        Thread threadA = new Thread(r);  
        Thread threadB = new Thread(r);  
        Thread threadC = new Thread(new Runnable() {  
            @Override            
            public void run() {  
                synchronized (resourceA) {  
                    resourceA.notifyAll();  
//                    resourceA.notify();  
                    System.out.println("ThreadC notified.");  
                }  
            }        });  
        threadA.start();  
        threadB.start();  
        Thread.sleep(200);  
        threadC.start();  
    }  
    @Override  
    public void run() {  
        synchronized (resourceA) {  
            System.out.println(Thread.currentThread().getName()+" got resourceA lock.");  
            try {  
                System.out.println(Thread.currentThread().getName()+" waits to start.");  
                resourceA.wait();  
                System.out.println(Thread.currentThread().getName()+"'s waiting to end.");  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}
```


调用wait后底层原理：进入对象monitor的等待列表
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205112016728.png)


wait/notify 与 sleep异同
相同点：
- 阻塞，都会释放CPU
- 响应中断

不同点：
- sleep不会释放锁，wait会释放锁
- wait/notify在同步代码块中存在，保证逻辑一致性
- sleep必须指定时间
- 所属的类不同：Object.wait(), Thread.sleep()




### 2.3.2 交替打印
两个线程交替打印至100
示范1：
分析：线程A先打印了0，然后就wait等待释放了锁；线程B此时拿到了锁，进行唤醒并打印了1
```java
// 结果：打印了0 1然后锁住了
synchronized (object) {  
    while (count <= 100) {  
        object.notify();  
        System.out.println(Thread.currentThread().getName()+" : "+count++);  
        if (count <= 100) {  
            try {  
                object.wait();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}
```

示范2：
```java
public class OddEven {  
    private static int count = 0;  
    private static final Object object = new Object();  
  
    public static void main(String[] args) {  
        myRunable r = new myRunable();  
        Thread t1 = new Thread(r);  
        Thread t2 = new Thread(r);  
        t1.start();  
        t2.start();  
    }  
  
    static class myRunable implements Runnable {  
  
        @Override  
        public void run() {  
            while (count <= 100) {  
                synchronized (object) {  
                    object.notify();  
                    System.out.println(Thread.currentThread().getName()+" : "+count++);  
                    if (count <= 100) {  
                        try {  
                            object.wait();  
                        } catch (InterruptedException e) {  
                            e.printStackTrace();  
                        }  
                    }  
                }  
            }  
        }  
    }  
  
  
}
```


### 2.3.3 生产者消费者模型
```java
package threadcoreknowledge.My;  
  
import java.util.Date;  
import java.util.LinkedList;  
  
public class ProducerConsumer {  
    public static void main(String[] args) {  
        Storage storage = new Storage();  
        Thread producer = new Thread(new Producer(storage));  
        Thread consumer = new Thread(new Consumer(storage));  
        producer.start();  
        consumer.start();  
    }  
}  
  
class Producer implements Runnable {  
  
    Storage storage;  
    public Producer(Storage storage) {  
        this.storage = storage;  
    }  
  
    @Override  
    public void run() {  
        for (int i = 0; i < 100; i++) {  
            storage.put();  
        }  
    }  
}  
  
class Consumer implements Runnable {  
  
    Storage storage;  
    public Consumer(Storage storage) {  
        this.storage = storage;  
    }  
  
    @Override  
    public void run() {  
        for (int i = 0; i < 100; i++) {  
            storage.get();  
        }  
    }  
}  
  
class Storage {  
    private int maxSize;  
    private LinkedList<Date> list;  
    public Storage() {  
        maxSize = 10;  
        list = new LinkedList<>();  
    }  
  
    public synchronized void put () {  
        while(list.size() == maxSize) {  
            try {  
                wait();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
        list.add(new Date());  
        System.out.println("storage " + list.size());  
        notify();  
    }  
  
    public synchronized void get() {  
        while (list.size() == 0) {  
            try {  
                wait();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
        list.removeLast();  
        System.out.println("storage" + list.size());  
        notify();  
    }  
}
```


### 2.3.4 join
join：主线程等子线程运行完后结束
也就是主 等 子
底层源码：当子线程还存活 且 还有剩余时间时，会调用wait方法等待子线程。
可以加时间方法`join(long millis)`，不加时间默认为`join(0)`
```java
// 结果：先输出：子线程运行，然后等三秒后，输出：主线程等子线程运行完后结束
// 如果不加join：直接输出：子线程运行 主线程等子线程运行完后结束, 然后等三秒后进程结束
public static void main(String[] args) throws InterruptedException {  
    Thread thread = new Thread(new Runnable() {  
        @Override        public void run() {  
            try {  
                Thread.sleep(3000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }    });  
    thread.start();  
    System.out.println("子线程运行");  
    thread.join();  
    System.out.println("主线程等子线程运行完后结束");  
}
```


### 2.3.5 sleep
`Thread.sleep(3000)`使程序释放CPU，但是不会释放锁

使用`TimeUnit.HOURS.sleep(3);`方式比使用`sleep(3000)`更加直观
比如当要求等待时间3小时3分钟时
```java
TimeUnit.HOURS.sleep(3);  
TimeUnit.MINUTES.sleep(3);
```
而且当出现负数时不会报错，会跳过什么都不做。
但是sleep会直接报错。

## 2.4 属性
|属性名称 | 途用|
|-- | --|
|编号（ID）|每个线程有自己的ID，用于标识不同的线程|
|名称（Name）| 作用让用户或程序员在开发、调试或运行过程中更容易区分每个不同的线程、定位问题等。|
|是否是守护线程(isDaemon ) |true代表该线程是【守护线程】，false代表线程是非守护线程，也就是【用户线程】。|
|优先级（Priority ）| 优先级这个属性的目的是告诉线程调度器，用户希望哪些线程相对多运行、哪些少运行。|

守护线程和普通线程区别：
守护线程结束了不影响JVM退出，但用户线程不结束JVM不会正常结束

程序不应该依赖于优先级：
不同操作系统优先级映射不一样，但是java一次编译多平台运行，所以对应不同平台后优先级都不同，Linux会忽略，Windows默认7个优先级所以会对应。
但是优先级较低可能会出现饿死现象，所以避免使用优先级，而采用默认方式。


## 2.5 异常
为什么需要`UncaughtExceptionHandler`
- 主线程可以捕获异常并程序报错，子线程会打印出错误信息，不会终止程序，无法直观显示
- 子线程异常无法通过常规try/catch捕获
```java
// 对于这两个线程都有异常，常规会出现线程1 报错后抛出Caught Exception，但是线程12都报错且没有抛出Caught Exception
try {  
    new Thread(new CantCatchDirectly(), "MyThread-1").start();  
    Thread.sleep(300);  
    new Thread(new CantCatchDirectly(), "MyThread-2").start();  
} catch (RuntimeException e) {  
    System.out.println("Caught Exception.");  
}
```

具体使用策略：
- 程序统一设置
- 给每个子线程单独设置
- 线程池设置

```java
// 1. 先定义处理异常处理器实现UncaughtExceptionHandler
public class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {  
  
    private String name;  
  
    public MyUncaughtExceptionHandler(String name) {  
        this.name = name;  
    }  
  
    @Override  
    public void uncaughtException(Thread t, Throwable e) {  
        Logger logger = Logger.getAnonymousLogger();  
        logger.log(Level.WARNING, "线程异常，终止啦" + t.getName());  
        System.out.println(name + "捕获了异常" + t.getName() + "异常");  
    }  
}
```

```java
// 2. 然后在使用子线程前先设置异常处理器为刚才定义的即可
Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler("捕获器1"));
```


## 2.6 死锁

### 2.6.1 死锁检测
通过`ThreadMXBeanDetection`检查并在程序中显示。
```java
public static void main(String[] args) throws InterruptedException {  
    ThreadMXBeanDetection r1 = new ThreadMXBeanDetection();  
    ThreadMXBeanDetection r2 = new ThreadMXBeanDetection();  
    r1.flag = 1;  
    r2.flag = 0;  
    Thread t1 = new Thread(r1);  
    Thread t2 = new Thread(r2);  
    t1.start();  
    t2.start();  
    Thread.sleep(1000);  
    ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();  
    long[] deadlockedThreads = threadMXBean.findDeadlockedThreads();  
    if (deadlockedThreads != null && deadlockedThreads.length > 0) {  
        for (int i = 0; i < deadlockedThreads.length; i++) {  
            ThreadInfo threadInfo = threadMXBean.getThreadInfo(deadlockedThreads[i]);  
            System.out.println("发现死锁" + threadInfo.getThreadName());  
        }  
    }  
}
```

### 2.6.2 死锁避免
转账时避免死锁 -- 不在乎多个锁获取的顺序
但是根据锁的hash值排序来确定获取顺序，如果冲突再加一把锁hash后来判断
或者通过主键来确定大小





# 3. 锁
## 3.1 Lock
```java
	Lock lock = new ReentrantLock();
	lock.lock();
	try {
	
	} finally {
		lock.unlock();
	}
```


Lock接口提供的synchronized关键字不具备的主要特性
| 特性描述           | 方法名称描述                                                               |
| ------------------ | -------------------------------------------------------------------------- |
| 尝试非阻塞地获取锁 | 当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁 |
| 能被中断地获取锁   |       与 synchronized 不同，获取到锁的线程能够响应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放                                                                     |
| 超时获取锁                   |   在指定的截止时间之前获取锁，如果截止时间到了仍旧无法获取锁，则返回                                                                         |

`Lock API`
|  方法名称                                                                                                                      |  描述                                                                                                                                                                                  |
|:---------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  void lock()                                                                                                               |  获取锁，调用该方法当前线程将会获取锁，当锁获得后，从该方法返回                                                                                                                                                     |
|  void lockInterruptibly() throws InterruptedException                                                                      | 可中断地获取锁，和lockO方法的不同之处在于该方法会响应中断，即在锁的获取中可以中断当前线程                                                                                                                                      |
| boolean tryLock()                                                                                                          | 尝试非阻塞的获取锁，调用该方法后立刻返回，如果能够获取则返回 true，否则返回 false                                                                                                                                       |
| boolean tryLock(longtime. TimeUnit unit) throws InterruptedException                                                       | <div>超时的获取锁，当前线程在以下3种情况下会返回∶</div><div>①当前线程在超时时间内获得了锁</div><div>②当前线程在超时时间内被中断</div><div>③超时时间结束，返回 false 释放锁</div>                                                                 |
| void unlock(）                                                                                                              |                                                                 <div>释放锁</div>                                                                                                       |
| Condition newCondition()                                                                                                   | 获取等待通知组件，该组件和当前的锁绑定，当前线程只有获得了锁，才能调用该组件的 wait（） 方法，而调用后，当前线程将释放锁                                                                                                                      |  



## 3.2 AQS
队列同步器`AbstractQueuedSynchronizer`
用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作

锁是面向使用者的，它定义了使用者与锁交互的接口（比如可以允许两个线程并行访问），隐藏了实现细节；

同步器面向的是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

### 3.2.1 API
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092159241.png)

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092159004.png)

同步器提供的模板方法基本上分为3类：
独占式获取与释放同步状态
共享式获取与释放同步状态
查询同步队列中的等待线程情况

### 3.2.2 实现分析

同步器依赖内部的同步队列（一个<mark style="background: #FF5582A6;">FIFO双向队列</mark> ）来完成同步状态的管理
当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成为一个<mark style="background: #FF5582A6;">节点（Node</mark> ）并将其加入同步队列
同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092208385.png)



![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092202704.png)

## 3.3 重入锁ReentrantLock
表示该锁能够支持一个线程对资源的重复加锁。
该锁的还支持获取锁时的公平和非公平性选择。
默认非公平锁：`Lock lock = new ReentrantLock();`

公平锁：`Lock lock = new ReentrantLock(true);`

公平锁：锁的获取顺序就应该符合请求的绝对时间顺序，也就是FIFO
非公平锁：线程来了就尝试CAS抢占，如果成功了就拿到锁，与时间顺序无关

获取锁后必须要释放
```java
lock.lock();  
try{  
    //获取本锁保护的资源  
    System.out.println(Thread.currentThread().getName()+"开始执行任务");  
}finally {  
	// 释放该锁
    lock.unlock();  
}
```


`tryLock(3000,TimeUnit.MILLISECONDS )`锁重试
在一定时间内如果拿不到锁，就放弃该锁
```java
if(lock.tryLock(3000, TimeUnit.MILLISECONDS)){  
    try {  
          ...
    }finally {  
        lock.unlock();  
    }  
}
```






## 3.4 读写锁
ReentrantReadWriteLock

## 3.5 Condition
不同Condition下有自己的队列
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092227780.png)


![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092230924.png)




# 4. 并发容器和框架

## 4.1 Concurrent包

### 4.1.1 ConcurrentHashMap
HashMap：
jdk1.7时，多线程环境下，使用HashMap进行put操作出现扩容时，由于头结点插入，会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。但是1.8后采用尾插法解决这个问题。

HashTable
HashTable容器使用synchronized来保证线程安全，对整张表加锁，效率低

ConcurrentHashMap：
所使用的锁分段技术。首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问

由Segment数组结构和HashEntry数组结构组成
Segment是一种可重入锁（ReentrantLock）
Segment的结构和HashMap类似，是一种数组和链表结构。
一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得与它对应的Segment锁。
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205090902936.png)


get操作
get操作的高效之处在于整个get过程不需要加锁，除非读到的值是空才会加锁重读。
原因是它的get方法里将要使用的共享变量都定义成volatile类型
这是用volatile替换锁的经典应用场景

put操作
put方法首先定位到Segment，然后在Segment里进行插入操作
第一步判断是否需要对Segment里的HashEntry数组进行扩容，第二步定位添加元素的位置，然后将其放在HashEntry数组里。
1. 在插入元素前会先<mark style="background: #BBFABBA6;">判断Segment里的HashEntry数组是否超过容量（threshold）</mark> ，如果超过阈值，则对数组进行扩容
2. 为了高效，ConcurrentHashMap不会对整个容器进行扩容，而<mark style="background: #BBFABBA6;">只对某个segment进行扩容</mark> 。先创建两倍原容量的数组，然后将原数组元素散列放入。

size操作
每个segment中都含有count统计本节点大小，还有个modCount来表示每次put、remove、clean进行与否。
统计整表的size时，先尝试2次不锁全表统计累加count值，如果modCount产生变化的话就锁表后统计。





### 4.1.2 ConcurrentLinkedQueue

基于链接节点的无界线程安全且非阻塞队列

入队时：先找尾节点，使用CAS将入队节点设为尾节点，每次插入前判断当前尾节点的next是否为null，为null表明为尾节点。


## 4.2 阻塞队列
阻塞队列有两个附加条件
- 支持阻塞的插入方法：意思是当队列满时，队列会阻塞插入元素的线程，直到队列不满。
- 支持阻塞的移除方法：意思是在队列为空时，获取元素的线程会等待队列变为非空。

阻塞队列常用于生产者和消费者的场景
| 方法 | 抛出异常  | 返回特殊值 | 一直阻塞 | 超时退出             |
| ---- | --------- | ---------- | -------- | -------------------- |
| 插入 | add(e)    | offer(e)   | put(e)   | offer(e, time, unit) |
| 移除 | remove()  | poll()     | take()   | poll(time, unit)     |
| 检查 | element() | peek()     | /        | /                     |
put和take分别尾首含有字母t
offer和poll都含有字母o。

- ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。
- LinkedBlockingQueue：一个由链表结构组成的无界阻塞队列。
- PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。
- DelayQueue：一个使用优先级队列实现的无界阻塞队列。
- SynchronousQueue：一个不存储元素的阻塞队列。
- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

DelayQueue非常有用，可以将DelayQueue运用在以下应用场景。
- 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。
- 定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从- DelayQueue中获取到任务就开始执行，比如TimerQueue就是使用DelayQueue实现的

SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。
SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合传递性场景。SynchronousQueue的吞吐量高于LinkedBlockingQueue和ArrayBlockingQueue。



## 4.3 Fork/Join框架
Fork/Join框架是Java 7提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205091018841.png)

工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行
通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行
优点：充分利用线程进行并行计算，减少了线程间的竞争。
缺点：在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且该算法会消耗了更多的系统资源，比如创建多个线程和多个双端队列。

# 5. 原子操作类
基本类型
- AtomicBoolean：原子更新布尔类型。
- AtomicInteger：原子更新整型。
- AtomicLong：原子更新长整型。

数组类型
- AtomicIntegerArray：原子更新整型数组里的元素。
- AtomicLongArray：原子更新长整型数组里的元素。
- AtomicReferenceArray：原子更新引用类型数组里的元素。

引用类型
- AtomicReference：原子更新引用类型。
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段。
- AtomicMarkableReference：原子更新带有标记位的引用类型。

类字段
- AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。
- AtomicLongFieldUpdater：原子更新长整型字段的更新器。
- AtomicStampedReference：原子更新带有版本号的引用类型。 


# 6. 线程池

## 6.1 原理

优点：
1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
3. 提高线程的可管理性。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。但是，要做到合理利用线程池，必须对其实现原理了如指掌

线程池处理流程：
1. 线程池判断<mark style="background: #FF5582A6;">核心线程池</mark> 里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。
2. 线程池判断<mark style="background: #FF5582A6;">工作队列</mark> 是否已经满。如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。
3. 线程池判断<mark style="background: #FF5582A6;">线程池的线程</mark> 是否都处于工作状态。如果没有，则创建一个新的工作线程来执行任务。
4. 如果已经满了，则交给<mark style="background: #FF5582A6;">饱和策略</mark> 来处理这个任务
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205091046958.png)


`ThreadPoolExecutor`执行`execute方法`分下面4种情况。
1. 如果当前运行的线程少于`corePoolSize`，则创建新线程来执行任务（注意，执行这一步骤需要获取全局锁）。
2. 如果运行的线程等于或多于corePoolSize，则将任务加入`BlockingQueue`。
3. 如果无法将任务加入BlockingQueue（队列已满），则创建新的线程来处理任务（<mark style="background: #FF5582A6;">注意，执行这一步骤需要获取全局锁</mark> ）。
4. 如果创建新线程将使当前运行的线程超出maximumPoolSize，任务将被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法。
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205091049463.png)


## 6.2 使用

### 6.2.1 创建
```java
public ThreadPoolExecutor(int corePoolSize,  
                          int maximumPoolSize,  
                          long keepAliveTime,  
                          TimeUnit unit,  
                          BlockingQueue<Runnable> workQueue,  
                          ThreadFactory threadFactory,  
                          RejectedExecutionHandler handler)
```

1. corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。
2. maximumPoolSize（线程池最大数量）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如果使用了无界的任务队列这个参数就没什么效果
3. keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率
4. BlockingQueue[4 2 阻塞队列](#4%202%20阻塞队列)
5. RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。在JDK 1.5中Java线程池框架提供了以下4种策略。
		1. AbortPolicy：直接抛出异常。
		2. CallerRunsPolicy：只用调用者所在线程来运行任务。
		3. DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
		4. DiscardPolicy：不处理，丢弃掉
6. TimeUnit（线程活动保持时间的单位）


### 6.2.2 执行

execute()方法用于提交不需要返回值的任务

submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象


### 6.2.3 关闭

shutdown或shutdownNow方法来关闭线程池。
它们的原理是遍历线程池中的工作线程，然后逐个调用线程的<mark style="background: #FF5582A6;">interrupt方法来中断线程</mark> ，所以无法响应中断的任务可能永远无法终止。
shutdownNow首先将线程池的状态设置成STOP，然后尝试<mark style="background: #FF5582A6;">停止所有的正在执行或暂停任务的线程</mark> ，并返回等待执行任务的列表
shutdown只是将线程池的状态设置成SHUTDOWN状态，然后<mark style="background: #FF5582A6;">中断所有没有正在执行任务的线程</mark> 。


### 6.2.4 合理配置

分析角度：
- 任务的性质：CPU密集型任务、IO密集型任务和混合型任务。
	- CPU密集型任务应配置尽可能小的线程，如配置N cpu +1个线程的线程池。
	- 由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如2N cpu 。
	- 通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数
- 任务的优先级：高、中和低。
	- 使用优先级队列PriorityBlockingQueue来处理
- 任务的执行时间：长、中和短。
- 任务的依赖性：是否依赖其他系统资源，如数据库连接。
	- 依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，等待的时间越长，则CPU空闲时间就越长，那么线程数应该设置得越大，这样才能更好地利用CPU。

建议使用有界队列。有界队列能增加系统的稳定性和预警能力

### 6.2.6 监控
- taskCount：线程池需要执行的任务数量。
- completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。
- largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过。
- getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。
- getActiveCount：获取活动的线程数

## 6.3 状态

- `RUNNING`∶接受新任务并处理排队任务
- `SHUTDOWN`∶不接受新任务，但处理排队任务
- `STOP`∶不接受新任务，也不处理排队任务，并中断正在进行的任务
- `TIDYING`，中文是整洁，理解了中文就容易理解这个状态了所有任务都已终止，workerCount为零时，线程会转换TIDYING状态，并将运行terminate（）钩子方法。
- `TERMINATED`∶terminate（）运行完成







# 7. Executor

>【强制】线程池<mark style="background: #FF5582A6;">不允许使用 Executors 去创建</mark> ，<mark style="background: #FF5582A6;">而是通过 ThreadPoolExecutor 的方式</mark> ，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
说明：Executors 返回的线程池对象的弊端如下：
1） FixedThreadPool 和 SingleThreadPool：
允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
2） CachedThreadPool：
允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

## 7.1 组成

Java的线程既是工作单元，也是执行机制。从JDK 5开始，把工作单元与执行机制分离开来。工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

组成部分
- <mark style="background: #BBFABBA6;">任务</mark> ：包括被执行任务需要实现的接口：Runnable接口或Callable接口。
- <mark style="background: #BBFABBA6;">任务的执行</mark> ：包括任务执行机制的核心接口Executor，以及继承自Executor的ExecutorService接口。Executor框架有两个关键类实现了ExecutorService接口（ThreadPoolExecutor和ScheduledThreadPoolExecutor）。
- <mark style="background: #BBFABBA6;">异步计算的结果</mark> ：包括接口Future和实现Future接口的FutureTask类。

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205091951898.png)

通过`Executor`框架可以创建出一下三种类型线程池

- FixedThreadPool适用于为了满足资源管理的需求，而需要限制当前线程数量的应用场景，它适用于负载比较重的服务器
```java
public static ExecutorService newFixedThreadPool(int nThreads) {  
    return new ThreadPoolExecutor(nThreads, nThreads,  
                                  0L, TimeUnit.MILLISECONDS,  
                                  new LinkedBlockingQueue<Runnable>());  
}
```

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092025672.png)

1）如果当前运行的线程数少于corePoolSize，则创建新线程来执行任务。
2）在线程池完成预热之后（当前运行的线程数等于corePoolSize），将任务加入LinkedBlockingQueue。
3）线程执行完1中的任务后，会在循环中反复从LinkedBlockingQueue获取任务来执行。

FixedThreadPool使用无界队列LinkedBlockingQueue作为线程池的工作队列（队列的容量为Integer.MAX_VALUE）。使用无界队列作为工作队列会对线程池带来如下影响。
1）当线程池中的线程数达到corePoolSize后，新任务将在无界队列中等待，因此线程池中的线程数不会超过corePoolSize。
2）由于1，使用无界队列时maximumPoolSize将是一个无效参数。
3）由于1和2，使用无界队列时keepAliveTime将是一个无效参数。
4）由于使用无界队列，运行中的FixedThreadPool（未执行方法shutdown()或shutdownNow()）不会拒绝任务（不会调用RejectedExecutionHandler.rejectedExecution方法）。


- SingleThreadExecutor适用于需要保证顺序地执行各个任务；并且在任意时间点，不会有多个线程是活动的应用场景
```java
public static ExecutorService newSingleThreadExecutor() {  
    return new FinalizableDelegatedExecutorService  
        (new ThreadPoolExecutor(1, 1,  
                                0L, TimeUnit.MILLISECONDS,  
                                new LinkedBlockingQueue<Runnable>()));  
}
```
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092026537.png)
1. 如果当前运行的线程数少于corePoolSize（即线程池中无运行的线程），则创建一个新线程来执行任务。
2. 在线程池完成预热之后（当前线程池中有一个运行的线程），将任务加入Linked-BlockingQueue。
3. 线程执行完1中的任务后，会在一个无限循环中反复从LinkedBlockingQueue获取任务来执行。


- CachedThreadPool是大小无界的线程池，适用于执行很多的短期异步任务的小程序，或者是负载较轻的服务器
```java
public static ExecutorService newCachedThreadPool() {  
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,  
                                  60L, TimeUnit.SECONDS,  
                                  new SynchronousQueue<Runnable>());  
}
```
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092026432.png)

CachedThreadPool的corePoolSize被设置为0，即corePool为空；
maximumPoolSize被设置为Integer.MAX_VALUE，即maximumPool是无界的。
keepAliveTime设置为60L，意味着CachedThreadPool中的空闲线程等待新任务的最长时间为60秒，空闲线程超过60秒后将会被终止。

| parameter       | newFixedThreadPool | newCachedThreadPool | newSingleThreadExecutor | ScheduledThreadPoolExecutor |
| --------------- | ------------------ | ------------------- | ----------------------- | --------------------------- |
| corePoolSize    | n                  | 0                   | 1                       | n                           |
| maximumPoolSize | n                  |Integer.MAX_VALUE    |    1                     | Integer.MAX_VALUE           |
| keepAliveTime   | 0                  | 60                    |     0                    | 0                           |






## 7.2 ThreadPoolExecutor


## 7.3 ScheduledThreadPoolExecutor
- ScheduledThreadPoolExecutor。包含若干个线程的ScheduledThreadPoolExecutor。适用于需要多个后台线程执行周期任务，同时为了满足资源管理的需求而需要限制后台线程的数量的应用场景
- SingleThreadScheduledExecutor。只包含一个线程的ScheduledThreadPoolExecutor。适用于需要单个后台线程执行周期任务，同时需要保证顺序地执行各个任务的应用场景。

DelayQueue是一个无界队列，所以ThreadPoolExecutor的maximumPoolSize在Scheduled-ThreadPoolExecutor中没有什么意义（设置maximumPoolSize的大小没有什么效果，所以默认为最大）。
```java
public ScheduledThreadPoolExecutor(int corePoolSize,  
                                   ThreadFactory threadFactory,  
                                   RejectedExecutionHandler handler) {  
    super(corePoolSize, Integer.MAX_VALUE,  
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,  
          new DelayedWorkQueue(), threadFactory, handler);  
}
```
ScheduledThreadPoolExecutor的执行主要分为两大部分。
1. 当调用ScheduledThreadPoolExecutor的scheduleAtFixedRate()方法或者scheduleWith-FixedDelay()方法时，会向ScheduledThreadPoolExecutor的DelayQueue添加一个实现了RunnableScheduledFutur接口的ScheduledFutureTask。
2. 线程池中的线程从DelayQueue中获取ScheduledFutureTask，然后执行任务
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092115204.png)

具体执行过程：
1. 线程1从DelayQueue中获取已到期的ScheduledFutureTask（DelayQueue.take()）。到期任务是指ScheduledFutureTask的time大于等于当前时间。
2. 线程1执行这个ScheduledFutureTask。
3. 线程1修改ScheduledFutureTask的time变量为下次将要被执行的时间。
4. 线程1把这个修改time之后的ScheduledFutureTask放回DelayQueue中（Delay-Queue.add()）。
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092121289.png)




## 7.4 Future

### 7.4.1 Future与Runnable区别
Runnable缺点：
- 没有返回值
- 无法上抛异常

而Future就改变了这两点。

Future方法
| Modifier and Type | Method and Description |
| ----------------- | --------------------------------|
| `boolean`         | `cancel(boolean mayInterruptIfRunning)`  试图取消执行这一任务。 |
| `V`               | `get()`  如果有必要等待计算完成,然后获取它的结果。           |
| `V`               | `get(long timeout,  TimeUnit unit)`  如果有必要等待最多计算给定的时间完成,然后获取它的结果,如果可用。 |
| `boolean`         | `isCancelled()`  返回 `true`如果这个任务被取消之前完成正常。 |
| `boolean`         | `isDone()`  返回 `true`如果这个任务完成                      |

### 7.4.2 FutureTask

FutureTask除了实现Future接口外，还实现了Runnable接口。因此，FutureTask可以交给Executor执行，也可以由调用线程直接执行（FutureTask.run()）

使用Cancel方法在不同状态下结果。
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092132375.png)

当一个线程需要等待另一个线程把某个任务执行完后它才能继续执行，此时可以使用FutureTask。

FutureTask的实现基于AbstractQueuedSynchronizer（以下简称为AQS）
基于AQS实现的同步器包括：ReentrantLock、Semaphore、ReentrantReadWriteLock、CountDownLatch和FutureTask


`Future`使用方式：
方式一：通过线程池执行。
方式二：先提交给FutureTask，让它去执行，执行完后从它那拿结果。
```java
public class MyFuture {  
  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        // 1. 线程池执行，通过submit返回结果  
        // 1.1 正常函数  
        LinkedBlockingQueue<Runnable> runnables = new LinkedBlockingQueue<>();  
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 4, 60L, TimeUnit.SECONDS, runnables);  
        Future<Integer> future1 = executor.submit(new myCall());  
        Integer integer1 = future1.get();  
        System.out.println(integer1);  
  
        // 1.2 匿名函数方式  
        Future<Integer> future2 = executor.submit(new Callable<Integer>() {  
            @Override  
            public Integer call() throws Exception {  
                System.out.println("use anonymous call");  
                return 20;  
            }  
        });  
        Integer integer2 = future2.get();  
        System.out.println(integer2);  
  
        // 1.3 use lamada  
        Callable<Integer> call = () -> {  
            System.out.println("use lamada");  
            return 30;  
        };  
        Future<Integer> future3 = executor.submit(() -> {  
            System.out.println("use lamada");  
            return 30;  
        });  
        System.out.println(future3.get());  
  
  
        // 2. FutureTask执行  
        FutureTask<Integer> futureTask = new FutureTask<>(new myCall());  
        // 也可以将futureTask提交给线程池执行或者建立个线程执行
//        executor.submit(futureTask);  
        new Thread(futureTask).start();  
        Integer integer = futureTask.get();  
        System.out.println("FutureTask " + integer);  
  
        executor.shutdown();  
    }  
  
    public static class myCall implements Callable<Integer> {  
  
        @Override  
        public Integer call() throws Exception {  
            System.out.println("use myCall");  
            return 10;  
        }  
    }  
}
```





# 8.ThreadLocal
每个线程内部独享的变量，存储在线程内存空间，
## 8.1 基本使用
| Modifier and Type           | Method and Description                 |
| --------------------------- | ---------------------------------------|
| `T`                         | `get()`  返回的值在这个线程局部变量的当前线程副本。          |
| `protected T`               | `initialValue()`  返回当前线程的初始值为这个线程局部变量。   |
| `void`                      | `remove()`  删除这个线程局部变量的当前线程的值。             |
| `void`                      | `set(T value)`  设置这个线程局部变量的当前线程的复制设置为指定的值。 |
| `static <S> ThreadLocal<S>` | `withInitial(Supplier<? extends  S> supplier)`  创建一个线程局部变量 |

`initialValue`方法只会调用一次，当首次使用`get`方法时才使用。

如果`remove`了之后就可以再次通过`set`来设置

两大典型场景：
1. 每个线程需要独享对象（通常工具类：SimpleDateFormat和Random）
2. 每个线程要保存全局变量，供线程内多个方法直接调用，避免参数传递麻烦

1. initialValue在第一个get时初始化，对象初始化时机有我们控制
2. set方式：通过拦截器生成用户信息传递


场景一：
```java
public class MyThreadLocal {  
    public static ExecutorService threadPool =  Executors.newFixedThreadPool(10);  
  
    public static void main(String[] args) {  
        for (int i = 0; i < 1000; i++) {  
            int innerI = i;  
            threadPool.execute(new Runnable() {  
                @Override  
                public void run() {  
                    String ss = new MyThreadLocal().date(innerI);  
                    System.out.println(ss);  
                }  
            });  
        }  
        threadPool.shutdown();  
    }  
  
    public String date(int s) {  
        Date date = new Date(1000 * s);  
        // 原先本地定义工具类，每个线程来了都创建一个工具类使用
//        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        SimpleDateFormat simpleDateFormat = SafeDate.safeDate.get();  
        String format = simpleDateFormat.format(date);  
        return format;  
    }  
}  
  
class SafeDate {  
    public static ThreadLocal<SimpleDateFormat> safeDate = ThreadLocal.withInitial(  
            ()-> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));  
}
```

场景二：演示ThreadLocal用法2：避免传递参数的麻烦
```java
public class ThreadLocalNormalUsage06 {  
  
    public static void main(String[] args) {  
        new Service1().process("");  
  
    }  
}  

// 从第一个类开始传递
class Service1 {  
  
    public void process(String name) {  
        User user = new User("超哥");  
        // 线程本地变量设置值
        UserContextHolder.holder.set(user);  
        // 调用第二类
        new Service2().process();  
    }  
}  
// 第二个类对该本线程内变量操作  
class Service2 {  
  
    public void process() {  
        User user = UserContextHolder.holder.get();  
        ThreadSafeFormatter.dateFormatThreadLocal.get();  
        System.out.println("Service2拿到用户名：" + user.name); 
        // 调用第三个类 
        new Service3().process();  
    }  
}  
 // 第三个类操作，不用传递，直接获取本线程内变量操作  
class Service3 {  
  
    public void process() {  
        User user = UserContextHolder.holder.get();  
        System.out.println("Service3拿到用户名：" + user.name);  
        // 使用完成后记得手动remove，防止内存泄漏
        UserContextHolder.holder.remove();  
    }  
}  
  
class UserContextHolder {  
    public static ThreadLocal<User> holder = new ThreadLocal<>();  
}  
  
class User {  
    String name;  
    public User(String name) {  
        this.name = name;  
    }  
}
```











## 8.2 原理

Thread、ThreadLocalMap、ThreadLocal关系

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205132316343.png)


ThreadLocalMap与HashMap不同
冲突时采用线性探测法，而不是拉链法、红黑树
其他基本类似，如扩容，基本方法都相同


## 8.3 内存泄漏问题
原因：
- ThreadLocalMap的每个Entry都是一个对key的弱引用，同时，每个Entry都包含了一个对value的强引用
- 正常情况下，当线程终止，保存在ThreadLocal里的value会被垃圾回收，因为没有任何强引用了
- 但是，如果线程不终止（比如线程需要保持很久），那么key对应的value就不能被回收，因为有以下的调用链∶`Thread → ThreadLocalMap → Entry（key为null）→ Value`
- 因为value和Thread之间还存在这个强引用链路，所以导致value无法回收，就可能会出现OOM
- JDK已经考虑到了这个问题，所以在set， remove， rehash方法中会扫描key=null,但value！=null设置为null，这样value对象就可以被回收
- 但是如果一个ThreadLocal不被使用，那么实际上set，remove，rehash方法也不会被调用，如果同时线程又不停止，那么调用链就一直存在，那么就导致了value的内存泄漏

防止：用完ThreadLocal后需要手动remove


## 8.4 继承主线程变量

ThreadLocal不支持继承主线程的变量，也就无法获取主线程的变量
需要通过``

```java
public class MyInheritThreadLocal {  
	// 原先通过ThreadLocal获取结果子线程得到null
	// public static ThreadLocal<String> threadLocal = new ThreadLocal<>(); 
	// 修改为InheritableThreadLocal后，子线程得到了haha值
    public static ThreadLocal<String> threadLocal = new InheritableThreadLocal<>();  
	  
    public static void main(String[] args) {  
       threadLocal.set("haha");  
        System.out.println(Thread.currentThread().getName() + " " + threadLocal.get());  
       new Thread(new Runnable() {  
           @Override           public void run() {  
               System.out.println(Thread.currentThread().getName() + " "  + threadLocal.get());  
           }  
       }).start();  
    }  
}
```






















![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205092136250.png)















