---
title: Java并发编程(JUC)
date: 2022-07-25 22:06:57
tags: java
categories: 学习笔记-javase/javaee
cover: https://images.pexels.com/photos/12507122/pexels-photo-12507122.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2
---

# 一、线程与进程

##  1.1进程与线程 

### 进程 

- 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。
- 进程就可以视为程序的一个实例。

### 线程

- 一个进程之内可以分为一到多个线程。 
- 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给 CPU 执行 
- **Java 中，线程作为最小调度单位，进程作为资源分配的最小单位。 在 windows 中进程是不活动的，只是作为线程的容器**

### 二者对比 

- 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集 
- 进程拥有共享的资源，如内存空间等，供其内部的线程共享 
- 进程间通信较为复杂 

- - 同一台计算机的进程通信称为 IPC（Inter-process communication） 
  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如 HTTP 

- 线程通信相对简单，因为它们共享进程内的内存，一个例子是多个线程可以访问同一个共享变量 
- 线程更轻量，线程上下文切换成本一般上要比进程上下文切换低

## 1.2并行与并发 

并发就是一个人面对多个事情

并行就是多个人面对多个事情

单核 cpu 下，线程实际还是 `串行执行` 的。操作系统中有一个组件叫做任务调度器，将 cpu 的时间片（windows 下时间片最小约为 15 毫秒）分给不同的程序使用，只是由于 cpu 在线程间（时间片很短）的切换非常快，人类感觉是 `同时运行的` 。总结为一句话就是： **微观串行，宏观并行** ，

一般会将这种线程轮流使用 CPU 的做法称为**并发， concurrent**

![](https://cdn.nlark.com/yuque/0/2022/png/393192/1648650642294-3244b7b5-7504-48c7-bcc4-f8a14f4a64dc.png)

<img src="https://cdn.nlark.com/yuque/0/2022/png/393192/1648650712710-c1b91ccb-3983-4229-8249-d0a71756bb1a.png?x-oss-process=image%2Fresize%2Cw_548%2Climit_0" width="30%;" />

多核 cpu下，每个 `核（core）` 都可以调度运行线程，这时候线程可以是**并行**的

![](https://cdn.nlark.com/yuque/0/2022/png/393192/1648650756667-a3a43425-081f-4760-a4db-be025953aa48.png)

<img src="https://cdn.nlark.com/yuque/0/2022/png/393192/1648650811806-d990d762-4670-4f7d-8c76-f6b22f42e9a7.png?x-oss-process=image%2Fresize%2Cw_541%2Climit_0" width="30%;" />

### 结论 

1.  单核 cpu 下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同线程轮流使用 cpu ，不至于一个线程总占用 cpu，别的线程没法干活 
2. 多核 cpu 可以并行跑多个线程，但能否提高程序运行效率还是要分情况的 

- - 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任 务都能拆分。
  - 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义 

3. IO 操作不占用 cpu，只是我们一般拷贝文件使用的是【阻塞 IO】，这时相当于线程虽然不用 cpu，

​      但需要一 直等待 IO 结束，没能充分利用线程。所以才有后面的【非阻塞 IO】和【异步 IO】优化

## 1.3 Java线程

Thread计和Runnable配合Thread创建线程，也可以通过lambda表达式创建

```java
 public static void main(String[] args) {
        // 第一种方式 构造方法的参数是给线程指定名字，推荐
        Thread t1 = new Thread("t1") {
            @Override
            // run 方法内实现了要执行的任务
            public void run() {
                log.debug("hello");
            }
        };
        t1.start();
        //第二种方式 使用 Runnable 配合 Thread
        Runnable r = new Runnable() {
            @Override
            public void run() {
                log.debug("hello runnable");
            }
        };
        Thread t2 = new Thread(r,"t2");
        t2.start();

        //第三种方式 使用lambda表达式
        Runnable r2 = () -> {log.debug("hello lambda");};
        Thread t3 = new Thread(r2,"t3");
        t3.start();
    }
```

FutureTask 配合 Thread 创建线程

```java
// 创建任务对象
FutureTask<Integer> task3 = new FutureTask<>(() -> {
    log.debug("hello");
    return 100;
});

// 参数1 是任务对象; 参数2 是线程名字，推荐
new Thread(task3, "t3").start();

// 主线程阻塞，同步等待 task 执行完毕的结果
Integer result = task3.get();
log.debug("结果是:{}", result);
```

```
19:22:27 [t3] c.ThreadStarter - hello
19:22:27 [main] c.ThreadStarter - 结果是:100
```

## **1.4**  Java线程运行的原理

### 栈与栈帧 

Java Virtual Machine Stacks （Java 虚拟机栈） 

我们都知道 JVM 中由**堆、栈、方法区**所组成，其中栈内存是给谁用的呢？其实就是线程，每个线程启动后，虚拟机就会为其分配一块栈内存。 

- 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存 
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法
- 线程与线程之间的栈帧是相互独立的
- 程序计数器是线程隔离的,每一个线程在工作的时候都有一个独立的计数器。

![](https://s1.ax1x.com/2022/07/26/jx5ywQ.png)

### 线程上下文切换（Thread Context Switch） 

因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码 

- 线程的 cpu 时间片用完 
- 垃圾回收 
- 有更高优先级的线程需要运行 
- 线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法 

当 Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念就是程序计数器（Program Counter Register），它的作用是记住下一条 jvm 指令的执行地址，是线程私有的 

- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等 
- Context Switch 频繁发生会影响性能

## 1.5.1 sleep 与 yield 

Java多线程常用方法

![](https://cdn.nlark.com/yuque/0/2022/png/393192/1649054467515-1a813bc3-edd8-4ff1-9cbd-d859abc0cbf7.png?x-oss-process=image%2Fresize%2Cw_613%2Climit_0)

### sleep 

- \1. 调用 sleep 会让当前线程从 *Running*进入 *Timed Waiting* 状态（阻塞） 
- \2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException 
- \3. 睡眠结束后的线程未必会立刻得到执行 
- \4. 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性 

### yield 

- \1. 调用 yield 会让当前线程从 *Running* 进入 *Runnable*就绪状态，然后调度执行其它线程 
- \2. 具体的实现依赖于操作系统的任务调度器 

### 线程优先级 

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它 
- 如果 cpu 比较忙，那么优先级高的线程会获得更多的时间片，但 cpu 闲时，优先级几乎没作用

### 应用: 限制-1.限制对CPU的使用

**sleep 实现** 

在没有利用 cpu 来计算时，不要让 while(true) 空转浪费 cpu，这时可以使用 yield 或 sleep 来让出 cpu 的使用权 给其他程序

```java
while(true) {
    try {
        Thread.sleep(50);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

- 可以用 wait 或 条件变量达到类似的效果 
- 不同的是，后两种都需要加锁，并且需要相应的唤醒操作，一般适用于要进行同步的场景 
- sleep 适用于无需锁同步的场景 

## 1.5.2 interrupt 方法详解 

打断 sleep，wait，join 的线程 

打断这几个方法都会让线程进入阻塞状态 

- 打断 sleep 的线程, 会清空打断状态，也就是isInterrupted方法返回为false，会抛出异常`java.lang.InterruptedException: sleep interrupted`

- 打断正常运行的线程, 不会清空打断状态,也就是只能打断正常的线程，而不能打断sleep的线程

那么如果要想打断sleep线程，而且还想让isInterrupted返回为true，也就是被打断，就会用到**(终止)模式之两阶段终止**

**(终止)模式之两阶段终止**

Two Phase Termination 

在一个线程 T1 中如何“优雅”终止线程 T2？这里的【优雅】指的是给 T2 一个料理后事的机会。 

### 1. 错误思路 

- 使用线程对象的 stop() 方法停止线程 

- - stop 方法会真正杀死线程，如果这时线程锁住了共享资源，那么当它被杀死后就再也没有机会释放锁，其它线程将永远无法获取锁 

- 使用 System.exit(int) 方法停止线程 

- - 目的仅是停止一个线程，但这种做法会让整个程序都停止

### 2. 两阶段终止模式

![](https://cdn.nlark.com/yuque/0/2022/png/393192/1649067461901-fb5807a0-1563-4543-9147-6a0fd2d6e31a.png)

代码

```java
class TPTInterrupt {
    private Thread thread;
    public void start(){
        thread = new Thread(() -> {
            while(true) {
                Thread current = Thread.currentThread();
                if(current.isInterrupted()) {
                    log.debug("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    log.debug("将结果保存");
                } catch (InterruptedException e) {
                    current.interrupt();
                }
                // 执行监控操作 
            }
        },"监控线程");
        thread.start();
    }
    public void stop() {
        thread.interrupt();
    }
}
```

调用

```java
TPTInterrupt t = new TPTInterrupt();
t.start();

Thread.sleep(3500);
log.debug("stop");
t.stop();
```

结果

```java
11:49:42.915 c.TwoPhaseTermination [监控线程] - 将结果保存
11:49:43.919 c.TwoPhaseTermination [监控线程] - 将结果保存
11:49:44.919 c.TwoPhaseTermination [监控线程] - 将结果保存
11:49:45.413 c.TestTwoPhaseTermination [main] - stop 
11:49:45.413 c.TwoPhaseTermination [监控线程] - 料理后事
```

## 1.6五种状态 ( **操作系统** 层面)

这是从 **操作系统** 层面来描述的

![](https://cdn.nlark.com/yuque/0/2022/png/393192/1649066126899-0b988758-8c4a-4e7d-b882-85bfae2d61a8.png)

- 【初始状态】仅是在语言层面**创建了线程对象**，还未与操作系统线程关联 
- 【可运行状态】（就绪状态）指该线程已经被创建（与操作系统线程关联），**可以由 CPU 调度执行** 
- 【运行状态】指**获取了 CPU 时间片**运行中的状态 

- - 当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换 

- 【阻塞状态】 

- - 如果**调用了阻塞 API**，如 BIO 读写文件，这时该线程实际不会用到 CPU，会导致线程上下文切换，进入【阻塞状态】 
  - 等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】 
  - 与【可运行状态】的区别是，对【阻塞状态】的线程来说**只要它们一直不唤醒，调度器就一直不会考虑调度它们** 

- 【终止状态】表示线程已经**执行完毕**，生命周期已经结束，不会再转换为其它状态

## 1.7 六种状态（JavaApi层面）

这是从 **Java API** 层面来描述的 

根据 Thread.State 枚举，分为六种状态

![](https://cdn.nlark.com/yuque/0/2022/png/393192/1649066238858-9cf6c08d-c3aa-478c-a3b0-f32e64f1978a.png)

- `NEW`线程刚被创建，但是还没有调用 `start()` 方法 
- `RUNNABLE` 当调用了 `start()` 方法之后，注意，**Java API** 层面的 `RUNNABLE` 状态涵盖了 **操作系统** 层面的【可运行状态】、【运行状态】和【阻塞状态】（由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为是可运行） 
- `BLOCKED ， WAITING ， TIMED_WAITING` 都是 **Java API** 层面对【阻塞状态】的细分。`WAITING`是没有时间限制的等待比如join方法，而`TIMED_WAITING`方法是指定时间的等待，比如sleep方法，`BLOCKED`是拿不到锁的情况
- `TERMINATED` 当线程代码运行结束

----

# 二、共享模型值管程

**共享带来的问题**，当多个线程去操作一个资源时，由于线程的切换，会造成共享资源读写操作时发生指令交错，就会出现问题 

## ２.１synchronized 解决方案 

为了避免临界区的竞态条件发生，有多种手段可以达到目的。 

- 阻塞式的解决方案：synchronized，Lock 
- 非阻塞式的解决方案：原子变量 

synchronized 语法

```java
synchronized(对象) // 线程1， 线程2(blocked)
{
 	临界区
}
```

示例

```java

static int counter = 0;
static final Object room = new Object();//锁

public static void main(String[] args) throws InterruptedException {
    
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            synchronized (room) {//获取锁之后才能执行
                counter++;
            }
        }
    }, "t1");
    
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            synchronized (room) {//获取锁之后才能执行
                counter--;
            }
        }
    }, "t2");
    
    t1.start();
    t2.start();
    t1.join();
    t2.join();
    log.debug("{}",counter);
}
```

### synchronize 面向对象改进 

把需要保护的共享变量放入一个类

```java

class Room {//对象本身就是一把锁
    int value = 0;
    public void increment() {
        synchronized (this) {
            value++;
        }
    }
    public void decrement() {
        synchronized (this) {//等同于public synchronized void decrement
            value--;
        }
    }
    public int get() {
        synchronized (this) {
            return value;
        }
    }
}

@Slf4j
public class Test1 {

    public static void main(String[] args) throws InterruptedException {
        Room room = new Room();
        Thread t1 = new Thread(() -> {
            for (int j = 0; j < 5000; j++) {
                room.increment();
            }
        }, "t1");
        
        Thread t2 = new Thread(() -> {
            for (int j = 0; j < 5000; j++) {
                room.decrement();
            }
        }, "t2");
        
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        
        log.debug("count: {}" , room.get());
    }
}
```

上面的方法中synchronize加在了非静态方法上，相当于对`this`，也就是**对象**进行加锁

如果是在静态方法上加synchronize，则表示对这个**类**加锁

## 2.2**方法上的 synchronized**

与使用synchronized(X)等效

```java
class Test{//非静态方法上加锁
    public synchronized void test() {

    }
}
等价于
class Test{
    public void test() {
        synchronized(this) {

        }
    }
}
```

```java
class Test{//静态方法上加锁
    public synchronized static void test() {
        
    }
}
等价于
class Test{
    public static void test() {
        synchronized(Test.class) {

        }
    }
}
```

## 2.3 Monitor 概念 

以 32 位虚拟机为例，**普通对象**的结构

![](https://s1.ax1x.com/2022/07/29/vP0NsH.png)

**Mark Word** 主要用来存储对象自身的运行时数据

![image-20221004203813164](http://www.wanghongtao.xyz/typora/image-20221004203813164.png)

![](https://s1.ax1x.com/2022/07/29/vP0tQe.md.png)

**Klass Word** 指向Class对象

不同对象状态下结构和含义也不同

> Normal 表示正常状态，Biased表示偏向锁，LightWeight表示轻量级锁，HeavyWeight表示重量级锁

**加锁过程**

Monitor是由操作系统给我们提供的，翻译为监视器或者管程

比如当`Thread-2`执行`synchronize(obj)`方法时，obj对象中的MarkWord指向系统给我们提供的Monitor

Monitor中的Owner设置为`Thread-2`

<img src="http://www.wanghongtao.xyz/typora/image-20221004204722582.png" alt="image-20221004204722582" width="50%;" />

当有其他线程`Thread-1 Thread-3`执行到`synchronize(obj)`方法时，这两个线程会进入Monitor的EntryList队列中等待

<img src="http://www.wanghongtao.xyz/typora/image-20221004205212697.png" alt="image-20221004205212697" width="50%;" />

### 2.3.1 Monitor 结构



![](https://s1.ax1x.com/2022/07/29/vPBup8.png)

- 刚开始 Monitor 中 Owner 为 null 
- 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一个 Owner 
- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入EntryList BLOCKED 
- Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争的时是非公平的 
- 图中 WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程，Thread-2要使用Thread-3的某些资源，那么Thread就回到WAITING中去等待了，等到获取到了资源，会重新唤醒，去BLOCKED中排队

### 2.3.2 (不涉及Monitor的)轻量级锁 

轻量级锁的使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化。

在一个对象创建轻量级锁时，让锁记录中 Object reference 指向锁对象，并尝试用 cas 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录，如果 cas 替换成功，对象头中存储了 `锁记录地址和状态 00 `，表示由该线程给对象加锁

![](https://s1.ax1x.com/2022/07/29/vPBOgS.png)

此时如果有该线程内其他方法进行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数

![](https://s1.ax1x.com/2022/07/29/vPBr7R.png)

- 如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入**锁膨胀**过程 

### 2.3.3(轻量级)锁膨胀(为重量级锁) 

如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁。

- 当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁

![](https://s1.ax1x.com/2022/07/29/vPDbZ9.md.png)

- 这时 Thread-1 加轻量级锁失败，进入锁膨胀流程 

- - 即为 Object 对象申请 **Monitor 锁**，让 Object 指向重量级锁地址 
  - 然后自己进入 Monitor 的 EntryList BLOCKED

![](https://s1.ax1x.com/2022/07/29/vPrPZd.png)

- 当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程

> 这一段可以怎样理解呢，可以将其理解为一个屋子，小明进入了屋子，并加上了轻量级锁；这时小红也想获得加轻量级锁，但是发现已经被占用了，于是乎就加上了一把重量级锁，反正别人也进不去了，自己就在屋外守着；等小明要出来时，发现自己被重量级锁锁住了，于是他便把屋子的使用权交给小红，并唤醒小红的线程。

### 2.3.4 (竞争重量级锁时的)自旋优化 

上面的例子中，当轻量级锁被膨胀成为重量级锁时，Thread-1线程停止了工作，也就是进入了阻塞状态。当重量级锁竞争的时候，会出现阻塞。可以使用**自旋(循环尝试获取重量级锁)**来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞。 (**进入阻塞再恢复,会发生上下文切换,比较耗费性能**)

- 自旋会占用 CPU 时间，单核 CPU 自旋就是浪费，多核 CPU 自旋才能发挥优势。 
- 在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能。 
- Java 7 之后不能控制是否开启自旋功能

### 2.3.5 (比轻量级锁更轻的)偏向锁 

轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作。 

Java 6 中引入了**偏向锁**来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有 

**轻量级锁与偏向锁对比**

![](https://s1.ax1x.com/2022/07/29/vPsvuD.md.png)

![](https://s1.ax1x.com/2022/07/29/vPyC4I.png)

![](https://s1.ax1x.com/2022/07/29/vP0tQe.md.png)

**偏向锁的撤销**

偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。

偏向锁的撤销，需要等到全局安全点（在这个时间点没有正在执行的字节码）。

它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否存活，如果线程不处于活动状态，则将对象头设置为无锁状态；如果线程还活着，拥有偏向锁的栈会被执行，遍历偏向锁的锁记录，栈中的锁记录和对象头的MarkWord要么重新偏向于其他线程，要么恢复到无锁状态。最后唤醒暂停中的线程。



一个对象创建时： 

- 如果开启了偏向锁（默认开启），那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的 thread、epoch、age 都为 0 
- 偏向锁是默认是延迟的，不会在程序启动时立即生效，如果想避免延迟，可以加 VM 参数 `-XX:BiasedLockingStartupDelay=0` 来`禁用延迟` 
- 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、age 都为 0，第一次用到 hashcode 时才会赋值



- 调用了对象的 hashCode，但偏向锁的对象 MarkWord 中存储的是线程 id，如果调用 hashCode 会导致偏向锁被撤销 
- 当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁
- 当(某类型对象)撤销偏向锁阈值超过 20 次后，jvm 会这样觉得，我是不是偏向错了呢，于是会在给(所有这种类型的状态为偏向锁的)对象加锁时重新偏向至新的加锁线程
- 当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向。于是整个类的所有对象都会变为不可偏向的，新建的该类型对象也是不可偏向的 

## 2.4 原理之 LockSupport.park & unpark

**与 Object 的 wait & notify 相比** 
● wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必
● park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，就不那么【精确】 
● park & unpark 可以先 unpark，而 wait & notify 不能先 notify 

每个线程都有自己的一个(C代码实现的) Parker 对象，由三部分组成 `_counter` ， `_cond` 和`_mutex`

当调用park方法时，会首先查看线程中`_counter`是否为0，如果为0，则阻塞该线程(线程进入Parker对象的`_cont`)，如果为1，则`_counter--`

当调用unpark方法时，会使`_counter++`(`_counter`最多为1)，如果`_cond`中有阻塞线程，则线程唤醒，并使`_counter--`

## 2.5 重新理解线程中的状态

![](https://cdn.nlark.com/yuque/0/2022/png/393192/1649066238858-9cf6c08d-c3aa-478c-a3b0-f32e64f1978a.png)

**情况1 NEW --> RUNNABLE**

当调用 t.start() 方法时，由 NEW --> RUNNABLE



**情况2 RUNNABLE <--> WAITING**

**t 线程**用 `synchronized(obj)` 获取了对象锁后 

- 调用 obj.wait() 方法时，**t 线程**从 RUNNABLE --> WAITING 
- 调用 obj.notify() ， obj.notifyAll() ， t.interrupt() 时 

- - 竞争锁成功，**t 线程**从WAITING --> RUNNABLE 
  - 竞争锁失败，**t 线程**从WAITING --> BLOCKED

- 

**情况 3 RUNNABLE <--> WAITING**

- **当前线程**调用 t.join() 方法时，**当前线程**从 RUNNABLE --> WAITING 

- - 注意是**当前线程**在**t 线程对象**的监视器上等待 

- **t 线程**运行结束，或调用了**当前线程**的 interrupt() 时，**当前线程**从 WAITING --> RUNNABLE



**情况 4 RUNNABLE <--> WAITING**

- 当前线程调用 LockSupport.park() 方法会让当前线程从 RUNNABLE --> WAITING 
- 调用 LockSupport.unpark(目标线程) 或调用了线程 的 interrupt() ，会让目标线程从 WAITING -->RUNNABLE 



**情况 5 RUNNABLE <--> TIMED_WAITING**

**t** **线程**用 synchronized(obj) 获取了对象锁后 

- 调用 obj.wait(long n) 方法时，**t 线程**从 RUNNABLE --> TIMED_WAITING 
- **t 线程**等待时间超过了 n 毫秒，或调用 obj.notify() ， obj.notifyAll() ， t.interrupt() 时 

- - 竞争锁成功，**t 线程**从TIMED_WAITING --> RUNNABLE 
  - 竞争锁失败，**t 线程**从TIMED_WAITING --> BLOCKED



**情况 6 RUNNABLE <--> TIMED_WAITING**

- **当前线程**调用 t.join(long n) 方法时，**当前线程**从 RUNNABLE --> TIMED_WAITING 

- - 注意是**当前线程**在**t 线程对象**的监视器上等待 

- **当前线程**等待时间超过了 n 毫秒，或**t 线程**运行结束，或调用了**当前线程**的 interrupt() 时，**当前线程**从 TIMED_WAITING --> RUNNABLE



**情况 7 RUNNABLE <--> TIMED_WAITING**



- 当前线程调用 Thread.sleep(long n) ，当前线程从 RUNNABLE --> TIMED_WAITING 
- **当前线程**等待时间超过了 n 毫秒，**当前线程**从TIMED_WAITING --> RUNNABLE



**情况 8 RUNNABLE <--> TIMED_WAITING**

- 当前线程调用 LockSupport.parkNanos(long nanos) 或 LockSupport.parkUntil(long millis) 时，**当前线 程**从 RUNNABLE --> TIMED_WAITING 
- 调用 LockSupport.unpark(目标线程) 或调用了线程 的 interrupt() ，或是等待超时，会让目标线程从 TIMED_WAITING--> RUNNABLE



**情况 9 RUNNABLE <--> BLOCKED**

- **t 线程**用synchronized(obj) 获取对象锁时如果竞争失败，从RUNNABLE --> BLOCKED 
- 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 BLOCKED的线程重新竞争，如果其中 **t 线程**竞争 成功，从 BLOCKED --> RUNNABLE ，其它失败的线程仍然BLOCKED 

**情况 10 RUNNABLE --> TERMINATED** 

当前线程所有代码运行完毕，进入 TERMINATED 

## 2.6 多把锁&活跃性

### 2.6.1 多把锁

例如，学习和睡觉共用一把锁

```java
class BigRoom {
    public void sleep() {
        synchronized (this) {
            log.debug("sleeping 2 小时");
            Sleeper.sleep(2);
        }
    }
    public void study() {
        synchronized (this) {
            log.debug("study 1 小时");
            Sleeper.sleep(1);
        }
    } 
}
```

优化后（使用两把锁，使其互不干扰），这样的话，假如有小明和小红分别要学习和睡觉，那么他们就可以互不干扰的使用两把锁，而不是去竞争一把锁。

```java
class BigRoom {
    private final Object studyRoom = new Object();
    private final Object bedRoom = new Object();
    public void sleep() {
        synchronized (bedRoom) {
            log.debug("sleeping 2 小时");
            Sleeper.sleep(2);
        }
    } 
    public void study() {
        synchronized (studyRoom) {
            log.debug("study 1 小时");
            Sleeper.sleep(1);
        }
    }  
}
```

将锁的粒度细分 

- 好处，是可以增强并发度 
- 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁

### 2.6.2 活跃性

**死锁**

`t1 线程` 获得 `A对象` 锁，接下来想获取 `B对` 的锁 

`t2 线程` 获得 `B对象` 锁，接下来想获取 `A对象` 的锁

> 避免死锁要注意加锁顺序 

**定位死锁**

检测死锁可以使用 jconsole工具，或者使用 jps 定位进程 id，再用 jstack 定位死锁

**活锁** 

活锁出现在**两个线程互相改变对方的结束条件**，最后谁也无法结束

**饥饿** 

很多教程中把饥饿定义为，一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束

> 在解决死锁问题时一般会用顺序加锁的方法，但是顺序加锁会导致饥饿问题

## 2.7 ReentrantLock *

相对于 **synchronized** 它具备如下特点 

- 可中断 
- 可以设置超时时间 
- 可以设置为公平锁 
- 支持多个条件变量 
- Lock是一个接口，而synchronized是java中的关键字，synchronized是内置的语言实现

与 synchronized 一样，都支持**可重入** 

**基本语法**

```java
// 获取锁
reentrantLock.lock();
try {
    // 临界区
} finally {
    // 释放锁
    reentrantLock.unlock();
}
```

**可重入** :

可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁 

如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住

**可打断  lock.lockInterruptibly()**

```java
try {
        //没有竞争就会获取锁
        //有竞争就进入阻塞队列等待,但可以被打断
        lock.lockInterruptibly();
        //lock.lock(); //不可打断
    } catch (InterruptedException e) {
        e.printStackTrace();
        log.debug("等锁的过程中被打断");
        return;
    }
```

注意如果是不可中断模式(`lock.lock()`)，那么即使使用了 interrupt 也不会让等待中断

**锁(可设置)超时** 

立刻返回结果 lock.tryLock()

```java
if (!lock.tryLock()) {
        log.debug("获取锁失败，立即返回");
        return;
    }
```

尝试一定时间 lock.tryLock

```java
if (!lock.tryLock(1, TimeUnit.SECONDS)) {
            log.debug("获取等待 1s 后失败，返回");
            return;
        }
```

**(可设置是否为)公平锁** 

ReentrantLock 默认是**不公平**的

```java
ReentrantLock lock = new ReentrantLock(false);//不公平锁
ReentrantLock lock = new ReentrantLock(true);//公平锁
```

**(多个)条件变量** 

synchronized 中也有条件变量，就是我们讲原理时那个 waitSet 休息室，当条件不满足时进入 waitSet 等待 

ReentrantLock 的条件变量比 synchronized 强大之处在于，它是支持多个条件变量的，这就好比 

- synchronized 是那些不满足条件的线程都在一间休息室等消息 
- 而 ReentrantLock 支持多间休息室，有专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒 

使用要点： 

- await 前需要获得锁 
- await 执行后，会释放锁，进入 conditionObject 等待 
- await 的线程被唤醒（或打断、或超时）去重新竞争 lock 锁 
- 竞争 lock 锁成功后，从 await 后继续执行

```java
static ReentrantLock lock = new ReentrantLock();

static Condition waitCigaretteQueue = lock.newCondition();
static Condition waitbreakfastQueue = lock.newCondition();

static volatile boolean hasCigrette = false;//第一间休息室
static volatile boolean hasBreakfast = false;//第二间休息室

public static void main(String[] args) {
    new Thread(() -> {
        try {
            lock.lock();
            while (!hasCigrette) {
                try {
                    waitCigaretteQueue.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("等到了它的烟");
        } finally {
            lock.unlock();
        }
    }).start();
    
    new Thread(() -> {
        try {
            lock.lock();
            while (!hasBreakfast) {
                try {
                    waitbreakfastQueue.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("等到了它的早餐");
        } finally {
            lock.unlock();
        }
    }).start();
    
    sleep(1);
    sendBreakfast();
    sleep(1);
    sendCigarette();
}

private static void sendCigarette() {
    lock.lock();
    try {
        log.debug("送烟来了");
        hasCigrette = true;
        waitCigaretteQueue.signal();//唤醒CigaretteQueue的等待线程
    } finally {
        lock.unlock();
    }
}

private static void sendBreakfast() {
    lock.lock();
    try {
        log.debug("送早餐来了");
        hasBreakfast = true;
        waitbreakfastQueue.signal();//唤醒breakfastQueue的等待线程
    } finally {
        lock.unlock();
    }
}

```

## 2.8 同步模式之顺序控制 *

###  固定运行顺序 

比如，必须先 2 后 1 打印 

 wait notify 版

Park Unpark 版

交替输出

wait notify 版

Lock 条件变量版

Park Unpark 版

# 三、共享模型之内存

## 3.1 Java 内存模型 

**JMM** 即 Java Memory Model，它定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、缓存、硬件内存、CPU 指令优化等。 



JMM 体现在以下几个方面 

- 原子性 - 保证指令不会受到线程上下文切换的影响 
- 可见性 - 保证指令不会受 cpu 缓存的影响 
- 有序性 - 保证指令不会受 cpu 指令并行优化的影响

在并发编程中，有两个关键问题：

- 线程之间如何通信
- 线程之间如何同步



## 3.2 可见性 

先来看一个现象，main 线程对 run 变量的修改对于 t 线程不可见，导致了 t 线程无法停止：

```java
static boolean run = true;

public static void main(String[] args) throws InterruptedException {
    Thread t = new Thread(()->{
        while(run){
            // ....
        }
    });
    t.start();
    sleep(1);
    run = false; // 线程t不会如预想的停下来
}
```

原因：因为 t 线程要频繁从主内存中读取 run 的值，JIT 编译器会将 run 的值缓存至自己工作内存中的高速缓存中，减少对主存中 run 的访问，提高效率

![](https://s1.ax1x.com/2022/08/03/vZvffS.md.png)

所以在主线程中修改的变量值并不会影响缓存中的数据，那么如何解决这一问题呢？

![](https://s1.ax1x.com/2022/08/03/vZv5lQ.md.png)

### 解决方法 

**volatile**（易变关键字） 

它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存

也就是把`static boolean run = true;`修改为`static volatile boolean run = true;`

也可以使用synchronize

> synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是synchronized 是属于重量级操作，性能相对更低 
>
> 如果在前面示例的死循环中加入 System.out.println() 会发现即使不加 volatile 修饰符，线程 t 也能正确看到对 run 变量的修改了，想一想为什么？
>
> 因为其内部包含了synchronized 的使用

## 3.3 模式之两阶段终止

设计思路：设置一个线程共享的变量stop（设置为volatile） ，使得一个线程可以打断另一个线程（用while循环判断stop是否为真）

## 3.4 (同步)模式之 Balking(犹豫) 

#### 1. 定义 

Balking （犹豫）模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做了，直接结束返回 

#### 2. 实现 

```java
 // 用来表示是否已经有线程已经在执行启动了
    private volatile boolean starting;
    
    public void start() {
        log.info("尝试启动监控线程...");
        synchronized (this) {
            if (starting) {
                return;
            }
            starting = true;
        }
        // 真正启动监控线程...
    }
```

## 3.5 有序性 *

### 3.5.1 指令重排

JVM 会在不影响正确性的前提下，可以调整语句的执行顺序。

### 3.5.2 **为什么会有指令重排-(CPU)指令级并行**

**Clock Cycle Time  时钟周期时间**

主频的概念大家接触的比较多，而 CPU 的 Clock Cycle Time（时钟周期时间），等于主频的倒数，意思是 CPU 能够识别的最小时间单位，比如说 4G 主频的 CPU 的 Clock Cycle Time 就是 0.25 ns，作为对比，我们墙上挂钟的Cycle Time 是 1s 

例如，运行一条加法指令一般需要一个时钟周期时间 

**CPI 平均时钟周期数**

有的指令需要更多的时钟周期时间，所以引出了 CPI （Cycles Per Instruction）指令平均时钟周期数 

**IPC  即 CPI 的倒数**

IPC（Instruction Per Clock Cycle）即 CPI 的倒数，表示每个时钟周期能够运行的指令数 

**CPU 执行时间** 

程序的 CPU 执行时间，即我们前面提到的 user + system 时间，可以用下面的公式来表示 

```
程序 CPU 执行时间 = 指令数 * CPI * Clock Cycle Time 
```

事实上，现代处理器会设计为一个时钟周期完成一条执行时间最长的 CPU 指令。为什么这么做呢？

可以想到指令还可以再划分成一个个更小的阶段，

例如，每条指令都可以分为： `取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回` 这 5 个阶段

- instruction fetch (IF) 
- instruction decode (ID) 
- execute (EX) 
- memory access (MEM) 
- register write back (WB) 

![](https://s1.ax1x.com/2022/08/04/veA1qU.png)

在不改变程序结果的前提下，这些指令的各个阶段可以通过**重排序**和**组合**来实现**指令级并行**

![](https://s1.ax1x.com/2022/08/04/veEYm8.png)

大多数处理器包含多个执行单元，并不是所有计算功能都集中在一起，可以再细分为整数运算单元、浮点数运算单 

元等，这样可以把多条指令也可以做到并行获取、译码等，CPU 可以在一个时钟周期内，执行多于一条指令，IPC> 1 

![](https://s1.ax1x.com/2022/08/04/veE511.png)



### 3.5.3 解决方法 -volatile

volatile 修饰的变量，可以禁用指令重排（也包括该变量之前的变量也会禁用指令重排）

**原理之 volatile (写屏障和读屏障来保证可见性和有序性)**

volatile 的底层实现原理是内存屏障，Memory Barrier（Memory Fence）

- 对 volatile 变量的写指令后会加入写屏障
- 对 volatile 变量的读指令前会加入读屏障

**保证可见性的原理**

写屏障（sfence）保证在该屏障之前的，对共享变量的改动，都同步到主存当中

```java
public void actor2(I_Result r) {
    num = 2;
    ready = true; // ready 是 volatile 赋值带写屏障
    // 写屏障
}
```

而读屏障（lfence）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据

```java
public void actor1(I_Result r) {
    // 读屏障
    // ready 是 volatile 读取值带读屏障
    if(ready) {
        r.r1 = num + num;
    } else {
        r.r1 = 1;
    }
}
```

假设线程A首先执行`writer()`方法，线程B随后执行`reader()`方法

```java
	int a = 0;
    volatile boolean flag = false;
    public void writer() {
        a = 1;
        flag = true;
    }
    public void reader() {
        if(flag) {
            int i = a;
            System.out.println(i);
        }
    }
```

![image-20221007170910592](http://www.wanghongtao.xyz/typora/image-20221007170910592.png)

**保证有序性的原理** 

写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后

```java
public void actor2(I_Result r) {
    num = 2;
    // 写写屏障 
    ready = true; // ready 是 volatile 赋值带写屏障 
    // 写读屏障
}
```

读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前

```java
public void actor1(I_Result r) {
    //读读屏障
    if(ready) {// ready 是 volatile 读取值带读屏障
        // 读写屏障
        r.r1 = num + num;
    } else {
        r.r1 = 1;
    }
}
```

**不能解决指令交错问题：** 

- 写屏障仅仅是保证之后的读能够读到最新的结果，但不能保证读跑到它前面去 
- 而有序性的保证也只是保证了本线程内相关代码不被重排序

上面只是JMM保证指令有序性原理的一部分，实际上JMM保证有序性的策略比较保守，有以下四点

- 在每个volatile写操作之前插入一个`StoreStore`屏障
- 在每个volatile写操作之后插入一个`StoreLoad`屏障
- 在每个volatile读操作前插入`LoadLoad`屏障
- 在每个volatile读操作后插入`LoadStore`屏障

![image-20221007174932728](http://www.wanghongtao.xyz/typora/image-20221007174932728.png)

![image-20221007193824675](http://www.wanghongtao.xyz/typora/image-20221007193824675.png)

## 3. 6 (实现单例的)double-checked locking 问题 

double-checked locking单例模式

```java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    
    public static Singleton getInstance() { 
        if(INSTANCE == null) { // 防止多个线程都去获取synchronize导致性能下降
            // 首次访问会同步获取锁，但创建对象之后就会被if判断阻拦
            synchronized(Singleton.class) {
                if (INSTANCE == null) { 
                    INSTANCE = new Singleton(); //synchronize内部也会有指令重排，因为INSTANCE在synchronize外部有判断，所以不能保证有序性
                } 
            }
        }
        return INSTANCE;
    }
}
```

上面的代码可能有如下问题：

synchronize代码块内部由于发生指令重排序，原本的`INSTANCE`对象创建在字节码层面发生了重排

![](https://s1.ax1x.com/2022/08/04/vemtvn.md.png)

其中 

- 17 表示创建对象，将对象引用入栈 // new Singleton 
- 20 表示复制一份对象引用 // 引用地址 
- 21 表示利用一个对象引用，调用构造方法 
- 24 表示利用一个对象引用，赋值给 static INSTANCE

也许 jvm 会优化为：先执行 24，再执行 21。如果两个线程 t1，t2 按如下时间序列执行：

![](https://s1.ax1x.com/2022/08/04/vem72d.png)

也就是，在synchronize内部还未调用构造方法时，INSTANCE已经不为空（一个未初始化完毕的单例），但是其他线程在使用对象时就会出现问题。

### **解决**

对 INSTANCE 使用 **volatile** 修饰即可，可以禁用指令重排

```java
public final class Singleton {
    private Singleton() { }
    private static volatile Singleton INSTANCE = null;
    
    public static Singleton getInstance() {
        // 实例没创建，才会进入内部的 synchronized代码块
        if (INSTANCE == null) { 
            synchronized (Singleton.class) { // t2
                // 也许有其它线程已经创建实例，所以再判断一次
                if (INSTANCE == null) { // t1
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

# 四、共享模型值无锁 *

**本章内容** 

- CAS 与 volatile 
- 原子整数 
- 原子引用 
- 原子累加器 
- Unsafe

有如下需求，保证 `account.withdraw` 取款方法的线程安全

```java
class AccountUnsafe implements Account {
    private Integer balance;
    
    public AccountUnsafe(Integer balance) {
        this.balance = balance;
    }
    
    @Override
    public Integer getBalance() {
        return balance;
    }
    
    @Override
    public void withdraw(Integer amount) {
        balance -= amount;//线程不安全
    }
}
```

我们知道，由于在字节码层面，自增和自减并非原子操作，所以在多线程模式下是线程不安全的

## 4.1 CAS实现思路

要解决线程安全问题

**方式一、使用synchronize锁**

实现思路，将原有方法改为： `public synchronized void withdraw(Integer amount) `

**方式二、无锁(AtomicInteger)**

实现思路：将balance定义为AtomicInteger类型，并在自减方法上使用方法**compareAndSet(CAS)**

```java
private AtomicInteger balance; //原子整数
@Override
    public void withdraw(Integer amount) {
        while (true) {
            int prev = balance.get();
            int next = prev - amount;
            if (balance.compareAndSet(prev, next)) {//第一个参数为expected value，第二个为next value
                break;
            }
        }
        // 可以简化为下面的方法
        // balance.addAndGet(-1 * amount);
    }
```

> 在上面的方法`balance.compareAndSet(prev, next)`中，修改balance的值之前会首先判断当前被修改的值是否为最新值，如果不是，则重新进入循环，知道读取最新值为止。这样就能保证操作的原子性。
>
> **注意** 
>
> 
>
> 其实 CAS 的底层是 `lock cmpxchg` 指令（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的原子性。
>
> - 在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的

## 4.2 CAS 与 volatile 

无锁实现线程安全，其中的关键是 **compareAndSet**，它的简称就是 **CAS** （也有 Compare And Swap 的说法），它必须是原子操作

获取共享变量时，为了保证该变量的可见性，需要使用 volatile 修饰。它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存。即一个线程对 volatile 变量的修改，对另一个线程可见。 

CAS是借助了volatile，其内部的value存储是volatile的

## 4.3 为什么(相对而言)无锁效率高 

synchronized 和 cas 没有绝对的谁效率高,要看所处的场景

- 无锁情况下，即使重试失败，线程始终在高速运行，没有停歇，而 synchronized 会让线程在没有获得锁的时候，发生上下文切换，进入阻塞

- 但无锁情况下，因为线程要保持运行，需要额外 CPU 的支持，CPU 在这里就好比高速跑道，没有额外的跑道，线程想高速运行也无从谈起，虽然不会进入阻塞，但由于没有分到时间片，仍然会进入可运行状态，还是会导致上下文切换。

总结一下就是，无锁情况的优势在于可以避免线程切换导致的效率减少，但是如果CPU的核心数不够，还是会导致线程切换，所以在多核CPU下，无锁才能发挥优势。

## 4.4 CAS 的特点

结合 CAS 和 volatile 可以实现无锁并发，适用于线程数少、多核 CPU 的场景下。 



- CAS 是基于乐观锁的思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，我吃亏点再重试呗。 
- synchronized 是基于悲观锁的思想：最悲观的估计，得防着其它线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会。 
- CAS 体现的是**无锁并发、无阻塞并发**

## 4.5 原子整数 

J.U.C 并发包提供了： 

- AtomicBoolean 
- AtomicInteger 
- AtomicLong

三种数据类型

AtomicInteger的相关API

```java
AtomicInteger i = new AtomicInteger(0);

// 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
System.out.println(i.getAndIncrement());

// 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i
System.out.println(i.incrementAndGet());

// 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
System.out.println(i.decrementAndGet());

// 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
System.out.println(i.getAndDecrement());

// 获取并加值（i = 0, 结果 i = 5, 返回 0）
System.out.println(i.getAndAdd(5));

// 加值并获取（i = 5, 结果 i = 0, 返回 0）
System.out.println(i.addAndGet(-5));

// 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.getAndUpdate(p -> p - 2));

// 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.updateAndGet(p -> p + 2));

// 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
// getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
// getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));

// 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));
```

## 4.6 原子引用  AtomicXXXReference 

为什么需要原子引用类型？ 

- AtomicReference 
- AtomicMarkableReference 
- AtomicStampedReference 

例如 AtomicReference<> 实现BigDecimal线程安全

```java
class DecimalAccountSafeCas implements DecimalAccount {
    AtomicReference<BigDecimal> ref;
    
    public DecimalAccountSafeCas(BigDecimal balance) {
        ref = new AtomicReference<>(balance);
    }
    
    @Override
    public BigDecimal getBalance() {
        return ref.get();
    }
    
    @Override
    public void withdraw(BigDecimal amount) {
        while (true) {
            BigDecimal prev = ref.get();
            BigDecimal next = prev.subtract(amount);
            if (ref.compareAndSet(prev, next)) {
                break;
            }
        }
    }
    
}
```

## 4.7 ABA 问题及解决 

### ABA问题

我们已知，在cas中，我们对一个共享变量进行修改时，需要首先判断是否当前值为最新值，也就是compareAndSet，但是，想象以下情况：

线程t1和t2获取共享变量A，t2执行修改A的操作，并执行到compareAndSet这一步，但是在这期间t2线程完成了将A改为B，再将B改为A的操作，此时线程t1的compareAndSet就会认为当前值A是最新值，会继续执行set方法。

但是实际情况是这个A已经经历了**A->B->A**

这就是ABA问题

### ABA问题解决

**AtomicStampedReference(维护版本号)**

将我们的共享变量这样定义

`static AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);//变量值，版本号`

在修改时执行如下操作

```java
int stamp = ref.getStamp();//版本号
ref.compareAndSet(prev, "C", stamp, stamp + 1)//expectedValue,nextValue,expectedStamp,nextStamp
```

但是有时候，并不关心引用变量更改了几次，只是单纯的关心**是否更改过**,这时，就用到了**AtomicMarkableReference**  

与AtomicStampedReference不同的是，AtomicMarkableReference维护的是共享变量是否被改过

`ref.compareAndSet(prev, next, true, false);`

## 4.8 原子数组 

- AtomicIntegerArray 
- AtomicLongArray 
- AtomicReferenceArray 

有如下方法

```java
/**
    参数1，提供数组、可以是线程不安全数组或线程安全数组
    参数2，获取数组长度的方法
    参数3，自增方法，回传 array, index
    参数4，打印数组的方法
*/
// supplier 提供者 无中生有 ()->结果
// function 函数 一个参数一个结果 (参数)->结果 , BiFunction (参数1,参数2)->结果
// consumer 消费者 一个参数没结果 (参数)->void, BiConsumer (参数1,参数2)->
private static <T> void demo(
    Supplier<T> arraySupplier,
    Function<T, Integer> lengthFun,
    BiConsumer<T, Integer> putConsumer,
    Consumer<T> printConsumer ) {
    
    List<Thread> ts = new ArrayList<>();
    T array = arraySupplier.get();
    int length = lengthFun.apply(array);
    for (int i = 0; i < length; i++) {
        // 每个线程对数组作 10000 次操作
        ts.add(new Thread(() -> {
            for (int j = 0; j < 10000; j++) {
                putConsumer.accept(array, j%length);
            }
        }));
    }
    ts.forEach(t -> t.start()); // 启动所有线程
    
    ts.forEach(t -> {
        try {
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }); // 等所有线程结束
    printConsumer.accept(array);
}
```

**不安全的数组**

```java
demo(
    ()->new int[10],
    (array)->array.length,
    (array, index) -> array[index]++,
    array-> System.out.println(Arrays.toString(array))
);
```

结果 `[9870, 9862, 9774, 9697, 9683, 9678, 9679, 9668, 9680, 9698]`

**安全的数组**

```java
demo(
    ()-> new AtomicIntegerArray(10),//原子数组
    (array) -> array.length(),
    (array, index) -> array.getAndIncrement(index),
    array -> System.out.println(array)
);
```

结果 `[10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000] `

## 4.9 字段原子更新器  AtomicXXXFieldUpdater 

- AtomicReferenceFieldUpdater // 域字段 
- AtomicIntegerFieldUpdater 
- AtomicLongFieldUpdater 



利用字段更新器，可以针对对象的某个域（Field）进行原子操作，只能配合 volatile 修饰的字段使用，

否则会出现异常 `Exception in thread "main" java.lang.IllegalArgumentException: Must be volatile type`

```java
public class Test5 {
    private volatile int field;
    
    public static void main(String[] args) {
        AtomicIntegerFieldUpdater fieldUpdater =AtomicIntegerFieldUpdater.newUpdater(Test5.class, "field");
        
        Test5 test5 = new Test5();
        fieldUpdater.compareAndSet(test5, 0, 10);
        // 修改成功 field = 10
        System.out.println(test5.field);
        // 修改成功 field = 20
        fieldUpdater.compareAndSet(test5, 10, 20);
        System.out.println(test5.field);
        // 修改失败 field = 20
        fieldUpdater.compareAndSet(test5, 10, 30);
        System.out.println(test5.field);
    }
}
```

## 4.10原子累加器 

**累加器性能比较**

假设以下方法时分别对LongAdder类型的变量和AtomicLong类型的变量进行累加操作，统计其经历时长，分别统计五次

```java
for (int i = 0; i < 5; i++) {
    demo(() -> new LongAdder(), adder -> adder.increment());
}
for (int i = 0; i < 5; i++) {
    demo(() -> new AtomicLong(), adder -> adder.getAndIncrement());
}
```

```java
private static <T> void demo(Supplier<T> adderSupplier, Consumer<T> action) {
    T adder = adderSupplier.get();
    
    long start = System.nanoTime();
    
    List<Thread> ts = new ArrayList<>();
    // 4 个线程，每人累加 50 万
    for (int i = 0; i < 40; i++) {
        ts.add(new Thread(() -> {
            for (int j = 0; j < 500000; j++) {
                action.accept(adder);
            }
        }));
    }
    ts.forEach(t -> t.start());
    
    ts.forEach(t -> {
        try {
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    long end = System.nanoTime();
    System.out.println(adder + " cost:" + (end - start)/1000_000);
}
```

```
1000000 cost:43 //LongAdder
1000000 cost:9 
1000000 cost:7 
1000000 cost:7 
1000000 cost:7 
    
1000000 cost:31 //AtomicLong
1000000 cost:27 
1000000 cost:28 
1000000 cost:24 
1000000 cost:22
```

性能提升的原因很简单，就是LongAdder在有竞争时 最后将结果汇总。

这样它们在累加时操作的不同的 Cell 变量，因此**减少了 CAS 重试失败**，从而提高性能。



## 4.11LongAddr 伪共享问题

**Cell需要防止防止缓存行伪共享问题，其中 Cell 即为累加单元**

![](https://s1.ax1x.com/2022/08/06/vu4ejx.png)

因为 CPU 与 内存的速度差异很大，需要靠预读数据至缓存来提升效率。 

而缓存以缓存行为单位，每个缓存行对应着一块内存，一般是 64 byte（8 个 long） 

缓存的加入会造成数据副本的产生，即同一份数据会缓存在不同核心的缓存行中 

CPU 要保证数据的一致性，如果某个 CPU 核心更改了数据，其它 CPU 核心对应的整个缓存行必须失效

![](https://s1.ax1x.com/2022/08/06/vu4uDK.png)

因为 Cell 是数组形式，在内存中是连续存储的，一个 Cell 为 24 字节（16 字节的对象头和 8 字节的 value），因此缓存行可以存下 2 个的 Cell 对象。

这样问题来了： 

- Core-0 要修改 Cell[0] 
- Core-1 要修改 Cell[1] 

无论谁修改成功，都会导致对方 Core 的缓存行失效，

#### 解决方法: @sun.misc.Contended

`@sun.misc.Contended` 用来解决这个问题，它的原理是在使用此注解的对象或字段的前后各增加 128 字节大小的padding，从而让 CPU 将对象预读至缓存时占用不同的缓存行，这样，不会造成对方缓存行的失效 

![](https://s1.ax1x.com/2022/08/06/vu4lUe.png)



## 4.12 unsafe



unsafe的作用：

![](https://s1.ax1x.com/2022/08/06/vu4LqK.png)

unsafe并不是线程不安全的意思，是因为他是系统底层的类，使用起来可能有安全隐患

LockSupport 类中有各种版本 pack 方法，但最终都调用了`Unsafe.park()`方法

# 五、共享模型之不可变

## 5.1 不可变设计 

不可变设计是为了解决对象的线程不安全问题

比如String 类是不可变的，以它为例，说明一下不可变设计的要素

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    /** Cache the hash code for the string */
    private int hash; // Default to 0
    // ...
    
}
```

- 属性用 final 修饰保证了该属性是只读的，不能修改 
- 类用 final 修饰保证了该类中的方法不能被覆盖，防止子类无意间破坏不可变性

### 保护性拷贝 （defensive copy）

但有同学会说，使用字符串时，也有一些跟修改相关的方法啊，比如 substring 等，

那么下面就看一看这些方法是如何实现的，就以 substring 为例：

```java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

发现其内部是调用 String 的构造方法创建了一个新字符串，构造新字符串对象时，会生成新的 char[] value，对内容进行复制 。

这种通过创建副本对象来避免共享的手段称之为【保护性拷贝（defensive copy）】 

### 设置 final 变量的原理 

理解了 volatile 原理，再对比 fifinal 的实现就比较简单了

![](https://s1.ax1x.com/2022/08/06/vuI5cR.png)

在对final域写之后，构造函数return之前，会加一个**写屏障**，这个屏障会禁止把final域的写重排序到构造函数之外。     这样写final域的重排序规则可以确保，在对象引用为任意线程可见之前，对象的final已经被正确的初始化了。

同样，在读之前，会加**读屏障**，初次读对象引用与初次读该对象包含的final，这两个之间不会重排序这两个操作

### 匿名内部类访问的局部变量为什么必须要用final修饰

匿名内部类之所以可以访问局部变量，是因为在底层将这个局部变量的值传入到了匿名内部类中，

并且以匿名内部类的成员变量的形式存在，这个值的传递过程是通过匿名内部类的构造器完成的

# 六、共享模型之工具

**线程池**

1、什么是线程池

用一句话来概述就是：线程池是指在初始化一个多线程应用程序过程中创建一个线程集合，然后再需要执行新的任务时重用这些线程而不是新建线程。
2、为什么使用线程池

使用线程池最大的原因就是可以根据系统的需求和硬件环境灵活的控制线程的数量，且可以对所有线程进行统一的管理和控制，从而提高系统的运行效率，降低系统的运行压力。
3、线程池有那些优势

降低资源消耗：线程和任务分离，提高线程重用性
控制线程并发数量，降低服务器压力，统一管理所有线程
提高系统响应速度。假如创建线程用的时间为T1，执行任务的时间为T2，销毁线程的时间为T3，那么使用线程池就免去了T1和T3的时间。

## 6.1**ThreadPoolExecutor**

![](https://s1.ax1x.com/2022/08/10/v1j11I.png)

### 1) 线程池状态 

ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量

![](https://s1.ax1x.com/2022/08/10/v1vKbT.png)

从数字上比较，TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING .

因为第一位是符号位,RUNNING 是负数,所以最小.

这些信息存储在一个**原子变量 ctl** 中，目的是将线程池状态与线程个数合二为一，这样就可以用一次 cas 原子操作进行赋值

```java
// c 为旧值， ctlOf 返回结果为新值
ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))));

// rs 为高 3 位代表线程池状态， wc 为低 29 位代表线程个数，ctl 是合并它们
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

### 2) 构造方法

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                         RejectedExecutionHandler handler)
```

- corePoolSize 核心线程数目 (最多保留的线程数) 
- maximumPoolSize 最大线程数目 
- keepAliveTime 生存时间 - 针对救急线程 
- unit 时间单位 - 针对救急线程 
- workQueue 阻塞队列 
- threadFactory 线程工厂 - 可以为线程创建时起个好名字 
- handler 拒绝策略

**线程池的工作方式**

![](https://s1.ax1x.com/2022/08/10/v3PzBn.png)

- 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务。 
- 当线程数达到 corePoolSize 并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue 队列排队，直到有空闲的线程。 
- 如果队列选择了有界队列，那么任务超过了队列大小时，会创建 maximumPoolSize - corePoolSize 数目的线程来救急。

**线程池的拒绝策略-当线程数量达到maxmumPoolSize**

- 如果线程到达 maximumPoolSize 仍然有新任务这时会执行拒绝策略。拒绝策略 jdk 提供了 4 种实现，其它著名框架也提供了实现

- **拒绝策略**

- - AbortPolicy 让调用者抛出 RejectedExecutionException 异常，这是默认策略
  - CallerRunsPolicy 让调用者运行任务 
  - DiscardPolicy 放弃本次任务 
  - DiscardOldestPolicy 放弃队列中最早的任务，本任务取而代之 

- **框架实现**

- - Dubbo 的实现，在抛出 RejectedExecutionException 异常之前会记录日志，并 dump 线程栈信息，方便定位问题 
  - Netty 的实现，是创建一个新线程来执行任务 
  - ActiveMQ 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略 
  - PinPoint 的实现，它使用了一个拒绝策略链，会逐一尝试策略链中每种拒绝策略

- 当高峰过去后，超过corePoolSize 的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由keepAliveTime 和 unit 来控制。 

> 也就是当救急线程空闲，超过keepAliveTime +时间单位unit后就会释放线程资源



### 3) Java五种常用的线程池使用方法：

```java
    //创建单核心的线程池
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    //创建固定核心数的线程池，这里核心数 = 2
    ExecutorService executorService = Executors.newFixedThreadPool(2);
    //创建一个按照计划规定执行的线程池，这里核心数 = 2
    ExecutorService executorService = Executors.newScheduledThreadPool(2);
    //创建一个自动增长的可缓存线程池
    ExecutorService executorService = Executors.newCachedThreadPool();
    //创建一个具有抢占式操作的线程池
    ExecutorService executorService = Executors.newWorkStealingPool();
```
1. newSingleThreadExecutor

创建一个**单线程的线程池**。这个线程池只有一个线程在工作，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行，若线程出现问题，会创建一个新的线程，也就是保证线程池中只有一个正常工作的线程

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

![](https://img-blog.csdnimg.cn/20190715141812231.gif)

2. newFixedThreadPool

创建**固定大小的线程池**。可控制线程最大并发数，超出的线程会在队列中等待。线程池的大小一旦达到最大值就会保持不变。

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

![](https://img-blog.csdnimg.cn/20190715142714484.gif)

3. newCachedThreadPool

创建一个**可缓存的线程池**。如果线程池的大小超过了处理任务所需要的线程，可灵活回收空闲线程，若无可回收，则新建线程。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统能够创建的最大线程大小。

```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

情况一、并发执行五个线程

![](https://img-blog.csdnimg.cn/20190715144005908.gif)

情况二、当有空闲线程时，会复用空闲线程

![](https://img-blog.csdnimg.cn/20190715152904461.gif)

情况三、当没有空闲线程时会创建新的线程

![](https://img-blog.csdnimg.cn/20190715153120901.gif)

4. **newScheduledThreadPool**

创建一个**大小无限的线程池**。此线程池支持定时及周期性任务执行。

```java
    public ScheduledThreadPoolExecutor newScheduledThreadPool(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
```

内部有一个延时的阻塞队列来维护任务的进行，延时也就是在这里进行的。我们把创建 **newScheduledThreadPool** 的代码放出来，这样对比效果图的话，显得更加直观。

```java
    //参数2：延时的时长
    scheduledExecutorService.schedule(th_all_1, 3000, TimeUnit.MILLISECONDS);
    scheduledExecutorService.schedule(th_all_2, 2000, TimeUnit.MILLISECONDS);
    scheduledExecutorService.schedule(th_all_3, 1000, TimeUnit.MILLISECONDS);
    scheduledExecutorService.schedule(th_all_4, 1500, TimeUnit.MILLISECONDS);
    scheduledExecutorService.schedule(th_all_5, 500, TimeUnit.MILLISECONDS);
```

![](https://img-blog.csdnimg.cn/20190715155708900.gif)

5. **newWorkStealingPool**

这个是 JDK1.8 版本加入的一种线程池，stealing 翻译为抢断、窃取的意思，它实现的一个线程池和上面4种都不一样，用的是 **ForkJoinPool** 类，构造函数代码如下

```java
 	/**
     * Creates a thread pool that maintains enough threads to support
     * the given parallelism level, and may use multiple queues to
     * reduce contention. The parallelism level corresponds to the
     * maximum number of threads actively engaged in, or available to
     * engage in, task processing. The actual number of threads may
     * grow and shrink dynamically. A work-stealing pool makes no
     * guarantees about the order in which submitted tasks are
     * executed.
     *
     * @param parallelism the targeted parallelism level
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code parallelism <= 0}
     * @since 1.8
     */
    public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```

就是它是一个并行的线程池，参数中传入的是一个线程并发的数量，这里和之前就有很明显的区别，前面4种线程池都有核心线程数、最大线程数等等，而这就使用了一个并发线程数解决问题。从介绍中，还说明这个线程池不会保证任务的顺序执行，也就是 WorkStealing 的意思，抢占式的工作

![](https://img-blog.csdnimg.cn/20190715162058337.gif)

###　４)向线程池提交任务的几种方法

```java
// 执行任务
void execute(Runnable command);

// 提交任务 task，用返回值 Future 获得任务执行结果
<T> Future<T> submit(Callable<T> task);

// 提交 tasks 中所有任务
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
    throws InterruptedException;

// 提交 tasks 中所有任务，带超时时间
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                              long timeout, TimeUnit unit)
    throws InterruptedException;

// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
    throws InterruptedException, ExecutionException;

// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消，带超时时间
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
                long timeout, TimeUnit unit)
 throws InterruptedException, ExecutionException, TimeoutException;
```

### 5) 关闭线程池的几种方式

```java
/*
线程池状态变为 SHUTDOWN
- 不会接收新任务
- 但已提交任务会执行完
- 此方法不会阻塞调用线程的执行
*/
void shutdown();
```

```java
/*
线程池状态变为 STOP
- 不会接收新任务
- 会将队列中的任务返回
- 并用 interrupt 的方式中断正在执行的任务
*/
List<Runnable> shutdownNow();
```

### 6) **创建多少线程池合适** 

- 过小会导致程序不能充分地利用系统资源、容易导致饥饿 
- 过大会导致更多的线程上下文切换，占用更多内存 

#### 3.1 CPU 密集型运算 

通常采用 `cpu 核数 + 1` 能够实现最优的 CPU 利用率，+1 是保证当线程由于页缺失故障（操作系统）或其它原因 

导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费 

#### 3.2 I/O 密集型运算 

CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程 

RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。 

经验公式如下 

```
线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间
```

例如 4 核 CPU 计算时间是 50% ，其它等待时间是 50%，期望 cpu 被 100% 利用，套用公式 

```
4 * 100% * 100% / 50% = 8 
```

例如 4 核 CPU 计算时间是 10% ，其它等待时间是 90%，期望 cpu 被 100% 利用，套用公式 

```
4 * 100% * 100% / 10% = 40 
```

### 7) 任务调度线程池 ScheduledExecutorService

在『任务调度线程池』功能加入之前，可以使用 java.util.Timer 来实现定时功能，Timer 的优点在于简单易用，但 

由于所有任务都是由同一个线程来调度，因此所有任务都是串行执行的，同一时间只能有一个任务在执行，前一个任务的延迟或异常都将会影响到之后的任务。

**使用任务调度线程池**

scheduleAtFixedRate

```java
ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);

log.debug("start...");
pool.scheduleAtFixedRate(() -> {
    log.debug("running...");
}, 1, 1, TimeUnit.SECONDS);//每隔一秒执行一次
```

实际间隔时间取执行时间和间隔时间两者之间最大值

scheduleWithFixedDelay

```java
ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);
log.debug("start...");
pool.scheduleWithFixedDelay(()-> {
    log.debug("running...");
    sleep(2);
}, 1, 1, TimeUnit.SECONDS);
```

实际间隔时间等于执行时间与间隔时间相加

**利用任务调度线程池执行定时任务**

```java
// 获得当前时间
LocalDateTime now = LocalDateTime.now();
// 获取本周四 18:00:00.000
LocalDateTime thursday = 
    now.with(DayOfWeek.THURSDAY).withHour(18).withMinute(0).withSecond(0).withNano(0);
// 如果当前时间已经超过 本周四 18:00:00.000， 那么找下周四 18:00:00.000
if(now.compareTo(thursday) >= 0) {
    thursday = thursday.plusWeeks(1);
}

// 计算时间差，即延时执行时间
long initialDelay = Duration.between(now, thursday).toMillis();
// 计算间隔时间，即 1 周的毫秒值
long oneWeek = 7 * 24 * 3600 * 1000;

ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);
System.out.println("开始时间：" + new Date());

executor.scheduleAtFixedRate(() -> {
    System.out.println("执行时间：" + new Date());
}, initialDelay, oneWeek, TimeUnit.MILLISECONDS);
```



### 8) 使用多线程实现递归

Fork/Join 是 JDK 1.7 加入的新的线程池实现，它体现的是一种分治思想，适用于能够进行任务拆分的 cpu 密集型运算 

所谓的任务拆分，是将一个大任务拆分为算法上相同的小任务，直至不能拆分可以直接求解。跟递归相关的一些计算，如归并排序、斐波那契数列、都可以用分治思想进行求解 

Fork/Join 在分治的基础上加入了多线程，可以把每个任务的分解和合并交给不同的线程来完成，进一步提升了运算效率 

Fork/Join 默认会创建与 cpu 核心数大小相同的线程池

 **使用** 

提交给 Fork/Join 线程池的任务需要继承 RecursiveTask（有返回值）或 RecursiveAction（没有返回值），例如下面定义了一个对 1~n 之间的整数求和的任务

```java
@Slf4j(topic = "c.AddTask")
class AddTask1 extends RecursiveTask<Integer> {
    int n;
    
    public AddTask1(int n) {
        this.n = n;
    }
    
    @Override
    public String toString() {
        return "{" + n + '}';
    }
    
    @Override
    protected Integer compute() {
        // 如果 n 已经为 1，可以求得结果了
        if (n == 1) {
            log.debug("join() {}", n);
            return n;
        }
        
        // 将任务进行拆分(fork)
        AddTask1 t1 = new AddTask1(n - 1);
        t1.fork();
        log.debug("fork() {} + {}", n, t1);
        
        // 合并(join)结果
        int result = n + t1.join();
        log.debug("join() {} + {} = {}", n, t1, result);
        return result;
    }
}
```

```java
public static void main(String[] args) {
    ForkJoinPool pool = new ForkJoinPool(4);//提交给ForkJoinPool
    System.out.println(pool.invoke(new AddTask1(5)));
}
```

![](https://s1.ax1x.com/2022/08/10/v32Ezd.png)

**使用二分查找改进**

```java
class AddTask3 extends RecursiveTask<Integer> {
    
    int begin;
    int end;
    
    public AddTask3(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }
    
    @Override
    public String toString() {
        return "{" + begin + "," + end + '}';
    }
    
    @Override
    protected Integer compute() {
        // 5, 5
        if (begin == end) {
            log.debug("join() {}", begin);
            return begin;
        }
        // 4, 5
        if (end - begin == 1) {
            log.debug("join() {} + {} = {}", begin, end, end + begin);
            return end + begin;
        }
        
        // 1 5
        int mid = (end + begin) / 2; // 3
        AddTask3 t1 = new AddTask3(begin, mid); // 1,3
        t1.fork();
        AddTask3 t2 = new AddTask3(mid + 1, end); // 4,5
        t2.fork();
        log.debug("fork() {} + {} = ?", t1, t2);
        int result = t1.join() + t2.join();
        log.debug("join() {} + {} = {}", t1, t2, result);
        return result;
    }
}
```

![](https://s1.ax1x.com/2022/08/10/v325fe.png)

## 6.2 * AQS 原理

### 1）AQS(AbstractQueuedSynchronizer) 原理

AQS 全称是 AbstractQueuedSynchronizer，是阻塞式锁和相关的同步器工具的框架 

AQS 是**用来构建锁（ReentrantLock）或者其他同步器组件（如CountDownLatch）的重量级框架及整个JUC体系的基石**。通过内置的FIFO队列来完成资源获取线程的排队工作，并通过一个int类型变量表示持有锁的状态。

![用到AQS的工具类](https://s1.ax1x.com/2022/08/10/v3WigH.png)

**AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

CLH 队列的好处： 

- 无锁，使用自旋
- 快速，无阻塞 

![](https://s1.ax1x.com/2022/08/10/v3h0BR.md.png)

### 2）AQS设计思想

要点 

- 原子维护 state 状态 
- 阻塞及恢复线程 
- 维护队列 

1. 利用int类型标识状态。**在AQS类中有一个叫做state的成员变量**，基于AQS有一个同步组件，叫做ReentrantLock。**在这个组件里，stste表示获取锁的线程数，假如state=0，表示还没有线程获取锁，1表示有线程获取了锁。大于1表示重入锁的数量。**

```java
  /**
   * The synchronization state.
   */
  private volatile int state;
```

关于state的方法

- getState()
- setState()
- compareAndSetState()



2. AQS定义两种资源共享方式：**Exclusive**（独占，只有一个线程能执行，如ReentrantLock）和**Share**（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

- isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
- tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
- tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

### 3 ）AQS执行流程

线程会首先尝试获取锁，如果失败就将当前线程及等待状态等信息包装成一个node节点加入到同步队列sync queue里。 接着会不断的循环尝试获取锁，条件是当前节点为head的直接后继才会尝试。如果失败就会阻塞自己直到自己被唤醒。而当持有锁的线程释放锁的时候，会唤醒队列中的后继线程。

详见-> AQS源码解析 [AQS源码解析](https://www.cnblogs.com/waterystone/p/4920797.html)



## 6.3 ReentrantLock原理

### 加锁流程（非公平锁） 

**第一步**：通过cas获取锁，也就是改变status的状态 （失败） ==第一次尝试==

**第二步**：进入 tryAcquire 逻辑，这时 state 已经是1，结果仍然失败  ==第二次尝试==

```java
 protected boolean tryAcquire(int arg) {
         throw new UnsupportedOperationException();
    }//这是个接口，独占模式下只用实现tryAcquire-tryRelease，而共享模式下只用实现tryAcquireShared-tryReleaseShared
```

**第三步**：执行addWaiter，将当前线程加入到等待队列的队尾

```java
private Node addWaiter(Node mode) {
    //以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
    Node node = new Node(Thread.currentThread(), mode);
    //尝试快速方式直接放到队尾。
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //上一步失败则通过enq入队。
    enq(node);
    return node;
}
```
**第四步：**上一步快速入队失败则通过enq入队

```java
private Node enq(final Node node) {
    //CAS"自旋"，直到成功加入队尾
    for (;;) {
        Node t = tail;
        if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {//正常流程，放入队尾
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

**第五步：**执行 acquireQueued ，准备进入等待状态休息

通过tryAcquire()和addWaiter()，该线程获取资源失败，已经被放入等待队列尾部了，执行acquireQueued，判断当前节点，如果是队列中head指向的节点，那么就再次尝试获取锁 ==第三次尝试==

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;//标记是否成功拿到资源
    try {
        boolean interrupted = false;//标记等待过程中是否被中断过

        //又是一个“自旋”！
        for (;;) {
            final Node p = node.predecessor();//拿到前驱
            //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
            if (p == head && tryAcquire(arg)) {
                setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                p.next = null; // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                failed = false; // 成功获取资源
                return interrupted;//返回等待过程中是否被中断过
            }

            //如果自己可以休息了，就通过park()进入waiting状态，直到被unpark()。如果不可中断的情况下被中断了，那么会从park()中醒过来，发现拿不到资源，从而继续进入park()等待。
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
        }
    } finally {
        if (failed) // 如果等待过程中没有成功获取资源（如timeout，或者可中断的情况下被中断了），那么取消结点在队列中的等待。
            cancelAcquire(node);
    }
}
```

**第六步**：执行shouldParkAfterFailedAcquire()和parkAndCheckInterrupt()

shouldParkAfterFailedAcquire主要就是让前驱节点拿到锁以后通知自己一声

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//拿到前驱的状态
    if (ws == Node.SIGNAL)
        //如果已经告诉前驱拿完号后通知自己一下，那就可以安心休息了
        return true;
    if (ws > 0) {
        /*
         * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
         * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被保安大叔赶走了(GC回收)！
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
         //如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下。有可能失败，人家说不定刚刚释放完呢！
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

parkAndCheckInterrupt此方法就是让线程去休息，真正进入等待状态。

```java
 private final boolean parkAndCheckInterrupt() {
     LockSupport.park(this);//调用park()使线程进入waiting状态
     return Thread.interrupted();//如果被唤醒，查看自己是不是被中断的。
 }
```

### 解锁流程

**第一步**：调用unlock

```java
// 解锁实现
    public void unlock() {
        sync.release(1);
    }
    
```

**第二步**：执行relase

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```

**第三步**：执行tryRelase

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```

**第四步**：执行tryRelease(int)

```java
 protected boolean tryRelease(int arg) {
     throw new UnsupportedOperationException();
 }
```

跟tryAcquire()一样，这个方法是需要独占模式的自定义同步器去实现的，该线程来释放资源，因为是独占模式，所以直接释放资源就可以了，要注意返回值，**release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了**

**第五步**：执行unparkSuccessor(Node)

此方法用于唤醒等待队列中下一个线程

**最后一步**：unparkSuccessor(Node)

此方法用于唤醒等待队列中下一个线程

```java
private void unparkSuccessor(Node node) {
    //这里，node一般为当前线程所在的结点。
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev) // 从后向前找。
            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```

### 可重入原理

```java
static final class NonfairSync extends Sync {
    // ...
    
    // Sync 继承过来的方法, 方便阅读, 放在此处
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果已经获得了锁, 线程还是当前线程, 表示发生了锁重入
        else if (current == getExclusiveOwnerThread()) {
            // state++
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    
```

释放重入锁

```java
 // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryRelease(int releases) {
        // state-- 
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        // 支持锁重入, 只有 state 减为 0, 才释放成功
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
}
```



### 不可打断原理

不可打断就是interrupted之后，阻塞队列中的线程只有获得了锁之后才知道被打断了

```java
// Sync 继承自 AQS
static final class NonfairSync extends Sync {
    // ...
    
    private final boolean parkAndCheckInterrupt() {
        // 如果打断标记已经是 true, 则 park 会失效
        LockSupport.park(this);
        // interrupted 会清除打断标记
        return Thread.interrupted();
    }
    
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null;
                    failed = false;
                    // 还是需要获得锁后, 才能返回打断状态
                    return interrupted;
                }
                if (
                    shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt()
                ) {
                    // 如果是因为 interrupt 被唤醒, 返回打断状态为 true
                    interrupted = true;
                }
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    
    public final void acquire(int arg) {
        if (
            !tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        ) {
            // 如果打断状态为 true
            selfInterrupt();
        }
    }
    
    static void selfInterrupt() {
        // 重新产生一次中断
        Thread.currentThread().interrupt();
    }
}
```

### 可打断模式

```java
static final class NonfairSync extends Sync {
    public final void acquireInterruptibly(int arg) throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        // 如果没有获得到锁, 进入 ㈠
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }
    
    // ㈠ 可打断的获取锁流程
    private void doAcquireInterruptibly(int arg) throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt()) {
                    // 在 park 过程中如果被 interrupt 会进入此
                    // 这时候抛出异常, 而不会再次进入 for (;;)
                    throw new InterruptedException();
                }
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
}
```

### 公平锁实现原理

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;
    final void lock() {
        acquire(1);
    }
    
    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final void acquire(int arg) {
        if (
            !tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        ) {
            selfInterrupt();
        }
    }
    // 与非公平锁主要区别在于 tryAcquire 方法的实现
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 先检查 AQS 队列中是否有前驱节点, 没有才去竞争
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
    
    // ㈠ AQS 继承过来的方法, 方便阅读, 放在此处
    public final boolean hasQueuedPredecessors() {
        Node t = tail;
        Node h = head;
        Node s;
        // h != t 时表示队列中有 Node
        return h != t &&
            (
            // (s = h.next) == null 表示队列中还有没有老二
            (s = h.next) == null ||
            // 或者队列中老二线程不是此线程
            s.thread != Thread.currentThread()
        );
    }
}
```

也就是只有head后面的老二才能取获取锁

### **条件变量实现原理** 

每个条件变量其实就对应着一个等待队列，其实现类是 ConditionObject 

ConditionObject 中存放的是已经获取锁，但是由于条件不满足而再次陷入等待的线程

#### await 流程 

1. 开始 Thread-0 持有锁，调用 await，进入 ConditionObject 的 addConditionWaiter 流程 

2. 创建新的 Node 状态为 -2（Node.CONDITION），关联 Thread-0，加入等待队列尾部

![](https://s1.ax1x.com/2022/08/10/v8Vp80.md.png)

3. 接下来进入 AQS 的 fullyRelease 流程(有可能是重入锁)，释放同步器上的锁

![](https://s1.ax1x.com/2022/08/10/v8VCvT.md.png)

4.  unpark AQS 队列中的下一个节点，竞争锁，假设没有其他竞争线程，那么 Thread-1 竞争成功

5. park 阻塞 Thread-0

![](https://s1.ax1x.com/2022/08/10/v8Vkb4.md.png)

#### signal 流程 

1. 假设 Thread-1 要来唤醒 Thread-0

![](https://s1.ax1x.com/2022/08/10/v8Vkb4.md.png)

2. 进入 ConditionObject 的 doSignal 流程，取得等待队列中第一个 Node，即 Thread-0 所在 Node

![](https://s1.ax1x.com/2022/08/10/v8V8VH.png)

3. 执行 transferForSignal 流程，将该 Node 加入 AQS 队列尾部，将 Thread-0 的 waitStatus 改为 0，Thread-3 的waitStatus 改为 -1

   ![](https://s1.ax1x.com/2022/08/10/v8VURP.png)

源码部分

```java
public class ConditionObject implements Condition, java.io.Serializable {
    private static final long serialVersionUID = 1173984872572414699L;
    
    // 第一个等待节点
    private transient Node firstWaiter;
    
    // 最后一个等待节点
    private transient Node lastWaiter;
    public ConditionObject() { }
    // ㈠ 添加一个 Node 至等待队列
    private Node addConditionWaiter() {
        Node t = lastWaiter;
        // 所有已取消的 Node 从队列链表删除, 见 ㈡
        if (t != null && t.waitStatus != Node.CONDITION) {
            unlinkCancelledWaiters();
            t = lastWaiter;
        }
        // 创建一个关联当前线程的新 Node, 添加至队列尾部
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        if (t == null)
            firstWaiter = node;
        else
            t.nextWaiter = node;
        lastWaiter = node;
        return node;
    }
    // 唤醒 - 将没取消的第一个节点转移至 AQS 队列
    private void doSignal(Node first) {
        do {
            // 已经是尾节点了
            if ( (firstWaiter = first.nextWaiter) == null) {
                lastWaiter = null;
            }
            first.nextWaiter = null;
        } while (
            // 将等待队列中的 Node 转移至 AQS 队列, 不成功且还有节点则继续循环 ㈢
            !transferForSignal(first) &&
            // 队列还有节点
            (first = firstWaiter) != null
        );
    }
    
    // 外部类方法, 方便阅读, 放在此处
    // ㈢ 如果节点状态是取消, 返回 false 表示转移失败, 否则转移成功
    final boolean transferForSignal(Node node) {
        // 如果状态已经不是 Node.CONDITION, 说明被取消了
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
        // 加入 AQS 队列尾部
        Node p = enq(node);
        int ws = p.waitStatus;
        if (
            // 上一个节点被取消
            ws > 0 ||
            // 上一个节点不能设置状态为 Node.SIGNAL
            !compareAndSetWaitStatus(p, ws, Node.SIGNAL) 
        ) {
            // unpark 取消阻塞, 让线程重新同步状态
            LockSupport.unpark(node.thread);
        }
        return true;
    }
    // 全部唤醒 - 等待队列的所有节点转移至 AQS 队列
    private void doSignalAll(Node first) {
        lastWaiter = firstWaiter = null;
        do {
            Node next = first.nextWaiter;
            first.nextWaiter = null;
            transferForSignal(first);
            first = next;
        } while (first != null);
    }
    
    // ㈡
    private void unlinkCancelledWaiters() {
        // ...
    }
    // 唤醒 - 必须持有锁才能唤醒, 因此 doSignal 内无需考虑加锁
    public final void signal() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignal(first);
    }
    // 全部唤醒 - 必须持有锁才能唤醒, 因此 doSignalAll 内无需考虑加锁
    public final void signalAll() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignalAll(first);
    }
    // 不可打断等待 - 直到被唤醒
    public final void awaitUninterruptibly() {
        // 添加一个 Node 至等待队列, 见 ㈠
        Node node = addConditionWaiter();
        // 释放节点持有的锁, 见 ㈣
        int savedState = fullyRelease(node);
        boolean interrupted = false;
        // 如果该节点还没有转移至 AQS 队列, 阻塞
        while (!isOnSyncQueue(node)) {
            // park 阻塞
            LockSupport.park(this);
            // 如果被打断, 仅设置打断状态
            if (Thread.interrupted())
                interrupted = true;
        }
        // 唤醒后, 尝试竞争锁, 如果失败进入 AQS 队列
        if (acquireQueued(node, savedState) || interrupted)
            selfInterrupt();
    }
    // 外部类方法, 方便阅读, 放在此处
    // ㈣ 因为某线程可能重入，需要将 state 全部释放
    final int fullyRelease(Node node) {
        boolean failed = true;
        try {
            int savedState = getState();
            if (release(savedState)) {
                failed = false;
                return savedState;
            } else {
                throw new IllegalMonitorStateException();
            }
        } finally {
            if (failed)
                node.waitStatus = Node.CANCELLED;
        }
    }
    // 打断模式 - 在退出等待时重新设置打断状态
    private static final int REINTERRUPT = 1;
    // 打断模式 - 在退出等待时抛出异常
    private static final int THROW_IE = -1;
    // 判断打断模式
    private int checkInterruptWhileWaiting(Node node) {
        return Thread.interrupted() ?
            (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
    }
    // ㈤ 应用打断模式
    private void reportInterruptAfterWait(int interruptMode)
        throws InterruptedException {
        if (interruptMode == THROW_IE)
            throw new InterruptedException();
        else if (interruptMode == REINTERRUPT)
            selfInterrupt();
    }
    // 等待 - 直到被唤醒或打断
    public final void await() throws InterruptedException {
        if (Thread.interrupted()) {
            throw new InterruptedException();
        }
        // 添加一个 Node 至等待队列, 见 ㈠
        Node node = addConditionWaiter();
        // 释放节点持有的锁
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        // 如果该节点还没有转移至 AQS 队列, 阻塞
        while (!isOnSyncQueue(node)) {
            // park 阻塞
            LockSupport.park(this);
            // 如果被打断, 退出等待队列
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        // 退出等待队列后, 还需要获得 AQS 队列的锁
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        // 所有已取消的 Node 从队列链表删除, 见 ㈡
        if (node.nextWaiter != null) 
            unlinkCancelledWaiters();
        // 应用打断模式, 见 ㈤
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
    // 等待 - 直到被唤醒或打断或超时
    public final long awaitNanos(long nanosTimeout) throws InterruptedException {
        if (Thread.interrupted()) {
            throw new InterruptedException();
        }
        // 添加一个 Node 至等待队列, 见 ㈠
        Node node = addConditionWaiter();
        // 释放节点持有的锁
        int savedState = fullyRelease(node);
        // 获得最后期限
        final long deadline = System.nanoTime() + nanosTimeout;
        int interruptMode = 0;
        // 如果该节点还没有转移至 AQS 队列, 阻塞
        while (!isOnSyncQueue(node)) {
            // 已超时, 退出等待队列
            if (nanosTimeout <= 0L) {
                transferAfterCancelledWait(node);
                break;
            }
            // park 阻塞一定时间, spinForTimeoutThreshold 为 1000 ns
            if (nanosTimeout >= spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            // 如果被打断, 退出等待队列
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
            nanosTimeout = deadline - System.nanoTime();
        }
        // 退出等待队列后, 还需要获得 AQS 队列的锁
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        // 所有已取消的 Node 从队列链表删除, 见 ㈡
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        // 应用打断模式, 见 ㈤
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return deadline - System.nanoTime();
    }
    // 等待 - 直到被唤醒或打断或超时, 逻辑类似于 awaitNanos
    public final boolean awaitUntil(Date deadline) throws InterruptedException {
        // ...
    }
    // 等待 - 直到被唤醒或打断或超时, 逻辑类似于 awaitNanos
    public final boolean await(long time, TimeUnit unit) throws InterruptedException {
        // ...
    }
    // 工具方法 省略 ...
}
```



## 6.5 ReentrantReadWriteLock

当读操作远远高于写操作时，这时候使用 `读写锁` 让 `读-读` 可以并发，提高性能

### 1）**读写锁的用法：**

```java
ReadWriteLock readWriteLock = new ReentrantReadWriteLock(); 
Lock readLock = readWriteLock.readLock(); 
readLock.lock(); 
// 进行读取操作 
readLock.unlock(); 

Lock writeLock = readWriteLock.writeLock(); 
writeLock.lock(); 
// 进行写操作 
writeLock.unlock();
```

**StampedLock**

该类自 JDK 8 加入，是为了进一步优化读性能，它的特点是在使用读锁、写锁时可以**配合【戳】**使用加解读锁

戳和cas思想类似

乐观读，StampedLock 支持 `tryOptimisticRead()` 方法（乐观读），读取完毕后需要做一次 `戳校验` 如果校验通过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全。

```java
private final StampedLock lock = new StampedLock();
//加锁
long stamp = lock.readLock();
lock.unlockRead(stamp);
//释放锁
long stamp = lock.writeLock();
lock.unlockWrite(stamp);
```



也就是说，当使用 ReadWriteLock 的时候，并不是直接使用，而是获得其内部的读锁和写锁，然后分别调用lock/unlock



### 2）应用之缓存

1. 缓存更新策略 

更新时，是先清缓存还是先更新数据库 

![](https://s1.ax1x.com/2022/08/11/vGBLuV.png)

如果先清缓存，会出现以上问题，所以，先更新数据库是正确的选择

但是即使是这样，也有可能出现缓存不一致的现象，这时可以用读写锁来解决

### 3）* 读写锁原理 

读写锁用的是同一个 Sycn 同步器，因此等待队列、state 等也是同一个

==写锁状态占了 state 的低 16 位，而读锁使用的是 state 的高 16 位==

比如t1获取写锁，t2获取读锁

1. t1 成功上写锁，流程与 ReentrantLock 加锁相比没有特殊之处，

![](https://s1.ax1x.com/2022/08/11/vGcxUK.png)

2. t2 执行 r.lock，这时进入读锁的 sync.acquireShared(1) 流程，首先会进入tryAcquireShared 流程。如果有写锁占据，那么 tryAcquireShared 返回 -1 表示失败 

   ![](https://s1.ax1x.com/2022/08/11/vGgqJS.png)

   > tryAcquireShared 返回值表示
   >
   > - -1 表示失败
   > - 0 表示成功，但后继节点不会继续唤醒 
   > - 正数表示成功，而且数值是还有几个后继节点需要唤醒，读写锁返回 1

3. 这时会进入 sync.doAcquireShared(1) 流程，首先也是调用 addWaiter 添加节点，不同之处在于节点被设置为Node.SHARED 模式而非 Node.EXCLUSIVE 模式，注意此时 t2 仍处于活跃状态

   ![](https://s1.ax1x.com/2022/08/11/vG2SZq.png)

4. t2 会看看自己的节点是不是老二，如果是，还会再次调用 tryAcquireShared(1) 来尝试获取锁 

5. 如果没有成功，在 doAcquireShared 内 for (;;) 循环一次，把前驱节点的 waitStatus 改为 -1，再 for (;;) 循环一次尝试 tryAcquireShared(1) 如果还不成功，那么在 parkAndCheckInterrupt() 处 park

   ![](https://s1.ax1x.com/2022/08/11/vG2UTP.md.png)

   6. t3 r.lock，t4 w.lock 这种状态下，假设又有 t3 加读锁和 t4 加写锁，这期间 t1 仍然持有锁，就变成了下面的样子

      ![](https://s1.ax1x.com/2022/08/11/vG2rlQ.md.png)

   当t1释放锁，会唤醒t2以及后面所有的shared节点，变成下面的情况

   ![](https://s1.ax1x.com/2022/08/11/vGWu8O.md.png)

   下一个节点不是 shared 了，因此不会继续唤醒 t4 所在节点 

   当t2 t3都释放读锁之后，t4才会被唤醒

![](https://s1.ax1x.com/2022/08/11/vGWJat.png)

读锁能够共享的原因是

具体执行流程： [读写锁原理](https://www.yuque.com/mo_ming/gl7b70/yuw120#E4YVv)

源码： [读写锁源码](https://www.yuque.com/mo_ming/gl7b70/yuw120#Fvwax)

## 6.6 Semaphore

semaphore 信号量，用来限制能同时访问共享资源的线程上限。

### 1）基本用法

```java
public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);//创建Semaphore对象

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();//获得semaphore
                    log.debug("running...");
                    Thread.sleep(1000);
                    log.debug("end...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();//释放semaphore
                }
            }).start();
        }
    }
```

应用： 可以利用Semaphore实现限制线程池连接数的一个功能，在获取线程池连接时调用`semaphore.acquire();`

### 2）原理

semaphore类似于停车场

刚开始，permits（state）为 3，这时 5 个线程来获取资源

![](https://s1.ax1x.com/2022/08/12/vJ7nyV.md.png)

假设其中 Thread-1，Thread-2，Thread-4 cas 竞争成功，而 Thread-0 和 Thread-3 竞争失败，进入 AQS 队列 park 阻塞

![](https://s1.ax1x.com/2022/08/12/vJ7lo4.png)

当Thread执行完，释放semaphore时，Thread-0会被唤醒

![](https://s1.ax1x.com/2022/08/12/vJ73FJ.png)

![](https://s1.ax1x.com/2022/08/12/vJ7sfA.png)

实际上state底层是基于AQS实现的

**源码分析**

```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -2694183684443567898L;
    NonfairSync(int permits) {
        // permits 即 state
        super(permits);
    }
    
    // Semaphore 方法, 方便阅读, 放在此处
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);//////////////
    }
    
    // 尝试获得共享锁
    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
    
    // Sync 继承过来的方法, 方便阅读, 放在此处
    final int nonfairTryAcquireShared(int acquires) {
        for (;;) {
            int available = getState();
            int remaining = available - acquires; 
            if (
                // 如果许可已经用完, 返回负数, 表示获取失败, 进入 doAcquireSharedInterruptibly
                remaining < 0 ||
                // 如果 cas 重试成功, 返回正数, 表示获取成功
                compareAndSetState(available, remaining)
            ) {
                return remaining;
            }
        }
    }
    
    // AQS 继承过来的方法, 方便阅读, 放在此处
    private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    // 再次尝试获取许可
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        // 成功后本线程出队（AQS）, 所在 Node设置为 head
                        // 如果 head.waitStatus == Node.SIGNAL ==> 0 成功, 下一个节点 unpark
                        // 如果 head.waitStatus == 0 ==> Node.PROPAGATE 
                        // r 表示可用资源数, 为 0 则不会继续传播
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                // 不成功, 设置上一个节点 waitStatus = Node.SIGNAL, 下轮进入 park 阻塞
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    
    // Semaphore 方法, 方便阅读, 放在此处
    public void release() {
        sync.releaseShared(1);
    }
    
    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
    
    // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryReleaseShared(int releases) {
        for (;;) {
            int current = getState();
            int next = current + releases;
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            if (compareAndSetState(current, next))
                return true;
        }
    }
}
```



## 6.7 CountDownLatch

用来进行线程同步协作，等待所有线程完成倒计时。 

其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一

### 1）基本用法

```java
CountDownLatch latch = new CountDownLatch(3);//需要被等待的线程数为3
线程1{
    ...
    latch.countDown();//让countdownlatch计数减一
}
线程2{
    ...
    latch.countDown();//让countdownlatch计数减一
}
线程3{
    ...
    latch.countDown();//让countdownlatch计数减一
}
等待线程{
    latch.await();
}
```

为什么不用join？

1. join 属于底层API，使用繁琐
2. join不适用于线程池

### 2）应用

应用之同步等待多线程准备完毕

应用之同步等待多个远程调用结束

### 3）用Future实现

可以在main函数中接受线程传过来的参数，也可以达到与CountDownLatch相同的效果

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        //testCountDown();
        ExecutorService service = Executors.newFixedThreadPool(4);
        CountDownLatch latch = new CountDownLatch(4);
        Future<String> future = service.submit(() -> {
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Thread1";
        });
        Future<String> future2 = service.submit(() -> {
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Thread2";
        });
        Future<String> future3 = service.submit(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Thread3";
        });
        Future<String> future4 = service.submit(() -> {
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Thread4";
        });
        log.debug(future.get()+","+future2.get()+","+future3.get()+","+future4.get());
        service.shutdown();
    }
```



### 3）CyclicBarrier

循环栅栏，用来进行线程协作，等待线程满足某个计数。构造时设置『计数个数』，每个线程执行到某个需要“同步”的时刻调用 await() 方法进行等待，当等待的线程数满足『计数个数』时，继续执行.

```java
CyclicBarrier cb = new CyclicBarrier(2); // 个数为2时才会继续执行

new Thread(()->{
    System.out.println("线程1开始.."+new Date());
    try {
        cb.await(); // 当个数不足时，等待
    } catch (InterruptedException | BrokenBarrierException e) {
        e.printStackTrace();
    }
    System.out.println("线程1继续向下运行..."+new Date());
}).start();

new Thread(()->{
    System.out.println("线程2开始.."+new Date());
    try { 
        Thread.sleep(2000); 
    } catch (InterruptedException e) {
    }
    try {
        cb.await(); // 2 秒后，线程个数够2，继续运行
    } catch (InterruptedException | BrokenBarrierException e) {
        e.printStackTrace();
    }
    System.out.println("线程2继续向下运行..."+new Date());
}).start();
```

## 6.8 ConcurrentHashMap

**要点：**

- HashTable和Vector是线程安全，但内部使用synchronize，性能比较低，属于遗留类
- ConcurrentHashMap是线程安全，但是只是单个方法线程安全
- jdk7 中hashmap采用数组＋链表实现，头插法插入链表，当元素长度超过3/4总长度，发生扩容，多线程下可能发生死链
- jdk8 中hashmap采用数组＋链表＋红黑树实现，尾插法插入链表，当链表长度超过8，如果map长度没有超过64，则发生扩容，否则把链表转成红黑树
- jdk8中hashmap发生扩容时，会从map尾部发生转移元素，转移后标记ForwardingNode



### 1）JDK 8 ConcurrentHashMap 

重要属性和内部类

```java
// 默认为 0
// 当初始化时, 为 -1
// 当扩容时, 为 -(1 + 扩容线程数)
// 当初始化或扩容完成后，为 下一次的扩容的阈值大小
private transient volatile int sizeCtl;

// 整个 ConcurrentHashMap 就是一个 Node[]
static class Node<K,V> implements Map.Entry<K,V> {}

// hash 表
transient volatile Node<K,V>[] table;

// 扩容时的 新 hash 表
private transient volatile Node<K,V>[] nextTable;

// 扩容时如果某个 bin 迁移完毕, 用 ForwardingNode 作为旧 table bin 的头结点
static final class ForwardingNode<K,V> extends Node<K,V> {}

// 用在 compute 以及 computeIfAbsent 时, 用来占位, 计算完成后替换为普通 Node
static final class ReservationNode<K,V> extends Node<K,V> {}

// 作为 treebin 的头节点, 存储 root 和 first
static final class TreeBin<K,V> extends Node<K,V> {}

// 作为 treebin 的节点, 存储 parent, left, right
static final class TreeNode<K,V> extends Node<K,V> {}
```

**重要方法**

```java
// 获取 Node[] 中第 i 个 Node
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i)
 
// cas 修改 Node[] 中第 i 个 Node 的值, c 为旧值, v 为新值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v)
 
// 直接修改 Node[] 中第 i 个 Node 的值, v 为新值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v)
```

**构造器分析** 

可以看到实现了懒惰初始化，在构造方法中仅仅计算了 table 的大小，以后在第一次使用时才会真正创建

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel) // Use at least as many bins
        initialCapacity = concurrencyLevel; // as estimated threads
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    // tableSizeFor 仍然是保证计算的大小是 2^n, 即 16,32,64 ... 
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap; 
}
```



**get流程 （无锁）**

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // spread 方法能确保返回结果是正数
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 如果头结点已经是要查找的 key
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        // hash 为负数表示该 bin 在扩容中或是 treebin, 这时调用 find 方法来查找
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        // 正常遍历链表, 用 equals 比较
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

put流程 每个链表都有一把锁

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    // 其中 spread 方法会综合高位低位, 具有更好的 hash 性
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        // f 是链表头节点
        // fh 是链表头结点的 hash
        // i 是链表在 table 中的下标
        Node<K,V> f; int n, i, fh;
        // 要创建 table
        if (tab == null || (n = tab.length) == 0)
            // 初始化 table 使用了 cas, 无需 synchronized 创建成功, 进入下一轮循环
            tab = initTable();
        // 要创建链表头节点
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 添加链表头使用了 cas, 无需 synchronized
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;
        }
        // 帮忙扩容
        else if ((fh = f.hash) == MOVED)
            // 帮忙之后, 进入下一轮循环
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 锁住链表头节点
            synchronized (f) {
                // 再次确认链表头节点没有被移动
                if (tabAt(tab, i) == f) {
                    // 链表
                    if (fh >= 0) {
                        binCount = 1;
                        // 遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 找到相同的 key
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                // 更新
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            // 已经是最后的节点了, 新增 Node, 追加至链表尾
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 红黑树
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        // putTreeVal 会看 key 是否已经在树中, 是, 则返回对应的 TreeNode
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
                // 释放链表头节点的锁
            }
            
            if (binCount != 0) { 
                if (binCount >= TREEIFY_THRESHOLD)
                    // 如果链表长度 >= 树化阈值(8), 进行链表转为红黑树
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 增加 size 计数
    addCount(1L, binCount);
    return null; 
}

private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield();
        // 尝试将 sizeCtl 设置为 -1（表示初始化 table）
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            // 获得锁, 创建 table, 这时其它线程会在 while() 循环中 yield 直至 table 创建
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab; 
}

// check 是之前 binCount 的个数
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    if (
        // 已经有了 counterCells, 向 cell 累加
        (as = counterCells) != null ||
        // 还没有, 向 baseCount 累加
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)
    ) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (
            // 还没有 counterCells
            as == null || (m = as.length - 1) < 0 ||
            // 还没有 cell
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            // cell cas 增加计数失败
            !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
        ) {
            // 创建累加单元数组和cell, 累加重试
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        // 获取元素个数
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                // newtable 已经创建了，帮忙扩容
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            // 需要扩容，这时 newtable 未创建
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}

```

**size 计算流程** 

size 计算实际发生在 put，remove 改变集合元素的操作之中 

- 没有竞争发生，向 baseCount 累加计数 
- 有竞争发生，新建 counterCells，向其中的一个 cell 累加计数 

- - counterCells 初始有两个 cell 
  - 如果计数竞争比较激烈，会创建新的 cell 来累加计数

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}

final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    // 将 baseCount 计数与所有 cell 计数累加
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

总结：

**Java 8** 数组（Node） +（ 链表 Node | 红黑树 TreeNode ） 以下数组简称（table），链表简称（bin） 

- 初始化，使用 cas 来保证并发安全，懒惰初始化 table 
- 树化，当 table.length < 64 时，先尝试扩容，超过 64 时，并且 bin.length > 8 时，会将链表树化，树化过程会用 synchronized 锁住链表头 
- put，如果该 bin 尚未创建，只需要使用 cas 创建 bin；如果已经有了，锁住链表头进行后续 put 操作，元素添加至 bin 的尾部 
- get，无锁操作仅需要保证可见性，扩容过程中 get 操作拿到的是 ForwardingNode 它会让 get 操作在新 table 进行搜索 
- 扩容，扩容时以 bin 为单位进行，需要对 bin 进行 synchronized，但这时妙的是其它竞争线程也不是无事可做，它们会帮助把其它 bin 进行扩容，扩容时平均只有 1/6 的节点会把复制到新 table 中 
- size，元素个数保存在 baseCount 中，并发时的个数变动保存在 CounterCell[] 当中。最后统计数量时累加即可

### 2）JDK 7 ConcurrentHashMap 

它维护了一个 segment 数组，每个 segment 对应一把锁 

- 优点：如果多个线程访问不同的 segment，实际是没有冲突的，这与 jdk8 中是类似的 
- 缺点：Segments 数组默认大小为16，这个容量初始化指定后就不能改变了，并且不是懒惰初始化 

**构造器分析**

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // ssize 必须是 2^n, 即 2, 4, 8, 16 ... 表示了 segments 数组的大小
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // segmentShift 默认是 32 - 4 = 28
    this.segmentShift = 32 - sshift;
    // segmentMask 默认是 15 即 0000 0000 0000 1111
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;
    // 创建 segments and segments[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss; 
}
```

 ConcurrentHashMap 没有实现懒惰初始化，空间占用不友好 

其中 this.segmentShift 和 this.segmentMask 的作用是决定将 key 的 hash 结果匹配到哪个 segment 

例如，根据某一 hash 值求 segment 位置，先将高位向低位移动 this.segmentShift 位

![](https://s1.ax1x.com/2022/08/14/vU1A7d.png)

实际上就是将`key`与15取模

**put流程**

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    // 计算出 segment 下标
    int j = (hash >>> segmentShift) & segmentMask;
    
    // 获得 segment 对象, 判断是否为 null, 是则创建该 segment
    if ((s = (Segment<K,V>)UNSAFE.getObject 
         (segments, (j << SSHIFT) + SBASE)) == null) {
        // 这时不能确定是否真的为 null, 因为其它线程也发现该 segment 为 null,
        // 因此在 ensureSegment 里用 cas 方式保证该 segment 安全性
        s = ensureSegment(j);
    }
    // 进入 segment 的put 流程
    return s.put(key, hash, value, false);
}
```

segment 继承了可重入锁（ReentrantLock），它的 put 方法为

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 尝试加锁
    HashEntry<K,V> node = tryLock() ? null :
    // 如果不成功, 进入 scanAndLockForPut 流程
    // 如果是多核 cpu 最多 tryLock 64 次, 进入 lock 流程
    // 在尝试期间, 还可以顺便看该节点在链表中有没有, 如果没有顺便创建出来
    scanAndLockForPut(key, hash, value);
    
    // 执行到这里 segment 已经被成功加锁, 可以安全执行
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                // 更新
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
                // 新增
                // 1) 之前等待锁时, node 已经被创建, next 指向链表头
                if (node != null)
                    node.setNext(first);
                else
                    // 2) 创建新 node
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1; 
                // 3) 扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    // 将 node 作为链表头
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

**rehash 流程** 扩容与搬迁

发生在 put 中，因为此时已经获得了锁，因此 rehash 时不需要考虑线程安全

```java
private void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    int newCapacity = oldCapacity << 1;
    threshold = (int)(newCapacity * loadFactor);
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    int sizeMask = newCapacity - 1;
    for (int i = 0; i < oldCapacity ; i++) {
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            int idx = e.hash & sizeMask;
            if (next == null) // Single node on list
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                // 过一遍链表, 尽可能把 rehash 后 idx 不变的节点重用
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                newTable[lastIdx] = lastRun;
                // 剩余节点需要新建
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // 扩容完成, 才加入新的节点
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    
    // 替换为新的 HashEntry table
    table = newTable; 
}
```

**get方法**

get 时并未加锁，用了 UNSAFE 方法保证了可见性，扩容过程中，get 先发生就从旧表取内容，get 后发生就从新表取内容

```java
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    int h = hash(key);
    // u 为 segment 对象在数组中的偏移量
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // s 即为 segment
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
             (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null; 
}
```

**size计算流程**

- 计算元素个数前，先不加锁计算两次，如果前后两次结果如一样，认为个数正确返回 
- 如果不一样，进行重试，重试次数超过 3，将所有 segment 锁住，重新计算个数返回

```java
public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum; // sum of modCounts
    long last = 0L; // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            if (retries++ == RETRIES_BEFORE_LOCK) {
                // 超过重试次数, 需要创建所有 segment 并加锁
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size; 
}
```



### 3）总结

- JDK7 中，concurrenthashmap采用segment+entry数组+链表实现，每一个segment对应一把锁（ReentrantLock）
- JDK7 没有实现懒惰初始化，空间占用不友好，segment初始容量为16 ，且大小不可变
- JDK7中超过扩容阈值会扩容，链表转移采用头插法
- JDK8 中，concurrenthashmap采用node数组+链表+红黑树实现，每一个node节点对应一把锁（Synchronize）
- JDK8中实现了懒惰初始化，node数组大小可变
- 达到扩容阈值会扩容，链表长度超过8也会扩容，当数组长度达到64，且链表长度超过8，会将链表转化为红黑树，如果少于8个，会退化为链表

## 6.9 BlockingQueue

 ### 1) (**Linked**)BlockingQueue 原理

**1. 基本的入队出队**

```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    static class Node<E> {
        E item;
        /**
        * 下列三种情况之一
        * - 真正的后继节点
        * - 自己, 发生在出队时
        * - null, 表示是没有后继节点, 是最后了
        */
        Node<E> next;
        Node(E x) { item = x; }
    }
}
```

初始化链表 `last = head = new Node<E>(null);` Dummy 节点用来占位，item 为 null

![](https://s1.ax1x.com/2022/08/25/v2nEwt.png)

当一个节点入队` last = last.next = node;`

![](https://s1.ax1x.com/2022/08/25/v2nlOs.png)

再来一个节点入队` last = last.next = node;`

![](https://s1.ax1x.com/2022/08/25/v2n8wq.png)

**出队**

```java
/**出队
*/
Node<E> h = head;
Node<E> first = h.next; 
h.next = h; // help GC
head = first; 
E x = first.item;
first.item = null;
return x;
```

`h = head `

![](https://s1.ax1x.com/2022/08/25/v2n0X9.png)

`first = h.next`

![](https://s1.ax1x.com/2022/08/25/v2nDmR.png)

`h.next = h`

![](https://s1.ax1x.com/2022/08/25/v2nRpD.png)

`head = first`

![](https://s1.ax1x.com/2022/08/25/v2nf6H.png)

```java
E x = first.item;
first.item = null;
return x;
```

![](https://s1.ax1x.com/2022/08/25/v2nHtf.png)

**加锁分析** 

高明之处在于用了两把锁和 dummy 节点 

- 用一把锁，同一时刻，最多只允许有一个线程（生产者或消费者，二选一）执行 
- 用两把锁，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行 

- - 消费者与消费者线程仍然串行 
  - 生产者与生产者线程仍然串行

**线程安全分析** 

- 当节点总数大于 2 时（包括 dummy 节点），putLock 保证的是 last 节点的线程安全，takeLock 保证的是 head 节点的线程安全。两把锁保证了入队和出队没有竞争 
- 当节点总数等于 2 时（即一个 dummy 节点，一个正常节点）这时候，仍然是两把锁锁两个对象，不会竞争 
- 当节点总数等于 1 时（就一个 dummy 节点）这时 take 线程会被 notEmpty 条件阻塞，有竞争，会阻塞

```java
// 用于 put(阻塞) offer(非阻塞)
private final ReentrantLock putLock = new ReentrantLock();

// 用户 take(阻塞) poll(非阻塞)
private final ReentrantLock takeLock = new ReentrantLock();
```



**PUT操作**

```java
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    // count 用来维护元素计数
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        // 满了等待
        while (count.get() == capacity) {
            // 倒过来读就好: 等待 notFull
            notFull.await();
        }
        // 有空位, 入队且计数加一
        enqueue(node);
        c = count.getAndIncrement(); 
        // 除了自己 put 以外, 队列还有空位, 由自己叫醒其他 put 线程
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    // 如果队列中有一个元素, 叫醒 take 线程
    if (c == 0)
        // 这里调用的是 notEmpty.signal() 而不是 notEmpty.signalAll() 是为了减少竞争
        signalNotEmpty();
}
```

**Take操作**

```java
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    // 如果队列中只有一个空位时, 叫醒 put 线程
    // 如果有多个线程进行出队, 第一个线程满足 c == capacity, 但后续线程 c < capacity
    if (c == capacity)
        // 这里调用的是 notFull.signal() 而不是 notFull.signalAll() 是为了减少竞争
        signalNotFull()
        return x; 
}
```

**性能比较** 

主要列举 LinkedBlockingQueue 与 ArrayBlockingQueue 的性能比较 

- Linked 支持有界，Array 强制有界 
- Linked 实现是链表，Array 实现是数组 
- Linked 是懒惰的，而 Array 需要提前初始化 Node 数组 
- Linked 每次入队会生成新 Node，而 Array 的 Node 是提前创建好的 
- Linked 两把锁，Array 一把锁



### 2) ConcurrentLinkedQueue 

ConcurrentLinkedQueue 的设计与 LinkedBlockingQueue 非常像，也是 

- 两把【锁】，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行 
- dummy 节点的引入让两把【锁】将来锁住的是不同对象，避免竞争 
- 只是这【锁】使用了 cas 来实现 

[ConcurrentLinkedQueue原理](https://www.yuque.com/mo_ming/gl7b70/ug6h9y#X76Lv)

### 3） CopyOnWriteArrayList 

`CopyOnWriteArraySet` 是它的马甲 底层实现采用了 `写入时拷贝` 的思想，增删改操作会==将底层数组拷贝一份==，更改操作在新数组上执行，这时不影响其它线程的**并发读**，**读写分离**。 以新增为例：

```java
public boolean add(E e) {
    synchronized (lock) {
        // 获取旧的数组
        Object[] es = getArray();
        int len = es.length;
        // 拷贝新的数组（这里是比较耗时的操作，但不影响其它读线程）
        es = Arrays.copyOf(es, len + 1);
        // 添加新元素
        es[len] = e;
        // 替换旧的数组
        setArray(es);
        return true;
    }
}
```

> 这里的源码版本是 Java 11，在 Java 1.8 中使用的是可重入锁而不是 synchronized

其他读操作没有加锁，例如

```java
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    for (Object x : getArray()) {
        @SuppressWarnings("unchecked") E e = (E) x;
        action.accept(e);
    }
}
```

适合『读多写少』的应用场景

**缺点**

- get 弱一致性

![](https://s1.ax1x.com/2022/08/25/v2K6zD.png)

- 迭代器弱一致性

不要觉得弱一致性就不好 

- 数据库的 MVCC 都是弱一致性的表现 
- 并发高和一致性是矛盾的，需要权衡
