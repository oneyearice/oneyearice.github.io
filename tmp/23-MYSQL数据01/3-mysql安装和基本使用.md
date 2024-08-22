# 第3节. mysql安装和基本使用



### 基本概念

![image-20230313112622651](3-mysql安装和基本使用.assets/image-20230313112622651.png) 

![image-20230313113443370](3-mysql安装和基本使用.assets/image-20230313113443370.png) 





![image-20230313114347774](3-mysql安装和基本使用.assets/image-20230313114347774.png)

数据观察角度

1、物理层：这点是运维关注的，磁盘上怎么保存这些文件，将来性能更好。磁盘文件、分区、LVM、raid

2、逻辑层：数据库里存放的哪些数据，数据和数据之间的关系；表之间的关系咯，一对一，一对多，多对多。

3、视图层：用户看到的结果，比如电商网页上看到的页面上的商品价格，但是看不全，比如进货价用户必然看不到。哈哈

运维人员可能更多是关注 物理层和逻辑层，开发更多关注视图层。



#### ORM，对象关系模型

将面向对象的语言比如PY，和关系型数据库联系在一起

说人话就是，SQL语句你可能就是学一下，后面用python调用库去处理DB的时候都是面向对象的语句，就是比如xxx.value()，这种CLI命令行封装成了python的CLASS调用了。





### Mysql安装

1、yum源安装

一般测试环境要求不高的安装方法

2、源码编译安装

3、二进制安装

企业更多是使用2、3两种方式安装



 

![image-20230313134222246](3-mysql安装和基本使用.assets/image-20230313134222246.png)



yum -y install mariadb-server

rpm -ql mariadb-server

其中

/etc/my.cnf.d/mariadb-server.cnf是配置文件

/var/lib/mysql是数据库存放的路径

/usr/lib/systemd/system/mariadb.service这是服务



 

![image-20230313140007786](3-mysql安装和基本使用.assets/image-20230313140007786.png)



mariadb-server是一个单进程多线程的程序，mysql一样

![image-20230313144424860](3-mysql安装和基本使用.assets/image-20230313144424860.png)

花括号的就是线程



 1、yum源安装特定版本

https://mariadb.org/download/?t=repo-config&d=CentOS+Stream&v=10.11&r_m=neusoft

新建repo文件，复制后，yum repolist看下

![image-20230313181449022](3-mysql安装和基本使用.assets/image-20230313181449022.png)

安装的时候大小写注意下

![image-20230313181817881](3-mysql安装和基本使用.assets/image-20230313181817881.png) 

安装的默认行为，server顺带就安装client了，老的一些lib库会自动升级

![image-20230313182020490](3-mysql安装和基本使用.assets/image-20230313182020490.png)



rpm -ql MariaDB-server 查询注意大小写

client 包也安装好了

![image-20230314093705117](3-mysql安装和基本使用.assets/image-20230314093705117.png) 



之前的版本

![image-20230314094029859](3-mysql安装和基本使用.assets/image-20230314094029859.png)



我的报错

![image-20230314103025106](3-mysql安装和基本使用.assets/image-20230314103025106.png)

上图报错是因为之前安装过并启动过，解决方法就是删除/var/lib/mysql文件夹，重新安装mariadb-server就好了

![image-20230314103443483](3-mysql安装和基本使用.assets/image-20230314103443483.png) 

这样就可以了



可以做安全加固，后补



进入mysql的交互模式后的常用cli

![image-20230314105251856](3-mysql安装和基本使用.assets/image-20230314105251856.png)

查看帮助



查看状态

![image-20230314105636216](3-mysql安装和基本使用.assets/image-20230314105636216.png)

user: root@localhost

mysql的用户账号是带client端主机地址的，所你同一个用户在机器A上root进来，不代表同样一个用户在机器B上root就进得来。

mysql 服务端的连接，走的是socket，一种是tcp/ip，一种是UNIX socket也就是文件形式--这种就是同一台主机上的不同进程之间通信；



现在client和server都是在一台linux上的，所以没必要走tcp/ip网络socket套接字，直接使用文件套接字就行，节省报文开销。

<img src="3-mysql安装和基本使用.assets/image-20230314111527944.png" alt="image-20230314111527944" style="zoom:50%;" /> 

利用该文件实现了两个进程的通信，client和server

其实上文的mysql回车就是没有跨网络就是走的文件socket的

<img src="3-mysql安装和基本使用.assets/image-20230314111929355.png" alt="image-20230314111929355" style="zoom:50%;" /> 

none就是进来的时候不在任何一个DB里的

默认安装mysql后，默认就自动创建多个数据库。

数据库的概念 和 实例instance 的概念



![image-20230314112321566](3-mysql安装和基本使用.assets/image-20230314112321566.png)

mysql  Ver 15.1 Distrib 10.11.2-MariaDB, for Linux (x86_64) using  EditLine wrapper

10.11.2就是版本，不同版本安装在同一个linux就是实例了；可以一台机器上安装多个mysql的版本，错开监听端口，这就叫多实例；一般这样用的不太多，测试环境中使用下。mysql属于比较重的应用 数据的访问压力较大，所以不会使用多实例。



![image-20230314112953767](3-mysql安装和基本使用.assets/image-20230314112953767.png)

这就是默认安装后生成的数据库

<img src="3-mysql安装和基本使用.assets/image-20230314113112049.png" alt="image-20230314113112049" style="zoom:50%;" /> 

然后msyql这个默认的库里有哪些表

![image-20230314113218554](3-mysql安装和基本使用.assets/image-20230314113218554.png) 

同样在cli里看

<img src="3-mysql安装和基本使用.assets/image-20230314113312240.png" alt="image-20230314113312240" style="zoom:50%;" /> 

mysql是系统数据库。

用户后面自建创建的数据库，也会在/var/lib/mysql里生成自己的文件夹。

可以认为msyql里所谓数据库就是文件夹，然后在文件夹里有若干个表。



看下表的文件，每个表里有3个文件

比如视频版本里的

![image-20230314113850908](3-mysql安装和基本使用.assets/image-20230314113850908.png)

user.MYD就是MyISAM的引擎

user.MYI就是innoDB的引擎



现在的版本好像没有这么多引擎了

![image-20230314114021219](3-mysql安装和基本使用.assets/image-20230314114021219.png) 





cli命令行 分为client端cli和server端cli

刚才myql进入的\h看到的就是client命令，是没show的，show是server cli

![image-20230314114651739](3-mysql安装和基本使用.assets/image-20230314114651739.png)

show 这个命令，client上是没有的，是client端发送给server，在server上执行的。

注意：客户端命令，只能是用mysql-cllient工具也就是mysql命令进入才会有的，换个客户端工具就没有上图这些命令。

而服务端命令，不管是什么客户端，只要能连到server上来，就能执行的。



数据库的命令和客户端的命令 表现形式上一个很大的不同，就是行尾带不带分号

server cli要求加分号

<img src="3-mysql安装和基本使用.assets/image-20230314115117174.png" alt="image-20230314115117174" style="zoom:50%;" /> 

而客户端命令比如status不用加分号![image-20230314115207278](3-mysql安装和基本使用.assets/image-20230314115207278.png) 



然后通过show databases;可见有5个默认库，但是在/var/lib/mysql里只有4个，少了一个information_schema库文件

<img src="3-mysql安装和基本使用.assets/image-20230314115331377.png" alt="image-20230314115331377" style="zoom:50%;" /> 

![image-20230314115355294](3-mysql安装和基本使用.assets/image-20230314115355294.png) 

然后比早期版本多了一个sys库

information_schema不是磁盘文件数据库类型，是内存中的。

test 测试的，可删

performance_scheme放了数据库性能相关的数据





进一步学习如何查看当前有哪些用户，

通过上文知道在/var/lib/mysql/mysq/下面有一个user.frm表文件，可以进入数据库里查看该表内容

这个表列特别多，不可能一下都展现出来。需要赛选

不进入的查表:

![image-20230314120248698](3-mysql安装和基本使用.assets/image-20230314120248698.png)

进入数据库好比cd进入文件夹

![image-20230314120409180](3-mysql安装和基本使用.assets/image-20230314120409180.png) 

![image-20230314120517645](3-mysql安装和基本使用.assets/image-20230314120517645.png)

![image-20230314120529243](3-mysql安装和基本使用.assets/image-20230314120529243.png) 

其中就有user表



查看表结构和列

除了select * from user;以外还可以看有哪些列：

![image-20230314120626272](3-mysql安装和基本使用.assets/image-20230314120626272.png) 

Field就是域，就是user表的列，就是属性

Null里的NO，就是不允许为空

PRI就是主键，而且如果两个都是PRI就是复合组件，低版本的mariadb里还是看得到的

![image-20230314121012652](3-mysql安装和基本使用.assets/image-20230314121012652.png) 

host+user组合起来成为了一个主键。





<img src="3-mysql安装和基本使用.assets/image-20230314121135036.png" alt="image-20230314121135036" style="zoom:50%;" /> 

版本差异看下也

<img src="3-mysql安装和基本使用.assets/image-20230314121206102.png" alt="image-20230314121206102" style="zoom:50%;" /> 

只看3个字段，的专业术语叫做投影，哈哈了解一下

<img src="3-mysql安装和基本使用.assets/image-20230314121245523.png" alt="image-20230314121245523" style="zoom:33%;" /> 

所以python打开文件，读取文件，设计open，close就可能没有db来的专业方便。





 







