
&emsp;&emsp;在Redis中会涉及很多数据结构，比如SDS，双向链表、字典、压缩列表、整数集合等等。Redis会基于这些数据结构自定义一个对象系统，而且自定义的对象系统有很多好处。

通过对以下的Redis对象系统的学习，可以了解Redis设计原理以及初衷，为了我们在使用Redis的时候，更加能够理解到其原理和定位问题。

## Redis 对象
Redis基于上述的数据结构自定义一个Object 系统，Object结构：

	redisObject结构：
		 typedef struct redisObject{
		//类型
		unsigned type:4;
		//编码
		unsigned encoding:4;
		//指向底层实现数据结构的指针
		void *ptr;
		….. 
	} 


Object 系统包含五种Object：

+ String：字符串对象
+ List：列表对象
+ Hash：哈希对象
+ Set：集合对象
+ ZSet：有序集合  

Redis使用对象来表示数据库中的键和值，即每新建一个键值对，至少创建有两个对象，而且使用对象的具有以下好处：
	1. redis可以在执行命令前会根据对象的类型判断一个对象是否可以执行给定的命令
	2. 针对不同的使用场景，为对象设置不同的数据结构实现，从而优化对象的不同场景夏的使用效率
	3. 对象系统还可以基于引用计数计数的内存回收机制，自动释放对象所占用的内存，或者还可以让多个数据库键共享同一个对象来节约内存。
	4. redis对象带有访问时间记录信息，使用该信息可以进行优化空转时长较大的key，进行删除！


<br>
对象的ptr指针指向对象的底层现实数据结构，而这些数据结构由对象的encoding属性决定，对应关系：

|编码常量|编码对应的底层数据结构|
|-------|---------|
|REDIS_ENCODING_INT|long类型的整数|
|REDIS_ENCODING_EMBSTR|embstr编码的简单动态字符串|
|REDIS_ENCODING_RAW|简单动态字符串|
|REDIS_ENCODING_HT|字典|
|REDIS_ENCODING_LINKEDLIST|双向链表|
|REDIS_ENCODING_ZIPLIST|压缩列表|
|REDIS_ENCODING_INTSET|整数集合|
|REDIS_ENCODING_SKIPLIST|跳跃表和字典|

<br>
每种Object对象至少有两种不同的编码，对应关系：

|类型|	编码|	对象|
|-------|---------|--------|
|String|	int|	整数值实现|
|String|	embstr|	sds实现 <=39 字节|
|String	|raw|	sds实现 > 39字节|
|List|	ziplist	|压缩列表实现|
|List|	linkedlist|	双端链表实现|
|Set|	intset	|整数集合使用|
|Set|	hashtable	|字典实现|
|Hash|	ziplist	|压缩列表实现|
|Hash	|hashtable|	字典使用|
|Sorted set|	ziplist	|压缩列表实现|
|Sorted set	|skiplist|	跳跃表和字典|



<br>
## String 对象

字符串对象编码可以int 、raw或者embstr，如果保存的值为整数值且这个值可以用long类型表示，使用int编码，其他编码类似。

比如：int编码的String Object

	redis> set number 520 
	 ok
	 redis> OBJECT ENCODING number 
	"int"
String Object结构：

![file](https://user-gold-cdn.xitu.io/2019/10/22/16df4123cfed8227?w=441&h=344&f=png&s=11866)


 ### String 对象之间的编码转换
int编码的字符串对象和embstr编码的字符串对象在条件满足的情况下，会被转换为raw编码的字符串对象。   

比如：对int编码的字符串对象进行append命令时，就会使得原来是int变为raw编码字符串


<br>
## List对象

list对象可以为ziplist或者为linkedlist，对应底层实现ziplist为压缩列表，linkedlist为双向列表。

	Redis>RPUSH numbers “Ccww” 520 1
	
用ziplist编码的List对象结构：  
![file](https://user-gold-cdn.xitu.io/2019/10/22/16df4123fa901f09?w=768&h=337&f=png&s=16278)
	
用linkedlist编码的List对象结构：  

![file](https://user-gold-cdn.xitu.io/2019/10/22/16df41241bef9e8e?w=714&h=320&f=png&s=18026)


### List对象的编码转换：
当list对象可以同时满足以下两个条件时，list对象使用的是ziplist编码：
	1. list对象保存的所有字符串元素的长度都小于64字节
	2. list对象保存的元素数量小于512个，
不能满足这两个条件的list对象需要使用linkedlist编码。




## Hash对象
Hash对象的编码可以是ziplist或者hashtable
其中，ziplist底层使用压缩列表实现：
+ 保存同一键值对的两个节点紧靠相邻，键key在前，值vaule在后
+ 先保存的键值对在压缩列表的表头方向，后来在表尾方向

hashtable底层使用字典实现，Hash对象种的每个键值对都使用一个字典键值对保存：
+ 字典的键为字符串对象，保存键key
+ 字典的值也为字符串对象，保存键值对的值

比如：HSET命令

	redis>HSET author name  "Ccww"
	(integer)
	
	redis>HSET author age  18
	(integer)
	
	redis>HSET author sex  "male"
	(integer)
ziplist的底层结构：

![file](https://user-gold-cdn.xitu.io/2019/10/22/16df412442de8b11?w=800&h=421&f=jpeg&s=21580)


hashtable底层结构：  

![file](https://user-gold-cdn.xitu.io/2019/10/22/16df41247769f041?w=646&h=476&f=png&s=27910)


### Hash对象的编码转换：
当list对象可以同时满足以下两个条件时，list对象使用的是ziplist编码：
	1. list对象保存的所有字符串元素的长度都小于64字节
	2. list对象保存的元素数量小于512个，
不能满足这两个条件的hash对象需要使用hashtable编码

**Note**：这两个条件的上限值是可以修改的，可查看配置文件hash-max-zaiplist-value和hash-max-ziplist-entries




<br>
## Set对象：
Set对象的编码可以为intset或者hashtable
+ intset编码：使用整数集合作为底层实现，set对象包含的所有元素都被保存在intset整数集合里面
+ hashtable编码：使用字典作为底层实现，字典键key包含一个set元素，而字典的值则都为null

inset编码Set对象结构：

		redis> SAD number  1 3 5 
		
![file](https://user-gold-cdn.xitu.io/2019/10/22/16df41249d4f9132?w=800&h=222&f=jpeg&s=19834)

hashtable编码Set对象结构：

	redis> SAD Dfruits  “apple”  "banana" " cherry"
	
![file](https://user-gold-cdn.xitu.io/2019/10/22/16df4124d15a9517?w=631&h=258&f=png&s=72867)
	

### Set对象的编码转换：
使用intset编码：
	1. set对象保存的所有元素都是整数值
	2. set对象保存的元素数量不超过512个
不能满足这两个条件的Set对象使用hashtable编码






## ZSet对象
ZSet对象的编码 可以为ziplist或者skiplist
ziplist编码，每个集合元素使用相邻的两个压缩列表节点保存，一个保存元素成员，一个保存元素的分值，然后根据分数进行从小到大排序。

ziplist编码的ZSet对象结构：

	Redis>ZADD price 8.5 apple 5.0 banana 6.0 cherry

![file](https://user-gold-cdn.xitu.io/2019/10/22/16df4124fd22fa39?w=797&h=395&f=png&s=21828)

skiplist编码的ZSet对象使用了zset结构，包含一个字典和一个跳跃表

	Type struct zset{
	
		Zskiplist *zsl；
		dict *dict；
		...
	}
	
	skiplist编码的ZSet对象结构
![file](https://user-gold-cdn.xitu.io/2019/10/22/16df41251f85b784?w=800&h=411&f=jpeg&s=39372)



### ZSet对象的编码转换

当ZSet对象同时满足以下两个条件时，对象使用ziplist编码
	1. 有序集合保存的元素数量小于128个
	2. 有序集合保存的所有元素的长度都小于64字节
不能满足以上两个条件的有序集合对象将使用skiplist编码。  

**Note：** 可以通过配置文件中zset-max-ziplist-entries和zset-max-ziplist-vaule

> 各位看官还可以吗？喜欢的话，动动手指点个💗，点个关注呗！！谢谢支持！
>
> 欢迎关注公众号【**Ccww技术博客**】，原创技术文章第一时间推出



![](https://user-gold-cdn.xitu.io/2020/4/14/171792690b19e0b8?w=350&h=129&f=png&s=17519)
