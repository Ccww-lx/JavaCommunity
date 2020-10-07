>该系列文章收录在公众号【Ccww技术博客】，原创技术文章早于博客推出

### 前言
集合在基础面试中是必备可缺的一部分，其中重要的HashMap更是少不了，那面试官会面试中提问那些问题呢，这些在JDK1.7和1.8有什么区别？？

- **HashMap的底层原理**
- **HashMap的hash哈希函数的设计原理，以及HashMap下标获取方式？**
- **HashMap扩容机制，hashMap中什么时候需要进行扩容，扩容resize()又是如何实现的**
- **hashMap中put是如何实现的 ，JDK1.7和1.8有什么区别？**
- **hashMap中get是如何实现的** 
- **其他涉及问题**
  - **HashMap具备的特性**
  - **为什么Hash的底层数据长度总为2的N次方？如果输入值不是2的幂比如10会怎么样？**
  - **加载因子为什么是 0.75？**
  - **哈希表如何解决Hash冲突**
  - **当有哈希冲突时，HashMap 是如何查找并确认元素的？**
  - **HashMap 是线程安全的吗，为什么不是线程安全的？**
       

### 1. HashMap的底层原理

JDK1.7使用的是数组+ 单链表的数据结构。JDK1.8之后，使用的是数组+链表+红黑树的数据结构

#### HashMap数据结构图（jdk1.8）
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b459755e05842e38c5039e204e1096e~tplv-k3u1fbpfcp-zoom-1.image)

```java
//解决hash冲突，链表转成树的阈值，当桶中链表长度大于8时转成树 
static final int TREEIFY_THRESHOLD = 8;
//进行resize操作时，若桶中数量少于6则从树转成链表
static final int UNTREEIFY_THRESHOLD = 6;
/* 当需要将解决 hash 冲突的链表转变为红黑树时，需要判断下此时数组容量，若是由于数组容量太小（小于　MIN_TREEIFY_CAPACITY　）导致的 hash 冲突太多，则不进行链表转变为红黑树操作，转为利用　resize() 函数对　hashMap 扩容　*/
static final int MIN_TREEIFY_CAPACITY = 64;
```

从HashMap常量中可以看出，当链表的深度达到8的时候，也就是默认阈值TREEIFY_THRESHOLD=8，就会自动扩容把链表转成红黑树的数据结构来把时间复杂度从O(n)变成O(logN)提高了效率，而且当进行resize操作时，若桶中数量少于6则从树转成链表。

#### 那为什么数据结构需要从JDK1.7换成JDK1.8的数组+链表+红黑树？

在JDK1.7中，当相同的hash值时，HashMap不断地产生碰撞，那么相同key位置的链表就会不断增长，当查询HashMap的相应key值的Vaule值时，就会去循环遍历这个超级大的链表，查询性能非常低下。

但在JDK1.8当链表超过8个节点数时，将会让红黑树来替代链表，查询性能得到了很好的提升，从原来的是O(n)到O(logn)。



### 2. HashMap的hash哈希函数的设计原理，以及HashMap下标获取 hash &（n - 1）？

#### hash哈希函数的设计原理

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

1. 首先获取hashcode，一个32位的int值
2. 然后将hashcode左移16位的值进行与或，即将高位与低位进行异或运算，减少碰撞机率。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13d9ace343d64343b34be65d5312cb52~tplv-k3u1fbpfcp-zoom-1.image)

#### HashMap下标获取h % n = h &（n - 1）

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c078bf47fed74fa6ac51d2372c5f8294~tplv-k3u1fbpfcp-zoom-1.image)

3. 取余运算，但在计算机运算中&肯定比%快，又因为h % n = h &（n - 1），所以最终将第二步得到的hash跟n-1进行与运算。n是table中的长度。

**设计原因：**

1. 一定要尽可能降低hash碰撞，越分散越好；
2. 算法一定要尽可能高效，因为这是高频操作, 因此采用位运算；



### 3. HashMap扩容机制resize（）

**HashMap扩容步骤分成两步：**

+ 获取新值：新的容量值newCap ，新的扩容阀界值newThr获取
+ 数据迁移：如果oldTab老数组不为空，说明是扩容操作，那么涉及到元素的转移操，遍历老数组，如果当前位置元素不为空，那么需要转移该元素到新数组



#### 获取新值：新的容量值newCap ，新的扩容阀界值newThr获取

- 扩容变量 

  ```java
  //原的元素数组
  Node<K,V>[] oldTab = table; 
  //老的元素数组长度
  int oldCap = (oldTab == null) ? 0 : oldTab.length; 
  // 老的扩容阀值设置
  int oldThr = threshold;
  // 新数组的容量，新数组的扩容阀值都初始化为0
  int newCap, newThr = 0;
  // 设置map的扩容阀值为 新的阀值
   threshold = newThr; 
   //创建新的数组（对于第一次添加元素，那么这个数组就是第一个数组；对于存在oldTab的时候，那么这个数组就是要需要扩容到的新数组）
   Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
   // 将该map的table属性指向到该新数组
    table = newTab;	
  ```

  

- 当如果老数组长度oldCap > 0，说明已经存在元素，

  - 如果此时oldCap>=MAXIMUM_CAPACITY(1 << 30)，表示已经到了最大容量，这时还要往map中put数据，则阈值设置为整数的最大值 Integer.MAX_VALUE，直接返回这个oldTab的内存地址
  - 如果扩容之后的新容量小于最大容量 ，且老的数组容量大于等于默认初始化容量（16），那么新数组的扩容阀值设置为老阀值的2倍（左移1位相当于乘以2，newCap = oldCap << 1），阈值也double（newThr= oldThr << 1）;

  ```java
   // 如果老数组长度大于0，说明已经存在元素
   if (oldCap > 0) {	
        if (oldCap >= MAXIMUM_CAPACITY) { 
               threshold = Integer.MAX_VALUE;	
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                      oldCap >= DEFAULT_INITIAL_CAPACITY)
              newThr = oldThr << 1; // double threshold
      }
  ```

  

- 当老数组没有任何元素，如果老数组的扩容阀值大于0，那么设置新数组的容量为该阀值，`newCap = oldThr`。当`newThr`扩容阀值为0 ，`newThr = (float)newCap * loadFactor`（**这一步也就意味着构造该map的时候，指定了初始化容量构造函数**）;

  ```java
   else if (oldThr > 0) // initial capacity was placed in threshold
   newCap = oldThr;
   ....
   if (newThr == 0) {
       float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);  
    }
  ```

  

- 其他情况**，设置新数组容量 为 16，且设置新数组扩容阀值为 16*0.75 = 12。0.75为负载因子，newCap =16，newThr=12（*****使用默认参数创建的该map，并且第一次添加元素**）

  ```java
   else { // zero initial threshold signifies using defaults
              // 设置新数组容量 为 16
              newCap = DEFAULT_INITIAL_CAPACITY;
               // 设置新数组扩容阀值为 16*0.75 = 12。0.75为负载因子
              newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
         }
  ```

#### 数据迁移

如果oldTab老数组不为空，说明是扩容操作，那么涉及到元素的转移操，遍历老数组，如果当前位置元素不为空，那么需要转移该元素到新数组。

- 如果元素没有有下一个节点，说明该元素不存在hash冲突，因将元素存储到新的数组中，存储到数组的哪个位置需要根据hash值和数组长度来进行取模         

  ```java
   // 如果元素没有有下一个节点，说明该元素不存在hash冲突
    if (e.next == null)   
  		newTab[e.hash & (newCap - 1)] = e;
  ```

- 如果该节点为TreeNode类型，插入红黑树中

  ```java
     // 如果该节点为TreeNode类型
      else if (e instanceof TreeNode)  
             ((TreeNode<K,V>)e).split(this, newTab, j, oldCap); 
  ```

- 遍历链表，并将链表节点按原顺序进行分组

  - 将元素的hash值 和 老数组的长度做与运算`e.hash & oldCap`，判断出是在原位置还是在原位置再移动2次幂的位置（`loTail`低位指的是新数组的 0  到 `oldCap-1 `、`hiTail`高位指定的是`oldCap `到 `newCap - 1`）

    - `(e.hash & oldCap) == 0`原位置，循环到链表尾端，赋值低位的元素loTail

    - `(e.hash & oldCap) != 0` 原位置再移动2次幂的位置，循环到链表尾端，赋值高位的元素hiTail

      

      ```java
       Node<K,V> loHead = null, loTail = null;  // 低位首尾节点
       Node<K,V> hiHead = null, hiTail = null;  // 高位首尾节点
       Node<K,V> next;
       // 遍历链表
       do {  
           next = e.next;                 
           //如果hash值和该原长度做与运算等于0，说明该元素可以放置在低位链表中。
           if ((e.hash & oldCap) == 0) {  
                // 如果没有尾，说明链表为空
                if (loTail == null) 
                        loHead = e; 
                 // 如果有尾，那么链表不为空，把该元素挂到链表的最后。
                 else
                     loTail.next = e; 
               // 把尾节点设置为当前元素
                 loTail = e; 
               }
               // 如果与运算结果不为0，说明hash值大于老数组长度（例如hash值为17）
               // 此时该元素应该放置到新数组的高位位置上
               else {  
                     if (hiTail == null)
                             hiHead = e;
                      else
                          hiTail.next = e;
                       hiTail = e;
                    }
        } while ((e = next) != null);
      ```

  - 将分组后的链表映射到新桶中

    - 低位的元素组成的链表还是放置在原来的位置，
    - 高位的元素组成的链表放置的位置只是在原有位置上偏移了老数组的长度个位置

    ```java
     // 低位的元素组成的链表还是放置在原来的位置
     if (loTail != null) { 
          loTail.next = null;
          newTab[j] = loHead;
     }
     // 高位的元素组成的链表放置的位置只是在原有位置上偏移了老数组的长度个位置。
      if (hiTail != null) {  
           hiTail.next = null;
           newTab[j + oldCap] = hiHead;                  
      }
    ```

JDK1.8对`resize()`扩容方法进行了优化，**经过rehash之后，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。** 

是不是有点不明白呢？那我们来用图来解析一下：

结合`e.hash & oldCapn`取值判断是在高位还是在低位，即如图（a）表示扩容前的key1和key2两种key确定索引位置的示例，![img](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ec1dcbbd06045758efbc469bd59ca0a~tplv-k3u1fbpfcp-zoom-1.image)
图（b）表示扩容后key1和key2两种key确定索引，元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：
![img](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a24f7ce22974235bbe3df1c5578c5b5~tplv-k3u1fbpfcp-zoom-1.image)
因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“`原索引+oldCap`”，可以看看下图为16扩充为32的resize示意图：
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f628559346754a38816b9bc41a7b5c28~tplv-k3u1fbpfcp-zoom-1.image)
在JDK1.7中rehash扩容的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同的链表元素会倒置，但是在JDK1.8进行了优化，从上图可以看出，JDK1.8链表元素不会倒置。因此不会出现链表死循环的问题。

由于篇幅过长，将分成两篇来介绍，接下来内容看 **《面试：为了进阿里，必须掌握HashMap源码原理和面试题（图解版二）》**


