# 第7节. 实现galaracluster和性能测试

![image-20230919135007319](7-实现galeracluster和性能测试.assets/image-20230919135007319.png)

<img src="7-实现galeracluster和性能测试.assets/image-20230919135102822.png" alt="image-20230919135102822" style="zoom:50%;" />



老版本的安装就是uyum install MariaDB-Galera-server，老版本的方法这里不管了，直接集成到新版里的。



然后就是看配置文件，在/etc/my.cnf里明确说了include

<img src="7-实现galeracluster和性能测试.assets/image-20230919135910659.png" alt="image-20230919135910659" style="zoom:50%;" /> 

galeracluster的配置就在这里

![image-20230919140028596](7-实现galeracluster和性能测试.assets/image-20230919140028596.png)



### wsrep_provider=

![image-20230919141456692](7-实现galeracluster和性能测试.assets/image-20230919141456692.png)

其中wsrep_provider=要填写一个叫做libgalera_smm.so的库文件，而这个文件是安装mariadb自带安装的，

![image-20230919141059885](7-实现galeracluster和性能测试.assets/image-20230919141059885.png)



![image-20230919141116807](7-实现galeracluster和性能测试.assets/image-20230919141116807.png)

![image-20230919141140225](7-实现galeracluster和性能测试.assets/image-20230919141140225.png)

而且就是mariadb的yum源安装的@mariadb可见。

👇这个源就是按官网的yum源复制过来就行啦，详情将前文mysql安装吧。

![image-20230919141214764](7-实现galeracluster和性能测试.assets/image-20230919141214764.png)

好了，基本零件都不缺了。



### wsrep_cluster_address=

![image-20230919141428096](7-实现galeracluster和性能测试.assets/image-20230919141428096.png)

三个节点的各自地址

**binlog_format=row**，就是二进制格式，一般默认新版都是MIXED，这里取消注释用ROW吧，忘记了，看前文👇

![image-20230919141620982](7-实现galeracluster和性能测试.assets/image-20230919141620982.png)



default_storage_engine=InooDB肯定是InnoDB了，MyISAM都不支持，官方就是说仅仅实验环境吧我看到好像。

innodb_autoinc_lock_mode=2这个不管默认就好

**其实就是改wsrep_provider和wsrep_cluster_address以及binlog_format=row。**



### 具体配置选项👇

https://mariadb.com/kb/en/configuring-mariadb-galera-cluster/

![image-20230919143040906](7-实现galeracluster和性能测试.assets/image-20230919143040906.png)

## 实际需要的操作

```bash
vim /etc/my.cnf.d/server.cnf
[galera]
wsrep_on=ON  # 10.1.1多了个开关
wsrep_provider=/usr/lib64/galera/libgalera_smm.so wsrep_cluster_address="gcomm://192.168.126.129,192.168.126.130,192.168.126.131"
binlog_format=row  
```

这个gcomm是协议，就好比ftp://还比https://一样，gcomm是glaeraCluster内部通讯的协议。

![image-20230919143202776](7-实现galeracluster和性能测试.assets/image-20230919143202776.png)

👆就改这么多，其他不动

优化设置

```bash
wsrep_cluster_name = 'mycluster'  # 默认my_wsrep_cluster
wsrep_node_name = 'node1'  # 本机器的名称，这里暂时不加
wsrep_node_address = '192.168.126.x’  # 当前机器的IP，不用写，这里只是强调写出，实际不写
```



通过scp复制到其他2个节点

![image-20230919143732435](7-实现galeracluster和性能测试.assets/image-20230919143732435.png)



## 启动注意点

* 三台不是一样的方法启动；

* 集群存在/不存在，启动方式也是不同的。

  

1、原来没有cluster的启动方法和原来有cluster集群的启动方法是不同！

![image-20230919145150254](7-实现galeracluster和性能测试.assets/image-20230919145150254.png)





![image-20230919145527141](7-实现galeracluster和性能测试.assets/image-20230919145527141.png)





2、首次启动，其他节点，systemctl restart mariadb就行



本实验是初始化集群，开始启动

```bash
任意一个节点，来初始化集群
galera_new_cluster

其他节点就
systemctl restart mariadb
```



![image-20230919150244962](7-实现galeracluster和性能测试.assets/image-20230919150244962.png)

这里我就用192.168.126.129，这里的master主机名，不要在意，不用纠结，这是以前实验 主从起的名字，这里多主，每个节点既是master也是slave。



其他节点重启报错

![image-20230919150841898](7-实现galeracluster和性能测试.assets/image-20230919150841898.png)



而且初始化集群的node，直接重启也会报错

![image-20230919150932693](7-实现galeracluster和性能测试.assets/image-20230919150932693.png)



此时处理方法，在没有头绪的情况下，俺是第一次接触这个集群，所以注释掉/etc/my.cnf.d/server里的galer模块配置，重启mariadb。

然后再重新走一遍就行了。注意重新启动就用galera_recovery来启动集群



![image-20230919152257276](7-实现galeracluster和性能测试.assets/image-20230919152257276.png)

![image-20230919152357434](7-实现galeracluster和性能测试.assets/image-20230919152357434.png)

然后莫名其妙再敲一遍就OK了？

![image-20230919152154647](7-实现galeracluster和性能测试.assets/image-20230919152154647.png)



然后去重启其他节点的服务，看看能不能成功，![image-20230919152516094](7-实现galeracluster和性能测试.assets/image-20230919152516094.png)

不行

排错

![image-20230919152942842](7-实现galeracluster和性能测试.assets/image-20230919152942842.png)

之前的配置注释掉



没用，重新做吧，把mariadb remove掉重弄？

重新rm -rf /var/lib/mysql/*   反复3次后倒是成功了

![image-20230919164358450](7-实现galeracluster和性能测试.assets/image-20230919164358450.png)

mb的怎么弄都OK了现在！！

应该就是/var/lib/mysql/下要干净，

### 然后有个明确的故障点处理方式就是

![image-20230919165029327](7-实现galeracluster和性能测试.assets/image-20230919165029327.png)

👆这个文件要么删掉，要么修改safe_to_bootstra:1就可以启动服务成功。



### 错误-2

![image-20230919165422809](7-实现galeracluster和性能测试.assets/image-20230919165422809.png)

这是cli用错了，重启不能这么干，

![image-20230919165938323](7-实现galeracluster和性能测试.assets/image-20230919165938323.png)

好像所有的node 都stop mariadb就起不来了，因为可能没有gcomm的通信方了

![image-20230919170223216](7-实现galeracluster和性能测试.assets/image-20230919170223216.png)

我的理解，就是集群里必须有一个活着才能重启成功，测试下



现在3个node都挂了，如何起来了，简单，

![image-20230919170253505](7-实现galeracluster和性能测试.assets/image-20230919170253505.png)

这样处理就行了

随便找一台node，一般是最优的吧

![image-20230919170359736](7-实现galeracluster和性能测试.assets/image-20230919170359736.png)

果然就好了，所以前面的配置OK就是这里的处理手法要注意，这里你可以理解为是技术细节，也可以理解为产品不傻瓜化。



此时理论上另外2个node直接restart就行了

![image-20230919170516424](7-实现galeracluster和性能测试.assets/image-20230919170516424.png)



本来创建集群的那一台tm的竟然起不来

![image-20230919170555058](7-实现galeracluster和性能测试.assets/image-20230919170555058.png)

判断失误，没事，老方法，狗屁啊，safe_to_bootstrap=1这个参数是针对the most advanced node最优节点也就是数据最新的节点的操作，就是确定后要敲galera_recovery的，前面已经有一台node初始化了cluster，此时其他只能是加入了，所以操作就是systemctl restart mariadb，通过报错日志可见

![image-20230919171357131](7-实现galeracluster和性能测试.assets/image-20230919171357131.png)

说明是SSL证书问题，关掉ssl

![image-20230919171423222](7-实现galeracluster和性能测试.assets/image-20230919171423222.png)

重启就好了

![image-20230919171445213](7-实现galeracluster和性能测试.assets/image-20230919171445213.png)





## 总结

1、初始化cluster

2、重启cluster

操作要小心



### galera_recover重启cluster，报错没关系，集群不受影响👇

![image-20230919171725860](7-实现galeracluster和性能测试.assets/image-20230919171725860.png)





## 总结2：重启集群

![image-20230919172031212](7-实现galeracluster和性能测试.assets/image-20230919172031212.png)

上图👆就告诉你了，如果nodes及群里的节点全部都stop了，此时就需要有一个node去做bootstrapped创建集群，如果只是简单的started normally就是systemctl start mariadb就会找wsrep_cluster_address里的IP地址去做gcomm协议连接，如果没有一个nodes是起来的，那么就会启动服务失败，这很关键。



## 测试下：

当前集群OK

![image-20230919172348760](7-实现galeracluster和性能测试.assets/image-20230919172348760.png)

### 停掉3台中的2台node，尝试将两台stop的start

![image-20230919172438926](7-实现galeracluster和性能测试.assets/image-20230919172438926.png)

没问题，处理方式OK

![image-20230919172549768](7-实现galeracluster和性能测试.assets/image-20230919172549768.png)

### 集群里的所有nodes全部停掉，这里就是3台咯

此时就需要恢复集群，

1、galera_recovery

![image-20230919172736925](7-实现galeracluster和性能测试.assets/image-20230919172736925.png)

按上文的说明，就是要修改

![image-20230919172843149](7-实现galeracluster和性能测试.assets/image-20230919172843149.png)



还是起不来

![image-20230919173002141](7-实现galeracluster和性能测试.assets/image-20230919173002141.png)





![image-20230919172952742](7-实现galeracluster和性能测试.assets/image-20230919172952742.png)

再次查看，发现人家让你找最优的node，何为最优，就是seqno

![image-20230919173039811](7-实现galeracluster和性能测试.assets/image-20230919173039811.png)



![image-20230919173056930](7-实现galeracluster和性能测试.assets/image-20230919173056930.png)

![image-20230919173113533](7-实现galeracluster和性能测试.assets/image-20230919173113533.png)



找到了，去84这台敲

galera_recovery

![image-20230919173145345](7-实现galeracluster和性能测试.assets/image-20230919173145345.png)

不行啊，操！

不过通过上面的操作已经知道了 不是恢复而是创建，用galera_new_cluster就可以了

![image-20230919173306841](7-实现galeracluster和性能测试.assets/image-20230919173306841.png)

而且集群ID也不会变

![image-20230919173341787](7-实现galeracluster和性能测试.assets/image-20230919173341787.png)

剩下两台node就简单了，重启服务就行了

![image-20230919173420275](7-实现galeracluster和性能测试.assets/image-20230919173420275.png)



到此不为此，基本的处理方法就有了。重点看总结就行，然后之前不行的原因就是①ssl没有弄好，之前实验遗留的，只配置了2个node的ssl，这里涉及3个呢，不过有一次ssl没关也OK，不过通过上文的报错可见确实有问题的；②就是集群的启动和重启有讲究的，重启有问题还是用的初始化创建的命令来解决问题的。



### 继续看集群的数据读写处理



说明：hostname无所谓master slave，这仅仅是之前实验的遗留，这里是多主，都是master。

导入一个sql脚本，会创建新库和表，这是在第一个节点上做的。

![image-20230919174143167](7-实现galeracluster和性能测试.assets/image-20230919174143167.png)

然后其他所有nodes也就自动同步了👇

![image-20230919174352331](7-实现galeracluster和性能测试.assets/image-20230919174352331.png)

表同步自然也OK

![image-20230919174507277](7-实现galeracluster和性能测试.assets/image-20230919174507277.png)

![image-20230919174538275](7-实现galeracluster和性能测试.assets/image-20230919174538275.png)



## 变量查看

### SHOW VARIABLES LIKE 'wsrep_%\G'; 



有当前自己的IP信息👇

![image-20230919174753051](7-实现galeracluster和性能测试.assets/image-20230919174753051.png)



有/etc/my.cnf.d/server.conf里配置的信息👇

![image-20230919174917759](7-实现galeracluster和性能测试.assets/image-20230919174917759.png)





### SHOW STATUS LIKE 'wsrep_%';   # 状态变量

![image-20230919175111726](7-实现galeracluster和性能测试.assets/image-20230919175111726.png)

![image-20230919175129421](7-实现galeracluster和性能测试.assets/image-20230919175129421.png)



### SHOW STATUS LIKE 'wsrep_cluster_size';   

![image-20230919175214090](7-实现galeracluster和性能测试.assets/image-20230919175214090.png)

查看集群nodes数👆



![image-20230919175319080](7-实现galeracluster和性能测试.assets/image-20230919175319080.png)





## 添加新成员node

![image-20230919180533521](7-实现galeracluster和性能测试.assets/image-20230919180533521.png)

不过人家官方说了，the first node has x.x.x.x，才能加入x.x.x.x，呵呵~。👇

![image-20230919180903800](7-实现galeracluster和性能测试.assets/image-20230919180903800.png)

其实就是每台node都补一个IP，然后一个个重启就行了

![image-20230919181220875](7-实现galeracluster和性能测试.assets/image-20230919181220875.png)

只要cluster里有一个活着，就能重启服务OK，除非同时down了

![image-20230919181251335](7-实现galeracluster和性能测试.assets/image-20230919181251335.png)

人家说的就是这个意思，所有nodes都down了，才需要重新初始化(这个我走成功了），或者没有走成功的galera_recovery；



然后新加入的node，也会很快同步db的

![image-20230919181437863](7-实现galeracluster和性能测试.assets/image-20230919181437863.png)





### 建表也不会冲突，因为有wsrep机制啊

![image-20230919181837269](7-实现galeracluster和性能测试.assets/image-20230919181837269.png)

只会有一个成功👇

![../_images/certificationbasedreplication.png](7-实现galeracluster和性能测试.assets/certificationbasedreplication.png)



## 然后看看大量数据写入的一个速度，是明显比主从慢很多的，因为👆

下班关机

算了明天继续弄吧，由于机器关机了，之前就是临时做实验，所以mysql服务都停了，这里正好重演一次cluster的启动

![image-20230919192042082](7-实现galeracluster和性能测试.assets/image-20230919192042082.png)

可见👆所有服务都没起来

随便选一台初始化集群

查看seqno虚拟号，👇下图注意哦，135的seqno最优秀，所以safe_to_bootstrap:1就是1，其他都是0，这是系统给默认设置好的，然后有个node3的1是我改的！我之前准备用node3初始化新建的，所以看到的是1。

![image-20230919192410975](7-实现galeracluster和性能测试.assets/image-20230919192410975.png)

node2最优秀，尝试不初始化，重启的专用cli试试

![image-20230919192638961](7-实现galeracluster和性能测试.assets/image-20230919192638961.png)

起不来啊~

直接new吧哈哈

![image-20230919192752265](7-实现galeracluster和性能测试.assets/image-20230919192752265.png)

![image-20230919192801825](7-实现galeracluster和性能测试.assets/image-20230919192801825.png)

然后其他node 都systemctl start mariadb就行了

![image-20230919192911714](7-实现galeracluster和性能测试.assets/image-20230919192911714.png)



同时创建的cli的没问题👇

![image-20230919193127367](7-实现galeracluster和性能测试.assets/image-20230919193127367.png)



### 测试开始



![image-20230919193544970](7-实现galeracluster和性能测试.assets/image-20230919193544970.png)



![image-20230919193647977](7-实现galeracluster和性能测试.assets/image-20230919193647977.png)



然后就看看数据的增长

![image-20230919194040967](7-实现galeracluster和性能测试.assets/image-20230919194040967.png)

发现这个速度比简单的主从还要慢很多，然后其实可以做成事务，事务就是一起提交，会快一点。

像这种就好比大量用户的写咯，所以具体用的时候还存在问题，需要优化吧应该！好像他们业务实际也不这么玩。



### 还发现一个现象👇

就是galera cluster默认就给你做了table的插入的自动间隔，以前是<img src="7-实现galeracluster和性能测试.assets/image-20230919195153835.png" alt="image-20230919195153835" style="zoom:33%;" />这么配置的。

![image-20230919195117332](7-实现galeracluster和性能测试.assets/image-20230919195117332.png)



**👇12分钟终于结束了：**

![image-20230919195714518](7-实现galeracluster和性能测试.assets/image-20230919195714518.png)



试试 事务的方式，理论上会快一些

![image-20230919195955669](7-实现galeracluster和性能测试.assets/image-20230919195955669.png)

这TM也太夸张了，不对吧

第二十运行事务，一样是6秒种，一共插入了3次，每次99999行，300000-3=299997行，对的👇

![image-20230919200224539](7-实现galeracluster和性能测试.assets/image-20230919200224539.png)

再次不跑事务看看



结论，galeraCluster集群，大数据并发写入，使用事务就很快！不使用事务就灰常慢！







## 复制的问题和解决方案

其实以下这段文字算不得什么问题和方案，聊胜于无，看看吧👇



(1)数据损坏或丢失

Master： MHA + semi repl

Slave：重新复制



(2)混合使用存储引擎

MyISAM：不支持事务 

InnoDB： 支持事务



(3)不惟一的server id 

 重新复制



(2)复制延迟：

需要额外的监控工具的辅助

一从多主：mariadb10版后支持 

多线程复制：对多个数据库复制，好像是高版本的特性。



## TiDb概述

**主从、多主，对于写操作，本质上都是在一台上操作的，所以mysql这种HA，本质上就是有瓶颈的。**

**而更优的解决方案就是TiDb分布式数据库**。

![image-20230919200851709](7-实现galeracluster和性能测试.assets/image-20230919200851709.png)



RDBMS关系型数据的 数据一致性ACID特性；NoSQL性能好但是没有保证数据的一致性；TiDb结合了两者的优点，又叫做NewSQL

所以DB分为了：RDBMS、NoSQL、NewSQL

<img src="7-实现galeracluster和性能测试.assets/image-20230919201306845.png" alt="image-20230919201306845" style="zoom:50%;" />





据说：mysql的业务迁到TiDb，基本上不用改动，直接搬过去就能用，不用改代码。



### TiDB的核心特点

1 高度兼容 MySQL	大多数情况下，无需修改代码即可从 MySQL 轻松迁移至 TiDB，分库分
表后的 MySQL 集群亦可通过 TiDB 工具进行实时迁移
2 水平弹性扩展	通过简单地增加新节点即可实现 TiDB 的水平扩展，按需扩展吞吐或存储，轻 松应对高并发、海量数据场景。
3 **分布式事务**	TiDB 100% 支持标准的 ACID 事务
4 真正金融级高可用	相比于传统主从 (M-S) 复制方案，基于 Raft 的多数派选举协议可以提 **供金融级的 100% 数据强一致性保证**，且在不丢失大多数副本的前提下，可以实现故障的自动 恢复 (auto-failover)，无需人工介入。
5 一站式 HTAP 解决方案	TiDB 作为典型的 OLTP 行存数据库，同时兼具强大的 OLAP 性能，  配合 TiSpark，可提供一站式 HTAP解决方案，一份存储同时处理OLTP & OLAP(OLAP、OLTP  的介绍和比较 )无需传统繁琐的 ETL 过程。
6 云原生 SQL 数据库	TiDB 是为云而设计的数据库，同 Kubernetes （十分钟带你理解 Kubernetes核心概念 ）深度耦合，支持公有云、私有云和混合云，使部署、配置和维护变得十 分简单。	TiDB 的设计目标是 100% 的 OLTP 场景和 80% 的 OLAP 场景，更复杂的 OLAP  分析可以通过 TiSpark 项目来完成。 TiDB 对业务没有任何侵入性，能优雅的替换传统的数据 库中间件、数据库分库分表等 Sharding 方案。同时它也让开发运维人员不用关注数据库 Scale  的细节问题，专注于业务开发，极大的提升研发的生产力.





**数据库的整理差不多了就**
下面介绍以下压力测试



# 性能衡量指标



**数据库服务衡量指标**

QPS:	query per second   # **查询性能**： 每秒处理的查询次数，简单的单表select和多表join对系统资源的消耗是截然不同！所以压力测试的时候是要事先定义select查询规则--涉及哪些查询方法。

TPS:	transaction per second  # 事务的处理，主要指的就是**数据的修改性能**了，涉及增删改。



**压力测试工具**
mysqlslap  # 系统自带，无需安装
Sysbench：功能强大 https://github.com/akopytov/sysbench
tpcc-mysql
MySQL Benchmark Suite
MySQL super-smack
MyBench



### mysqlslap使用

该工具来源于MariaDB-client软件👇

<img src="7-实现galeracluster和性能测试.assets/image-20230919202805799.png" alt="image-20230919202805799" style="zoom:50%;" /> 



![image-20230919202945144](7-实现galeracluster和性能测试.assets/image-20230919202945144.png)

![image-20230919204046401](7-实现galeracluster和性能测试.assets/image-20230919204046401.png)

![image-20230919204208689](7-实现galeracluster和性能测试.assets/image-20230919204208689.png)

注意：虽然这个工具测试完成后不会在DB中留痕，但是binlog肯定会大量被它修改的，所以测试的时候binlog要么关闭，要么单独存放。

binlog除了在/etc/my.cnd里定义，也可以放到/etc/my.cnf.d/server.conf里一样的，👇涉及集群galera里定义了binglog的格式row，所以也可以log-bin开启也放在这个配置文件里。

<img src="7-实现galeracluster和性能测试.assets/image-20230919203412460.png" alt="image-20230919203412460" style="zoom:44%;" />



binlog是否启用，最好还是看变量，而不是ll /var/lib/mysql/去看相关文件有没有对吧，你这样还得去先看看cnf人家配置在哪里了。

![image-20230919204001405](7-实现galeracluster和性能测试.assets/image-20230919204001405.png)





![image-20230919204226654](7-实现galeracluster和性能测试.assets/image-20230919204226654.png)

![image-20230919204240003](7-实现galeracluster和性能测试.assets/image-20230919204240003.png)





![image-20230919204606178](7-实现galeracluster和性能测试.assets/image-20230919204606178.png)

生产了大量的binlog

![image-20230919204636782](7-实现galeracluster和性能测试.assets/image-20230919204636782.png)



如果要停binlog，set 变量这种挺不掉，因为是基于你当前cli交互进去的session的，而压力测试是多session并发的，完全没有一点效果，因为session完全撞不到一起去，哈哈。而且sql_log_bin是session级别的变量，没有全局的。真的是session的，看看官方

![image-20230919205427595](7-实现galeracluster和性能测试.assets/image-20230919205427595.png)

![image-20230919205437537](7-实现galeracluster和性能测试.assets/image-20230919205437537.png)

再试试

![image-20230919205545717](7-实现galeracluster和性能测试.assets/image-20230919205545717.png)

果然👆，无法实现：关闭全局binlog的效果，只能去cnf文件了。



等等👇这TM什么回事：

![image-20230919205645119](7-实现galeracluster和性能测试.assets/image-20230919205645119.png)

问👆binlog基于本会话到底是关了还是没关？



![image-20230919205820257](7-实现galeracluster和性能测试.assets/image-20230919205820257.png)

![image-20230919210011549](7-实现galeracluster和性能测试.assets/image-20230919210011549.png)

确实关了



