# 第2节. mysql备份还原mysqldump使用



# mysqldump

mysqldump是mariadb-client安装的时候就自带的

![image-20230719150441673](2-mysql备份还原mysqldump使用.assets/image-20230719150441673.png)

查看mysqldump的帮助信息：

![image-20230719150957100](2-mysql备份还原mysqldump使用.assets/image-20230719150957100.png)

![image-20230719151016824](2-mysql备份还原mysqldump使用.assets/image-20230719151016824.png)

![image-20230719151121884](2-mysql备份还原mysqldump使用.assets/image-20230719151121884.png)

-A 也不是真的如上图所说是all-databases，不会备份information_schema，也不会备份performance_schema，

<img src="2-mysql备份还原mysqldump使用.assets/image-20230719151440765.png" alt="image-20230719151440765" style="zoom:50%;" />



比如可以这么写

mysqldump hellodb   		# 必须指定一个数据库

mysqldump hellodb students

mysqldump hellodb



mysqldump -uroot -pxxx mysql 对 mysql这个系统库进行备份👇

![image-20230719155506000](2-mysql备份还原mysqldump使用.assets/image-20230719155506000.png)

1、mysqldump db_name 是把数据库里的东西做了个一个显示，可以看到user的视图，slowlog的insert等等

![image-20230719170313496](2-mysql备份还原mysqldump使用.assets/image-20230719170313496.png)

![image-20230719170345245](2-mysql备份还原mysqldump使用.assets/image-20230719170345245.png)



![image-20230719171635867](2-mysql备份还原mysqldump使用.assets/image-20230719171635867.png)

看提示，关闭binlog

![image-20230719172734478](2-mysql备份还原mysqldump使用.assets/image-20230719172734478.png)



这样就可以了

![image-20230719172758457](2-mysql备份还原mysqldump使用.assets/image-20230719172758457.png)

然后去用mysqldump 看看是否能看到这个函数

![image-20230719172920154](2-mysql备份还原mysqldump使用.assets/image-20230719172920154.png)



## 先看下一种错误的mysqldump

全备可以使用重定向到一个文件里

mysqldump -uroot -pxxxx hellodb > /data/hellodb.sql

![image-20230719174253475](2-mysql备份还原mysqldump使用.assets/image-20230719174253475.png)

所谓mysqldump db其实就是保存了之前配置的各种命令

![image-20230719174701959](2-mysql备份还原mysqldump使用.assets/image-20230719174701959.png)

下面开始利用这个dump出来的文件进行还原测试，

drop 掉先

<img src="2-mysql备份还原mysqldump使用.assets/image-20230719174830630.png" alt="image-20230719174830630" style="zoom:50%;" /> 



由于mysqldump出来的备份文件，你们没有创建库的命令，所以还原之前要手动创建库。

![image-20230719175136730](2-mysql备份还原mysqldump使用.assets/image-20230719175136730.png)



同样由于mysql dump出来的内容里没有创建数据库，所以你手动创建的时候其实可以改个名字的，

![image-20230719181131116](2-mysql备份还原mysqldump使用.assets/image-20230719181131116.png)

进去看一下

<img src="2-mysql备份还原mysqldump使用.assets/image-20230719180254467.png" alt="image-20230719180254467" style="zoom:50%;" />

<img src="2-mysql备份还原mysqldump使用.assets/image-20230719180340044.png" alt="image-20230719180340044" style="zoom:67%;" /> 



1、这种方法备份数据就不太好了。万一原来的db名称你忘了呢。随便写一个，用户那边前端底层sql里面人家还是老的库名，业务肯定恢复不了的。

2、还一个，由于这种mysqldump出来的sql内容里没有涉及库，也就是不知道人家当初创建库的时候是否涉及了一些特性，比如字符集，比如引擎，等。这些都是问题。





## mysqldump --databases | -B

原来是这样👇

![image-20230719181627156](2-mysql备份还原mysqldump使用.assets/image-20230719181627156.png)

现在改成这样👇

![image-20230719181737703](2-mysql备份还原mysqldump使用.assets/image-20230719181737703.png)

再来看备份的文件，此时-B就会有创建数据库的命令了

![image-20230719181834909](2-mysql备份还原mysqldump使用.assets/image-20230719181834909.png)

删库

![image-20230719182028795](2-mysql备份还原mysqldump使用.assets/image-20230719182028795.png)

<img src="2-mysql备份还原mysqldump使用.assets/image-20230719182054440.png" alt="image-20230719182054440" style="zoom:50%;" />



然后恢复一下，此时就不需要手动创建一个可能都不知道的具体参数的数据库了

![image-20230719182250859](2-mysql备份还原mysqldump使用.assets/image-20230719182250859.png)

![image-20230719182333475](2-mysql备份还原mysqldump使用.assets/image-20230719182333475.png)

![image-20230719182351105](2-mysql备份还原mysqldump使用.assets/image-20230719182351105.png)

同样也可以不用cli交互模式，不过-e的方式，无需考虑结尾的分号了;就

![image-20230719182635419](2-mysql备份还原mysqldump使用.assets/image-20230719182635419.png)



### 如果不是drop删库，而是库里的表格丢了，也一样可以用这种方式，

因为mysqldump -rtoot -pXXX -B hellodb里的create语句里是由IF NOT EXISTS条件判断的。

![image-20230719182852320](2-mysql备份还原mysqldump使用.assets/image-20230719182852320.png)



### 备份多个一样是-B

![image-20230719183037926](2-mysql备份还原mysqldump使用.assets/image-20230719183037926.png)

注意上图一样hello mysql其实不加-B就判定未hello库里的mysql表了。

![image-20230719183337051](2-mysql备份还原mysqldump使用.assets/image-20230719183337051.png)

![image-20230719183432639](2-mysql备份还原mysqldump使用.assets/image-20230719183432639.png)



### 再看看-A全备，就是不备份information_schema和performance_schema

![image-20230719184026104](2-mysql备份还原mysqldump使用.assets/image-20230719184026104.png)

果然少了那两个：一个元数据，一个性能

![image-20230719184742018](2-mysql备份还原mysqldump使用.assets/image-20230719184742018.png)

进一步思考-A不会备份information_schema和performance_schema，那么会不会丢失数据呢？

存储过程、触发器是放在mysql这个系统库里的，所以不会丢的。



![image-20230720094222942](2-mysql备份还原mysqldump使用.assets/image-20230720094222942.png)

上图就是说明一下函数、存储过程、触发器这些可能都是跟这个某一个数据库走的，drop database hello;后，里面的函数也就没了。

![image-20230720094751435](2-mysql备份还原mysqldump使用.assets/image-20230720094751435.png)

-A就包含了-E 和-R因为这些东西基本都在mysql库里，包括--triggers触发器也是在-A里就包含了，都是在mysql系统库里。



## mysqldump和binlog

![image-20230720100257409](2-mysql备份还原mysqldump使用.assets/image-20230720100257409.png)

1、夜里2点mysqldump做了全备，第二天18点数据库崩了；

2、二进制是独立的日志记录，是独立于全备的另外一条线索

3、此时用mysqldump的全备文件，只能恢复到夜里2点的数据，还需要结合binglog才能补全恢复数据。

4、此时就需要知道2点全备的时间节点(也不一定是用时间来比对，可能就是记录下全备对应的binlog位置就行了)对应的binlog日志里的位置

5、为了记录下来当时做全备的二进制哪个位置点，就需要mysqldump --master-data这个选项。

![image-20230720101827383](2-mysql备份还原mysqldump使用.assets/image-20230720101827383.png)

--master-data=1和=2的有意义，没有涉及主从复制的时候=2就行了。=1就是加了个change master的命令；=2就是只是一个comment symbol注释而已，不会像=1一样生成一个change master to这个命令。=2是加注释，注释里面记录全备动作对应的binlog的位置。

![image-20230720102344946](2-mysql备份还原mysqldump使用.assets/image-20230720102344946.png)

这样就能在注释里看到binllog的位置position了

![image-20230720102444092](2-mysql备份还原mysqldump使用.assets/image-20230720102444092.png)

-- 两个横线，被注释掉了；这就是--master-data=1的效果，=2就是注释取消--删掉。

![image-20230720102601224](2-mysql备份还原mysqldump使用.assets/image-20230720102601224.png)



![image-20230720102748844](2-mysql备份还原mysqldump使用.assets/image-20230720102748844.png)

![image-20230720102827153](2-mysql备份还原mysqldump使用.assets/image-20230720102827153.png)



所以看到这个MASTER_LOG_POS=2998040;就知道mysqldump全备的动作是对应在binlog里的2998040这个位置的，对应这个位置做了全备，也就是之前的binlog是老日志，之后的就是新日志了。全备要+上从这个位置以后的binlog。



比如现在开始有数据更新了，模拟从这个位置开始以后存在数据变动

![image-20230720103508160](2-mysql备份还原mysqldump使用.assets/image-20230720103508160.png)

数据变动了，binlog就增长了

![image-20230720103616026](2-mysql备份还原mysqldump使用.assets/image-20230720103616026.png)

再加一条

![image-20230720103705847](2-mysql备份还原mysqldump使用.assets/image-20230720103705847.png)

此时二进制日志被我们独立的存放了，也不太担心数据库崩坏。

![image-20230720103930324](2-mysql备份还原mysqldump使用.assets/image-20230720103930324.png)

现在就模拟故障场景

<img src="2-mysql备份还原mysqldump使用.assets/image-20230720100257409.png" alt="image-20230720100257409" style="zoom:30%;" />

有全备，然后又二进制的新增，然后坏了

删库模拟数据库崩了

![image-20230720104148524](2-mysql备份还原mysqldump使用.assets/image-20230720104148524.png)

就只删除mysql/下面的文件，mysql文件夹保留了，这样就无需再手动创建文件夹+修改所有者。

![image-20230720104310505](2-mysql备份还原mysqldump使用.assets/image-20230720104310505.png)

虽然此时数据库没了，但是二进制文件binlog分开放的，还在

还原操作开始

1、重启数据库服务，会生成崭新的数据库文件。

我实验的时候报错了，

![image-20230720104535537](2-mysql备份还原mysqldump使用.assets/image-20230720104535537.png)

![image-20230720105327235](2-mysql备份还原mysqldump使用.assets/image-20230720105327235.png)



![image-20230720105337096](2-mysql备份还原mysqldump使用.assets/image-20230720105337096.png)

我先重启下看看，重启还是不行，修改

![image-20230720105719795](2-mysql备份还原mysqldump使用.assets/image-20230720105719795.png)

没用，还得谷歌

https://stackoverflow.com/questions/60248748/could-not-increase-number-of-max-open-files-to-more-than-4096-request-4214

![image-20230720110045596](2-mysql备份还原mysqldump使用.assets/image-20230720110045596.png)

找到了，AI有时候还是不靠谱

不过，重置mariadb也行的，啊哈哈

改一下

![image-20230720110219134](2-mysql备份还原mysqldump使用.assets/image-20230720110219134.png)



有进展，但还不够



![image-20230720110307192](2-mysql备份还原mysqldump使用.assets/image-20230720110307192.png)



重装算了，操！

yum -y remove mariadb-server

yum -y install mariadb-server

![image-20230720111301229](2-mysql备份还原mysqldump使用.assets/image-20230720111301229.png)

好奇怪啊，重装后，rm -rf /var/lib/mysql/*    后重启立马就起不来了

算了故障演示就不能用rm -rf /var/lib/myqls/*了，就算用这中方式，也只能用重装来弄，否则查这个错太麻烦了

![image-20230720111532047](2-mysql备份还原mysqldump使用.assets/image-20230720111532047.png)

反正重装后，![image-20230720111947286](2-mysql备份还原mysqldump使用.assets/image-20230720111947286.png)

这个值也不用动了。



好了，小插曲，继续恢复数据吧

现在重装mariadb-server后的数据都是光的，开始恢复

![image-20230720112148922](2-mysql备份还原mysqldump使用.assets/image-20230720112148922.png)

### 1、准备好全备文件，binlog文件

完了，我没有备份配置文件，哈哈哈!反面教材，算了手动意思意思吧

![image-20230720112438141](2-mysql备份还原mysqldump使用.assets/image-20230720112438141.png)

### 2、要还原之前假设好数据库一部分用户还在用着，此时需要停服维护

①再一个停服维护的文本里写上这个机器的信息，让相关人员可以看到，其实优化出来就是一个web展示页面

②修改/etc/my.cnf里的选项，补一个skip-networking=1就关闭3306端口

![image-20230720112750552](2-mysql备份还原mysqldump使用.assets/image-20230720112750552.png)

![image-20230720112758977](2-mysql备份还原mysqldump使用.assets/image-20230720112758977.png)

### 3、先把全备文件还原掉

注意，mysql < /data/all.sql  就可以直接还原了，但是这个还原动作 本身也就是大量sql语言，也会记录到binlog里的，而这部分还原操作，一般无需记录到binlog里。

①停止数据库的binlog先

一般生成中binlog已经开了，两个开关，都需要打开，所以关闭binlog就是进入cli交互模式去关一个就行了。之前我是重装了mariadb，所以 binlog没开，我先开了去配置文件里。

![image-20230720113500125](2-mysql备份还原mysqldump使用.assets/image-20230720113500125.png)

![image-20230720113520263](2-mysql备份还原mysqldump使用.assets/image-20230720113520263.png)

OK了，然后，使用cli关闭binlog，导入全备文件

使用 set sql_log_bin=off；就行了，这个变量是会话级的，不会影响别人，且退出失效。

![image-20230720113701144](2-mysql备份还原mysqldump使用.assets/image-20230720113701144.png)

此时binlog就临时禁用了。



此时binlog的位置在

![image-20230720113916128](2-mysql备份还原mysqldump使用.assets/image-20230720113916128.png)

而全备的动作所在binlog的位置在

![image-20230720113954470](2-mysql备份还原mysqldump使用.assets/image-20230720113954470.png)

上图可见是从mariadb-bin.000004这个文件的2990804这个位置之前的binlog有全备。

这个时候的一个规范操作就是，①由于你停服了，所以数据库不会变化了，binlog不新增了②所以此时可以刷新一个binlog，从新的binlog之前到--master-data那个位置就是全备以后需要补充的binlog

![image-20230720114809499](2-mysql备份还原mysqldump使用.assets/image-20230720114809499.png)

所以就是需要从全备补充的binlog涉及这些文件

![image-20230720114925573](2-mysql备份还原mysqldump使用.assets/image-20230720114925573.png)

呵呵，虽然好多就是空的，但是这里是演示嘛，严谨些也是要这么做的。 

关键cli： mysqlbinlog mariadb-bin.000004 >> /data/inc.sql

下图写的有点问题，就是第一个文件它只是精确到了文件，没有精确到位置，所以cli要修改

```
mysqlbinlog --start-postion=2998040 mariadb-bin.000004 > /data/inc.sql
mysqlbinlog mariadb-bin.000005 >> /data/inc.sql
mysqlbinlog mariadb-bin.000006 > /data/inc.sql
mysqlbinlog mariadb-bin.000007 >> /data/inc.sql
mysqlbinlog mariadb-bin.000008 >> /data/inc.sql
mysqlbinlog mariadb-bin.000009 >> /data/inc.sql
mysqlbinlog mariadb-bin.000010 >> /data/inc.sql
mysqlbinlog mariadb-bin.000011 >> /data/inc.sql
mysqlbinlog mariadb-bin.000012 >> /data/inc.sql
```

![image-20230720115224419](2-mysql备份还原mysqldump使用.assets/image-20230720115224419.png)

马上就开始还原了，

![image-20230720115335161](2-mysql备份还原mysqldump使用.assets/image-20230720115335161.png)

all.sql全备+inc.sql里的all.sql里的--master-data的位置到行尾

source /data/all.sql  先还原全备

![image-20230720115548124](2-mysql备份还原mysqldump使用.assets/image-20230720115548124.png)

你看就已经恢复全备的东西了

![image-20230720115615930](2-mysql备份还原mysqldump使用.assets/image-20230720115615930.png)

![image-20230720115632199](2-mysql备份还原mysqldump使用.assets/image-20230720115632199.png)

在还原inc.sql这个binlog补全的

source /data/inc.sql

![image-20230720115739247](2-mysql备份还原mysqldump使用.assets/image-20230720115739247.png)



两个数据就回来了

![image-20230720115827240](2-mysql备份还原mysqldump使用.assets/image-20230720115827240.png)

退出，sql_log_bin就直接恢复成ON啦。或者set sql_log_bin=on;

然后修改/etc/my.cnf里的选项打开对外服务

![image-20230720133220990](2-mysql备份还原mysqldump使用.assets/image-20230720133220990.png)

![image-20230720133248816](2-mysql备份还原mysqldump使用.assets/image-20230720133248816.png)

以上就是数据库的备份和恢复，

不过前提就是，binlog得安全得保存起来，以及全备一样得妥善保存。

1、二进制日志启用

2、msqldup -A --master-data=2 > /data/all.sql

数据修改

insert students (name,age) values ('a',20)

insert students (name,age) values ('b',40)

3、删库 rm -rf /var/lib/mysql/*  模拟故障，实际上可能生成中就是部分业务故障。

4、还原

卸载mariadb-server，重装

4.1、通告维护

4.2、对外停服，修改skip-network=1

4.3、systemc restart mariadb

4.4、mysql > show master logs; 查看当前binlog位置，就是为了取--master-data到这个位置的binlog去还原。记录此位置值

```mysql
mysqlbinlog --start-postion=xxxx mariadb-bin.0000x >> /data/inc.sql
mysqlbinlog mariadb-bin.0000x >> /data/inc.sql
mysqlbinlog mariadb-bin.0000x >> /data/inc.sql
```



5、mysql > set sql_log_bon=off;

5.1、mysql > source /data/all.sql 

5.2、mysql > source /data/inc.sql



6、mysql > set sql_log_bin=on;  开个屁，直接\q就行了，session级退出就没了。

7、恢复配置文件里的skip-networking，或放开防火墙对外。

8、检查数据，确认恢复。





## 如果是人为故障比如删表了

![image-20230720135424202](2-mysql备份还原mysqldump使用.assets/image-20230720135424202.png)

下午2点完全备份

18点老6删表

18点10你开始处理去恢复数据



好了故事讲完了，开始恢复吧

思路和之前一样，就是

①全备文件恢复到2点

②找到binlog，vim进去去掉18点的那个drop table students;这个删表的命令，恩？你问我为什么去掉，你不去掉，待会恢复binlog一样还是删了啊

③恢复的操作就和上面一样了，这样的好处是能将数据恢复到18点10分。

再来一遍实验

![image-20230720145444652](2-mysql备份还原mysqldump使用.assets/image-20230720145444652.png)

看着像靠谱的，就是mysqldump默认是单表事务，但是有外链就要多表一个事务了。👆后文会讲事务下备份。

1、全备-模拟图上2点的全备

![image-20230720144403339](2-mysql备份还原mysqldump使用.assets/image-20230720144403339.png)

![image-20230720145607493](2-mysql备份还原mysqldump使用.assets/image-20230720145607493.png)

2、接着模拟2点以后数据变化，以及删表操作。

![image-20230720145812593](2-mysql备份还原mysqldump使用.assets/image-20230720145812593.png)

模拟18点删表了

![image-20230720150300742](2-mysql备份还原mysqldump使用.assets/image-20230720150300742.png)

模拟后续新的数据变化，你只是删了students表，其他业务可能还是可以正常，比如teacher表的写入。

![image-20230720150544269](2-mysql备份还原mysqldump使用.assets/image-20230720150544269.png)



这个时候你收到故障告警了，或者有人找你，删库的老6电话你了，这个肯定不比告警慢了~

①禁止用户访问，就是skip-networking=1或者iptables deny掉；或者加锁flush tables with read lock；这样不是太好，因为还是能读，数据都被删了，读也有问题了；然后skip-networking=1也不是太好，因为还要重启服务，iptables不需要重启服务，比较方便，其实无所谓。一般都停服维护~

![image-20230720151330754](2-mysql备份还原mysqldump使用.assets/image-20230720151330754.png)

![image-20230720151340089](2-mysql备份还原mysqldump使用.assets/image-20230720151340089.png)

②开始还原

要注意，还原的时候，要去掉18点的drop删表cli，然后就可以还原到18点10分停服的时候了。

去全备的文件里看下--master-data里的binlog位置。

![image-20230720151549971](2-mysql备份还原mysqldump使用.assets/image-20230720151549971.png)

是15编号的344位置做的全备。

也就是从15binlog344位置往后一直到停服的时候

其实停服的时候binlog也要停掉了，不过停服了binlog自然也不会有了，待会还原的时候停掉binlog就行了。

![image-20230720152239701](2-mysql备份还原mysqldump使用.assets/image-20230720152239701.png)

我们可以刷一下日志

![image-20230720152534253](2-mysql备份还原mysqldump使用.assets/image-20230720152534253.png)

就是17以前的，15以后的都是要处理的文件。也就是看起来舒服点的操作。

就是说17这个新binlog，是我们拒绝了别人访问后刷出来的，不应该再有什么新的数据了，不用管了就。

```mysql
mysqlbinlog --start-position=344 /data/logs/logbin/mariadb-bin.000015 > /data/inc.sql

mysqlbinlog /data/logs/logbin/mariadb-bin.000016 >> /data/inc.sql
```



![image-20230720153817937](2-mysql备份还原mysqldump使用.assets/image-20230720153817937.png)

找到drop那一行，删掉，规范点就是先复制一份出来，然后再改。

![image-20230720160547167](2-mysql备份还原mysqldump使用.assets/image-20230720160547167.png)



![image-20230720153906637](2-mysql备份还原mysqldump使用.assets/image-20230720153906637.png)

<img src="2-mysql备份还原mysqldump使用.assets/image-20230720154007111.png" alt="image-20230720154007111" style="zoom:50%;" /> 



关闭binlog

![image-20230720160717589](2-mysql备份还原mysqldump使用.assets/image-20230720160717589.png)

现在有个问题，你全备里面是普遍存在删表后重建操作，

1、你删掉所有库，拿全备去恢复

2、你直接不删所有看，直接用全备覆盖，这个就有一个问题：如果全备动作之后创建了一个表，你拿全备去恢复，它就不会DROP掉，不知道会不会有什么影响。

然后你还有binlog去补差价，到时候这个表就有问题可能，所以可能还是推荐删掉所有的库。

```linux
cp -a /etc/my.cnf /root   备份配置文件
```

![image-20230720161403291](2-mysql备份还原mysqldump使用.assets/image-20230720161403291.png)

![image-20230720161424443](2-mysql备份还原mysqldump使用.assets/image-20230720161424443.png)

![image-20230720161436901](2-mysql备份还原mysqldump使用.assets/image-20230720161436901.png)



```
systemc	restart mariadb
```

![image-20230720161714221](2-mysql备份还原mysqldump使用.assets/image-20230720161714221.png)

17以后都不用管，包括17，都是rm和yum自动生成的。

关掉binlog

![image-20230720161848341](2-mysql备份还原mysqldump使用.assets/image-20230720161848341.png)



```
source /data/all_2023-07-20.sql
```

![image-20230720162108377](2-mysql备份还原mysqldump使用.assets/image-20230720162108377.png)

```
source /data/inc.sql
```

![image-20230720162244632](2-mysql备份还原mysqldump使用.assets/image-20230720162244632.png)

![image-20230720162442124](2-mysql备份还原mysqldump使用.assets/image-20230720162442124.png)



然后就是退出，修改注释配置文件里的skip-networking，对外服务。
