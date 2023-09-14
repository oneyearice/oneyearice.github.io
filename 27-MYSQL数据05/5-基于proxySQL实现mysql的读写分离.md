# 第5节. 基于proxySQL实现mysql的读写分离



# 简单介绍下工作原理



![image-20230908173634354](5-基于proxySQL实现mysql的读写分离.assets/image-20230908173634354.png)

proxySQL通过read-only选项来识别谁是slave的，然后讲DML写道主上，DQL查询往从上发。



官网的指南：

https://proxysql.com/documentation/installing-proxysql/

我用的是rockylinux，通过它的这个URL

![image-20230908174134701](5-基于proxySQL实现mysql的读写分离.assets/image-20230908174134701.png)

网上层看

![image-20230908174151934](5-基于proxySQL实现mysql的读写分离.assets/image-20230908174151934.png)



![image-20230908174202461](5-基于proxySQL实现mysql的读写分离.assets/image-20230908174202461-1694166124950-3.png)

所以待会可能用centos8来试试



通过上述就知道直接用他的指南肯定报错，因为我的rocky和他的URL对不上的

![image-20230908174520664](5-基于proxySQL实现mysql的读写分离.assets/image-20230908174520664.png)



直接epel里就有这个proxysql

![image-20230908174612825](5-基于proxySQL实现mysql的读写分离.assets/image-20230908174612825.png)

版本也不低了2.4.8，直接用吧



# ProxySQL组成

服务脚本：/etc/init.d/proxysql 

配置文件：/etc/proxysql.cnf 

主程序：/usr/bin/proxysql

基于SQLITE的数据库文件：/var/lib/proxysql/    # 读写规则调度策略配置信息都是放到这个小型数据库里的。不是有配置文件吗，奇怪了，继续看吧，据说是配置文件仅仅可以改一小部分，基本上全部的配置信息都是放到SQLITE里的。所以将来管理proxySQL不是修改配置文件，而是通过像连接mysql一样去配置SQLITE。而且sql修改了SQLITE其实也会自动不配置文件里的内容给改了。



启 动 ProxySQL：service proxysql start  启动后会监听两个默认端口

​	6032：ProxySQL的管理端口 就是进入SQLITE的途径。

​	6033：ProxySQL对外提供服务的端口

![image-20230908180034440](5-基于proxySQL实现mysql的读写分离.assets/image-20230908180034440.png)

6033默认面向用户的，可以改成3306这样用户那边就不要变动了，直接把proxySQL当做mysql连就行了，透明处理--对于用户来讲他只会以为连接的就是DB。



# proxySQL默认数据库



**main** 是默认的”数据库”名，表里存放后端db实例、用户验证、路由规则等信息。 表名以 runtime开头的表示proxysql当前运行的配置内容，不能通过dml语句修改， 只能修改对应的不以 runtime 开头的（在内存）里的表，然后 LOAD 使其生效， SAVE 使其存到硬盘以供下次重启加载



**disk** 是持久化到硬盘的配置，sqlite数据文件



**stats** 是proxysql运行抓取的统计信息，包括到后端各命令的执行次数、流量、 processlist、查询种类汇总/执行时间，等等



**monitor** 库存储 monitor 模块收集的信息，主要是对后端db（主从）的健康/延迟检查





# 开始配置

192.168.126.129 - master

192.168.126.130 -  slave

192.168.126.131 - proxySQL



## 搭建主从

master的配置：在129上

```shell
vim /etc/my.cnf
    server-id=129
    log-bin
wr

systemctl restart mariadb

mysql -e 'show master logs'  # 从这里开始复制，把下面的创建账号也复制过去，其实正儿八经是先dump在开主从 ，就是按前面专门讲主从的章节来配置就行了

mysql -e "grant replication slave on *.* to repluser@'192.168.%.%' identified by 'centos' "
mysql -e "select user,host,password from mysql.user"  #检查下上一行的用户是否创建成功



```



slave的配置：在130上

```shell
vim /etc/my.cnf
    server-id=130
    read-only
wr
systemctl restart mariadb

msyql  # 进入mysql的交互cli模式

MariaDB[(none)]> CHANGE MASTER TO
     -> MASTER_HOST='192.168.126.129',
     -> MASTER_USER='repluser',
     -> MASTER_PASSWORD='centos',
     -> MASTER_PORT=3306,
     -> MASTER_LOG_FILE='mariadb-bin.000001',
     -> MASTER_LOG_POS=xxx;  # xxx就填上面master里的log的位置。

MariaDB[(none)]> start slave;  # 如果不是干净的slave之前配置过，就stop slave;reset slave all;再start slave;
MariaDB[(none)]> show slave status\G;  # 看到两个yes就基本OK了。

再测试一下同步是否OK，去master上创建也给库看看同步与否

```

## 到此主从就搭建好了，下面开始弄proxySQL

安装直接yum，epel里就有版本不低，反正没有rockylinux9的，有aws的新版倒是。这里选择直接yum了。

这里👇有最新版

https://github.com/sysown/proxysql/releases

![image-20230912110844913](5-基于proxySQL实现mysql的读写分离.assets/image-20230912110844913.png)

也许rokcylinux可以用almalinux9的，试试~

```
[root@bind-child ~]# curl -OL https://github.com/sysown/proxysql/releases/download/v2.5.5/proxy                                                                               sql-2.5.5-1-almalinux9.x86_64.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 22.0M  100 22.0M    0     0  1458k      0  0:00:15  0:00:15 --:--:-- 1690k


[root@bind-child ~]# ls
anaconda-ks.cfg  go   lib64                                   pyvenv.cfg
bin              lib  proxysql-2.5.5-1-almalinux9.x86_64.rpm  tcping
[root@bind-child ~]# yum -y install proxysql-2.5.5-1-almalinux9.x86_64.rpm  # 当前目录下的rpm包直接yum它就行了


再安装一个mysql-client用来做msyql db的client连接用，不知道是不是必须的哦，继续梳理
[root@vpn001 ~]# rpm -qf `which mysql`
mariadb-5.5.64-1.el7.x86_64

[root@bind-child ~]# yum -y install mariadb  # 注意不是mariadb-client，直接mariadb就行，mysql这个命令就是来自mariadb这个包的。

[root@master ~]# rpm -qf `which mysql`
MariaDB-client-10.11.5-1.el9.x86_64
[root@master ~]#
虽然显示mysql这个客户端命令就是来自于MariaDB-client,但是也可以
yum -y install MariaDB-client
同时也可以
yum -y install mariadb  一个效果。
```

有个用户组没得，先不管它

![image-20230912111635267](5-基于proxySQL实现mysql的读写分离.assets/image-20230912111635267.png)

![image-20230912111722802](5-基于proxySQL实现mysql的读写分离.assets/image-20230912111722802.png)

有的吖，操，瞎报什么...安装后脚本看看

![image-20230912111932610](5-基于proxySQL实现mysql的读写分离.assets/image-20230912111932610.png)

哈哈，果然，这标志着我的这种慢慢吞吞就是吞吞怎么滴，学的法子，就不差，就能反映过来，yum的时候是安装报错没有group，就是一定是没有gorup的，但是结果id又看到了group，说明是安装后脚本，对吧，我是查看之前判断的，还不错。



![image-20230912112424146](5-基于proxySQL实现mysql的读写分离.assets/image-20230912112424146.png)

版本还可以这是目前最新的了，毕竟是AlmaLinux9用的proxysql。

https://www.51cto.com/article/705594.html

![image-20230912133818945](5-基于proxySQL实现mysql的读写分离.assets/image-20230912133818945.png)



启动proxysql服务后可见监听端口6032 6033

![image-20230912134224249](5-基于proxySQL实现mysql的读写分离.assets/image-20230912134224249.png)

老版本是通过service启动的

![image-20230912134345588](5-基于proxySQL实现mysql的读写分离.assets/image-20230912134345588.png)

6032是管理端口，进行配置proxysql的

6033是用户去连接的。



**proxy的配置文件一般不用去改，一般都是通过sqlite这个小库去dml修改配置，sql语句修改了sqlite，就会自动把这个配置文件/etc/proxysql给改了。**可以看一下配置文件

<img src="5-基于proxySQL实现mysql的读写分离.assets/image-20230912135245437.png" alt="image-20230912135245437" style="zoom:50%;" /> 

配置文件里就有端口号，可以自定义修改。

### 备份配置文件

![image-20230912135500452](5-基于proxySQL实现mysql的读写分离.assets/image-20230912135500452.png)

①注意用-a

②覆盖不想看到交互让你y，就用\cp   \就是转义转成原始的cp，要知道cp一般是alias

![image-20230912135551816](5-基于proxySQL实现mysql的读写分离.assets/image-20230912135551816.png)



<img src="5-基于proxySQL实现mysql的读写分离.assets/image-20230912135743525.png" alt="image-20230912135743525" style="zoom:40%;" /> 



但是没有改过来

![image-20230912141256796](5-基于proxySQL实现mysql的读写分离.assets/image-20230912141256796.png)

改不过来就算，直接进SQLITE去用sql去改吧，反正后面都是用这种方式。

恢复一下

![image-20230912145534627](5-基于proxySQL实现mysql的读写分离.assets/image-20230912145534627.png)





注意

![image-20230912145848311](5-基于proxySQL实现mysql的读写分离.assets/image-20230912145848311.png)

移除的是server，其实client也会被当做依赖一并移除。

可以直接yum mariadb就是client

![image-20230912145947006](5-基于proxySQL实现mysql的读写分离.assets/image-20230912145947006.png)

也可以MariaDB-client,一样的效果。

![image-20230912150209818](5-基于proxySQL实现mysql的读写分离.assets/image-20230912150209818.png)



有了mysql 这个client命令，就可以进入proxysql的sqlite了👇

### 连接proxysql的sqlite

```
mysql -uadmin -padmin -P6032 -h127.0.0.1
```

![image-20230912150532284](5-基于proxySQL实现mysql的读写分离.assets/image-20230912150532284.png)

默认db

![image-20230912150911679](5-基于proxySQL实现mysql的读写分离.assets/image-20230912150911679.png)

默认看的就是main库👇

![image-20230912150942269](5-基于proxySQL实现mysql的读写分离.assets/image-20230912150942269.png)



**monitor库可以看log了解相关信息，比如复制是否成功**

![image-20230914140650513](5-基于proxySQL实现mysql的读写分离.assets/image-20230914140650513.png)



### 

看下这个表

select * from sqlite_master ;这个命令中sqlite_master没有指定哪个库，就是默认的main库

通过select * from db_name.sqlite_manster;可以看到每个db都有一个默认的表，且内容不同，且show tables from db是看不到的！

select * from main.sqlite_master where name='mysql_servers'\G;可以看到当初是怎么创建定义字段的。

![image-20230914141946948](5-基于proxySQL实现mysql的读写分离.assets/image-20230914141946948.png)

同时mysql里的desc命令在sqilte里是没有的，就是通过👆上图的方式来查看类似信息的。



## 开始配置proxySQL



### 添加mysql节点，默认就是操作的main，无需use main，等价于use main。

```sqlite
MySQL > insert into mysql_servers(hostgroup_id,hostname,port) values(10,'192.168.126.129',3306);
MySQL > insert into mysql_servers(hostgroup_id,hostname,port)  values(20,'192.168.126.130',3306);
```

insert后检查：

![image-20230914142834738](5-基于proxySQL实现mysql的读写分离.assets/image-20230914142834738.png)

load和save，一个是加载到内存，一个是保存到磁盘，类似juniper，vyos的commit和save；

```sqlite
MySQL > load mysql servers to runtime;
MySQL > save mysql servers to disk;
```



### 创建登入mysql节点们的账号，用来查看read-only选项，以判断主从。

在mysql节点上

```mysql
master节点：
MySQL> grant replication client on *.* to monitor@'192.168.%.%'  identified by 'cisco';     
```

一般此时主从都是正常同步的，所以slave上也会自动创建同样的账号密码。



### 在proxySQL上使用该密码用来做监控，其实就包含主从的判断

```mysql
MySQL [main]> set mysql-monitor_username='monitor';
MySQL [main]> set mysql-monitor_password='cisco';

mysql > load mysql variables to runtime;
mysql > save mysql variables to disk;

MySQL [main]> select * from runtime_mysql_users;

```

<img src="5-基于proxySQL实现mysql的读写分离.assets/image-20230914145001071.png" alt="image-20230914145001071" style="zoom:50%;" /> 

但是找不到配置的信息在哪里查看，虽然可以肯定已经生效了，因为后买你可以通过log看到连接OK的。



本质上就是proxysql通过这种方式来判断主从的👇

<img src="5-基于proxySQL实现mysql的读写分离.assets/image-20230914144006119.png" alt="image-20230914144006119" style="zoom:50%;" /> 



<img src="5-基于proxySQL实现mysql的读写分离.assets/image-20230914143931759.png" alt="image-20230914143931759" style="zoom:45%;" /> 



### 查看相关日志，看连接有无问题

```
查看监控连接是否正常的 (对connect指标的监控)：(如果connect_error的结果
为NULL则表示正常)
MySQL> select * from mysql_server_connect_log;
查看监控心跳信息 (对ping指标的监控)：
MySQL> select * from mysql_server_ping_log;
查看read_only和replication_lag的监控日志
MySQL> select * from mysql_server_read_only_log;
MySQL> select * from mysql_server_replication_lag_log;
```

![image-20230914145249151](5-基于proxySQL实现mysql的读写分离.assets/image-20230914145249151.png)

从上往下插入的，上面很多都是一开始没有配置密码的时候的报错，账号是一致的，没有配置前默认就是用monitor作为账号的。

默认密码在proxysql的配置文件里可见👇

![image-20230914145554054](5-基于proxySQL实现mysql的读写分离.assets/image-20230914145554054.png)



![image-20230914150028653](5-基于proxySQL实现mysql的读写分离.assets/image-20230914150028653.png)

但是这两个日志还没有记录👇

![image-20230914155859124](5-基于proxySQL实现mysql的读写分离.assets/image-20230914155859124.png)

通过前面的配置此时proxysql已经能识别出来谁是master，谁是slave了。



### 读写分离：分组，那个是写组，哪些事读组

```
MySQL> insert into mysql_replication_hostgroups values(10,20,"test"); 
修正为：
MySQL> insert into mysql_replication_hostgroups values(10,20,"read_only","test"); 

MySQL> load mysql servers to runtime;
MySQL> save mysql servers to disk;

通过下图可知，前面写的是写的，后面是读的，不过疑问就来了这里的10，20是否一定要和前面定义节点的时候的10，20保持一致，为了得到验证，这里先把10，20改成100，200

```

按PPT来，报错了，自行排错

![image-20230914172807043](5-基于proxySQL实现mysql的读写分离.assets/image-20230914172807043.png)

![image-20230914173333982](5-基于proxySQL实现mysql的读写分离.assets/image-20230914173333982.png)

好了加了ready_only的变种关键字，应该是新版本的变动。

![image-20230914173640691](5-基于proxySQL实现mysql的读写分离.assets/image-20230914173640691.png)

别忘了load和save

![image-20230914173739666](5-基于proxySQL实现mysql的读写分离.assets/image-20230914173739666.png)



为了测试10，20是不是要和前面的定义节点的时候保持一致，我觉得肯定要一致就是把定义节点的hostGroupID写到这里来的，才符合逻辑，这里先改掉，改个屁，给老子改回来，明摆着的事情不要瞎弄。

![image-20230914174528614](5-基于proxySQL实现mysql的读写分离.assets/image-20230914174528614.png)

特意将两处的id改为不一致

![image-20230914174726822](5-基于proxySQL实现mysql的读写分离.assets/image-20230914174726822.png)

改回来，不浪费时间！

![image-20230914203913022](5-基于proxySQL实现mysql的读写分离.assets/image-20230914203913022.png)





这个不知道怎么查看主从判断的结果。





### 读写分离：proxySQL使用的业务用户做DML和DQL用



配置发送SQL语句的用户，在**master**节点上创建访问用户，让它同步到**slave**上。

```sqlite
MySQL> grant all on *.* to sqluser@'192.168.%.%' identified by 'cisco';
```

**在ProxySQL配置**，将用户sqluser添加到mysql_users表中， default_hostgroup默认组设置为写组10，当读写分离的路由规则不符合时，会访问10这个默认组的数据库；关注这里的10，从①定义节点②分组写读③默认路由要串起来理解。

```
MySQL> insert into mysql_users(username,password,default_hostgroup)  values('sqluser','cisco',10);
MySQL> load mysql users to runtime;
MySQL> save mysql users to disk;
```

使用sqluser用户测试是否能路由到默认的10写组实现读、写数据

```
mysql -usqluser -pcisco -P6033 -h127.0.0.1 -e 'select @@server_id'
mysql -usqluser -pcisco -P6033 -h127.0.0.1 -e 'create database testdb'
mysql -usqluser -pcisco testdb -P6033 -h127.0.0.1 -e 'create table t(id int)'
```

![image-20230914204712872](5-基于proxySQL实现mysql的读写分离.assets/image-20230914204712872.png)

就是在proxysql上通过127环回口连自己，模拟用户进行DQL查询，发现的到server_id都是129，说明都是走的proxySQL的默认路由其实人家叫default_hostgroup 10，10里放的就是192.168.126.129这个master。

![image-20230914205016417](5-基于proxySQL实现mysql的读写分离.assets/image-20230914205016417.png)

