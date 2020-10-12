&emsp;&emsp;在使用MongoDB时，在创建索引会涉及到在复制集(replication)以及分片(Shard)中创建，为了最大限度地减少构建索引的影响,在副本和分片中创建索引，使用滚动索引构建过程。如果不使用滚动索引构建过程：
+ **主服务器上的前台索引构建需要数据库锁定。它复制为副本集辅助节点上的前台索引构建，并且复制工作程序采用全局数据库锁定，该锁定将读取和写入排序到索引服务器上的所有数据库。**
+ **主要的后台索引构建复制为后台索引构建在辅助节点上。复制工作程序不会进行全局数据库锁定，并且辅助读取不会受到影响。**
+ **对于主服务器上的前台和后台索引构建，副本集辅助节点上的索引操作在主节点完成构建索引之后开始。**
+ **在辅助节点上构建索引所需的时间必须在oplog的窗口内，以便辅助节点可以赶上主节点。
那么该如何创建呢？具体步骤呢?请看接下来的具体过程。**



## 1. 在副本集创建索引	
### 准备  
&emsp;&emsp;**必须在索引构建期间停止对集合的所有写入，否则可能会在副本集成员中获得不一致的数据。**
### 具体过程
**在副本集中以滚动方式构建唯一索引包括以下过程：**

1. **停止一个Secondary节点（从节点）并以单机模式重新启动，可以使用配置文件更新配置以单机模式重新启动：**
+ **注释掉replication.replSetName选项。**
+ **将net.port更改为其他端口。将原始端口设置注释掉。**
+ **在setParameter部分中将参数disableLogicalSessionCacheRefresh设置为true。**
	
**例如：**

        //修改配置
        net:
        bindIp: localhost,<hostname(s)|ip address(es)>
        port: 27217
        #port: 27017
        #replication:
        #replSetName: myRepl
        setParameter:
        disableLogicalSessionCacheRefresh: true
        //重新启动
        mongod --config <path/To/ConfigFile>
2. **创建索引：在单机模式下进行索引创建**
3. **重新开启Replica Set 模式：索引构建完成后，关闭mongod实例。撤消作为独立启动时所做的配置更改，以返回其原始配置并作为副本集的成员重新启动。**

        //回退原来的配置：net:
        bindIp: localhost,<hostname(s)|ip address(es)>
        port: 27017
        replication:
        replSetName: myRepl
        //重新启动：
        mongod --config <path/To/ConfigFile>
    
4. **在其他从节点中重复1、2、3步骤的过程操作。**
5. **主节点创建索引，当所有从节点都有新索引时，降低主节点，使用上述过程作为单机模式重新启动它，并在原主节点上构建索引：**
	+ **使用mongo shell中的rs.stepDown（）方法来降低主节点为从节点，** 
	+ **成功降级后，当前主节点成为从节点，副本集成员选择新主节点，并进行从节点创建方式进行创建索引。**
	
## 2. 分片集群创建唯一索引
### 准备
&emsp;&emsp;**创建唯一索引，必须在索引构建期间停止对集合的所有写入。 否则，您可能会在副本集成员中获得不一致的数据。如果无法停止对集合的所有写入，请不要使用以下过程来创建唯一索引。**
### 具体过程 

1. **停止Balancer：将mongo shell连接到分片群集中的mongos实例，然后运行sh.stopBalancer（）以禁用Balancer。如果正在进行迁移，系统将在停止平衡器之前完成正在进行的迁移。**
2. **确定Collection的分布：刷新该mongos的缓存路由表，以避免返回该Collection旧的分发信息。刷新后，对要构建索引的集合运行db.collection.getShardDistribution()。**

例如：在test数据库中的records字段中创建上升排序的索引

        db.adminCommand( { flushRouterConfig: "test.records" } );
        db.records.getShardDistribution();
	
 例如，考虑一个带有3个分片shardA，shardB和shardC的分片集群，db.collection.getShardDistribution（）返回以下内容 
 
	   Shard shardA at shardA/s1-mongo1.example.net:27018,s1-mongo2.example.net:27018,s1-mongo3.example.net:27018 
       data : 1KiB docs : 50 chunks : 1
	   estimated data per chunk : 1KiB 
	   estimated docs per chunk : 50 Shard shardC at shardC/s3-mongo1.example.net:27018,s3-mongo2.example.net:27018,s3-mongo3.example.net:27018 
	   data : 1KiB docs : 50 chunks : 1 
       estimated data per chunk : 1KiB 
	   estimated docs per chunk : 50 
	   Totals data : 3KiB docs : 100 chunks : 2 
	   Shard shardA contains 50% data, 50% docs in cluster, avg obj size on shard : 40B 
	   Shard shardC contains 50% data, 50% docs in cluster, avg obj size on shard : 40B
	  从输出中，您只在shardA和shardC上为test.records构建索引。

3. **在包含集合Chunks的分片创建索引**
	+ **C1.停止从节点，并以单机模式重新启动：对于受影响的分片，停止从节点与其中一个分区相关联的mongod进程，进行配置文件/命令模式更新后重新启动。**
	
    **配置文件：** 
	+ **将net.port更改为其他端口。 注释到原始端口设置。**  
	+ **注释掉replication.replSetName选项。**  
	+ **注释掉sharding.clusterRole选项。**  
	+ **在setParameter部分中将参数skipShardingConfigurationChecks设置为true。**  
	+ **在setParameter部分中将参数disableLogicalSessionCacheRefresh设置为true。**
		
    		net:
    		bindIp: localhost,<hostname(s)|ip address(es)>
    		port: 27218
    		#   port: 27018
    		#replication:
    		#   replSetName: shardA
    		#sharding:
    		#   clusterRole: shardsvr
    	    setParameter:
    		skipShardingConfigurationChecks: true
    		disableLogicalSessionCacheRefresh: true
    		//重启：
            mongod --config <path/To/ConfigFile>
	+ **C2.创建索引：直接连接到在新端口上作为独立运行的mongod实例，并为此实例创建新索引。**

    		//例如：在record Collection的username创建索引
    		db.records.createIndex( { username: 1 } )

	+ **C3.恢复C1的配置，并作为 Replica Set成员启动：索引构建完成后，关闭mongod实例。 撤消作为单机模式时所做的配置更改，以返回其原始配置并重新启动。**
	
	**配置文件模式：**
	 + **恢复为原始端口号。**
	+ **取消注释replication.replSetName。**
	+ **取消注释sharding.clusterRole。**
	+ **删除setParameter部分中的参数skipShardingConfigurationChecks。**
	+ **在setParameter部分中删除参数disableLogicalSessionCacheRefresh。**
	
    		net:
    		   bindIp: localhost,<hostname(s)|ip address(es)>
    		   port: 27018
    		replication:
    		   replSetName: shardA
    		sharding:
    		   clusterRole: shardsvr
    		重启：mongod --config <path/To/ConfigFile>
	+ **C4.其他从节点分片重复C1、C2、C3过程创建索引。**
	+ **C5.主节点创建索引：当所有从节点都有新索引时，降低主节点，使用上述过程作为单机模式重新启动它，并在原主节点上构建索引
		使用mongo shell中的rs.stepDown（）方法来降低主节点为从节点，成功降级后，当前主节点成为从节点，副本集成员选择新主节点，并进行从节点创建方式进行创建索引。**
4. **在其他受影响的分片重复C步骤;**
5. **重启Balancer，一旦全部分片创建完索引，重启Balancer：sh.startBalancer()。**

## 总结
&emsp;&emsp;**后续还有关于实践中复制集以及分片的搭建过程，复制集成员节点增加删除等一系列实战操作。**

<br>

 **最后可关注公众号，一起学习,每天会分享干货，还有学习视频领取！**

![](https://user-gold-cdn.xitu.io/2020/4/14/171792690b19e0b8?w=350&h=129&f=png&s=17519)