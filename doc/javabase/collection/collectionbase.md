## 一、集合基础

### 1.1 集合框架有哪些优点如下：

+ 使用核心集合类降低开发成本，而非实现我们自己的集合类。
+ 随着使用经过严格测试的集合框架类，代码质量会得到提高。
+ 通过使用 JDK 附带的集合类，可以降低代码维护成本。
+ 复用性和可操作性。

### 1.2 Java集合类框架的基本接口有哪些？

Java 集合类提供了一套设计良好的支持对一组对象进行操作的接口和类。Java集合类里面最基本的接口有：

+ Collection：代表一组对象，每一个对象都是它的子元素。
+ Set：不包含重复元素的 Collection。
+ List：有顺序的 collection，并且可以包含重复元素。
+ Map：可以把键(key)映射到值(value)的对象，键不能重复。
+ 还有其它接口 Queue、Dequeue、SortedSet、SortedMap 和 ListIterator。

### 1.3 为什么集合类没有实现 Cloneable 和 Serializable 接口？

集合类接口指定了一组叫做元素的对象。集合类接口的每一种具体的实现类都可以选择以它自己的方式对元素进行保存和排序，可以使得集合类很灵活，可以实现自定义集合类属性，比如有的集合类允许重复的键，有些不允许。

### 1.4 集合框架中的泛型有什么优点？

Java5 引入了泛型，所有的集合接口和实现都大量地使用它。泛型允许我们为集合提供一个可以容纳的对象类型。因此，如果你添加其它类型的任何元素，它会在编译时报错。这避免了在运行时出现 ClassCastException，因为你将会在编译时得到报错信息。

泛型也使得代码整洁，我们不需要使用显式转换和 instanceOf 操作符。它也给运行时带来好处，因为不会产生类型检查的字节码指令。

### 1.5 Collection 和 Collections 的区别？

+ Collection ，是集合类的上级接口，继承与他的接口主要有 Set 和List 。
+ Collections ，是针对集合类的一个工具类，它提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。

### 1.6 什么是迭代器(Iterator)？

Iterator 接口提供了很多对集合元素进行迭代的方法。每一个集合类都包含了可以返回迭代器实例的 迭代方法。迭代器可以在迭代的过程中删除底层集合的元素。

克隆(cloning)或者是序列化(serialization)的语义和含义是跟具体的实现相关的。因此，应该由集合类的具体实现来决定如何被克隆或者是序列化。

### 1.7 Iterator和ListIterator的区别是什么？

下面列出了他们的区别：

+ Iterator 可用来遍历 Set 和 List 集合，但是 ListIterator 只能用来遍历 List 。
+ Iterator 对集合只能是前向遍历，ListIterator 既可以前向也可以后向。
+ ListIterator 实现了 Iterator 接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。

### 1.8 快速失败(fail-fast)和安全失败(fail-safe)的区别是什么？

Iterator 的安全失败是基于对底层集合做拷贝，因此，它不受源集合上修改的影响。

java.util 包下面的所有的集合类都是快速失败的，而 java.util.concurrent 包下面的所有的类都是安全失败的。快速失败的迭代器会抛出 ConcurrentModificationException 异常，而安全失败的迭代器永远不会抛出这样的异常。

### 1.9 Enumeration 接口和 Iterator 接口的区别有哪些？

Enumeration 速度是 Iterator 的2倍，同时占用更少的内存。但是，Iterator 远远比 Enumeration 安全，因为其他线程不能够修改正在被 iterator 遍历的集合里面的对象。同时，Iterator 允许调用者删除底层集合里面的元素，这对 Enumeration 来说是不可能的。

### 1.10 Java集合类框架的最佳实践有哪些？

根据应用的需要正确选择要使用的集合的类型对性能非常重要，比如：假如元素的大小是固定的，而且能事先知道，我们就应该用 Array 而不是 ArrayList。 有些集合类允许指定初始容量。因此，如果我们能估计出存储的元素的数目，我们可以设置初始容量来避免重新计算 hash 值或者是扩容。

为了类型安全，可读性和健壮性的原因总是要使用泛型。同时，使用泛型还可以避免运行时的 ClassCastException。

使用 JDK 提供的不变类(immutable class)作为Map的键可以避免为我们自己的类实现 hashCode() 和 equals() 方法。

编程的时候接口优于实现。

底层的集合实际上是空的情况下，返回长度是0的集合或者是数组，不要返回 null。

## 二、HashMap、Hashtable

### 2.1 HashMap的结构（jdk1.8）

HashMap（数组+链表+红黑树）的结构，利用了红黑树，所以其由 数组+链表+红黑
树组成：

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7e915afd4201c?w=1383&h=638&f=png&s=85707)
HashMap 里面是一个数组，然后数组中每个元素是一个单向链表。上图中，每个绿色
的实体是嵌套类 Entry 的实例， Entry 包含四个属性： key, value, hash 值和用于单向链表的 next。

### 2.2 Java 中的 HashMap

HashMap 根据键的 hashCode 值存储数据，大多数情况下可以直接定位到它的值，因而具有很快
的访问速度，但遍历顺序却是不确定的。  

HashMap 最多只允许一条记录的键为 null，允许多条记
录的值为 null。   

HashMap 非线程安全，即任一时刻可以有多个线程同时写 HashMap，可能会导
致数据的不一致。如果需要满足线程安全，可以用 Collections 的 synchronizedMap 方法使
HashMap 具有线程安全的能力，或者使用 ConcurrentHashMap。



### 2.3 HashMap重要参数

1. capacity：当前数组容量16 ，始终保持 2^n，可以扩容，扩容后数组大小为当前的 2 倍。
2. loadFactor：负载因子，默认为 0.75
3. threshold：扩容的阈值，等于 capacity * loadFactor

### 2.4 HashMap查询

查找的时候，根据 hash 值我们能够快速定位到数组的
具体下标，但是之后的话， 需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决
于链表的长度，为 O(n)。 

为了降低这部分的开销，在 Java8 中， 当链表中的元素超过了 8 个以后，
会将链表转换为红黑树，在这些位置进行查找的时候可以降低时间复杂度为 O(logN)。

### 2.5 hashCode() 和 equals() 方法的重要性体现在什么地方？

Java 中的 HashMap 使用 hashCode() 和 equals() 方法来确定键值对的索引，当根据键获取值的时候也会用到这两个方法。如果没有正确的实现这两个方法，两个不同的键可能会有相同的 hash 值。

因此，可能会被集合认为是相等的。而且，这两个方法也用来发现重复元素。所以这两个方法的实现对 HashMap 的精确性和正确性是至关重要的。

### 2.6 Hashtable

哈希表（HashTable）又叫做散列表，根它通过把key映射到表中一个位置来访问记录，以加快查找速度。这个映射函数就叫做散列（哈希）函数，存放记录的数组叫做散列表。

哈希表是一个时间和空间上平衡的例子。如果没有空间的限制，我们可以直接用键来作为数组的索引，这样可以将查找时间做到最快（O(1)）。如果没有时间的限制，我们可以使用无序链表进行顺序查找，这样只需要很少的内存

### 2.7 为什么Hashtable的速度快？

Hashtable是由数组与链表。数组的特点就是查找容易，插入删除困难；而链表的特点就是查找困难，但是插入删除容易。既然两者各有优缺点，那么Hashtable查找容易，插入删除也会快起来。

### 2.8 Hashtable如何根据key查找？

使用哈希函数将被查找的key转化为数组的索引。在理想的状态下，不同的键会被转化成不同的索引值。但是那是理想状态，我们实践当中是不可能一直是理想状态的。当不同的键生成了相同的索引的时候，即是哈希冲突，处理冲突方式：

+ 拉链法
+ 线性探索法

### 2.9 LinkHashMap

LinkHashMapshi=HashMap + LinkedList

LinkedHashMap 是基于 HashMap 实现的一种集合，具有 HashMap 集合上面所说的所有特点，除了 HashMap 无序的特点，LinkedHashMap 是有序的，因为 LinkedHashMap 在 HashMap 的基础上单独维护了一个具有所有数据的双向链表，该链表保证了元素迭代的顺序。


![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ebe4cc3f1e86?w=739&h=390&f=png&s=101271)

+ LinkedHashMap是继承于HashMap，是基于HashMap和双向链表来实现的。
+ HashMap无序；LinkedHashMap有序，可分为插入顺序和访问顺序两种。如果是访问顺序，那put和get操作已存在的Entry时，都会把Entry移动到双向链表的表尾(其实是先删除再插入)。
+ LinkedHashMap存取数据，还是跟HashMap一样使用的Entry[]的方式，双向链表只是为了保证顺序。
+ LinkedHashMap是线程不安全的

## 三、ArrayList、 Vector 和 LinkedList

### 3.1 ArrayList（数组）

ArrayList 是最常用的 List 实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。

数组的缺点是每个元素之间不能有间隔， 当数组大小不满足时需要增加存储能力，就要将已经有数
组的数据复制到新的存储空间中。 

当从 ArrayList 的中间位置插入或者删除元素时，需要对数组进
行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。

ArrayList支持序列化功能，支持克隆（浅拷贝）功能，排序功能等

### 3.2 ArrayList 是如何扩容的？

如果通过无参构造的话，初始数组容量为 0 ，当真正对数组进行添加时，才真正分配容量。**每次按照 1.5 倍（位运算）的比率通过 copeOf 的方式扩容**。  

**在 JKD6 中实现是，如果通过无参构造的话，初始数组容量为10，每次通过 copeOf 的方式扩容后容量为原来的 1.5 倍**

### 3.3 ArrayList 集合加入 1 万条数据，应该怎么提高效率？

ArrayList 的默认初始容量为 10 ，要插入大量数据的时候需要不断扩容，而扩容是非常影响性能的。因此，现在明确了 10 万条数据了，我们可以直接在初始化的时候就设置 ArrayList 的容量！

### 3.4 Vector（数组实现、 线程同步）

Vector 与 ArrayList 一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一
个线程能够写 Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，
访问它比访问 ArrayList 慢。

### 3.5 LinkList（链表）

LinkedList 是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较
慢。另外，他还提供了 List 接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆
栈、队列和双向队列使用


## 四、HashSet、TreeSet以及LinkHashSet

Set 注重独一无二的性质,该体系集合用于存储无序(存入和取出的顺序不一定相同)元素， 值不能重复。

对象的相等性本质是对象 hashCode 值（ java 是依据对象的内存地址计算出的此序号） 判断的， 如果想要让两个不同的对象视为相等的，就必须覆盖 Object 的 hashCode 方法和 equals 方法。

### 4.1 HashSet

哈希表边存放的是哈希值。 HashSet 存储元素的顺序并不是按照存入时的顺序（和 List 显然不同） 而是按照哈希值来存的所以取数据也是按照哈希值取得。元素的哈希值是通过元素的hashcode 方法来获取的, HashSet 首先判断两个元素的哈希值，如果哈希值一样，接着会比较equals 方法 如果 equls 结果为 true ， HashSet 就视为同一个元素。如果 equals 为 false 就不是同一个元素。

HashSet 通过 hashCode 值来确定元素在内存中的位置。 一个 hashCode 位置上可以存放多个元素。


哈希值相同 equals 为 false 的元素是怎么存储呢,就是在同样的哈希值下顺延（可以认为哈希值相同的元素放在一个哈希桶中）。也就是哈希一样的存一列。 如图 1 表示 hashCode 值不相同的情况； 图 2 表示 hashCode 值相同，但 equals 不相同的情况。

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7eb3674785cc6?w=1224&h=491&f=png&s=99145)

### 4.2 TreeSet（二叉树）

1. TreeSet()是使用二叉树的原理对新add()的对象按照指定的顺序排序（升序、降序），每增加一个对象都会进行排序，将对象插入的二叉树指定的位置。
2. Integer 和 String 对象都可以进行默认的 TreeSet 排序，而自定义类的对象是不可以的， 自己定义的类必须实现 Comparable 接口，并且覆写相应的 compareTo()函数，才可以正常使用。
3. 在覆写 compare()函数时，要返回相应的值才能使 TreeSet 按照一定的规则来排序
4. 比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则分别返回负整数、零或正整数


### 4.3 LinkHashSet（ HashSet+LinkedHashMap）

对于 LinkedHashSet 而言，它继承与 HashSet、又基于 LinkedHashMap 来实现的。LinkedHashSet 底层使用 LinkedHashMap 来保存所有元素，它继承与 HashSet，其所有的方法操作上又与 HashSet 相同.

因此 LinkedHashSet的实现上非常简单，只提供了四个构造方法，并通过传递一个标识参数，调用父类的构造器，底层构造一个 LinkedHashMap 来实现，在相关操作上与父类 HashSet 的操作相同，直接调用父类 HashSet 的方法即可。

## 五、集合的区别

### 5.1 HashMap 和 Hashtable 有什么区别？

HashMap 和 Hashtable 都实现了 Map 接口，因此很多特性非常相似。但是，他们有以下不同点： HashMap 允许键和值是 null，而 Hashtable 不允许键或者值是 null。

Hashtable 是同步的，而 HashMap 不是。因此， HashMap 更适合于单线程环境，而 Hashtable 适合于多线程环境。

HashMap 提供了可供应用迭代的键的集合，因此，HashMap 是快速失败的。另一方面，Hashtable 提供了对键的列举(Enumeration)。

一般认为 Hashtable 是一个遗留的类。

### 5.2 数组(Array)和列表(ArrayList)有什么区别？什么时候应该使用 Array 而不是 ArrayList？

下面列出了 Array 和 ArrayList 的不同点：

Array 可以包含基本类型和对象类型，ArrayList 只能包含对象类型。

Array 大小是固定的，ArrayList 的大小是动态变化的。

ArrayList 提供了更多的方法和特性，比如：addAll()，removeAll()，iterator()等等。 对于基本类型数据，集合使用自动装箱来减少编码工作量。但是，当处理固定大小的基本数据类型的时候，这种方式相对比较慢。

### 5.3 ArrayList 和 LinkedList 有什么区别？

ArrayList 和 LinkedList 都实现了 List 接口，他们有以下的不同点：

ArrayList 是基于索引的数据接口，它的底层是数组。它可以以O(1)时间复杂度对元素进行随机访问。与此对应，LinkedList 是以元素列表的形式存储它的数据，每一个元素都和它的前一个和后一个元素链接在一起，在这种情况下，查找某个元素的时间复杂度是O(n)。

相对于 ArrayList，LinkedList 的插入，添加，删除操作速度更快，因为当元素被添加到集合任意位置的时候，不需要像数组那样重新计算大小或者是更新索引。

LinkedList 比 ArrayList 更占内存，因为 LinkedList 为每一个节点存储了两个引用，一个指向前一个元素，一个指向下一个元素。

也可以参考 ArrayList vs. LinkedList。

### 5.4 Comparable 和Comparator 接口是干什么的？列出它们的区别。

Java 提供了只包含一个 compareTo() 方法的 Comparable 接口。这个方法可以个给两个对象排序。具体来说，它返回负数，0，正数来表明输入对象小于，等于，大于已经存在的对象。

Java 提供了包含 compare() 和 equals() 两个方法的 Comparator 接口。compare() 方法用来给两个输入参数排序，返回负数，0，正数表明第一个参数是小于，等于，大于第二个参数。equals() 方法需要一个对象作为参数，它用来决定输入参数是否和 comparator 相等。只有当输入参数也是一个 comparator 并且输入参数和当前 comparator 的排序结果是相同的时候，这个方法才返回 true。


### 5.5 HashSet 和 TreeSet 有什么区别？

HashSet 是由一个 hash 表来实现的，因此，它的元素是无序的。add()，remove()，contains()方法的时间复杂度是 O(1)。

另一方面，TreeSet 是由一个树形的结构来实现的，它里面的元素是有序的。因此，add()，remove()，contains() 方法的时间复杂度是 O(logn)。

### 5.6 HashMap 和 ConcurrentHashMap 的区别？

ConcurrentHashMap 是线程安全的 HashMap 的实现。主要区别如下：

1. ConcurrentHashMap 对整个桶数组进行了分割分段(Segment)，然后在每一个分段上都用 lock 锁进行保护，相对 于Hashtable 的 syn 关键字锁的粒度更精细了一些，并发性能更好。而 HashMap 没有锁机制，不是线程安全的。

2. HashMap 的键值对允许有 null ，但是 ConCurrentHashMap 都不允许

>JDK8 之后，ConcurrentHashMap 启用了一种全新的方式实现,利用 CAS 算法。

### 5.7 List、Set、Map 是否继承自 Collection 接口？

List、Set 是，Map 不是。Map 是键值对映射容器，与 List 和 Set 有明显的区别，而 Set 存储的零散的元素且不允许有重复元素（数学中的集合也是如此），List 是线性结构的容器，适用于按数值索引访问元素的情形。

### 5.8 说出 ArrayList、Vector、LinkedList 的存储性能和特性？

ArrayList 和 Vector 都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引娶元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector 由于使用了 synchronized 方法（线程安全），通常性能上较 ArrayList 差。

而 LinkedList 使用双向链表实现存储（将内存中零散的内存单元通过附加的引用关联起来，形成一个可以按序号索引的线性结构，这种链式存储方式与数组的连续存储方式相比，其实对内存的利用率更高），按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。

Vector 属于遗留容器（早期的 JDK 中使用的容器，除此之外 Hashtable、Dictionary、BitSet、Stack、Properties 都是遗留容器），现在已经不推荐使用，但是由于 ArrayList 和 LinkedListed 都是非线程安全的，如果需要多个线程操作同一个容器，那么可以通过工具类 Collections 中的 synchronizedList 方法将其转换成线程安全的容器后再使用（这其实是装潢模式最好的例子，将已有对象传入另一个类的构造器中创建新的对象来增加新功能）。

### 5.9 List、Map、Set 三个接口存储元素时各有什么特点？

+ List 是有序的 Collection，使用此接口能够精确的控制每个元素插入的位置。用户能够使用索引（元素在 List 中的位置，类似于数组下标）来访问 List 中的元素，这类似于 Java 的数组。
+ Set 是一种不包含重复的元素的 Collection，即任意的两个元素 e1 和 e2 都有e1.equals(e2)=false，Set 最多有一个 null 元素。
+ Map 接口 ：请注意，Map 没有继承 Collection 接口，Map 提供 key 到 value 的映射
