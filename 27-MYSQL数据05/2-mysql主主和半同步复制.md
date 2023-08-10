# 第2节. mysql主主和半同步复制



# 补充几个选项

![image-20230810134026214](2-mysql主主和半同步复制.assets/image-20230810134026214.png)

### 5个写磁盘的及时性：

sync_binlog=1     binlog的及时写磁盘，

innodb_flush_log_at_trx_commit=1       事务的及时写磁盘，当然不是说及时就一定是好的，也要考虑IO的负担。

sync_master_info是  多少次 事件后 master.info的信息 写磁盘

sync_relay_log   多少次 后同步到relaylog磁盘文件，应该也是指relay-log.info文件吧。

sync_relay_log_info    多少次事务后 后同步到relay-log.info磁盘文件。



skip-slave-start=ON  # 要知道一个事实--重启mariadb服务后，slave的两个线程IO和SQL都是OK的，也就是第一次手动start slave;后面都不用管的，都是随服务启动就启动的。这个特性就是依赖于skip-slave-start这个选项，如果置为ON，就不会随服务启动了，到时候就只能手动每次start slave了。







# 主-主 复制，需要调度固定分配成主-从



主-主前面一篇就讲过了，这种的好处，是多活吗，无需从节点提升 这么一个动作。



所谓主主，其实是互为主从，A-B 两个机器，A是B的主，又是B的从，B一样。



---

不过有人就是要真正意义上的主主，多活，A/A，类似ASA的A/A，人家就是要并发写，我那个去，怎么搞，这么搞。

![image-20230810140651676](2-mysql主主和半同步复制.assets/image-20230810140651676.png)

所谓的并发写，A/A，其实就是通过

![image-20230810141138154](2-mysql主主和半同步复制.assets/image-20230810141138154.png)

auto_increment：也就是奇偶区分，不同的主起始不通，间隔一样，错开序号了就。

然后再配合前置调度器，比如，来了2条写操作，调度器就直接并发的扔给2个主，而2个主处理的时候又是错开PRI主键ID的，于是就实现了多主。



```
节点A上
------
yum -y remove mariadb-server
rm -rf /var/lib/mysql/*
cp -a /etc/my.cnf /etc/my.cnf.bak

yum -y install mariadb-server
vim /etc/my.cnf
	[mysqld]
	server-id=1
	log-bin
	auto_increment_offset=1
	auto_increment_increment=2
wr

systemctl start mariadb

mysql > grant replication slave on *.* to repluser@'192.168.126.%' identified by 'cisco';


```

```
节点B上
------
yum -y remove mariadb-server
rm -rf /var/lib/mysql/*
cp -a /etc/my.cnf /etc/my.cnf.bak

yum -y install mariadb-server
vim /etc/my.cnf
	[mysqld]
	server-id=2
	log-bin
	auto_increment_offset=2
	auto_increment_increment=2
wr

systemctl start mariadb

 // mysql > grant replication slave on *.* to repluser@'192.168.126.%' identified by 'cisco';   # 这句就不用敲了，直接复制过来从A。
 
 mysql > 
 CHANGE MASTER TO
 MASTER_HOST='192.168.126.129',
 MASTER_USER='repluser',
 MASTER_PASSWORD='cisco',
 MASTER_PORT=3306,
 MASTER_CONNECT_RETRY=10,
 MASTER_LOG_FILE='mariadb-bin.000001', MASTER_LOG_POS=330;   # 去A看下show master logs;将binglog文件和pos位置复制过来。不过这个要配合mysqldump 从A备份还原到B，如果不想这么麻烦，直接将这里的binglog设置成第一个文件的第一个位置一般是330左右。可以刷新一个binlog，看看初始值多少。放屁！这样从头复制直接报错！哈哈因为复制的时候可不会想mysqldump出来的有IF判断存在就DROP的语句，主从复制如果存在就直接报错卡住了。
 
 所以这里的binlog和pos需要先去master上show master logs;复制过来的，然后master前面的数据需要dump过来的。
 

```

报错啦

![image-20230810161837911](2-mysql主主和半同步复制.assets/image-20230810161837911.png)

```
还是去A上dump吧

mysqldump -A --single-transaction --master-data=1 > /data/all.sql
scp /data/all.sql root@192.168.126.130:/data/
```

![image-20230810162257932](2-mysql主主和半同步复制.assets/image-20230810162257932.png)

```
去B上还原下
msyql > stop slave;
mysql < /data/all.sql
msql > show databaes;   可见hellodb同步过来了
mysql > start slave;
mysql > show slave status\G;   这回就OK了。

```

![image-20230810162412356](2-mysql主主和半同步复制.assets/image-20230810162412356.png)

![image-20230810162422644](2-mysql主主和半同步复制.assets/image-20230810162422644.png)

这就好了，果然是primary主键可能不让写奇数吧，因为/etc/my.cnf里是做了偶数限制

<img src="2-mysql主主和半同步复制.assets/image-20230810162621208.png" alt="image-20230810162621208" style="zoom:50%;" />



果然个屁，再仔细看看报错，是dupicate

![image-20230810163449550](2-mysql主主和半同步复制.assets/image-20230810163449550.png)

这就是已经说主从复制的时候，已经有的数据库，里面的表，再次写的时候就报错了，

但是mysqldump 出来的all.sql里其实也是大量的sql语言，但是人家是写了IF 条件判断的，如果存在这个表格就DROP的啊，哈哈，对吧mydqldump是先铲除后还原啊，所以不会报错啊👇

![image-20230810163920576](2-mysql主主和半同步复制.assets/image-20230810163920576.png)

而复制就不是啦，存在的就冲突了。现在知道为什么每次都dump而不是从头做复制了吧。



现在传统的主-从就OK，测试一下

![image-20230810171520308](2-mysql主主和半同步复制.assets/image-20230810171520308.png)

![image-20230810171509543](2-mysql主主和半同步复制.assets/image-20230810171509543.png)





## 然后把节点B也就是slave也变成master

```
B上也就是slave：
------
mysql > show master logs;


```

![image-20230810171541654](2-mysql主主和半同步复制.assets/image-20230810171541654.png)

```
A上也就是master：
---------
 
mysql > 
 CHANGE MASTER TO
 MASTER_HOST='192.168.126.130',
 MASTER_USER='repluserB',
 MASTER_PASSWORD='cisco',
 MASTER_PORT=3306,
 MASTER_CONNECT_RETRY=10,
 MASTER_LOG_FILE='mariadb-bin.000002', MASTER_LOG_POS=2539205;
 
 上面的binglog文件和pos位置就是B上看到的复制过来的，敲完A就是从了，当然还得start slave一下；
 
mysql > start slave;
mysql > show slave status\G;
```

然后就报错了

![image-20230810172640127](2-mysql主主和半同步复制.assets/image-20230810172640127.png)

奇了怪了，用户确认同步过去了啊，密码通过哈希值都能判断是一致的👇

![image-20230810172745859](2-mysql主主和半同步复制.assets/image-20230810172745859.png)





![image-20230810172705241](2-mysql主主和半同步复制.assets/image-20230810172705241.png)



不管先在B上新建一个用户试试

```
在B上
-----
 grant replication slave on *.* to repluserB@'192.168.126.%' identified by 'cisco';
```

![image-20230810173046337](2-mysql主主和半同步复制.assets/image-20230810173046337.png)



![image-20230810173136519](2-mysql主主和半同步复制.assets/image-20230810173136519.png)



靠！好了，难道用户要单独创建咯，好吧。难道主主，复制用户必须各自单独创建。不是，后面我又把A的复制用户换回去又可以了。感觉是这个用户没有在B上生效。

<img src="2-mysql主主和半同步复制.assets/image-20230810174357148.png" alt="image-20230810174357148" style="zoom:50%;" />

好像flush privileges;就好了，不过我记得我刷过啊，估计就是没生效当时，然后过一会生效了，或者是创建repluserB的时候相当于又刷了一次，我当时应该多刷几次的，(￣▽￣)"。



![image-20230810174556864](2-mysql主主和半同步复制.assets/image-20230810174556864.png)





A->B 和A<-B 双向同步就都测试到了👇

![image-20230810173356188](2-mysql主主和半同步复制.assets/image-20230810173356188.png)



![image-20230810173427835](2-mysql主主和半同步复制.assets/image-20230810173427835.png)





## 模拟写表看看是否冲突-主主的时候

```
A上
-----
mysql > use hellodb
mysql > create table test (id int auto_increment primary key, name char(10));

mysql > desc test;

```



![image-20230810175251661](2-mysql主主和半同步复制.assets/image-20230810175251661.png)

此时B上也有了

![image-20230810175344445](2-mysql主主和半同步复制.assets/image-20230810175344445.png)



```
通过xshel的批量下发cli，给A和B同事敲入insert

insert test (name) values ('wang');

```

![image-20230810180052019](2-mysql主主和半同步复制.assets/image-20230810180052019.png)



这样就同时键入了sql insert

![image-20230810180224614](2-mysql主主和半同步复制.assets/image-20230810180224614.png)



![image-20230810180533473](2-mysql主主和半同步复制.assets/image-20230810180533473.png)

不冲突，左边A是1，右边B插入的是2偶数，奇偶错开来的。



![image-20230810193755718](2-mysql主主和半同步复制.assets/image-20230810193755718.png)

还是一样左边1 3 5 ，右边2 4 6



创建表，理应冲突

![image-20230810194117844](2-mysql主主和半同步复制.assets/image-20230810194117844.png)

但是高版本确实不会了，不知道怎么实现的。



我理解一定是有一个被回退了，内部忽略了，去找找相关资料。



![image-20230810195508311](2-mysql主主和半同步复制.assets/image-20230810195508311.png)

中文会翻译掉了一写 重要的 专业词汇，就是人家起的名字吧，通过英文去问得到英文的回答

![image-20230810200423244](2-mysql主主和半同步复制.assets/image-20230810200423244.png)

通过GPT找到了关键信息，然后通过关键信息去官网再找

![image-20230810200534191](2-mysql主主和半同步复制.assets/image-20230810200534191.png)

https://mariadb.com/kb/en/about-galera-replication/

![image-20230810200504940](2-mysql主主和半同步复制.assets/image-20230810200504940.png)

然后看看默认的值就是1👇

![image-20230810200251384](2-mysql主主和半同步复制.assets/image-20230810200251384.png)

确认了👆

还有这一篇也写的不错

https://galeracluster.com/library/documentation/certification-based-replication.html

![image-20230810201309719](2-mysql主主和半同步复制.assets/image-20230810201309719.png)



