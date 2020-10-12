&emsp;&emsp;在面试中遇到美女面试官时，我们以为面试会比较容易过，也能好好表现自己技术的时候了。然而却出现以下这一幕，当美女面试官听说你使用过Redis时，那么问题来了。

**👩面试官**：**Q1，你知道Redis设置key过期时间的命令吗？**   

**👧你**：你毫不犹豫的巴拉巴拉说了一堆命令，以及用法，比如expire <key> <ttl>等等命令

（🎈这时候你想问得那么简单？但真的那么简单吗？美女面试官停顿了一下，接着问）

**👩面试官**：**Q2，那你说说Redis是怎么实现过期时间设置呢？以及怎么判断键过期的呢？**  
**👧你**：（这时候想这还难不倒我），然后又巴拉巴拉的说一通，Redis的数据库服务器中redisDb数据结构以及过期时间的判定

（🎈你又在想应该不会问了吧，换个Redis的话题了吧，那你就错了）

**👩面试官**:**（抬头笑着看了看你）Q3，那你说说过期键的删除策略以及Redis过期键的删除策略以及实现？**   
**🤦‍️你**：这时你回答的就不那么流畅了，有时头脑还阻塞了。

（🎈这是你可能就有点蒙了，或者只知道一些过期键的删除策略，但具体怎么实现不知道呀，你以为面试官的提问这样就完了吗？）

**👩面试官**：**Q4,那你再说说其他环节中是怎么处理过期键的呢（比如AOF、RDB）？**  
**🤦🤦你**：...........

（🎈这更加尴尬了，知道的不全，也可能不知道，本来想好好表现，也想着面试比较简单，没想到会经历这些）

**为了避免这尴尬的场景出现，那现在需要你记录下以下的内容，这样就可以在美女面试官面前好好表现了。**

## 1. Redis Expire Key基础
redis数据库在数据库服务器中使用了`redisDb`数据结构，结构如下：

    typedef struct redisDb {
         dict *dict;     /* 键空间 key space */
         dict *expires;    /* 过期字典 */
         dict *blocking_keys;  /* Keys with clients waiting for data (BLPOP) */
         dict *ready_keys;   /* Blocked keys that received a PUSH */
         dict *watched_keys;   /* WATCHED keys for MULTI/EXEC CAS */
         struct evictionPoolEntry *eviction_pool; /* Eviction pool of keys */
         int id;      /* Database ID */
         long long avg_ttl;   /* Average TTL, just for stats */
    } redisDb;
其中，
+ **键空间(`key space`)：dict字典用来保存数据库中的所有键值对**
+ **过期字典(`expires`):保存数据库中所有键的过期时间，过期时间用`UNIX`时间戳表示，且值为`long long`整数** 




### 1.1 设置过期时间命令
+ **`EXPIRE \<key> \<ttl>`**：命令用于将键key的过期时间设置为ttl秒之后
+ **`PEXPIRE \<key> \<ttl>`**：命令用于将键key的过期时间设置为ttl毫秒之后
+ **`EXPIREAT \<key> \<timesramp>`**：命令用于将key的过期时间设置为timrestamp所指定的秒数时间戳
+ **`PEXPIREAT \<key> \<timesramp>`**：命令用于将key的过期时间设置为timrestamp所指定的毫秒数时间戳

**设置过期时间：**  

    redis> set Ccww   5 2 0  
    ok  
    redis> expire Ccww 5  
    ok  
**使用redisDb结构存储数据图表示：**   
![](https://user-gold-cdn.xitu.io/2019/10/17/16dd8410e06b4d43?w=1052&h=548&f=png&s=66186)
### 1.2过期时间保存以及判定
过期键的判定，其实通过**过期字典**进行判定，步骤：
1. 检查给定键是否存在于过期字典，如果存在，取出键的过期时间
2. 通过判断当前UNIX时间戳是否大于键的过期时间，是的话，键已过期，相反则键未过期。
   

    




## 2. 过期键删除策略
### 2.1 三种不同删除策略
1. **定时删除**：在设置键的过期时间的同时，创建一个定时任务，当键达到过期时间时，立即执行对键的删除操作
2. **惰性删除**：放任键过期不管，但在每次从键空间获取键时，都检查取得的键是否过期，如果过期的话，就删除该键，如果没有过期，就返回该键
3. **定期删除**：每隔一点时间，程序就对数据库进行一次检查，删除里面的过期键，至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。

### 2.2 三种删除策略的优缺点
#### 2.2.1 定时删除
+ **优点：** 对内存友好，定时删除策略可以保证过期键会尽可能快地被删除，并释放国期间所占用的内存
+ **缺点：** 对cpu时间不友好，在过期键比较多时，删除任务会占用很大一部分cpu时间，在内存不紧张但cpu时间紧张的情况下，将cpu时间用在删除和当前任务无关的过期键上，影响服务器的响应时间和吞吐量

#### 2.2.2 惰性删除
+ **优点：** 对cpu时间友好，在每次从键空间获取键时进行过期键检查并是否删除，删除目标也仅限当前处理的键，这个策略不会在其他无关的删除任务上花费任何cpu时间。
+ **缺点：** 对内存不友好，过期键过期也可能不会被删除，导致所占的内存也不会释放。甚至可能会出现内存泄露的现象，当存在很多过期键，而这些过期键又没有被访问到，这会可能导致它们会一直保存在内存中，造成内存泄露。

#### 2.2.4 定期删除
&emsp;&emsp;由于定时删除会占用太多cpu时间，影响服务器的响应时间和吞吐量以及惰性删除浪费太多内存，有内存泄露的危险，所以出现一种整合和折中这两种策略的定期删除策略。
1. 定期删除策略每隔一段时间执行一次删除过期键操作，并通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。
2. 定时删除策略有效地减少了因为过期键带来的内存浪费。


**定时删除策略难点就是确定删除操作执行的时长和频率：**  

&emsp;&emsp;删除操作执行得太频繁。或者执行时间太长，定期删除策略就会退化成为定时删除策略，以至于将cpu时间过多地消耗在删除过期键上。相反，则惰性删除策略一样，出现浪费内存的情况。
所以使用定期删除策略，需要根据服务器的情况合理地设置删除操作的执行时长和执行频率。

## 3. Redis的过期键删除策略
&emsp;&emsp;Redis服务器结合惰性删除和定期删除两种策略一起使用，通过这两种策略之间的配合使用，使得服务器可以在合理使用CPU时间和浪费内存空间取得平衡点。

### 3.1 惰性删除策略的实现
&emsp;&emsp;Redis在执行任何读写命令时都会先找到这个key，惰性删除就作为一个切入点放在查找key之前，如果key过期了就删除这个key。

![](https://user-gold-cdn.xitu.io/2019/10/16/16dd4ce4716ff077?w=321&h=280&f=png&s=36691)

    robj *lookupKeyRead(redisDb *db, robj *key) {
              robj *val;
    	 expireIfNeeded(db,key); // 切入点
    	 val = lookupKey(db,key);
    	 if (val == NULL)
    	  server.stat_keyspace_misses++;
    	 else
    	  server.stat_keyspace_hits++;
    	 return val;
    }
    
**通过`expireIfNeeded`函数对输入键进行检查是否删除**

    int expireIfNeeded(redisDb *db, robj *key) {
         /* 取出键的过期时间 */
        mstime_t when = getExpire(db,key);
        mstime_t now;
        
         /* 没有过期时间返回0*/
        if (when < 0) return 0; /* No expire for this key */
    
        /* 服务器loading时*/
        if (server.loading) return 0;
    
        /* 根据一定规则获取当前时间*/
        now = server.lua_caller ? server.lua_time_start : mstime();
        /* 如果当前的是从(Slave)服务器
         * 0 认为key为无效
         * 1 if we think the key is expired at this time. 
         * */
        if (server.masterhost != NULL) return now > when;
    
         /* key未过期，返回 0 */
        if (now <= when) return 0;
    
         /* 删除键 */
        server.stat_expiredkeys++;
        propagateExpire(db,key,server.lazyfree_lazy_expire);
        notifyKeyspaceEvent(NOTIFY_EXPIRED,
            "expired",key,db->id);
        return server.lazyfree_lazy_expire ? dbAsyncDelete(db,key) :
                                             dbSyncDelete(db,key);
    }

### 3.2 定期删除策略的实现
&emsp;&emsp;key的定期删除会在Redis的周期性执行任务（`serverCron`，默认每100ms执行一次）中进行，而且是发生Redis的`master`节点，因为`slave`节点会通过主节点的DEL命令同步过来达到删除key的目的。

    for (j = 0; j < dbs_per_call; j++) {
     int expired;
     redisDb *db = server.db+(current_db % server.dbnum);
     
     current_db++;
     
     /* 超过25％的key已过期，则继续. */
     do {
      unsigned long num, slots;
      long long now, ttl_sum;
      int ttl_samples;
     
      /* 如果该db没有设置过期key，则继续看下个db*/
      if ((num = dictSize(db->expires)) == 0) {
       db->avg_ttl = 0;
       break;
      }
      slots = dictSlots(db->expires);
      now = mstime();
     
      /*但少于1%时，需要调整字典大小*/
      if (num && slots > DICT_HT_INITIAL_SIZE &&
       (num*100/slots < 1)) break;
     
      expired = 0;
      ttl_sum = 0;
      ttl_samples = 0;
     
      if (num > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP)
       num = ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP;// 20
     
      while (num--) {
       dictEntry *de;
       long long ttl;
     
       if ((de = dictGetRandomKey(db->expires)) == NULL) break;
       ttl = dictGetSignedIntegerVal(de)-now;
       if (activeExpireCycleTryExpire(db,de,now)) expired++;
       if (ttl > 0) {
        /* We want the average TTL of keys yet not expired. */
        ttl_sum += ttl;
        ttl_samples++;
       }
      }
     
      /* Update the average TTL stats for this database. */
      if (ttl_samples) {
       long long avg_ttl = ttl_sum/ttl_samples;
     
       /样本获取移动平均值 */
       if (db->avg_ttl == 0) db->avg_ttl = avg_ttl;
       db->avg_ttl = (db->avg_ttl/50)*49 + (avg_ttl/50);
      }
      iteration++;
      if ((iteration & 0xf) == 0) { /* 每迭代16次检查一次 */
       long long elapsed = ustime()-start;
     
       latencyAddSampleIfNeeded("expire-cycle",elapsed/1000);
       if (elapsed > timelimit) timelimit_exit = 1;
      }
     /* 超过时间限制则退出*/
      if (timelimit_exit) return;
      /* 在当前db中，如果少于25%的key过期，则停止继续删除过期key */
     } while (expired > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP/4);
    }
&emsp;&emsp;依次遍历每个db（默认配置数是16），针对每个db，每次循环随机选择20个（`ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP`）key判断是否过期，如果一轮所选的key少于25%过期，则终止迭次，此外在迭代过程中如果超过了一定的时间限制则终止过期删除这一过程。

## 4. AOF、RDB和复制功能对过期键的处理
### 4.1 RDB
**生成RDB文件**  
&emsp;程序会数据库中的键进行检查，已过期的键不会保存到新创建的RDB文件中  

**载入RDB文件**
1. 主服务载入RDB文件，会对文件中保存的键进行检查会忽略过期键加载未过期键
2. 从服务器载入RDB文件，会加载文件所保存的所有键（过期和未过期的），但从主服务器同步数据同时会清空从服务器的数据库。

### 4.2 AOF
+ AOF文件写入：当过期键被删除后，会在AOF文件增加一条DEL命令，来显式地记录该键已被删除。
+ AOF重写：已过期的键不会保存到重写的AOF文件中

### 4.3 复制
&emsp;当服务器运行在复制模式下时，从服务器的过期键删除动作由主服务器控制的，这样的好处主要为了保持主从服务器数据一致性：
1. 主服务器在删除一个过期键之后，会显式地向所有的从服务器发送一个DEL命令，告知从服务器删除这个过期键
2. 从服务器在执行客户端发送的读取命令时，即使碰到过期键也不会将过期键删除，不作任何处理。
3. 只有接收到主服务器 DEL命令后，从服务器进行删除处理。


> 各位看官还可以吗？喜欢的话，动动手指点个💗，点个关注呗！！谢谢支持！
>
> 欢迎关注公众号【**Ccww技术博客**】，原创技术文章第一时间推出



![](https://user-gold-cdn.xitu.io/2020/4/14/171792690b19e0b8?w=350&h=129&f=png&s=17519)
