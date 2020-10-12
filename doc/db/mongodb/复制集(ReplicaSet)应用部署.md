&emsp;&emsp;通过在不同的计算机上托管mongod实例来尽可能多地保持成员之间的分离。将虚拟机用于生产部署时，应将每个mongod实例放置在由冗余电源电路和冗余网络路径提供服务的单独主机服务器上，而且尽可能的将副本集的每个成员部署到自己的计算机绑定到标准的MongoDB端口27017。  
&emsp;&emsp;其中三个成员节点的副本集提供足够的冗余以承受大多数网络分区和其他系统故障。这些collection集合还具有足够的容量用于许多分布式读取操作。副本集应始终具有奇数个成员。这确保选举顺利进行。  

部署时考虑的问题：
+ **建立虚拟专用网络。确保您的网络拓扑通过局域网路由单个站点内的成员节点之间的所有流量。**
+ **配置访问控制以防止从未知客户端到副本集的连接。**
+ **配置网络和防火墙规则，以便仅在默认的MongoDB端口上允许传入和传出的数据包，并且仅在部署中允许。**
+ **确保可通过可解析的DNS或主机名访问副本集的每个成员节点。您应该正确配置DNS名称，或者设置系统的/etc/hosts文件以反映此配置。**
+ **每个成员必须能够连接到每个其他成员。**

**NOTE：如果可能，请使用逻辑DNS主机名而不是IP地址，尤其是在配置副本集成员或分片AC集群成员时。逻辑DNS主机名的使用避免了由于IP地址更改而导致的配置更改。**
## 生产环境中禁用访问控制时部署副本集的步骤

1. IP绑定：使用bind_ip选项可确保MongoDB侦听来自配置地址上的应用程序的连接。在绑定到非本地主机（例如可公开访问的）IP地址之前，请确保已保护您的群集免受未经授权的访问。例如，以下mongod实例绑定到localhost和主机名TestHostname，它与ip地址198.51.100.1相关联：  

        mongod --bind_ip localhost,TestHostname
    要连接到此实例，远程客户端必须指定主机名或其关联的IP地址198.51.100.1：  

        mongo --host TestHostname
        mongo --host 198.51.100.1

2. 创建MongoDB存储数据文件的目录,在/etc/mongod.conf中存储的配置文件或相关位置中指定mongod配置。

3. 使用适当的选项启动副本集的每个成员。对于每个成员，使用以下设置启动mongod实例：
+ 将replication.replSetName选项设置为副本集名称（如果您的应用程序连接到多个副本集，则每个集合应具有不同的名称，某些驱动程序按副本集名称对副本集进行连接）。 
+ 将net.bindIp选项设置为hostname / ip或逗号分隔的hostnames / ips列表，以及配置部署所需的其他配置。  

    以下示例通过--replSet和--bind_ip命令行选项指定副本集名称和ip绑定：

         mongod --replSet "rs0" --bind_ip localhost,<hostname(s)|ip address(es)>
    对于<hostname（s）| ip address（es）>，指定远程客户端（包括副本集的其他成员）可用于连接的mongod实例的主机名和/或IP地址。
或者，也可以在配置文件中指定副本集名称和IP地址，要使用配置文件启动mongod，请使用--config选项指定配置文件的路径：  

        replication:
           replSetName: "rs0"
        net:
           bindIp: localhost,<hostname(s)|ip address(es)>
    
        mongod --config <path-to-config>

4. 使用mongo shell连接到其中一个mongod实例，从运行其中一个mongod的同一台机器（在本教程中为mongodb0.example.net）启动mongo shell。 要在默认端口27017上连接到监听localhost的mongod，只需指令mongo。
5. 启动副本集，仅在副本集的一个且仅一个mongod实例上运行rs.initiate（），如在副本集成员0上运行rs.initiate()：  

         rs.initiate( {
           _id : "rs0",
           members: [
              { _id: 0, host: "mongodb0.example.net:27017" },
              { _id: 1, host: "mongodb1.example.net:27017" },
              { _id: 2, host: "mongodb2.example.net:27017" }
           ]
        })

6. 查看副本集配置，使用rs.conf()显示副本集配置对象。如副本集配置对象类似于以下内容： 

          {
           "_id" : "rs0",
           "version" : 1,
           "protocolVersion" : NumberLong(1),
           "members" : [
          {
             "_id" : 0,
             "host" : "mongodb0.example.net:27017",
             "arbiterOnly" : false,
             "buildIndexes" : true,
             "hidden" : false,
             "priority" : 1,
             "tags" : {
    
             },
             "slaveDelay" : NumberLong(0),
             "votes" : 1
          },
          {
             "_id" : 1,
             "host" : "mongodb1.example.net:27017",
             "arbiterOnly" : false,
             "buildIndexes" : true,
             "hidden" : false,
             "priority" : 1,
             "tags" : {
    
             },
             "slaveDelay" : NumberLong(0),
             "votes" : 1
          },
          {
             "_id" : 2,
             "host" : "mongodb2.example.net:27017",
             "arbiterOnly" : false,
             "buildIndexes" : true,
             "hidden" : false,
             "priority" : 1,
             "tags" : {
    
             },
             "slaveDelay" : NumberLong(0),
             "votes" : 1
          }
    
       ],
       "settings" : {
          "chainingAllowed" : true,
          "heartbeatIntervalMillis" : 2000,
          "heartbeatTimeoutSecs" : 10,
          "electionTimeoutMillis" : 10000,
          "catchUpTimeoutMillis" : -1,
          "getLastErrorModes" : {
    
          },
          "getLastErrorDefaults" : {
             "w" : 1,
             "wtimeout" : 0
          },
          "replicaSetId" : ObjectId("585ab9df685f726db2c6a840")
         }
        }

7. 确保副本集具有主副本，使用rs.status()查看复制集的主节点。

## 测试以及开发环境中复制集的应用部署
在此测试以及开发部署中，三个成员在同一台计算机上运行。

1. IP绑定：使用bind_ip选项可确保MongoDB侦听来自配置地址上的应用程序的连接。在绑定到非本地主机（例如可公开访问的）IP地址之前，请确保已保护您的群集免受未经授权的访问。例如，以下mongod实例绑定到localhost和主机名TestHostname，它与ip地址198.51.100.1相关联：  

        mongod --bind_ip localhost,TestHostname
    要连接到此实例，远程客户端必须指定主机名或其关联的IP地址198.51.100.1：  

        mongo --host TestHostname
        mongo --host 198.51.100.1
2. 通过发出类似于以下内容的命令为每个成员创建必要的数据目录，这将创建名为“rs0-0”，“rs0-1”和“rs0-2”的目录，它们将包含实例的数据库文件，示例如下：  

        mkdir -p / srv / mongodb / rs0-0 / srv / mongodb / rs0-1 / srv / mongodb / rs0-2

3. 通过发出以下命令在自己的shell窗口中启动mongod实例： 

        //不同的三个节点：
        mongod --replSet rs0 --port 27017 --bind_ip localhost,<hostname(s)|ip address(es)> --dbpath /srv/mongodb/rs0-0 --smallfiles --oplogSize 128
        mongod --replSet rs0 --port 27018 --bind_ip localhost,<hostname(s)|ip address(es)> --dbpath /srv/mongodb/rs0-1 --smallfiles --oplogSize 1284
        mongod --replSet rs0 --port 27019 --bind_ip localhost,<hostname(s)|ip address(es)> --dbpath /srv/mongodb/rs0-2 --smallfiles --oplogSize 128

+ 这将每个实例作为名为rs0的副本集的成员启动，每个副本集都在不同的端口上运行，并使用--dbpath设置指定数据目录的路径。 如果已在使用上述端口，请选择其他的端口。
+ 实例绑定到localhost和主机的ip地址。
+ --smallfiles和--oplogSize设置可减少每个mongod实例使用的磁盘空间。 [1]这是测试和开发部署的理想选择，因为它可以防止机器过载。 有关这些和其他配置选项的更多信息，请参阅<配置文件选项>(https://docs.mongodb.com/manual/reference/configuration-options/)

4. 通过mongo shell连接到其中一个mongod实例，需要通过指定端口号来指示哪个实例。 假设选择第一个，如下面的命令：  

        mongo --port 27017

5. 在mongo shell中，使用rs.initiate（）来启动副本集，主要是在mongo shell环境中创建副本集配置对象，并用系统的主机名替换<hostname>，然后将rsconf文件传递给rs.initiate（），如以下示例所示：  

        rsconf = {
         _id: "rs0",
          members: [
        {
          _id: 0,
          host: "<hostname>:27017"
        },
        {
          _id: 1,
          host: "<hostname>:27018"
        },
        {
          _id: 2,
          host: "<hostname>:27019"
            }
           ]
         }
         rs.initiate( rsconf )

6.通过发出rs.conf()命令显示当前副本配置：rs.conf()，使用rs.status（）操作随时检查副本集的状态。副本集配置对象类似于以下内容：  

    {
        "_id" : "rs0",
         "version" : 1,
        "protocolVersion" : NumberLong(1),
        "members" : [
      {
         "_id" : 0,
         "host" : "<hostname>:27017",
         "arbiterOnly" : false,
         "buildIndexes" : true,
         "hidden" : false,
         "priority" : 1,
         "tags" : {

         },
         "slaveDelay" : NumberLong(0),
         "votes" : 1
      },
      {
         "_id" : 1,
         "host" : "<hostname>:27018",
         "arbiterOnly" : false,
         "buildIndexes" : true,
         "hidden" : false,
         "priority" : 1,
         "tags" : {

         },
         "slaveDelay" : NumberLong(0),
         "votes" : 1
      },
      {
         "_id" : 2,
         "host" : "<hostname>:27019",
         "arbiterOnly" : false,
         "buildIndexes" : true,
         "hidden" : false,
         "priority" : 1,
         "tags" : {

         },
         "slaveDelay" : NumberLong(0),
         "votes" : 1
      }
       ],
       "settings" : {
          "chainingAllowed" : true,
          "heartbeatIntervalMillis" : 2000,
          "heartbeatTimeoutSecs" : 10,
          "electionTimeoutMillis" : 10000,
          "catchUpTimeoutMillis" : -1,
          "getLastErrorModes" : {

      },
      "getLastErrorDefaults" : {
         "w" : 1,
         "wtimeout" : 0
      },
      "replicaSetId" : ObjectId("598f630adc9053c6ee6d5f38")
     }
    }


## 仲裁节点添加到副本集  

&emsp;&emsp;仲裁节点是mongod实例，它们是副本集的一部分但不保存数据。仲裁节点参加选举以打破关系。如果副本集具有偶数个成员，请添加仲裁者。  
&emsp;&emsp;仲裁节点具有最低的资源要求，不需要专用硬件。您可以在应用程序服务器或监视主机上部署仲裁程序。不要在也承载副本集的主成员节点或辅助成员节点的系统上运行仲裁程序。  
&emsp;&emsp;仲裁节点不存储数据，但是在将仲裁节点的mongod进程添加到副本集之前，仲裁节点将像任何其他mongod进程一样运行并启动一组数据文件和一个完整大小的日志。

1. 为仲裁节点创建数据目录（例如storage.dbPath）。 mongod实例使用该目录进行配置数据。 该目录不会保存数据集。 例如，创建/ data / arb目录：  

        mkdir /data/arb
2. 启动仲裁节点，指定数据目录和要加入的副本集的名称。 以下开始使用/ data / arb作为dbPath和rs作为副本集名称的仲裁： 

    mongod --port 27017 --dbpath /data/arb --replSet rs --bind_ip localhost,<hostname(s)|ip address(es)>

3. 连接到主服务器并将仲裁服务器添加到副本集。，使用rs.addArb（）方法，如以下示例所示，该示例假定m1.example.net是与仲裁节点的指定ip地址关联的主机名：  

        rs.addArb("m1.example.net:27017")  //此操作添加在m1.example.net主机上的端口27017上运行的仲裁器。

## 单实例转变为复制集
&emsp;&emsp;将独立mongod实例转换为副本集的过程。使用独立实例进行测试和开发，但始终在生产中使用副本集。该过程特定于不属于分片群集的实例。具体过程如下：
1. 关闭单实例的mongod。

2. 重启实例。 使用--replSet选项指定新副本集的名称。例如，以下命令将独立实例作为名为rs0的新副本集的成员启动。 该命令使用独立的/ srv / mongodb / db0的现有数据库路径：  

        mongod --port 27017 --dbpath /srv/mongodb/db0 --replSet rs0 --bind_ip localhost,<hostname(s)|ip address(es)>

3. 将mongo shell连接到mongod实例。

4. 使用rs.initiate（）启动新的副本集：rs.initiate()
副本集现在可以运行了。 要查看副本集配置，请使用rs.conf（）。 要检查副本集的状态，请使用rs.status（）。

## 增加新节点
&emsp;&emsp;副本集最多可以有七个投票成员。要将成员添加到已有七个投票成员的副本集，您必须将该成员添加为无投票成员或从现有成员中删除投票。您可以使用这些过程将新成员添加到现有集。您还可以使用相同的过程“重新添加”已删除的成员。如果删除的成员的数据仍然相对较新，很简单地恢复。   
&emsp;&emsp;如果您有现有成员的备份或快照，则可以将数据文件（例如dbPath目录）移动到新系统，并使用它们快速启动新成员。文件必须是来自同一副本集的成员的数据文件的有效副本。

### 准备数据目录
在将新成员添加到现有副本集之前，请使用以下策略之一准备新成员的数据目录：

+ 确保新成员的数据目录不包含数据。新成员将复制现有成员的数据。
+ 如果新成员处于恢复状态，则必须先退出并成为辅助成员，然后MongoDB才能在复制过程中复制所有数据。此过程需要时间，但不需要管理员干预。
+ 从现有成员手动复制数据目录。新成员将成为辅助成员，并将同步副本集的当前状态。复制数据可能会缩短新成员成为最新成员的时间。
+ 确保您可以将数据目录复制到新成员，并在oplog允许的窗口内开始复制。否则，新实例必须执行初始同步，这将完全重新同步数据，如重新同步副本集的成员中所述。
+ 使用rs.printReplicationInfo（）检查副本集成员关于oplog的当前状态。

### 具体步骤
1. 启动新的mongod实例。 指定数据目录和副本集名称。 以下示例指定/ srv / mongodb / db0数据目录和rs0副本集：

        mongod --dbpath /srv/mongodb/db0 --replSet rs0  --bind_ip localhost,<hostname(s)|ip address(es)>

    可以在mongod.conf配置文件中指定数据目录，副本集名称和ip绑定，并使用以下命令启动mongod：  

         mongod --config /etc/mongod.conf

2. 连接到副本集的主节点，只能在连接到主节点时添加成员。 如果不知道哪个成员是主成员节点，请登录到副本集的任何成员并发出db.isMaster（）命令。

3. 使用rs.add（）将新成员添加到副本集。 将成员配置文档传递给方法。 例如，要在主机mongodb3.example.net上添加成员，请发出以下命令：

        rs.add( { host: "mongodb3.example.net:27017", priority: 0, votes: 0 } )

4. 确保新成员已达到SECONDARY状态。 要检查副本集成员的状态，请运行rs.status（）：rs.status()

5. 一旦新添加的成员转换为SECONDARY状态，请使用rs.reconfig（）更新新添加的成员的优先级并根据需要进行投票。例如，如果rs.conf（）返回mongodb3.example.net:27017的配置文档作为members数组中的第五个元素，要更新其优先级并投票为1，请使用以下操作序列：

        var cfg = rs.conf();
        cfg.members[4].priority = 1
        cfg.members[4].votes = 1
        rs.reconfig(cfg)
**NOTE：当新添加的辅助节点的投票和优先级设置大于零时，在其初始同步期间，辅助节点仍然计为投票成员，即使它不能提供读取也不能成为主节点，因为其数据尚未一致。这可能导致大多数投票成员在线但不能选出主要成员的情况。 要避免这种情况，请考虑最初添加新的辅助优先级：0和投票：0。 然后，一旦成员转换到SECONDARY状态，使用rs.reconfig（）更新其优先级和投票。**
## 移除节点
### 使用rs.remove()移除过程
1. 关闭要删除的成员的mongod实例。 要关闭实例，请使用mongo shell和db.shutdownServer（）方法进行连接。

2. 连接到副本集的当前主节点。 要确定当前主节点，请在连接到副本集的任何成员时使用db.isMaster（）。

3. 使用以下任一形式的rs.remove（）删除该成员：

	rs.remove（ “mongod3.example.net:27017”）
	rs.remove（ “mongod3.example.net”）
NOTE：如果副本集需要选择新的主节点，MongoDB可能会短暂地断开shell。 在这种情况下，shell会自动重新连接。 即使命令成功，shell也可能显示DBClientCursor :: init call（）失败错误。

### 使用rs.reconfig()移除过程

1. 关闭要删除的成员的mongod实例。 要关闭实例，请使用mongo shell和db.shutdownServer（）方法进行连接。

2. 连接到副本集的当前主节点。 要确定当前主节点，请在连接到副本集的任何成员时使用db.isMaster（）。

3. 发出rs.conf（）方法以查看当前配置文档并确定要删除的成员的成员数组中的位置，示例mongod_C.example.net位于以下配置文件的第2位： 

        {
        "_id" : "rs",
        "version" : 7,
        "members" : [
            {
                "_id" : 0,
                "host" : "mongod_A.example.net:27017"
            },
            {
                "_id" : 1,
                "host" : "mongod_B.example.net:27017"
            },
            {
                "_id" : 2,
                "host" : "mongod_C.example.net:27017"
            }
        ]
    }

4. 将当前配置文档分配给变量cfg，修改cfg对象以删除该成员，并重启新配置，示例如下：

        cfg = rs.conf()
        cfg.members.splice(2,1)
        rs.reconfig(cfg)

5. 要确认新配置是否生效，请发出rs.conf（）。例如，输出将是： 

          {
            "_id" : "rs",
            "version" : 8,
            "members" : [
                {
                    "_id" : 0,
                    "host" : "mongod_A.example.net:27017"
                },
                {
                    "_id" : 1,
                    "host" : "mongod_B.example.net:27017"
                }
            ]
        }
        
## 总结
&emsp;&emsp;后续还有关于实践中分片的搭建过程，以及分片和复制集原理分析。  

**最后可关注公众号，一起学习,每天会分享干货，还有学习视频领取！**

![](https://user-gold-cdn.xitu.io/2020/4/14/171792690b19e0b8?w=350&h=129&f=png&s=17519)