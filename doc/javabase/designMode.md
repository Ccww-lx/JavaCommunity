## 设计模式常见面试题

### 1.设计模式分类

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNDg2MDc0LzIwMTkwOC80ODYwNzQtMjAxOTA4MTIxNjI1MzY5MDgtNDYyMzg0OTcucG5n?x-oss-process=image/format,png)

 

 

### 2.设计模式的优点

　　设计模式可在多个项目中重用。

　　设计模式提供了一个帮助定义系统架构的解决方案。

　　设计模式吸收了软件工程的经验。

　　设计模式为应用程序的设计提供了透明性。

　　设计模式是被实践证明切实有效的，由于它们是建立在专家软件开发人员的知识和经验之上的。

 

### 3.单例模式的七种实现

#### 第一种：懒汉式加载

> 懒汉式加载：最简单的单例模式：2步，1.把自己的构造方法设置为私有的，不让别人访问你的实例，2.提供一个static方法给别人获取你的实例.

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTliMDEzNmVlZGY5MzcwYWEucG5n?x-oss-process=image/format,png)

懒汉式加载版本单例模式

我们可以看到，这是一个简单的获取单例的一个类，首先我们定义一个静态实例 single, 如何将构造方法变成私有的。并且给外界一个静态获取实例的方法。如果对象不是null，就直接返回实例，从而保证实例。也可以保证不浪费内存。这是我们的第一个实现单例模式的例子。很简单。但是有问题，我们后面再讲。

#### 第二种：饿汉式加载

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTlmODkwNzVhN2I4MjM2OWUucG5n?x-oss-process=image/format,png)

饿汉式加载版本单例模式

我们看到第二种单例模式，代码量比第一个少了很多，而为什么叫饿汉式呢？我们看代码，我们定义了一个静态的final的实例，并且直接new了一个对象，这样就会导致Single2 类在加载字节码到虚拟机的时候就会实例化这个实例，当你调用getInstance方法的时候，就会直接返回，不必做任何判断，这样做的好处是代码量明显减少了，坏处是，在你没有使用该单例的时候，该单例却被加载了，如果该单例很大的话，将会浪费很多的内存。

#### 我们停下来思考一下

> 我们如何选择这两种实现方式呢？如果你的项目对性能没有要求，那么请直接使用饿汉式方法实现单例模式，既简单又方便。但是，大部分程序员都是有追求的，岂能不追求性能。那么我们看第一种方式，就是懒汉式，我们刚刚说过，懒汉式既保证了单例，又保证了性能。但是，他真的能保证单例吗？可以确定的是：在单线程模式下，毫无问题，但在复杂的多线程模式下，会怎么样呢？show me code .

#### 测试用例：我们测试一下  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTM1MWEzZTNjNjk2M2E1ODEucG5n?x-oss-process=image/format,png) 测试用例 

我们分析一下上面的代码，首先，我们验证的是什么呢？我们想验证多线程下获取懒汉式单例会不会出现错误。也就是出现一个以上的单例，我们如何做呢？首先我们定义一个Set对实例进行去重，然后创建1000个线程（Windows每个进程最多1000个线程，Linux每个进程最多2000个线程），每个线程都去获取实例，并添加到set中，实际上，我们应该使用Collections.synchronizedSet(set)获取一个线程安全的set，但是，这里为了方便，就直接使用HashSet了，然后main线程等待10秒，让1000个线程尽量都执行完毕。最后循环打印set的内容。在某些情况下，会出现2个实例，注意，是某些情况下，一定要多测试几次。下面是我们测试的结果：



![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLWUyYjQyODM4YzAwYTAwM2YucG5n?x-oss-process=image/format,png)

测试结果

#### 我们停下来思考一下：

> 我们通过测试用例发现：高并发情况下，我们的懒加载确实存在bug。为什么会这样呢？我们假设第一个线程进入getInstance方法，判断实例为null，准备进入if块内执行实例化，这时线程突然让出时间片，第二个线程也进入方法，判断实例也为null，并且进入if块执行实例化，第一个线程唤醒也进入if块进行实例化。这时就会出现2个实例。所以出现了bug。So,  我们想要性能（避免上面说的消耗不需要的内存），又要线程安全。那我们该怎么办呢？有点经验的同学心里肯定有数了。show me code.

#### 第三种方式：synchronized 同步式

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTI0YjgyOTE1MDk2ZjFiMTIucG5n?x-oss-process=image/format,png)

59{(V}0%M`G546FRI`F4(_9.png

这是我们的第三种方式，我们分析一下代码，我们可以看到，我们仅仅是在第一种懒汉式中加入了一个关键字，synchronized, 使用synchronized保证线程同步，保证同时只有一个进程进入此方法。从而保证并发安全。但是这样做完美吗？我们思考一下我们的代码：我们使用synchronized关键字，相当于每个想要进入该方法的获取实例的线程都要阻塞排队，我们仔细思考一下：需要吗？当实例已经初始化之后，我们还需要做同步控制吗？这对性能的影响是巨大的。是的，我们只需要在实例第一次初始化的时候同步就足够了。我们继续优化。

#### 第四种方式：双重检验锁：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTA2Y2IwM2IyOGJiYWI5MjUucG5n?x-oss-process=image/format,png)

双重检验锁

我们继续分析一下代码：首先看getInstance方法，我们在方法声明上去除了synchronized关键字，多线程进入方法内部，判断是否为null，如果为null，多个线程同时进入if块内，此时，我们是用Single4 Class对象同步一段方法。保证只有一个线程进入该方法。并且判断是否为null，如果为null，就进行初始化。我们想象一下，如果第一个线程进入进入同步块，发现该实例为null，于是进入if块实例化，第二个线程进入同步内则发现实例已经不是null，直接就返回 了，从而保证了并发安全。那么这个和第三种方式又什么区别呢？第三种方式的缺陷是：每个线程每次进入该方法都需要被同步，成本巨大。而第四种方式呢？每个线程最多只有在第一次的时候才会进入同步块，也就是说，只要实例被初始化了，那么之后进入该方法的线程就不必进入同步块了。就解决并发下线程安全和性能的平衡。虽然第一次还是会被阻塞。但相比较于第三种，已经好多了。

我们还对一个东西感兴趣，就是修饰变量的volatile关键字，为什么要用volatile关键字呢？这是个有趣的问题。我们好好分析一下：
 首先我们看，Java虚拟机初始化一个对象都干了些什么？总的来说，3件事情：

1. 在堆空间分配内存
2. 执行构造方法进行初始化
3. 将对象指向内存中分配的内存空间，也就是地址

但是由于当我们编译的时候，编译器在生成汇编代码的时候会对流程进行优化（这里涉及到happen-before原则和Java内存模型和CPU流水线执行的知识，就不展开讲了），优化的结果式有可能式123顺序执行，也有可能式132执行，但是，如果是按照132的顺序执行，走到第三步（还没到第二步）的时候，这时突然另一个线程来访问，走到if(single4 == null)块，会发现single4已经不是null了，就直接返回了，但是此时对象还没有完成初始化，如果另一个线程对实例的某些需要初始化的参数进行操作，就有可能报错。使用volatile关键字，能够告诉编译器不要对代码进行重排序的优化。就不会出现这种问题了。

我们看到，小小的单例模式被我们弄得很复杂。但这就是一个程序员的追求，追求最好的性能，追求最好的代码。

那还有没有别的更好的办法呢？这个代码也太多了，代码可读性也不好。而且线程第一次进入还会阻塞，还能更完美吗？

#### 第五种方式：既要懒汉式加载，又要线程安全：静态内部类。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTg4ODAzMTMwNzk5YmJiNDkucG5n?x-oss-process=image/format,png)

我们来分析一下代码：相比较饿汉式（也就是第二种），我们增加了一个内部类，内部类中有一个外部类的实例，并且已经初始化了。我们回忆一下饿汉式有什么问题，饿汉式的问题是：**在你没有使用该单例的时候，该单例却被加载了，如果该单例很大的话，将会浪费很多的内存**.但是，我们现在引入了内部类的方式，虚拟机的机制是，如果你没有访问一个类，那么是不会载入该类进入虚拟机的。当我们使用外部类的时候其他属性的时候，是不会浪费内存载入内部类中的单例的。从而也就保证了并发安全和防止内存浪费。
 但是，这样就能完美了吗？

#### 第六种方式：反射和反序列化破坏单例

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLWY4MjM0ZTkwY2MxYzcwZjMucG5n?x-oss-process=image/format,png)

我们知道Java的反射几乎是什么事情都能做，管你什么私有的公有的。都能破坏。我们是没有还手之力的。精心编写的代码就被破坏了，而反序列化也很厉害，但是稍微还有点办法遏制。什么办法呢？重写readResolve方法。show me code。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLWE1ODcyZGY1MmJlZmQ2MDIucG5n?x-oss-process=image/format,png)

我们看到：我们重写了readResolve方法，在该方法中直接返回了我们的内部类实例。重写readResolve() 方法，防止反序列化破坏单例机制，这是因为：反序列化的机制在反序列化的时候，会判断如果实现了serializable或者externalizable接口的类中包含readResolve方法的话，会直接调用readResolve方法来获取实例。这样我们就制止了反序列化破坏我们的单例模式。那反射呢？我们有办法吗？

#### 第七种方式：最后一招，使用枚举

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTBjZTZiMzdhNzNlMTk1ZjMucG5n?x-oss-process=image/format,png)

为什么使用枚举可以呢？枚举类型反编译之后可以看到实际上是一个继承自Enum的类。所以本质还是一个类。 因为枚举的特点，你只会有一个实例。我们看一下反编译的枚举类。



![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MjM2NTUzLTBkNzhkM2U2NzkzYmQ1MTIucG5n?x-oss-process=image/format,png)

反编译的class字节码

我们看到，我们的hello包下的Single7枚举继承了java.lang.Enum<> 类。事实上就是一个类，但是我们这样就能防止反射破坏我们辛苦写的单例模式了。因为枚举的特点，而他也能保证单例。堪称完美！！！

### 总结

回到开始，我们引用了一些维基百科的话，我们再看看维基百科关于并发是怎么说的：

> 单例模式在多线程的应用场合下必须小心使用。如果当唯一实例尚未创建时，有两个线程同时调用创建方法，那么它们同时没有检测到唯一实例的存在，从而同时各自创建了一个实例，这样就有两个实例被构造出来，从而违反了单例模式中实例唯一的原则。 解决这个问题的办法是为指示类是否已经实例化的变量提供一个互斥锁(虽然这样会降低效率).

我们看到维基百科还是靠谱的。告诉了我们可以使用互斥锁来防止并发出现的问题。

而单例模式带来了什么好处呢？

1. 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
2. 由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

#### 4.jdk中使用了哪些设计模式？

#### **Structural（结构模式）**

**Adapter:**

- java.util.Arrays#asList()
- javax.swing.JTable(TableModel)
- java.io.InputStreamReader(InputStream)
- java.io.OutputStreamWriter(OutputStream)
- javax.xml.bind.annotation.adapters.XmlAdapter#marshal()
- javax.xml.bind.annotation.adapters.XmlAdapter#unmarshal()

**Bridge:**
把抽象和实现解藕，于是接口和实现可在完全独立开来。

- AWT (提供了抽象层映射于实际的操作系统)
- JDBC

**Composite:**
让使用者把单独的对象和组合对象混用。

- javax.swing.JComponent#add(Component)
- java.awt.Container#add(Component)
- java.util.Map#putAll(Map)
- java.util.List#addAll(Collection)
- java.util.Set#addAll(Collection)

 

**Decorator:**
为一个对象动态的加上一系列的动作，而不需要因为这些动作的不同而产生大量的继承类。这个模式在JDK中几乎无处不在，所以，下面的列表只是一些典型的。

- java.io.BufferedInputStream(InputStream)
- java.io.DataInputStream(InputStream)
- java.io.BufferedOutputStream(OutputStream)
- java.util.zip.ZipOutputStream(OutputStream)
- java.util.Collections#checked[List|Map|Set|SortedSet|SortedMap]()

**Facade:**
用一个简单的接口包状一组组件，接口，抽象或是子系统。

- java.lang.Class
- javax.faces.webapp.FacesServlet

**Flyweight:**
有效率地存储大量的小的对象。

- java.lang.Integer#valueOf(int)
- java.lang.Boolean#valueOf(boolean)
- java.lang.Byte#valueOf(byte)
- java.lang.Character#valueOf(char)

**Proxy:**
用一个简单的对象来代替一个复杂的对象。

- java.lang.reflect.Proxy
- RMI

#### **Creational（创建模式）**

**Abstract factory:**

 

- java.util.Calendar#getInstance()
- java.util.Arrays#asList()
- java.util.ResourceBundle#getBundle()
- java.sql.DriverManager#getConnection()
- java.sql.Connection#createStatement()
- java.sql.Statement#executeQuery()
- java.text.NumberFormat#getInstance()
- javax.xml.transform.TransformerFactory#newInstance()

**Builder:**
主要用来简化一个复杂的对象的创建。这个模式也可以用来实现一个 [Fluent Interface](http://en.wikipedia.org/wiki/Fluent_interface)。

- java.lang.StringBuilder#append()
- java.lang.StringBuffer#append()
- java.sql.PreparedStatement
- javax.swing.GroupLayout.Group#addComponent()

**Factory:**
简单来说，按照需求返回一个类型的实例。

- java.lang.Proxy#newProxyInstance()
- java.lang.Object#toString()
- java.lang.Class#newInstance()
- java.lang.reflect.Array#newInstance()
- java.lang.reflect.Constructor#newInstance()
- java.lang.Boolean#valueOf(String)
- java.lang.Class#forName()

**Prototype:**
使用自己的实例创建另一个实例。有时候，创建一个实例然后再把已有实例的值拷贝过去，是一个很复杂的动作。所以，使用这个模式可以避免这样的复杂性。

- java.lang.Object#clone()
- java.lang.Cloneable

**Singleton:**
只允许一个实例。在 Effective Java中建议使用Emun.

- java.lang.Runtime#getRuntime()
- java.awt.Toolkit#getDefaultToolkit()
- java.awt.GraphicsEnvironment#getLocalGraphicsEnvironment()
- java.awt.Desktop#getDesktop()

#### **Behavioral(行为模式)**

**Chain of responsibility:**
把一个对象在一个链接传递直到被处理。在这个链上的所有的对象有相同的接口（抽象类）但却有不同的实现。

- java.util.logging.Logger#log()
- javax.servlet.Filter#doFilter()

**Command:**
把一个或一些命令封装到一个对象中。

- java.lang.Runnable
- javax.swing.Action

**Interpreter:**
一个语法解释器的模式。

- java.util.Pattern
- java.text.Normalizer
- java.text.Format

**Iterator:**
提供一种一致的方法来顺序遍历一个容器中的所有元素。

- java.util.Iterator
- java.util.Enumeration

**Mediator:**
用来减少对象单的直接通讯的依赖关系。使用一个中间类来管理消息的方向。

- java.util.Timer
- java.util.concurrent.Executor#execute()
- java.util.concurrent.ExecutorService#submit()
- java.lang.reflect.Method#invoke()

**Memento:**
给一个对象的状态做一个快照。Date类在内部使用了一个long型来做这个快照。

- java.util.Date
- java.io.Serializable

**Null Object:**
这个模式用来解决如果一个Collection中没有元素的情况。

- java.util.Collections#emptyList()
- java.util.Collections#emptyMap()
- java.util.Collections#emptySet()

**Observer:**
允许一个对象向所有的侦听的对象广播自己的消息或事件。

- java.util.EventListener
- javax.servlet.http.HttpSessionBindingListener
- javax.servlet.http.HttpSessionAttributeListener
- javax.faces.event.PhaseListener

**State:**
这个模式允许你可以在运行时很容易地根据自身内部的状态改变对象的行为。

- java.util.Iterator
- javax.faces.lifecycle.LifeCycle#execute()

**Strategy:**
定义一组算法，并把其封装到一个对象中。然后在运行时，可以灵活的使用其中的一个算法。

- java.util.Comparator#compare()
- javax.servlet.http.HttpServlet
- javax.servlet.Filter#doFilter()

**Template method:**
允许子类重载部分父类而不需要完全重写。

- java.util.Collections#sort()
- java.io.InputStream#skip()
- java.io.InputStream#read()
- java.util.AbstractList#indexOf()

**Visitor:**

作用于某个对象群中各个对象的操作. 它可以使你在不改变这些对象本身的情况下,定义作用于这些对象的新操作.

- javax.lang.model.element.Element 和javax.lang.model.element.ElementVisitor
- javax.lang.model.type.TypeMirror 和javax.lang.model.type.TypeVisitor

### 5. 如何编写线程安全的单例

双重检查锁（Double-checked locking）

```
public static synchronized Singleton getInstance() {
  if(singleton == null) {
     synchronized(Singleton.class) {
       if(singleton == null) {
         singleton = new Singleton();
       }
    }
  }
  return singleton;
}
```

### 6.  代理模式有哪些类型？

+ 保护代理: 
  + 它根据某些条件控制对真实主题的访问。

+ 虚拟代理
  + 虚拟代理用于实例化昂贵的对象。代理管理实现中真实主体的生命周期。
  + 它决定创建实例的需要以及何时重用实例。虚拟代理优化性能。

+ 缓存代理
  + 缓存代理用于缓存对真实主题的昂贵调用。代理可以使用许多缓存策略。
  +  其中一些是通读、写、缓存和基于时间的。缓存代理用于提高性能。

+ 远程代理
  + 远程代理用于分布式对象通信。远程代理通过调用本地对象方法在远程对象上执行。

+ 智能代理
  + 智能代理用于实现对对象的日志调用和引用计数

### 7. mvc模式

MVC 模式代表 Model-View-Controller（模型-视图-控制器） 模式。这种模式用于应用程序的分层开发。

- **Model（模型）** - 模型代表一个存取数据的对象或 JAVA POJO。它也可以带有逻辑，在数据变化时更新控制器。
- **View（视图）** - 视图代表模型包含的数据的可视化。
- **Controller（控制器）** - 控制器作用于模型和视图上。它控制数据流向模型对象，并在数据变化时更新视图。它使视图与模型分离开。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cucnVub29iLmNvbS93cC1jb250ZW50L3VwbG9hZHMvMjAxNC8wOC9NVkMxLnBuZw?x-oss-process=image/format,png)

 

### 8. 拦截过滤器模式及其有点

拦截过滤器设计模式用于在请求处理之前和之后拦截和操作请求和响应。过滤器执行请求的身份验证/授权/日志记录或跟踪，然后将请求转发给相应的处理程序。让我们看一下拦截设计模式的一些基本实体。

**过滤器**

　　它在请求处理程序执行请求之前或之后执行特定的任务。

**过滤器链**

　　它包含多个过滤器，有助于在目标上按定义的顺序执行它们。

**目标**

　　目标对象是请求处理程序

**过滤器管理器**

　　它管理过滤器和过滤器链。 

**客户端**

　　客户机对象是向目标对象发送请求的对象。

**拦截过滤器设计模式的好处**

　　过滤器模式使用松散耦合的处理程序提供中央控制。

　　它扩展了可重用性。

　　可以随时添加新的过滤器，而不影响客户机的代码。

　　过滤器可以在程序执行期间动态选择。

###  9. dao设计模式

数据访问对象模式用于将低级数据访问API或操作与高级业务服务隔离开来。下面是DAO模式中的组件。

数据存取对象接口

　　DAO接口描述要在模型对象上执行的标准操作。

数据访问对象的具体类

　　该类实现一个DAO接口。该类负责从数据源(可以是Xml/数据库或任何其他存储机制)获取数据。

模型对象或值对象

　　这个对象是一个普通的旧java对象，包含用于存储使用DAO类检索的数据的get/set方法。
