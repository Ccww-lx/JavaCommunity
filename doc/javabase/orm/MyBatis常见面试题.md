## 【面试官之你说我听】-MyBatis常见面试题

### 精讲#{}和${}的区别是什么？

+ mybatis在处理\#{}时，会将sql中的\#{}替换为?号，调用PreparedStatement的set方法来赋值。

+ mybatis在处理\${}时，就是把\${}替换成变量的值。

+ 使用#{}可以有效的防止SQL注入，提高系统安全性。原因在于：预编译机制。**预编译完成之后，SQL的结构已经固定，即便用户输入非法参数，也不会对SQL的结构产生影响，从而避免了潜在的安全风险。**

+ 预编译是提前对SQL语句进行预编译，而其后注入的参数将不会再进行SQL编译。我们知道，SQL注入是发生在编译的过程中，因为恶意注入了某些特殊字符，最后被编译成了恶意的执行操作。而预编译机制则可以很好的防止SQL注入。

> 既然\${}会引起sql注入，为什么有了#{}还需要有${}呢？那其存在的意义是什么？ 
>
>  \#{}主要用于预编译，而预编译的场景其实非常受限，而${}用于替换，很多场景会出现替换，而这种场景可不是预编译 



### 数据库链接中断如何处理

 数据库的访问底层是通过tcp实现的，当链接中断是程序是无法得知，导致程序一直会停顿一段时间在这，最终会导致用户体验不好，因此面对数据库连接中断的异常，该怎么设置mybatis呢？ 

connection操作底层是一个循环处理操作，因此可以进行时间有关的参数：

+  max_idle_time ： 表明最大的空闲时间，超过这个时间socket就会关闭 
+  connect_timeout ： 表明链接的超时时间 

> 数据库服务器活的杠杠的，但是因为网络用塞，客户端仍然连不上服务器端，这个时候就要设置timeout，别一直傻等着 



### 在开发过程中，经常遇到插入重复的现象，这种情况该如何解决呢？ 

> 插入的过程一般都是分两步的：先判断是否存在记录，没有存在则插入否则不插入。如果存在并发操作，那么同时进行了第一步，然后大家都发现没有记录，然后都插入了数据从而造成数据的重复 

 解决插入重复的思路 :

+  先判断数据库是否存在数据，有的话则不进行任何操作。没有数据的话，进行下一步；
+   向redis set key，其中只有一个插入操作A会成功，其他并发的操作（B和C...）都会失败的 ；
+  当set key 成功的操作A，开始执行插入数据操作，无论是否插入数据成功，都在需要将redis key删除。【注】插入不成功可以多尝试几次，增加成功的概率 ；
+  然而set key 失败的操作B和C，sleep一下，竞争赢的插入操作重复以上步骤。

 总结：多线程同时插入数据，谁获取锁并插入数据成功了其他线程不做任何操作。当插入数据失败后，其他线程抢锁进行插入数据。





### 事务执行过程中宕机的应对处理方式

> 数据库插入百万级数据的时候，还没操作完，但是把服务器重启了，数据库会继续执行吗？ 还是直接回滚了？ 

 不会自动继续执行，不会自动直接回滚 ,但可以依据事务日志进行回滚或者进行执行。

事务开启时，事务中的操作，都会先写入存储引擎的日志缓冲中，在事务提交之前，这些缓冲的日志都需要提前刷新到磁盘上持久化 ，两种类型：

> 在事务执行的过程中，除了记录redo log，还会记录一定量的undo log。 

+  redo log  ：按语句的执行顺序，依次交替的记录在一起 
+  undo log： 主要为事务的回滚服务。undo log记录了数据在每个操作前的状态，如果事务执行过程中需要回滚，就可以根据undo log进行回滚操作。  



### Java客户端中的一个Connection是不是在MySQL中就对应一个线程来处理这个链接呢？ 

 Java客户端中的一个Connection不是在MySQL中就对应一个线程来处理这个链接，而是：

> **监听socket的主线程+线程池里面固定数目的工作线程来处理的** 

 高性能服务器端端开发底层主要靠I/O复用来处理，这种模式：

> **单线程+事件处理机制**

在MySQL有一个主线程，这是单线程（与Java中处处强调多线程的思想有点不同哦），它不断的循环查看是否有socket是否有读写事件，如果有读写事件，再从线程池里面找个工作线程处理这个socket的读写事件，完事之后工作线程会回到线程池。



### Mybatis中的Dao接口和XML文件里的SQL是如何建立关系的？

+ 解析XML： 初始化SqlSessionFactoryBean会将mapperLocations路径下所有的XML文件进行解析
  + 创建SqlSource： Mybatis会把每个SQL标签封装成SqlSource对象，可以为动态SQL和静态SQL 
  + 创建MappedStatement： XML文件中的每一个SQL标签就对应一个MappedStatement对象 ，并由 Configuration解析XML 
+ Dao接口代理：  Spring中的FactoryBean 和 JDK动态代理返回了可以注入的一个Dao接口的代理对象 
+ 执行： 通过statement全限定类型+方法名拿到MappedStatement 对象，然后通过执行器Executor去执行具体SQL并返回 



### 当实体类中的属性名和表中的字段名不一样，怎么办 ？

+  通过在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致
+  通过\<resultMap>来映射字段名和实体类属性名的一一对应的关系。 



### 模糊查询like语句该怎么写?

+ 在Java代码中添加sql通配符 

```
string name = "%Ccww%"; 
list<name> names = mapper.selectName(name);
```

```
<select id="selectName"> 
	select * from users where name like #{value} 
</select>
```

+ 在sql语句中拼接通配符，会引起sql注入 

```
<select id="selectName">
    select * from users where name like "%"#{value}"%"
</select>
```



### 什么是MyBatis的接口绑定？有哪些实现方式？

 接口绑定 : 在MyBatis中任意定义接口，然后把接口里边的方法和SQL语句绑定，我们可以直接调用接口方法，比起SqlSession提供的方法我们可以有更加灵活的选择和设置 

 接口绑定有两种实现方式 ：

+   通过注解绑定： 在接口的方法上加上 @Select、@Update等注解，里面包含Sql语句来绑定； 
+  通过xml绑定 ： 要指定xml映射文件里面的namespace必须为接口的全路径名 。



### 使用MyBatis的mapper接口调用时要注意的事项

+  Mapper接口方法名和mapper.xml中定义的每个sql的id相同； 
+  Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同； 
+  Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同； 
+  Mapper.xml文件中的namespace即是mapper接口的类路径。 



### 通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？ 

+ Dao接口为Mapper接口。

+ 接口的全限名为映射文件中的namespace的值；

+ 接口的方法名为映射文件中Mapper的Statement的id值；

+ 接口方法内的参数为传递给sql的参数。

Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MapperStatement。在Mybatis中，每一个 \<select>、\<insert>、\<update>、\<delete>标签，都会被解析为一个MapperStatement对象

 Mapper接口里的方法，是不能重载的，因为是使用 全限名+方法名 的保存和寻找策略。Mapper 接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Mapper接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement所代表的sql，然后将sql执行结果返回。 



### Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？

**基于上面，可以得知**

> **Statement=namespace+id**

 如果配置了namespace可以重复的 ,但如果没有配置namespace的话，那么相同的id就会导致覆盖了。



### Mybatis的一级、二级缓存的作用是什么？

（1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存。

（2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态)，可在它的映射文件中配置 <cache /> ；

（3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。



### Mybatis 是如何进行分页的？分页插件的原理是什么？

Mybatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的**内存分页**，而非**数据库分页**。

在实际场景下，使用如下两种方案：

- 在 SQL 内直接书写带有数据库分页的参数来完成数据库分页功能
- 也可以使用分页插件来完成数据库分页。

这两者都是基于数据库分页，差别在于前者是工程师**手动**编写分页条件，后者是插件**自动**添加分页条件。

------

分页插件的基本原理是使用 Mybatis 提供的插件接口，实现自定义分页插件。在插件的拦截方法内，拦截待执行的 SQL ，然后重写 SQL ，根据dialect 方言，添加对应的物理分页语句和物理分页参数。

举例：`SELECT * FROM student` ，拦截 SQL 后重写为：`select * FROM student LIMI 0，10` 。

目前市面上目前使用比较广泛的 MyBatis 分页插件有：

- [Mybatis-PageHelper](https://github.com/pagehelper/Mybatis-PageHelper)
- [MyBatis-Plus](https://github.com/baomidou/mybatis-plus)

### Mybatis 动态 SQL 是做什么的？都有哪些动态 SQL ？能简述一下动态 SQL 的执行原理吗？

- Mybatis 动态 SQL ，可以让我们在 XML 映射文件内，以 XML 标签的形式编写动态 SQL ，完成逻辑判断和动态拼接 SQL 的功能。
- Mybatis 提供了 9 种动态 SQL 标签：  
  - \<if>
  - \<choose>
  - \<when>
  - \<otherwise>
  - \<trim>
  - \<where>、
  - \<set>
  - \<foreach>
  - \<bind>    
- 其执行原理为，使用 **OGNL** 的表达式，从 SQL 参数对象中计算表达式的值，根据表达式的值动态拼接 SQL ，以此来完成动态 SQL 的功能

### Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？

Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。

在Mybatis配置文件中，可以配置是否启用延迟加载:

> lazyLoadingEnabled=true|false。

原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法.

> 比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。

### Mybatis都有哪些Executor执行器？它们之间的区别是什么？

Mybatis有三种基本的Executor执行器，**SimpleExecutor、ReuseExecutor、BatchExecutor。**

+ **SimpleExecutor：**每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。

+ **ReuseExecutor：**执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。

+ **BatchExecutor：**执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内

 在Mybatis配置文件中，可以指定默认的ExecutorType执行器类型，也可以手动给DefaultSqlSessionFactory的创建SqlSession的方法传递ExecutorType类型参数。 

### MyBatis与Hibernate区别

+  hibernate是全自动，而mybatis是半自动 
+  hibernate数据库移植性远大于mybatis 
+  hibernate拥有完整的日志系统，mybatis则欠缺一些 
+  mybatis相比hibernate需要关心很多细节 
+  sql直接优化上，mybatis要比hibernate方便很多 
+  缓存机制上，hibernate要比mybatis更好一些 

**总结:**

- Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取。
- Mybatis 属于半自动 ORM 映射工具，在查询关联对象或关联集合对象时，需要手动编写 SQL 来完成。

参考文章:

 http://www.mybatis.cn/category/interview/ 

 https://www.cnblogs.com/huajiezh/p/6415388.html 

 http://svip.iocoder.cn/MyBatis/Interview/ 


