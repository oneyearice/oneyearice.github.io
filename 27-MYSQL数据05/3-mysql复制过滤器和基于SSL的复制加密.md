# 第3节. mysql复制过滤器和基于SSL的复制加密

# 部分数据复制，采用过滤器

两种实现方式：

1、仅写入需要复制的数据库，master节点配置

​		缺点：binlog不仅仅是用来实现主从复制，更多是用来做备份的一个增量还原用的。这样肯定不行，你本地binlog都不全了。

2、slave节点配置，在rely-log中继日志中挑取出，哪些特定数据库的特定表需要同步到本地。

​		缺点：rely-log中继日志里东西是全的，是从master复制过来了，然后再sql thread进行挑选的，所以已经从master复制过来，意味着网络和磁盘IO就花费出去了。

​		注意slave过滤需要在所有的slave上都要配置一边。





## 方法1：只生产特定binlog

```
master上：
-----------
vim /etc/my.cnf
  server-id=1
  log-bin
  # binlog-do-db = 
  binlog-ignore-db = hellodb
wr
systemctl restart mariadb

mysql > show master status;

可见binlog黑名单数据库👇
```

![image-20230814135651615](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814135651615.png)

此时hellodb的binlog都不会生产，自然就不会参与主从复制了，测试👇

![image-20230814135929346](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814135929346.png)

slave那边test003还在，确实不同步了

![image-20230814135954341](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814135954341.png)



其他数据还是同步的

![image-20230814140251345](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814140251345.png)

![image-20230814140303255](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814140303255.png)

此时就实现了数据的一个过滤过程





## 方法2：slave服务器上进行过滤

<img src="3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814140849855.png" alt="image-20230814140849855" style="zoom:40%;" />

```
slave上：
-------
vim /etc/my.cnf
  replicate-do-db=hellodb,db1  # 这里定义了一个白名单只从中继日志里取出helldb,db1两个库的binlog进行sql thread写入slave本地。
wr
systemctl restart mariadb


```

此时没有加白的就不会复制了👇

![image-20230814142853544](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814142853544.png)



![image-20230814142904220](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814142904220.png)

加白的就继续复制

![image-20230814143546438](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814143546438.png)



![image-20230814143602805](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814143602805.png)



![image-20230814143610666](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814143610666.png)

上图的"hellodb,db1"呈现出来的貌似是两个db用逗号分开，其实不一定的，要去my.cnf里看如果是replicate-do-db=hellodb,db1这种写法，系统可能就判定为一个叫"hellodb,db1"数据库了，鬼知道；如果是下面的写法，系统就虽然slave status里显示还是一样"hellodb,db1"，但是确实是逗号分开的两个数据库了。

结果就是加白的也没有复制，但是slave 状态时OK的啊，奇怪了。

就是写的格式有问题，不支持hellodb,db1这种逗号的方式，一行一个，

<img src="3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814144250082.png" alt="image-20230814144250082" style="zoom:50%;" />

此时slave的状态倒时用逗号分开的，貌似和以前一样，但事实上时两个db了，以前时一个db

![image-20230814144356127](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814144356127.png)





![image-20230814144044470](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814144044470.png)

此时test在slave那边就确实同步 也删掉了

![image-20230814144115664](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814144115664.png)



再看一个现象--就是目前的过滤器，必须切到use db进到哪个库里才能同步👇

![image-20230814150026822](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814150026822.png)

![image-20230814150107585](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814150107585.png)

你看master在none下操作的，没有use进到对应的库，所以就没有同步，下面对比测试use一下就同步了👇

![image-20230814150246283](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814150246283.png)

![image-20230814150253935](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814150253935.png)

实际工作中不要使用这种跨库操作就好，<font color=red>注意下，都使用use dbxxx进到某个库里进行操作</font>。





# 数据主从复制安全加密下



mysql -uroot -pxxxx -h xxxx这种是明文的，抓包可得

![image-20230814153425676](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814153425676.png)



![image-20230814153437885](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814153437885.png)

可见cli和查看的结果数据都能看得到。





## 利用SSL加密，就是要有SSL证书

实验，在192.168.126.129和192.168.126.130之间实现加密

证书怎么申请，

证书是用来证明公钥的安全性的，每个服务器都有自己的证书，将来才能利用证书交换彼此的公钥，才可以进一笔利用公钥来加密对称密钥，进而实现数据传输的安全。



mysql的ssl是做了双向认证的，和平时https上网-只做服务器的认证，不太一样。



1、ca自己给自己版本证书：生产私钥，生产自签名证书；

2、然后在颁发证书

3、这次做都在一台机器上做完，包括CA、各个节点的证书生成，然后分发下去。

-----------------

```shell
mkdir /etc/my.cnf.d/ssl  # 专门放证书信息，利用现成的my.cnf.d文件夹，ssl是创建的
cd /etc/my.cnf.d/ssl
```

我们需要3个证书：CA的证书、主服务器证书、从节点证书。

有证书，就得有私钥，

```shell
①生成CA的私钥：
openssl genrsa 2048 > cakey.pem    # 以前是专门一个目录，现在简单放一起就行
可能最好加个密，或者改个cakey.pem的权限，安全些。
```

![image-20230814163025210](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814163025210.png)

```
②利用私钥生成自签名证书
openssl req -new -x509 -key cakey.pem -out cacert.pem -days 3650

```

![image-20230814164039232](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814164039232.png)

![image-20230814164202761](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814164202761.png)

```shell
③生成master的私钥和证书申请文件
openssl req -newkey rsa:1024 -days 365 -nodes -keyout master.key > master.csr  # 利用一条命令生成私钥文件master.key，并利用该key生成证书申请文件。

注意！1024得改成2048，否则mysql起不来。可能是之前用得2048的CA私钥吧。
```

![image-20230814165624518](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814165624518.png)

```
④有了证书申请文件，就可以签名了--也就是颁发证书
openssl x509 -req -in master.csr -CA cacert.pem -CAkey cakey.pem -set_serial 01 > master.crt

利用CA的信息，根据证书申请文件，来实现生成证书。
_set_serial 01  指定证书编号？

```

![image-20230814170353681](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814170353681.png)

```shell
⑤同样再生成slave的私钥和证书申请文件
openssl req -newkey rsa:2048 -days 365 -nodes -keyout slave.key > slave.csr  # 利用一条命令生成私钥文件slave.key，并利用该key生成证书申请文件。

注意：-days 365 好像是默认就有的。

```

![image-20230814173701688](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814173701688.png)



```shell
⑥同样再给slave节点颁发证书
openssl x509 -req -in slave.csr -CA cacert.pem -CAkey cakey.pem -set_serial 02 > slave.crt

利用CA的信息，根据证书申请文件，来实现生成证书。
_set_serial 02  指定证书编号
```

![image-20230814173838129](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814173838129.png)



到此就有了master和slave的证书和私钥了，就可以利用证书来加密了

## 开始给mysql配置ssl证书

默认是未开启ssl加密的

![image-20230814174726652](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814174726652.png)

一些和加密相关的变量

![image-20230814174952359](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814174952359.png)

题外话：%ssl%不区分大小写。

需要指ssl_ca  CA的证书；ssl_cert自己的证书，ssl_key自己的私钥。

```
vim /etc/my.cnf
  ssl-ca=/etc/my.cnf.d/ssl/cacert.pem
  ssl-cert=/etc/my.cnf.d/ssl/matser.crt
  ssl-key=/etc/my.cnf.d/ssl/master.key
  
```

![image-20230814180628985](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814180628985.png)

但是服务起不来

![image-20230814180650916](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814180650916.png)

查看报错日志

![image-20230814180710663](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814180710663.png)

发现是SSL error: Unable to get certificate from '/etc/my.cnf.d/ssl/master.crt'

可能是之前的master.crt证书文件生成的有问题。

一般是文件权限问题，但是这里不是，需要修改一下相关命令。

https://stackoverflow.com/questions/42145925/mariadb-over-ssl-not-working-certificate-verify-failed

👆这篇是OK的，只不过sha1要去掉就行了，我怀疑只要将上面的cli里的后缀改成pem就行了，试试看。不是，破案了👇，还是用上面的原cli改成2048跑一遍再。



下面对比排障测试：

```shell
这段NG：
-----------------
openssl genrsa 2048 > cakey.pem
openssl req -new -x509 -key cakey.pem -out cacert.pem -days 3650
openssl req -newkey rsa:1024 -days 365 -nodes -keyout master-key.pem > master-req.pem  # 1024 改成2048就OK了

openssl x509 -req -in master-req.pem -CA cacert.pem -CAkey cakey.pem -set_serial 01 > master-cert.pem


```

```shell
这段OK：
-----------------
openssl genrsa 2048 > cakey.pem
openssl req -new -x509 -nodes -days 3650 -key cakey.pem > cacert.pem
openssl req -newkey rsa:2048 -days 730 -nodes -keyout master-key.pem > master-req.pem

openssl x509 -req -in master-req.pem -days 730  -CA cacert.pem -CAkey cakey.pem -set_serial 01 > master-cert.pem

chown mysql.mysql *

```

如果常规就是起个server名字👇

```
openssl genrsa 2048 > ca-key.pem
openssl req -new -x509 -nodes -days 3650 -key ca-key.pem > ca-cert.pem
openssl req -newkey rsa:2048 -days 730 -nodes -keyout server-key.pem > server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 730  -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 > server-cert.pem
chown mysql.mysql *

```



ssl路径配置，重启服务OK后，此时status和show variables like '%ssl%';就可见

![image-20230815111527667](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230815111527667.png)

通过SSL相关变量可见 两个都YES了，也有了相关路径👇

<img src="3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230815111544230.png" alt="image-20230815111544230" style="zoom:50%;" />



此时就把ssl的配置都弄了，但是还没有启用加密方式连接。瞎说，上上图status可见

![image-20230906104226492](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906104226492.png)

说明就已经启动了，至于有的低版本还需要这样启用，那是低版本太low。

不管是本地登入自己还是别人远程登入自己，还是本地用socket或是tcp套接字登入自己都是自动的启用了SSL的，status 可见SSL Cipher in use is TLS_AES_256_GCM_SHA384的。

![image-20230906105636989](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906105636989.png)



低版本才需要链接的时候调用配置里的ssl证书和key

```

mysql --ssl-ca=cacert.pem --ssl-cert=master.crt --ssl-key=master.key

```

![image-20230906113532587](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906113532587.png)

![image-20230906103846535](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906103846535.png)

然后强制某个账号必须使用ssl，高版本默认就是会用ssl，应该这也是低版本才需要的操作

```mysql
grant replication slave on *.* to repluser2@'192.168.%.%' identified by 'cisco' require ssl;
```

![image-20230906112456314](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906112456314.png)

此时使用repluser2 低版本就需要加上ssl选项，高版本不需要

![image-20230906112726708](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906112726708.png)





进一步实现主从复制用SSL，将之前在master上一并产生的slave的证书也复制到slave上

![image-20230906110126468](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906110126468.png)

其实用这个三个文件就行了

![image-20230906110204369](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906110204369.png)



<img src="3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906110347724.png" alt="image-20230906110347724" style="zoom:50%;" /> 

但是slave这样起不来！

查看报错，说是SSL路径不对，

![image-20230906114502207](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906114502207.png)

其实是权限没有，拿不到ssl文件

![image-20230906114423046](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906114423046.png)



![image-20230906114727784](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906114727784.png)

起来了，但是有错误

这个报错是主从复制的报错，

![image-20230906115223657](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906115223657.png)

也就是不同步的报错，按之前的处理方法试试

SLAVE IO都没有Yes啊，解决思路有2

```
1、重新从master dump 出来一份，记得带--master-date=1选项，导入后，启动slave即可
2、在master上看看当前的binlog 位置，直接在slave上stop; reset slave all; 充型配置同步信息CHANGE TO 。。。。;在start slave试试，差不多就能同步，当然这里推荐用1而不是2.因为这是你从最新的位置复制过来的，前面很多数据都没有同步，而且2还不一定成功，可能报错👇如下图，不过只要按提示改成'mariadb-bin.000003' at 4 也能同步，呵呵，实验就无所谓了，同步就行了。
```

![image-20230906154130117](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906154130117.png)

```
CHANGE MASTER TO  MASTER_HOST='192.168.126.129',  MASTER_USER='repluser',  MASTER_PASSWORD='cisco',  MASTER_PORT=3306,  MASTER_CONNECT_RETRY=10, MASTER_LOG_FILE='mariadb-bin.000003', MASTER_LOG_POS=4;
```

![image-20230906153758203](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906153758203.png)

此时搞同步了就，但是SSL还没有在主从同步环节启用，默认配置了ssl在/etc/my.cnf里只是登入(远程本地也好，socker/tcp也好)默认使用ssl。

```
Slave服务器配置
mysql>
CHANGE MASTER TO  MASTER_HOST='MASTERIP',  MASTER_USER='rep',  MASTER_PASSWORD='centos',
MASTER_LOG_FILE='mariadb-bin.0000xx',  MASTER_LOG_POS=xx,  #这里写master的
MASTER_SSL=1,
MASTER_SSL_CA = '/etc/my.cnf.d/ssl/cacert.pem',  
MASTER_SSL_CERT = '/etc/my.cnf.d/ssl/slave.crt',  
MASTER_SSL_KEY = '/etc/my.cnf.d/ssl/slave.key';
```

```
CHANGE MASTER TO  MASTER_HOST='192.168.126.129',  MASTER_USER='repluser',  MASTER_PASSWORD='cisco',
MASTER_LOG_FILE='mariadb-bin.000023',  MASTER_LOG_POS=691,
MASTER_SSL=1,
MASTER_SSL_CA = '/etc/my.cnf.d/ssl/cacert.pem',  
MASTER_SSL_CERT = '/etc/my.cnf.d/ssl/slave.crt',  
MASTER_SSL_KEY = '/etc/my.cnf.d/ssl/slave.key';
```

![image-20230906155554764](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906155554764.png)



![image-20230906155656281](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906155656281.png)



我看别人演示里不写三个ssl文件路径唉，我试试

```
CHANGE MASTER TO  MASTER_HOST='192.168.126.129',  MASTER_USER='repluser',  
MASTER_PASSWORD='cisco',
MASTER_LOG_FILE='mariadb-bin.000023',  
MASTER_LOG_POS=691,
MASTER_SSL=1;
```

结果一样的报错，而且，人家会自动去找到三个ssl文件路径并给你显示出来的--这是因为第一次配置了，这点和老版本不一样。放屁！stop slave; reset slave all;exit systemctl restart mariadb就好了，不用配置证书路径！



![image-20230906160302206](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906160302206.png)



尝试去掉ssl，发现去不掉👇，reset slave all;后并未配置ssl，也为启用，但是就启用了YES

![image-20230906162023615](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906162023615.png)

手动写MASTER_SSL=0来关闭，

![image-20230906162126405](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906162126405.png)start slave一下发现报错变了

![image-20230906162155000](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906162155000.png)

问问GPT

![image-20230906162213129](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906162213129.png)

实测操作，有效

![image-20230906162258650](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906162258650.png)

![image-20230906162316647](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906162316647.png)



此时在开启SSL看看

```
Last_IO_Error: error connecting to master 'repluser@192.168.126.129:3306' - retry-time: 10  maximum-retries: 100000  message: SSL connection error: error:00000000:lib(0)::reason(0)

还是报这个错，
master上敲mariadb-admin flush-hosts也没有用！

```

##### 如何清空ssl文件路径，需要重启服务

stop slave; reset slave all;exit systemctl restart mariadb就好了，不用配置证书路径！

![image-20230906163449853](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906163449853.png)

![image-20230906163704230](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906163704230.png)

你要说用户repluser2是只能ssl的，

![image-20230906163748263](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906163748263.png)

没事，换，，改成repluser也一样，至少这里repluser2 成功就说明已经使用了ssl加密复制了。

![image-20230906163907483](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906163907483.png)



那么问题来了，要不要配置，如果配置了是否应该配置master而不是slave呢，试试

不行，一样报错，只要配置ssl三个文件的路径，就有问题

![image-20230906164032406](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906164032406.png)



![image-20230906164827162](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906164827162.png)

这样就实现了带ssl的主从复制，上图hellodb002复制OK，001没有，是因为之前有个白名单

去掉就好了👇

![image-20230906165005320](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906165005320.png)





再次尝试一下

去掉配置里的ssl路径，改为配置slave的时候指定ssl文件

![image-20230906181110098](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230906181110098.png)

```
CHANGE MASTER TO  MASTER_HOST='192.168.126.129',  MASTER_USER='repluser',  MASTER_PASSWORD='cisco',
MASTER_LOG_FILE='mariadb-bin.000023',  MASTER_LOG_POS=691,
MASTER_SSL=1,
MASTER_SSL_CA = '/etc/my.cnf.d/ssl/cacert.pem',  
MASTER_SSL_CERT = '/etc/my.cnf.d/ssl/master.crt',  
MASTER_SSL_KEY = '/etc/my.cnf.d/ssl/master.key';

不管是master的ssl证书还是slave的证书都不行，报错一样。算了，就在/etc/my.cnf里配置就行了，别在slave里配置ssl文件了。

CHANGE MASTER TO  MASTER_HOST='192.168.126.129',  MASTER_USER='repluser',  MASTER_PASSWORD='cisco',
MASTER_LOG_FILE='mariadb-bin.000023',  MASTER_LOG_POS=691,
MASTER_SSL=1,
MASTER_SSL_CA = '/etc/my.cnf.d/ssl/cacert.pem',  
MASTER_SSL_CERT = '/etc/my.cnf.d/ssl/slave.crt',  
MASTER_SSL_KEY = '/etc/my.cnf.d/ssl/slave.key';
```



