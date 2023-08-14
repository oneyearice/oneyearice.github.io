# 第3节. mysql复制过滤器和基于SSL的复制加密

# 部分数据复制，采用过滤器

两种实现方式：

1、仅写入需要复制的数据库，master节点配置

​		缺点：binlog不仅仅是用来实现主从复制，更多是用来做备份的一个增量还原用的。

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

实际工作中不要使用这种跨库操作就好，注意下，都使用use dbxxx进到某个库里进行操作。









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

```
mkdir /etc/my.cnf.d/ssl  # 专门放证书信息，利用现成的my.cnf.d文件夹，ssl是创建的
cd /etc/my.cnf.d/ssl
```

我们需要3个证书：CA的证书、主服务器证书、从节点证书。

有证书，就得有私钥，

```
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

```
③生成master的私钥和证书申请文件
openssl req -newkey rsa:1024 -days 365 -nodes -keyout master.key > master.csr  # 利用一条命令生成私钥文件master.key，并利用该key生成证书申请文件。

```

![image-20230814165624518](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814165624518.png)

```
④有了证书申请文件，就可以签名了--也就是颁发证书
openssl x509 -req -in master.csr -CA cacert.pem -CAkey cakey.pem -set_serial 01 > master.crt

利用CA的信息，根据证书申请文件，来实现生成证书。
_set_serial 01  指定证书编号？

```

![image-20230814170353681](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814170353681.png)

```
⑤同样再生成的私钥和证书申请文件
openssl req -newkey rsa:1024 -days 365 -nodes -keyout slave.key > slave.csr  # 利用一条命令生成私钥文件slave.key，并利用该key生成证书申请文件。

注意：-days 365 好像是默认就有的。

```

![image-20230814173701688](3-mysql复制过滤器和基于SSL的复制加密.assets/image-20230814173701688.png)



```
⑥同样再给slave节点颁发证书
openssl x509 -req -in slave.csr -CA cacert.pem -CAkey cakey.pem -set_serial 02 > slave.crt

利用CA的信息，根据证书申请文件，来实现生成证书。
_set_serial 01  指定证书编号？
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



