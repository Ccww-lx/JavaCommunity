## Volatile与Synchronized分析

在深入理解使用Volatile与Synchronized时，应该先理解明白Java内存模型 （Java Memory Model，JMM）

------

### Java内存模型（Java Memory Model，JMM）

Java内存（JMM）模型是在硬件内存模型基础上更高层的抽象，它屏蔽了各种硬件和操作系统对内存访问的差异性，从而实现让Java程序在各种平台下都能达到一致的并发效果。

JMM的内部工作机制


- 主内存：存储共享的变量值（实例变量和类变量，不包含局部变量，因为局部变量是线程私有的，因此不存在竞争问题）

- 工作内存：CPU中每个线程中保留共享变量的副本，线程的工作内存，线程在变更修改共享变量后同步回主内存，在变量被读取前从主内存刷新变量值来实现的。
- 内存间的交互操作：不同线程之间不能直接访问不属于自己工作内存中的变量，线程间变量的值的传递需要通过主内存中转来完成。（lock，unlock，read，load，use，assign，store，write）

JMM内部会有指令重排，并且会有af-if-serial跟happen-before的理念来保证指令的正确性

- 为了提高性能，编译器和处理器常常会对既定的代码执行顺序进行指令重排序
- af-if-serial：不管怎么重排序，单线程下的执行结果不能被改变
- 先行发生原则(happen-before)：先行发生原则有很多，其中程序次序原则，在一个线程内，按照程序书写的顺序执行，书写在前面的操作先行发生于书写在后面的操作，准确地讲是控制流顺序而不是代码顺序

Java内存模型为了解决多线程环境下共享变量的一致性问题，包含三大特性，

- 原子性：操作一旦开始就会一直运行到底，中间不会被其它线程打断（这操作可以是一个操作，也可以是多个操作），在内存中原子性操作包括read、load、user、assign、store、write，如果需要一个更大范围的原子性可以使用synchronized来实现，synchronized块之间的操作。
- 可见性：一个线程修改了共享变量的值，其它线程能立即感知到这种变化，修改之后立即同步回主内存，每次读取前立即从主内存刷新，可以使用volatile保证可见性，也可以使用关键字synchronized和final。
- 有序性：在本线程中所有的操作都是有序的；在另一个线程中，看来所有的操作都是无序的，就可需要使用具有天然有序性的volatile保持有序性，因为其禁止重排序。

在理解了JMM的时，来讲讲Volatile与Synchronized的使用，Volatile与Synchronized到底有什么作用呢？

------

### Volatile

**Volatile 的特性**：

- 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。（实现可见性）
- 禁止进行指令重排序。（实现有序性）
- volatile 只能保证对单次读/写的原子性，i++ 这种操作不能保证原子性

#### Volatile可见性

当写一个volatile变量时，JMM会把该线程对应的工作内存中的共享变量值更新后刷新到主内存，

当读取一个volatile变量时，JMM会把该线程对应的工作内存置为无效，线程会从主内存中读取共享变量。

写操作:
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fbd6adf4216439cb910bd7ca2dc451a~tplv-k3u1fbpfcp-zoom-1.image)

读操作：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8bc545d9c3974f68a3d2038ace8fcdfa~tplv-k3u1fbpfcp-zoom-1.image)

#### Volatile 禁止指令重排

JMM对volatile的禁止指令重排采用内存屏障插入策略：

在每个volatile写操作的前面插入一个StoreStore屏障。在每个volatile写操作的后面插入一个StoreLoad屏障

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bcd8b709a174f4096db06849f855ffb~tplv-k3u1fbpfcp-zoom-1.image)

在每个volatile读操作的后面插入一个LoadLoad屏障。在每个volatile读操作的后面插入一个LoadStore屏障
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bf3b9c5ca8042ed85654068959da4e5~tplv-k3u1fbpfcp-zoom-1.image)
------

### Synchronized

Synchronized是Java中解决并发问题的一种最常用的方法，也是最简单的一种方法。Synchronized的作用主要有三个：

- 原子性：确保线程互斥的访问同步代码；
- 可见性：保证共享变量的修改能够及时可见，其实是通过Java内存模型中的 “对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值” 来保证的
- 有序性：有效解决重排序问题，即 “一个unlock操作先行发生(happen-before)于后面对同一个锁的lock操作”；

Synchronized总共有三种用法：

> 1. 当synchronized作用在实例方法时，监视器锁（monitor）便是对象实例（this）；
> 2. 当synchronized作用在静态方法时，监视器锁（monitor）便是对象的Class实例，因为Class数据存在于永久代，因此静态方法锁相当于该类的一个全局锁；
> 3. 当synchronized作用在某一个对象实例时，监视器锁（monitor）便是括号括起来的对象实例；

更加详细的解析看[Java并发之Synchronized](https://juejin.im/post/6844904031945490445)

理解了Volatile与Synchronized后，那我们来看看如何使用Volatile与Synchronized优化单例模式

------

### 单例模式优化-双重检测DCL（Double Check Lock）

先来看看一般模式的单例模式：

```java
class Singleton{
    private static Singleton singleton;    
    private Singleton(){}

    public static Singleton getInstance(){
            if(singleton == null){
                singleton = new Singleton();   // 创建实例
        }
        return singleton;
    }

}
```

可能出现问题：当有两个线程A和B，

- 线程A判断`if(singleton == null)`准备执行创建实例时，线程挂起，
- 此时线程B也会判断singleton为空，接着执行创建实例对象返回；
- 最后，由于线程A已进入也会创建了实例对象，这就导致多个单例对象的情况

首先想到是那就在使用synchronized作用在静态方法：

```java
public class Singleton {
    private static Singleton singleton;
    private Singleton(){}
    public static synchronized Singleton getInstance(){
        if(singleton == null){
       		 singleton = new Singleton();
        }
        return singleton;
    }
}
```

虽然这样简单粗暴解决，但会导致这个方法比较效率低效，导致程序性能严重下降，那是不是还有其他更优的解决方案呢？

可以进一步优化创建了实例之后，线程再同步锁之前检验singleton非空就会直接返回对象引用，而不用每次都在同步代码块中进行非空验证，

如果只有synchronized前加一个singleton非空，就会出现第一种情况多个线程同时执行到条件判断语句时，会创建多个实例

因此需要在synchronized后加一个singleton非空，就不会出现会创建多个实例，

```java
class Singleton{
    private static Singleton singleton;    
    private Singleton(){}
    
    public static Singleton getInstance(){
        if(singleton == null){
            synchronized(Singleton.class){
                if(singleton == null)
                    singleton = new Singleton();   
            }
        }
        return singleton;
    }
}
```

这个优化方案虽然解决了只创建单个实例，由于存在着指令重排，会导致在多线程下也是不安全的（当发生了重排后，后续的线程发现singleton不是null而直接使用的时候，就会出现意料之外的问题。）。导致原因`singleton = new Singleton()`新建对象会经历三个步骤：

- 1.内存分配
- 2.初始化
- 3.返回对象引用

由于重排序的缘故，步骤2、3可能会发生重排序，其过程如下：

- 1.分配内存空间
- 2.将内存空间的地址赋值给对应的引用
- 3.初始化对象

那么问题找到了，那怎么去解决呢？那就禁止不允许初始化阶段步骤2 、3发生重排序，刚好Volatile 禁止指令重排，从而使得双重检测真正发挥作用。

```java
public class Singleton {
    //通过volatile关键字来确保安全
    private volatile static Singleton singleton;
    private Singleton(){}
    public static Singleton getInstance(){
        if(singleton == null){
           synchronized (Singleton.class){
                if(singleton == null){
                singleton = new Singleton();
            }
        }
    }
    return singleton;
    }
}
```

最终我们这个完美的双重检测单例模式出来了

------

### 总结

- volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取； synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
- volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的
- volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性
- volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
- volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化
-  使用volatile而不是synchronized的唯一安全的情况是类中只有一个可变的域 
