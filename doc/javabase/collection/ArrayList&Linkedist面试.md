### 前言

在一开始基础面的时候，很多面试官可能会问List集合一些基础知识，比如：

- `ArrayList`默认大小是多少，是如何扩容的？
- `ArrayList`和`LinkedList`的底层数据结构是什么？
- `ArrayList`和`LinkedList`的区别？分别用在什么场景？
- 为什么说`ArrayList`查询快而增删慢？
- `Arrays.asList`方法后的List可以扩容吗？
- `modCount`在非线程安全集合中的作用？
- `ArrayList`和`LinkedList`的区别、优缺点以及应用场景



------

### ArrayList(1.8)

`ArrayList`是由动态再分配的`Object[]`数组作为底层结构，可设置`null`值，是非线程安全的。

#### ArrayList成员属性

```java
//默认的空的数组，在构造方法初始化一个空数组的时候使用
private static final Object[] EMPTY_ELEMENTDATA = {};

//使用默认size大小的空数组实例
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//ArrayList底层存储数据就是通过数组的形式，ArrayList长度就是数组的长度。
transient Object[] elementData; 

//arrayList的大小
private int size;
```

那么`ArrayList`底层数据结构是什么呢？

很明显，使用动态再分配的`Object[]`数组作为`ArrayList`底层数据结构了，既然是使用数组实现的，那么数组特点就能说明为什么ArrayList查询快而增删慢？

因为数组是根据下标查询不需要比较，查询方式为：首地址＋（元素长度＊下标），基于这个位置读取相应的字节数就可以了，所以非常快；但是增删会带来元素的移动，增加数据会向后移动，删除数据会向前移动，导致其效率比较低。



#### ArrayList的构造方法

- 带有初始化容量的构造方法
- 无参构造方法
- 参数为`Collection`类型的构造器

```java
//带有初始化容量的构造方法
public ArrayList(int initialCapacity) {
    //参数大于0，elementData初始化为initialCapacity大小的数组
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    //参数小于0，elementData初始化为空数组
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    //参数小于0，抛出异常
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

//无参构造方法
public ArrayList() {
    //在1.7以后的版本，先构造方法中将elementData初始化为空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    //当调用add方法添加第一个元素的时候，会进行扩容,扩容至大小为DEFAULT_CAPACITY=10
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

那么`ArrayList`默认大小是多少？

从无参构造方法中可以看出，一开始默认为一个空的实例`elementData`为上面的`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，当添加第一个元素的时候会进行扩容，扩容大小就是上面的默认容量`DEFAULT_CAPACITY`为**10**

#### ArrayList的Add方法

- `boolean add(E)`：默认直接在末尾添加元素
- `void add(int，E)`：在特定位置添加元素，也就是插入元素
- `boolean addAll(Collection<? extends E> c)`：添加集合
- `boolean addAll(int index, Collection<? extends E> c)`：在指定位置后添加集合


##### boolean add(E)

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

通过`ensureCapacityInternal`方法为确定容量大小方法。在添加元素之前需要确定数组是否能容纳下，`size`是数组中元素个数，添加一个元素size+1。然后再数组末尾添加元素。

其中，`ensureCapacityInternal`方法包含了`ArrayList`扩容机制`grow`方法，当前容量无法容纳下数据时**1.5倍**扩容，进行：

```java
private void ensureCapacityInternal(int minCapacity) {
    //判断当前的数组是否为默认设置的空数据，是否取出最小容量
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
	//包括扩容机制grow方法
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    	//记录着集合的修改次数，也就每次add或者remove它的值都会加1
        modCount++;

        //当前容量容纳不下数据时（下标超过时），ArrayList扩容机制：扩容原来的1.5倍
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
    	//ArrayList扩容机制：扩容原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

`ArrayList`是如何扩容的？

根据当前的容量容纳不下新增数据时，`ArrayList`会调用`grow`进行扩容：

```java
//相当于int newCapacity = oldCapacity + oldCapacity/2
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

扩容原来的1.5倍。



##### void add(int，E)

```java
public void add(int index, E element) {
    //检查index也就是插入的位置是否合理,是否存在数组越界
    rangeCheckForAdd(index);
	//机制和boolean add(E)方法一样
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

#### ArrayList的删除方法

- **remove(int)：**通过删除指定位置上的元素，
- **remove(Object)**：根据元素进行删除，
- **clear()：**将`elementData`中每个元素都赋值为null，等待垃圾回收将这个给回收掉，
- **removeAll(collection c)：**批量删除。



##### remove(int)

```java
public E remove(int index) {
    //检查下标是否超出数组长度，造成数组越界
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);
	//算出数组需要移动的元素数量
    int numMoved = size - index - 1;
    if (numMoved > 0)
        //数组数据迁移，这样会导致删除数据时，效率会慢
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //将--size上的位置赋值为null，让gc(垃圾回收机制)更快的回收它。
    elementData[--size] = null; // clear to let GC do its work
	//返回删除的元素
    return oldValue;
}
```

为什么说`ArrayList`删除元素效率低？

因为删除数据需要将数据后面的元素数据迁移到新增位置的后面，这样导致**性能下降很多**，效率低。

##### remove(Object)

```java
public boolean remove(Object o) {
    //如果需要删除数据为null时，会让数据重新排序，将null数据迁移到数组尾端
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                //删除数据，并迁移数据
                fastRemove(index);
                return true;
            }
    } else {
        //循环删除数组中object对象的值，也需要数据迁移
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

可以看出，**`arrayList`是可以存放`null`值。**



------

### LinkedList(1.8)

 `LinkedList`是一个继承于`AbstractSequentialList`的双向链表。它也可以被当做堆栈、队列或双端队列进行使用，而且`LinkedList`也为非线程安全， jdk1.6使用的是一个带有 `header`节头结点的双向循环链表， 头结点不存储实际数据 ，在1.6之后，就变更使用两个节点`first`、`last`指向首尾节点。

####  LinkedList的主要属性

```java
//链表节点的个数 
transient int size = 0; 
//链表首节点
 transient Node<E> first; 
//链表尾节点
 transient Node<E> last; 
//Node节点内部类定义
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

 一旦变量被`transient`修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问 

#### LinkedList构造方法

无参构造函数， 默认构造方法声明也不做，`first`和`last`节点会被默认初始化为null。 

```java
*
/** Constructs an empty list. \*/*

public LinkedList() {}

```



#### LinkedList插入

由于`LinkedList`由双向链表作为底层数据结构，因此其插入无非由三大种

+ 尾插： `add(E e)`、`addLast(E e)`、`addAll(Collection<? extends E> c)`

+ 头插： `addFirst(E e)`
+ 中插： `add(int index, E element)`

 可以从源码看出，在链表首尾添加元素很高效，在中间添加元素比较低效，首先要找到插入位置的节点，在修改前后节点的指针。 

![](https://user-gold-cdn.xitu.io/2020/6/26/172ee0f83a1d92d2?w=828&h=307&f=png&s=24403)

##### 尾插-add(E e)和addLast(E e)

```java
//常用的添加元素方法
public boolean add(E e) {
    //使用尾插法
    linkLast(e);
    return true;
}

//在链表尾部添加元素
public void addLast(E e) {
        linkLast(e);
    }

//在链表尾端添加元素
void linkLast(E e) {
    	//尾节点
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
    	//判断是否是第一个添加的元素
        //如果是将新节点赋值给last
        //如果不是把原首节点的prev设置为新节点
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        //将集合修改次数加1
        modCount++;
    }

```



##### 头插-addFirst(E e)

```java
public void addFirst(E e) {
    //在链表头插入指定元素
    linkFirst(e);
}

private void linkFirst(E e) {
   		 //获取头部元素,首节点
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
    	//链表头部为空，（也就是链表为空）
    	//插入元素为首节点元素
    	// 否则就更新原来的头元素的prev为新元素的地址引用
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
    	//
        size++;
        modCount++;
    }
```

##### 中插-add(int index, E element)

当`index`不为首尾的的时候，实际就在链表中间插入元素。

```java
 // 作用：在指定位置添加元素
    public void add(int index, E element) {
        // 检查插入位置的索引的合理性
        checkPositionIndex(index);

        if (index == size)
            // 插入的情况是尾部插入的情况：调用linkLast（）。
            linkLast(element);
        else
            // 插入的情况是非尾部插入的情况（中间插入）：linkBefore
            linkBefore(element, node(index));
    }

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;  // 得到插入位置元素的前继节点
        final Node<E> newNode = new Node<>(pred, e, succ);  // 创建新节点，其前继节点是succ的前节点，后接点是succ节点
        succ.prev = newNode;  // 更新插入位置（succ）的前置节点为新节点
        if (pred == null)
            // 如果pred为null，说明该节点插入在头节点之前，要重置first头节点 
            first = newNode;
        else
            // 如果pred不为null，那么直接将pred的后继指针指向newNode即可
            pred.next = newNode;
        size++;
        modCount++;
    }
```

#### LinkedList 删除

 删除和插入一样，其实本质也是只有三大种方式，

+ 删除首节点:`removeFirst()`
+ 删除尾节点:`removeLast()`
+ 删除中间节点 ：`remove(Object o)`、`remove(int index)`

 在首尾节点删除很高效，删除中间元素比较低效要先找到节点位置，再修改前后指针指引。  


![](https://user-gold-cdn.xitu.io/2020/6/26/172ee10ae541b839?w=831&h=282&f=png&s=21365)



##### 删除中间节点-remove(int index)和remove(Object o)

`remove(int index)`和`remove(Object o)`都是使用删除指定节点的`unlink`删除元素

```java
 public boolean remove(Object o) {
     //因为LinkedList允许存在null，所以需要进行null判断        
     if (o == null) {
         //从首节点开始遍历
         for (Node<E> x = first; x != null; x = x.next) {
             if (x.item == null) {
                 //调用unlink方法删除指定节点
                 unlink(x);
                 return true;
             }
         }
     } else {
         for (Node<E> x = first; x != null; x = x.next) {
             if (o.equals(x.item)) {
                 unlink(x);
                 return true;
             }
         }
     }
    return false;
 } 

//删除指定位置的节点，其实和上面的方法差不多
	//通过node方法获得指定位置的节点，再通过unlink方法删除
    public E remove(int index) {
        checkElementIndex(index);
       
        return unlink(node(index));
    }

 //删除指定节点
    E unlink(Node<E> x) {
        //获取x节点的元素，以及它上一个节点，和下一个节点
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;
		//如果x的上一个节点为null，说明是首节点，将x的下一个节点设置为新的首节点
        //否则将x的上一节点设置为next，将x的上一节点设为null
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
		//如果x的下一节点为null，说明是尾节点，将x的上一节点设置新的尾节点
        //否则将x的上一节点设置x的上一节点，将x的下一节点设为null
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
		//将x节点的元素值设为null，等待垃圾收集器收集
        x.item = null;
        //链表节点个数减1
        size--;
        //将集合修改次数加1
        modCount++;
        //返回删除节点的元素值
        return element;
    }
```

##### 删除首节点-removeFirst()

```java
//删除首节点
public E remove() {
        return removeFirst();
    }
 //删除首节点
 public E removeFirst() {
      final Node<E> f = first;
      //如果首节点为null，说明是空链表，抛出异常
      if (f == null)
          throw new NoSuchElementException();
      return unlinkFirst(f);
  }
  //删除首节点
  private E unlinkFirst(Node<E> f) {
      //首节点的元素值
      final E element = f.item;
      //首节点的下一节点
      final Node<E> next = f.next;
      //将首节点的元素值和下一节点设为null，等待垃圾收集器收集
      f.item = null;
      f.next = null; // help GC
      //将next设置为新的首节点
      first = next;
      //如果next为null，说明说明链表中只有一个节点，把last也设为null
      //否则把next的上一节点设为null
      if (next == null)
          last = null;
      else
          next.prev = null;
      //链表节点个数减1
      size--;
      //将集合修改次数加1
      modCount++;
      //返回删除节点的元素值
      return element;
 }
```

##### 删除尾节点-removeLast()

```java
    //删除尾节点
    public E removeLast() {
        final Node<E> l = last;
        //如果首节点为null，说明是空链表，抛出异常
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
    private E unlinkLast(Node<E> l) {
       	//尾节点的元素值
        final E element = l.item;
        //尾节点的上一节点
        final Node<E> prev = l.prev;
        //将尾节点的元素值和上一节点设为null，等待垃圾收集器收集
        l.item = null;
        l.prev = null; // help GC
        //将prev设置新的尾节点
        last = prev;
        //如果prev为null，说明说明链表中只有一个节点，把first也设为null
        //否则把prev的下一节点设为null
        if (prev == null)
            first = null;
        else
            prev.next = null;
        //链表节点个数减1
        size--;
        //将集合修改次数加1
        modCount++;
        //返回删除节点的元素值
        return element;
    }
```

其他方法也是类似的，比如查询方法 `LinkedList`提供了`get`、`getFirst`、`getLast`等方法获取节点元素值。 

#### modCount属性的作用？

 `modCount`属性代表为结构性修改（ 改变list的size大小、以其他方式改变他导致正在进行迭代时出现错误的结果）的次数，该属性被`Iterato`r以及`ListIterator`的实现类所使用，且很多非线程安全使用`modCount`属性。

​	初始化迭代器时会给这个modCount赋值，如果在遍历的过程中，一旦发现这个对象的modCount和迭代器存储的modCount不一样，`Iterator`或者`ListIterator` 将抛出`ConcurrentModificationException`异常，

这是jdk在面对迭代遍历的时候为了避免不确定性而采取的 fail-fast（快速失败）原则：

在线程不安全的集合中，如果使用迭代器的过程中，发现集合被修改，会抛出`ConcurrentModificationExceptions`错误，这就是fail-fast机制。对集合进行结构性修改时，`modCount`都会增加，在初始化迭代器时，`modCount`的值会赋给`expectedModCount`，在迭代的过程中，只要`modCount`改变了，`int expectedModCount = modCount`等式就不成立了，迭代器检测到这一点，就会抛出错误：`urrentModificationExceptions`。



------

### 总结

#### ArrayList和LinkedList的区别、优缺点以及应用场景

区别：

+ `ArrayList`是实现了基于动态数组的数据结构，`LinkedList`是基于链表结构。
+ 对于随机访问的`get`和`set`方法查询元素，`ArrayList`要优于`LinkedList`，因为`LinkedList`循环链表寻找元素。
+ 对于新增和删除操作`add`和`remove`，`LinkedList`比较高效，因为`ArrayList`要移动数据。

优缺点：

+ 对`ArrayList`和`LinkedList`而言，在末尾增加一个元素所花的开销都是固定的。对`ArrayList`而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；而对`LinkedList`而言，这个开销是 统一的，分配一个内部`Entry`对象。
+ 在`ArrayList`集合中添加或者删除一个元素时，当前的列表移动元素后面所有的元素都会被移动。而`LinkedList`集合中添加或者删除一个元素的开销是固定的。
+ `LinkedList`集合不支持 高效的随机随机访问（`RandomAccess`），因为可能产生二次项的行为。
+ `ArrayList`的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而`LinkedList`的空间花费则体现在它的每一个元素都需要消耗相当的空间

应用场景:

 `ArrayList`使用在查询比较多，但是插入和删除比较少的情况，而`LinkedList`用在查询比较少而插入删除比较多的情况 
