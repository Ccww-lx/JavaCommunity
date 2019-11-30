>**欢迎关注公众号【Ccww笔记】，原创文章第一时间推出！！！**   


&emsp;&emsp;多线程和并发是求职大小厂面试中必问的知识点，其涉及到点很多，难度很大。有些人面对这些问题有点迷茫，为了解决这情况，总结了一下java多线程并发的基础知识点。而且要想深入研究java多线程并发也必须先掌握基础知识，可为后续各个模块深入研究做好做好准备。现在废话不多说，各位看官请查看基础知识点，后续还有源码解析（`synchronize`底层原理，线程池原理，`Lock`，`AQS`，同步、并发容器等源码解析）。

### 1 基本概念

** 程序：** 是计算机指令的集合，它以文件的形式存储在磁盘上,即程序是静态的代码

** 进程：**  

+ 是一个程序在其自身的地址空间中的一次执行活动,是系统运行程序的基本单位
+ 进程是资源申请、调度和独立运行的单位

** 线程：**

+ 是进程中的一个单一的连续控制流程。一个进程可以拥有多个线程。
+ 线程又称为轻量级进程，它和进程一样拥有独立的执行控制，由操作系统负责调度，区
  别在于线程没有独立的存储空间，而是和所属进程中的其它线程共享一个存储空间，这
  使得线程间的通信远较进程简单。

** 三者之间的关系:**

+ 线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。
+ 从另一角度来说，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执行一个以上的程序段。


![](https://user-gold-cdn.xitu.io/2019/11/7/16e4609b7176e34c?w=646&h=502&f=png&s=26960)

### 2 线程组成

组成部分：虚拟CPU、执行的代码以及处理的数据。
![](https://user-gold-cdn.xitu.io/2019/11/7/16e460d1dd037ff1?w=250&h=270&f=png&s=9047)

### 3 线程与进程区别

** 进程：** 指系统中正在运行中的应用程序，它拥有自己独立的内存空间；

** 线程：** 是指进程中一个执行流程，一个进程中允许同时启动多个线程，他们分别执行不同的任务,多个线程共享内存，从而极大地提高了程序的运行效率;

** 主要区别：**

+ 每个进程都需要操作系统为其分配独立的内存地址空间
+ 而同一进程中的所有线程在同一块地址空间中，这些线程可以共享数
  据，因此线程间的通信比较简单，消耗的系统开销也相对较小


### 4 为什么要使用多线程

** 使用多线程好处：**

+ 可以同时并发执行多个任务
+ 程序的某个功能部分正在等待某些资源的时候，此时又不愿意因为等待而造成程序暂停，那么就可以创建另外的线程进行其它的工作；
+ 多线程可以最大限度地减低CPU的闲置时间，从而提高CPU的利用率；

### 5 主线程

Java程序启动时，一个线程立刻运行，它执行main方法，这个线程称为程序的主线程，任何Java程序都至少有一个线程，即主线程。

** 主线程的特殊之处在于：**

+ 它是产生其它线程子线程的线程；
+ 通常它必须最后结束，因为它要执行其它子线程的关闭工作。

### 6 线程优先级

单核计算机只有一个CPU，各个线程轮流获得CPU的使用权，才能执行任务：

+ 优先级较高的线程有更多获得CPU的机会，反之亦然；
+ 优先级用整数表示，取值范围是1~10，一般情况下，线程的默认
+ 优先级都是5，但是也可以通过setPriority和getPriority方法来设置或返回优先级；

** `Thread`类有如下3个静态常量来表示优先级：**

+ MAX_PRIORITY：取值为10，表示最高优先级
+ MIN_PRIORITY：取值为1，表示最低优先级
+ NORM_PRIORITY：取值为5，表示默认的优先级

### 7 线程的生命周期

![](https://user-gold-cdn.xitu.io/2019/11/10/16e532817f707795?w=982&h=706&f=png&s=99918)

** 线程状态(`State`枚举值代表线程状态)：**

+ **新建状态（ NEW）：** 线程刚创建, 尚未启动。`Thread thread = new Thread()`。
+ **可运行状态（RUNNABLE）：** 线程对象创建后，其他线程(比如 main 线程）调用了该对象的 `start` 方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取 cpu 的使用权。
+ **运行(running)：** 线程获得 CPU 资源正在执行任务（`run()` 方法），此时除非此线程自动放弃 CPU 资源或者有优先级更高的线程进入，线程将一直运行到结束
+ **阻塞状态（Blocked）：** 线程正在运行的时候，被暂停，通常是为了等待某个时间的发生(比如说某项资源就绪)之后再继续运行。`sleep`,`suspend`，`wait`等方法都可以导致线程阻塞
+ **等待（WAITING）：** 进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
+ **超时等待(TIMED_WAITING)：** 该状态不同于`WAITING`，它可以在指定的时间后自行返回。
+ **终止(TERMINATED)：** 表示该线程已经执行完毕，如果一个线程的`run`方法执行结束或者调用`stop`方法后，该线程就会死亡。对于已经死亡的线程，无法再使用`start`方法令其进入就绪。


** 线程在Running的过程中可能会遇到阻塞(Blocked)情况：**

+ 调用`join()`和`sleep()`方法，`sleep()`时间结束或被打断，`join()`中断,IO完成都会回到`Runnable`状态，等待JVM的调度。
+ 调用`wait()`，使该线程处于等待池(wait blocked pool),直到`notify()`/`notifyAll()`，线程被唤醒被放到锁定池(lock blocked pool )，释放同步锁使线程回到可运行状态（Runnable）
+ 对Running状态的线程加同步锁(Synchronized)使其进入(lock blocked pool ),同步锁被释放进入可运行状态(Runnable)。


### 8 线程创建方式

** 线程创建方式：**

+ **实现Runnable接口，重载`run()`，无返回值**
+ **继承Thread类，复写`run()`**
+ **实现Callable接口，通过FutureTask/Future来创建有返回值的Thread线程，通过Executor执行**
+ **使用Executors创建ExecutorService，入参Callable或Future**

**1.实现Runnable接口，重载`run()`，无返回值，Runnable接口的存在主要是为了解决Java中不允许多继承的问题。**

    public class ThreadRunnable implements Runnable {
      public void run() {
        for (int i = 0; i < 10; i++) {
          System.out.println(Thread.currentThread().getName() + ":" + i);
        }
      }
    }
      
    public class ThreadMain {
      public static void main(String[] args) throws Exception {
        ThreadRunnable threadRunnable1 = new ThreadRunnable();
        ThreadRunnable threadRunnable2 = new ThreadRunnable();
        ThreadRunnable threadRunnable3 = new ThreadRunnable();
        Thread thread1 = new Thread(threadRunnable1);
        Thread thread2 = new Thread(threadRunnable2);
        Thread thread3 = new Thread(threadRunnable3);    
        thread1.start();
        thread2.start();
        thread3.start();
      }
    }

**2.继承Thread类，重写`run()`，通过调用Thread的`start()`会调用创建线程的`run()`，不同线程的run方法里面的代码交替执行。但由于Java不支持多继承.因此继承Thread类就代表这个子类不能继承其他类.**

    public class ThreadCustom extends Thread {
      public void run() {
        for (int i = 0; i < 10; i++) {
          System.out.println(Thread.currentThread() + ":" + i);
        }
      }
    }


​      

    public class ThreadTest {
      public static void main(String[] args)
      {
        ThreadCustom thread = new ThreadCustom();
        thread.start();
      }
    }

**3.实现Callable接口，通过FutureTask/Future来创建有返回值的Thread线程，通过Executor执行，该方式有返回值，可以获得异步。**


    public class ThreadCallableCustom {
      public static void main(String[] args) throws Exception {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
          public Integer call() throws Exception {
            for (int i = 0; i < 10; i++) {
              System.out.println(Thread.currentThread().getName() + ":" + i);
            }
            return 1;
          }
        });
        Executor executor = Executors.newFixedThreadPool(1);
        ((ExecutorService) executor).submit(futureTask);
      
        //获得线程执行状态
        System.out.println(Thread.currentThread().getName() + ":" + futureTask.get());
      }
    }

**4.使用Executors创建ExecutorService，入参Callable或Future，适用于线程池和并发**


    public class ThreadExecutors {
      private final String threadName;
      
      public ThreadExecutors(String threadName) {
        this.threadName = threadName;
      }
      
      private ThreadFactory createThread() {
        ThreadFactory tf = new ThreadFactory() {
          public Thread newThread(Runnable r) {
            Thread thread = new Thread();
            thread.setName(threadName);
            thread.setDaemon(true);
            try {
              sleep(1000);
            }
            catch (InterruptedException e) {
              e.printStackTrace();
            }
            return thread;
          }
        };
        return tf;
      }
      
      public Object runCallable(Callable callable) {
        return Executors.newSingleThreadExecutor(createThread()).submit(callable);
      }
      
      public Object runFunture(Runnable runnable) {
        return Executors.newSingleThreadExecutor(createThread()).submit(runnable);
      }
    }
      
    public class ThreadTest {
      public static void main(String[] args) throws Exception {
        ThreadExecutors threadExecutors = new ThreadExecutors("callableThread");
        threadExecutors.runCallable(new Callable() {
          public String call() throws Exception {
            return "success";
          }
        });
      
        threadExecutors.runFunture(new Runnable() {
          public void run() {
            System.out.println("execute runnable thread.");
          }
        });
      }
    }

### 9 Runnable接口和Callable接口区别

1）两个接口需要实现的方法名不一样，Runnable需要实现的方法为`run()`，Callable需要实现的方法为`call()`。  
2）实现的方法返回值不一样，Runnable任务执行后无返回值，Callable任务执行后可以得到异步计算的结果。  
3）抛出异常不一样，Runnable不可以抛出异常，Callable可以抛出异常。  


### 10 线程安全

**线程安全定义**   

当多个线程访问某个一类(对象或方法)时，这个类始终都能表现出正确的行为，那么这个类(对象或方法)就是线程安全的(即在多线程环境中被调用时，能够正确地处理多个线程之间的共享变量，使程序功能正确完成)。

**线程安全示例**

**饿汉式单例模式-线程安全**


    public class EagerSingleton(){
     
        private static final EagerSingleton instance = new EagerSingleton();
     
        private EagerSingleton(){};
        
        public static EagerSingleton getInstance(){
           return instance;
        }
    }

**如何解决线程安全问题？**

 可以通过加锁的方式：

+ 同步(synchronized)代码块：只需要将操作共享数据的代码放在synchronized
+ 同步(synchronized)方法：将操作共享数据的代码抽取出来放到一个synchronized方法里面就可以了
+ Lock锁：加同步锁 `lock()` 以及释放同步锁`unlock()`



### 11 什么是死锁、活锁？

死锁，是指两个或两个以上的进程（或线程）在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。

活锁，任务或者执行者没有被阻塞，由于某些条件没有满足，导致一直重复尝试，失败，尝试，失败。

** 产生死锁的必要条件：**

+ 互斥条件：所谓互斥就是进程在某一时间内独占资源。
+ 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
+ 不剥夺条件：进程已获得资源，在末使用完之前，不能强行剥夺。
+ 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

** 死锁的解决方法：**

+  撤消陷于死锁的全部进程。
+  逐个撤消陷于死锁的进程，直到死锁不存在。
+  从陷于死锁的进程中逐个强迫放弃所占用的资源，直至死锁消失。
   从另外一些进程那里强行剥夺足够数量的资源分配给死锁进程，以解除死锁状态。


### 12 什么是悲观锁、乐观锁？

**1）悲观锁**  

悲观锁，总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。

+ 传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。
+ Java 里面的同步原语 synchronized 关键字的实现也是悲观锁。

**2）乐观锁** 

乐观锁，顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。



### 13 多个线程间锁的并发控制

多个线程间锁的并发控制,对象锁多个线程、每个线程持有该方法所属对象的锁以及类锁。synchronized, wait, notify 是任何对象都具有的同步工具

**对象锁的同步和异步**

+ 同步：synchronized，同步的概念就是共享，只需要针对共享的资源，才需要考虑同步。
+ 异步：asynchronized，异步的概念就是独立，相互之间不受到任何制约。

同步的目的就是为线程安全，其实对于线程安全来说，需要满足两个特性：原子性(同步)、可见性。

### 14 Volatile关键字

Volatile作用，实现变量在多个线程间可见，保证内存可见性和禁止指令重排

**多线程的内存模型：main memory（主存）、working**  

memory（线程栈），在处理数据时，线程会把值从主存load到本地栈，完成操作后再save回去(volatile关键词的作用：每次针对该变量的操作都激发一次load and save)。

![](https://user-gold-cdn.xitu.io/2019/11/10/16e5360a93f8c504?w=592&h=493&f=png&s=39606)


### 15 ThreadLocal

&emsp;&emsp;线程局部变量，以空间换时间的手段，为每个线程提供变量的独立副本，以无锁的情况下保障线程安全。主要解决的就是让每个线程执行完成之后再结束，这个时候就要用到join()方法。

**适用场景：**

+ 在并发不是很高的时候，加锁的性能会更好
+ 在高并发量场景下，使用ThreadLocal可以在一定程度上减少锁竞争。

### 16 多线程同步和互斥实现方法

&emsp;&emsp;1). 线程同步，是指线程之间所具有的一种制约关系，一个线程的执行依赖另一个线程的消息，当它没有得到另一个线程的消息时应等待，直到消息到达时才被唤醒。

** 线程间的同步方法，大体可分为两类：用户模式和内核模式。顾名思义：**

内核模式，就是指利用系统内核对象的单一性来进行同步，使用时需要切换内核态与用户态。内核模式下的方法有：

+ 事件
+ 信号量
+ 互斥量

用户模式，就是不需要切换到内核态，只在用户态完成操作。用户模式下的方法有：

+ 原子操作（例如一个单一的全局变量）
+ 临界区


&emsp;&emsp;2). 线程互斥，是指对于共享的进程系统资源，在各单个线程访问时的排它性。

当有若干个线程都要使用某一共享资源时，任何时刻最多只允许一个线程去使用，其它要使用该资源的线程必须等待，直到占用资源者释放该资源。
线程互斥可以看成是一种特殊的线程同步。

### 17 线程之间通信

线程是操作系统中独立的个体，但这些个体之间如果不经过特殊的协作就不能成为一个整体，线程间的通信就成为整体的必用方式之一。

线程间通信的几种方式？

线程之间的通信方式:

+ 共享内存
+ 消息传递

**共享内存：在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信。典型的共享内存通信方式，就是通过共享对象进行通信。**

![](https://user-gold-cdn.xitu.io/2019/11/10/16e5368b83241ef5?w=600&h=520&f=png&s=36212)
**消息传递：在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。在 Java 中典型的消息传递方式，就是 `wait()` 和 `notify()` ，或者 BlockingQueue 。**

![](https://user-gold-cdn.xitu.io/2019/11/10/16e53725deed33fc?w=559&h=582&f=png&s=39503)



### 18 什么是 Java Lock 接口？

java.util.concurrent.locks.Lock 接口，比 synchronized 提供更具拓展行的锁操作。它允许更灵活的结构，可以具有完全不同的性质，并且可以支持多个相关类的条件对象。它的优势有：

+ 可以使锁更公平。
+ 可以使线程在等待锁的时候响应中断。
+ 可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间。
+ 可以在不同的范围，以不同的顺序获取和释放锁。


![](https://user-gold-cdn.xitu.io/2019/11/10/16e537ff5b306b7a?w=1071&h=573&f=png&s=162121)

### 19 Java AQS

AQS ，AbstractQueuedSynchronizer ，即队列同步器。它是构建锁或者其他同步组件的基础框架（如 ReentrantLock、ReentrantReadWriteLock、Semaphore 等），J.U.C 并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。它是 J.U.C 并发包中的核心基础组件。

** 优势：**  

AQS 解决了在实现同步器时涉及当的大量细节问题，例如获取同步状态、FIFO 同步队列。基于 AQS 来构建同步器可以带来很多好处。它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。

在基于 AQS 构建的同步器中，只能在一个时刻发生阻塞，从而降低上下文切换的开销，提高了吞吐量。同时在设计 AQS 时充分考虑了可伸缩性，因此 J.U.C 中，所有基于 AQS 构建的同步器均可以获得这个优势。





### 20 同步类容器

何为同步容器？可以简单地理解为通过synchronized来实现同步的容器，如果有多个线程调用同步容器的方法，它们将会串行执行。

** 特点：**

+ 是线程安全的
+ 某些场景下可能需要加锁来保护复合操作

** 常见同步类容器：**

+ 如Vector、HashTable
+ 使用JDK的Collections.synchronized等工厂方法去创建实现的。
+ 底层是用传统的synchronized关键字对方法进行同步。
+ 无法满足高并发场景下的性能需求


### 21 并发类容器

jdk5.0以后提供了多种并发类容器来替代同步类容器从而改善性能。

**同步类容器局限性：**

+ 都是串行化的。
+ 虽实现了线程安全，却降低了并发性
+ 在多线程环境时，严重降低了应用程序的吞吐量。


**常用的并发类容器:**

+ ConcurrentHashMap
+ Copy-On-Write容器



** ConcurrentHashMap原理**

+ 内部使用段(Segment)来表示这些不同的部分
+ 每个段相当于一个小的HashTable，它们有自己的锁。
+ 把一个整体分成了16个段(Segment)。也就是最高支持16个线程的并发修改操作。
+ 这也是在多线程场景时通过减小锁的粒度从而降低锁竞争的一种方案。

** Copy-On-Write容器** 

Copy-On-Write简称COW，是一种用于程序设计中的优化策略。

+ 读写分离，读和写分开
+ 最终一致性
+ 使用另外开辟空间的思路，来解决并发冲突

** JDK里的COW容器有两种：**

+ CopyOnWriteArrayList:适用于读操作远远多于写操作的场景,例如，缓存.
+ CopyOnWriteArraySet:线程安全的无序的集合，可以将它理解成线程安全的HashSet,适用于Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突


### 22 并发Queue

** 并发Queue:** 

+ ConcurrentLinkedQueue,高性能队列，当许多线程共享访问一个公共集合时，ConcurrentLinkedQueue 是一个恰当的选择。
+ BlockingQueue,阻塞队列,是一个支持两个附加操作的队列,常用于生产者和消费者的场景。

** ConcurrentLinkedQueue**

+ 适用于高并发场景
+ 使用无锁的方式，实现了高并发状态下的高性能
+ 其性能好于BlockingQueue
+ 遵循先进先出的原则

**常用方法：**

+ Add()和offer()都是加入元素的方法
+ Poll()和peek()都是取头元素节点，区别在于前者会删除元素，后者不会。

** BlockingQueue接口实现：**

+ ArrayBlockingQueue：基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长的数组，以便缓存队列中的数据对象，其内部没实现读写分离，也就意味着生产和消费者不能完全并行，适用很多场景。
+ LinkedBlockingQueue：基于链表的阻塞队列，同ArrayBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），LinkedBlockingQueue之所以能够高效地处理并发数据，是因为其内部实现采用分离锁（读写分离两个锁），从而实现生产者和消费者操作完全并行运行。
+ PriorityBlockingQueue：基于优先级别的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定，也就是说传入队列的对象必须实现Comparable接口），在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁
+ DelayQueue：带有延迟时间的Queue，其中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue中的元素必须先实现Delayed接口，DelayQueue是一个没有大小限制的队列，应用场景很多，比如对缓存超时的数据进行移除、任务超时处理、空闲连接的关闭等等。
+ SynchronousQueue：一种没有缓冲的队列，生产者产生的数据直接会被消费者获取并且立刻消费

### 23 多线程的设计模式

+ 基于并行设计模式演变而来
+ 属于设计优化的一部分
+ 是对一些常用的多线程结构的总结和抽象
+ 常见的多线程设计模式有哪些？

### 24 Concurrent.util常用类

**CountDownLatch：** 用于监听某些初始化操作，等初始化执行完毕后，通知主线程继续工作。

**CycilcBarrier：** 所有线程都准备好后，才一起出发，只要有一个人没有准备好，大家都等待。

**Concurrent.util常用类** 
定义：实现异步回调，jdk针对该场景提供了一个实现的封装，简化了调用
适合场景：处理耗时的业务逻辑时，可有效的减少系统的响应时间，提高系统的吞吐量。

**Concurrent.util常用类**
Semaphore：信号量，适合高并发访问， 用于进行访问流量的控制

**ReentrantLock(重入锁)**
重入锁，在需要进行同步的代码部分加上锁定，但不要忘记最后一定要释放锁定，不然会造成锁永远无法释放，其他线程永远也进不来的结果。

** 锁与等待/通知**

+ 多线程间进行协作工作则需要Object的wait()和notify()、notifyAll()方法进行配合工作
+ 在使用锁的时候，可以使用一个新的等待/通知的类，它就是Condition
+ 这个Condition是针对具体某一把锁的

** 多Condition**

+ 可通过一个Lock对象产生多个Condition进行多线程间的交互
+ 使得部分需要唤醒的线程唤醒，其他线程则继续等待通知。

** ReentrantReadWriteLock(读写锁)**

+ 其核心就是实现读写分离的锁。尤其适应在在高并发访问下读多写少的情况下，性能要远高于重入锁。
+ 将读写锁分离为读锁和写锁
+ 在读锁下，多个线程可以并发进行访问
+ 在写锁下，只能一个一个的顺序访问
+ 锁优化


### 25 线程池

![](https://user-gold-cdn.xitu.io/2019/11/10/16e55f1bd227e5cc?w=1259&h=958&f=png&s=132196)

** 使用 Executor 框架的原因：**

+ 每次执行任务创建线程 new Thread() 比较消耗性能，创建一个线程是比较耗时、耗资源的。
+ 调用 new Thread() 创建的线程缺乏管理，被称为野线程，而且可以无限制的创建，线程之间的相互竞争会导致过多占用系统资源而导致系统瘫痪，还有线程之间的频繁交替也会消耗很多系统资源。
+ 接使用 new Thread() 启动的线程不利于扩展，比如定时执行、定期执行、定时定期执行、线程中断等都不便实现


** 线程池的创建方式：**

**普通任务线程池**

+ newFixedThreadPool(int nThreads) 方法，创建一个固定长度的线程池。
  + 每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程规模将不再变化。
  + 当线程发生未预期的错误而结束时，线程池会补充一个新的线程。
+ newCachedThreadPool() 方法，创建一个可缓存的线程池。
  + 如果线程池的规模超过了处理需求，将自动回收空闲线程。
  + 当需求增加时，则可以自动添加新线程。线程池的规模不存在任何限制。
+ newSingleThreadExecutor() 方法，创建一个单线程的线程池。
  + 它创建单个工作线程来执行任务，如果这个线程异常结束，会创建一个新的来替代它。
  + 它的特点是，能确保依照任务在队列中的顺序来串行执行。

**定时任务线程池** 

+ newScheduledThreadPool(int corePoolSize) 方法，创建了一个固定长度的线程池，而且以延迟或定时的方式来执行任务，类似 Timer 。
+ newSingleThreadExecutor() 方法，创建了一个固定长度为 1 的线程池，而且以延迟或定时的方式来执行任务，类似 Timer 。


** 线程池的关闭方式**

ThreadPoolExecutor 提供了两个方法，用于线程池的关闭，分别是：

+ shutdown() 方法，不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
+ shutdownNow() 方法，立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务


### 26 总结

&emsp;&emsp;由于java多线程并发涉及到的知识点太多了，这边不可能一一列全，不过我会在后续的更新中一一去补充完善，而且会涉及原理以及源码层次的解析。谢谢观看，有错误欢迎指出更改！！



欢迎关注公众号【Ccww笔记】，原创文章第一时间推出！！！   
![](https://user-gold-cdn.xitu.io/2019/7/18/16c058521c9cf39d?w=258&h=258&f=png&s=42777)

