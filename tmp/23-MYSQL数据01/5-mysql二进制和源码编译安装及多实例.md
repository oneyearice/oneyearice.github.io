# 第5节. mysql二进制和源码编译安装及多实例



### 二进制安装

<img src="5-mysql二进制和源码编译安装及多实例.assets/image-20230316110539246.png" alt="image-20230316110539246" style="zoom:40%;" /> 



MSI格式点击安装就好

![image-20230315103736071](5-mysql二进制和源码编译安装及多实例.assets/image-20230315103736071.png) 



#### 一、创建账号以及数据库的目录

确定没有使用该账号，再创建

```
(1) 准备用户
groupadd -r -g 306 mysql  # 没必要，删掉这句，本身useradd -r 就会创建同名组，id都是<1000的系统ID
useradd -r –d /data/mysql mysql  # 没必要-d /data/mysql ，因为-r本身也不会创建家目录写了也白写
(2) 准备数据目录，建议使用逻辑卷
mkdir /data/mysql
chown mysql:mysql /data/mysql

```

<img src="5-mysql二进制和源码编译安装及多实例.assets/image-20230315104501402.png" alt="image-20230315104501402" style="zoom:50%;" /> 

-r系统用户了，-d就不会创建家目录了，所以没必要-d带上目录，

所以创建账号优化为

```
useradd -r mysql
mkdir /data/mysql
chown mysql.mysql /data/mysql
```

或者

```
useradd -r -s /sbin/nologin -m -d /data/mysql mysql  # -m就是-r的情况下不会-d所以就-m
cd /data/mysql
rm -rf /data/mysql/.*  # 删除家目录里的隐藏文件，否则会有问题

```

![image-20230315110417550](5-mysql二进制和源码编译安装及多实例.assets/image-20230315110417550.png) 



然后这里强调下 useradd -s /bin/false 和/sbin/nologin一个意思，不过false的本质还是

<img src="5-mysql二进制和源码编译安装及多实例.assets/image-20230907164449447.png" alt="image-20230907164449447" style="zoom:50%;" /> 

你给了用户/bin/false 的shell后，用户登入使用这个/bin/false后发现返回的是false是错误，于是就不会继续登入了。





#### 二、解压二进制和创建软连接以及文件权限

注意/usr/local路径是固定的，当初官方编译的时候指定的

![image-20230315110614365](5-mysql二进制和源码编译安装及多实例.assets/image-20230315110614365.png)

解压后就得到带版本号的文件夹，大小从600M+变成了1.5G

![image-20230315110808010](5-mysql二进制和源码编译安装及多实例.assets/image-20230315110808010.png)

但是人家编译的时候的路径是/usr/local/mysql，所以创建一个软连接

![image-20230315110939922](5-mysql二进制和源码编译安装及多实例.assets/image-20230315110939922.png)

权限的问题

![image-20230315111441349](5-mysql二进制和源码编译安装及多实例.assets/image-20230315111441349.png)

![image-20230315111554390](5-mysql二进制和源码编译安装及多实例.assets/image-20230315111554390.png) 

![image-20230315111615945](5-mysql二进制和源码编译安装及多实例.assets/image-20230315111615945.png)

解释：加不加斜线

1、chown -R root.root mysql   # msql是软连接文件，-R也是不认为是个文件夹，不存在递归、

2、chown -R root.root mysql/  # 这样就会判定为文件夹，-R就会做递归了

这里就完成了二进制文件的解压，得到了一堆mysql程序。然后考虑使用这些二进制文件生成DB文件。



#### 三、生成数据库文件

现在/data/mysql里是空的

![image-20230315112151485](5-mysql二进制和源码编译安装及多实例.assets/image-20230315112151485.png) 

![image-20230315112230093](5-mysql二进制和源码编译安装及多实例.assets/image-20230315112230093.png) 

关注这两个文件夹

![image-20230315112349674](5-mysql二进制和源码编译安装及多实例.assets/image-20230315112349674.png) 

/usr/local/msyql/scripts里的mysql_install_db是用来创建数据库文件



创建的时候要指明，在哪个目录下生成DB文件，并这些文件属于哪个用户

执行的时候报错了

![image-20230315113455024](5-mysql二进制和源码编译安装及多实例.assets/image-20230315113455024.png) 



![image-20230315113731171](5-mysql二进制和源码编译安装及多实例.assets/image-20230315113731171.png) 

解决方法是到/usr/local/mysql下使用相对路径执行

![image-20230315114030148](5-mysql二进制和源码编译安装及多实例.assets/image-20230315114030148.png) 

![image-20230315134035600](5-mysql二进制和源码编译安装及多实例.assets/image-20230315134035600.png)

这就是安全加固的指令👆运行完后，就生成了DB文件👇

![image-20230315134119983](5-mysql二进制和源码编译安装及多实例.assets/image-20230315134119983.png)



配置文件一般系统自带就有，也要看什么系统呢，如果是最小化rocky-linux就没有

![image-20230315135915680](5-mysql二进制和源码编译安装及多实例.assets/image-20230315135915680.png) 

可见该/etc/my.cnf文件不是二进制安装得来的。



#### 四、从二进制安装包里找到配置文件推荐模板复制到常用配置文件路径下

![image-20230315140139209](5-mysql二进制和源码编译安装及多实例.assets/image-20230315140139209.png)

![image-20230315140243332](5-mysql二进制和源码编译安装及多实例.assets/image-20230315140243332.png) 

![image-20230315140334239](5-mysql二进制和源码编译安装及多实例.assets/image-20230315140334239.png) 

这些参数就是老文件保留下来的，现在不会给这么小的内存。

![image-20230315140652697](5-mysql二进制和源码编译安装及多实例.assets/image-20230315140652697.png) 

和原来的/etc/my.cnf区别开来，放到/etc/mysql/my.cnf👆

修改/etc/mysql/my.cnf里的datadir为之前指定的/data/mysql，其他不用动

<img src="5-mysql二进制和源码编译安装及多实例.assets/image-20230315140956584.png" alt="image-20230315140956584" style="zoom:60%;" /> 



####  五、启动服务

由于是二进制安装的，启动靠脚本

![image-20230315141348789](5-mysql二进制和源码编译安装及多实例.assets/image-20230315141348789.png)



脚本里有start status stop这些动作的

![image-20230315141324297](5-mysql二进制和源码编译安装及多实例.assets/image-20230315141324297.png) 



![image-20230315141525647](5-mysql二进制和源码编译安装及多实例.assets/image-20230315141525647.png) 

![image-20230315141633402](5-mysql二进制和源码编译安装及多实例.assets/image-20230315141633402.png) 

设置为开机启动了已经👆

然后启动服务

![image-20230315141740330](5-mysql二进制和源码编译安装及多实例.assets/image-20230315141740330.png)





#### 六、客户端连接



修改一下PATH路径，便于使用mysql命令

![image-20230315141842822](5-mysql二进制和源码编译安装及多实例.assets/image-20230315141842822.png)

![image-20230315142058883](5-mysql二进制和源码编译安装及多实例.assets/image-20230315142058883.png) 

PATH变量写到独立的/etc/profile.d/mysql.sh里

![image-20230315142200340](5-mysql二进制和源码编译安装及多实例.assets/image-20230315142200340.png) 



此时就可以连接mysqld了

![image-20230315142221195](5-mysql二进制和源码编译安装及多实例.assets/image-20230315142221195.png)



#### 七、开头提到的家目录里的隐藏文件的问题

如果当初创建用户使用-m -d生成了家目录来作为DB文件路径，然后又没有删除里面的隐藏文件夹，就会遇到问题

![image-20230315142613253](5-mysql二进制和源码编译安装及多实例.assets/image-20230315142613253.png)

删掉就好了

![image-20230315142653243](5-mysql二进制和源码编译安装及多实例.assets/image-20230315142653243.png) 

![image-20230315142925485](5-mysql二进制和源码编译安装及多实例.assets/image-20230315142925485.png)

system是在mysql交互界面里调用linux系统命令👆，当然这里仅仅是演示知识点，工作中千万不要这么操作，太危险了，数据库文件都在那呢。

```shell
rm -rf .[^.]* 这是删除隐藏文件的准确方法。	
```

![image-20230315143250509](5-mysql二进制和源码编译安装及多实例.assets/image-20230315143250509.png)



#### 八、安全加固

![image-20230315143452748](5-mysql二进制和源码编译安装及多实例.assets/image-20230315143452748.png) 

![image-20230315143555409](5-mysql二进制和源码编译安装及多实例.assets/image-20230315143555409.png) 

以上就是二进制安装方法的详情👆



### 源码编译安装

考虑到数据库后面可能需要扩容，所以这次使用逻辑卷。

##### 分区

![image-20230315144555766](5-mysql二进制和源码编译安装及多实例.assets/image-20230315144555766.png)![image-20230315144629573](5-mysql二进制和源码编译安装及多实例.assets/image-20230315144629573.png) 

![image-20230315145443643](5-mysql二进制和源码编译安装及多实例.assets/image-20230315145443643.png)

![image-20230315145601333](5-mysql二进制和源码编译安装及多实例.assets/image-20230315145601333.png) 

注意上图👆error 需要同步一下

![image-20230315145758854](5-mysql二进制和源码编译安装及多实例.assets/image-20230315145758854.png) 



#### 创建物理卷并加入卷组

加组的时候，指定单个PE大小为16M，将来空间就是16M，16M地分出去的。

![image-20230316095357311](5-mysql二进制和源码编译安装及多实例.assets/image-20230316095357311.png) 



#### 创建逻辑卷

磁盘空间用光100%free

![image-20230316101035597](5-mysql二进制和源码编译安装及多实例.assets/image-20230316101035597.png) 



#### 挂载文件夹

创建文件夹，记得格式化LVM

![image-20230316101220484](5-mysql二进制和源码编译安装及多实例.assets/image-20230316101220484.png) 

![image-20230316101412040](5-mysql二进制和源码编译安装及多实例.assets/image-20230316101412040.png) 

持续挂载👆



查看确认下

![image-20230316101449574](5-mysql二进制和源码编译安装及多实例.assets/image-20230316101449574.png) 



#### 修改文件夹属性

创建账号先

![image-20230316101620933](5-mysql二进制和源码编译安装及多实例.assets/image-20230316101620933.png) 



#### -------------开始源码编译------------

#### **安装依赖包👆**

```shell
yum install bison bison-devel zlib-devel libcurl-devel libarchive-devel boost-  devel gcc gcc-c++ cmake ncurses-devel gnutls-devel libxml2-devel openssl-  devel libevent-devel libaio-devel
```



#### 解压缩源码压缩包并进行编译

![image-20230316101950830](5-mysql二进制和源码编译安装及多实例.assets/image-20230316101950830.png) 

![image-20230316102004655](5-mysql二进制和源码编译安装及多实例.assets/image-20230316102004655.png) 

![image-20230316102022180](5-mysql二进制和源码编译安装及多实例.assets/image-20230316102022180.png)

编译采用cmake而不是configure预编译，记得CPU给够，否则待会make编译的时候耗时更长；

```shell
cd mariadb-10.2.18/  cmake . \
-DCMAKE_INSTALL_PREFIX=/app/mysql \           # 这个是二进制程序安装路径
-DMYSQL_DATADIR=/data/mysql/ \                # 这是数据库文件存放路径
-DSYSCONFDIR=/etc/ \
-DMYSQL_USER=mysql \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITHOUT_MROONGA_STORAGE_ENGINE=1 \
-DWITH_DEBUG=0 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_UNIX_ADDR=/data/mysql/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
make && make install
```

提示：如果出错，执行rm -f CMakeCache.txt

```shell
make -j 16 && make install && echo -e '\a' && date
```

![image-20230316103020251](5-mysql二进制和源码编译安装及多实例.assets/image-20230316103020251.png)

#### 后续安装和二进制安装一样👇

```
准备环境变量
echo 'PATH=/app/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
.	/etc/profile.d/mysql.sh
生成数据库文件 cd	/app/mysql/
scripts/mysql_install_db --datadir=/data/mysql/ --user=mysql
准备配置文件
cp  /app/mysql/support-files/my-huge.cnf	/etc/my.cnf
[mysqld]中添加三个选项：  
datadir = /data/mysql  
innodb_file_per_table = on
skip_name_resolve = on	禁止主机名解析，建议使用

准备启动脚本
cp /app/mysql/support-files/mysql.server /etc/init.d/mysqld
启动服务
chkconfig --add mysqld ;service mysqld start

```

记录编译安装遇到的问题

![image-20230316140057953](5-mysql二进制和源码编译安装及多实例.assets/image-20230316140057953.png) 

![image-20230331101324556](5-mysql二进制和源码编译安装及多实例.assets/image-20230331101324556.png)



### 多实例mysql

其实也可以直接用docker咯，这里先不管容器的事

不管是哪种，端口号必须分开来

#### 开始操作，使用的是yum安装的mariadb进行实验

当前单个实例的情况

##### 1、端口一个3306，要分开

![image-20230316112402636](5-mysql二进制和源码编译安装及多实例.assets/image-20230316112402636.png) 

##### 2、数据文件路径一个，要分开

![image-20230316112539364](5-mysql二进制和源码编译安装及多实例.assets/image-20230316112539364.png) 

数据库的路径yum安装的是这里👆，待会要做多实例，路径重新规划成三个独立的。

----

### ①创建独立的目录用来存放DB文件、bin文件、etc配置文件、log日志文件

![image-20230316112925342](5-mysql二进制和源码编译安装及多实例.assets/image-20230316112925342.png) 

![image-20230316112944307](5-mysql二进制和源码编译安装及多实例.assets/image-20230316112944307.png) 

socket就是mysql通过本地socket连接的文件。

![image-20230316113514834](5-mysql二进制和源码编译安装及多实例.assets/image-20230316113514834.png) 



### ②跑数据库生成的脚本，生成多个DB文件

就是把yum安装的/var/lib/mysql/下的文件，生成到自己创建的多个分开的路径里

![image-20230316113839359](5-mysql二进制和源码编译安装及多实例.assets/image-20230316113839359.png) 

yum安装就会自动生成的一个文件mysql_install_db，用它来生成DB文件

![image-20230316113947400](5-mysql二进制和源码编译安装及多实例.assets/image-20230316113947400.png) 

如果是二进制安装的就是在scripts路径下，注意不要进去运行，就是在外面运行。

生成动作👇

![image-20230316114147794](5-mysql二进制和源码编译安装及多实例.assets/image-20230316114147794.png)yum安装的时候自带脚本就会创建mysql用户，所以直接用就行了👆

--datadir 指定DB文件目录

tree可见 已生成了

<img src="5-mysql二进制和源码编译安装及多实例.assets/image-20230316114317478.png" alt="image-20230316114317478" style="zoom:50%;" /> 

同样另外两份生成一下：

![image-20230316114419611](5-mysql二进制和源码编译安装及多实例.assets/image-20230316114419611.png) 

![image-20230316114437740](5-mysql二进制和源码编译安装及多实例.assets/image-20230316114437740.png)



### ③修改目录的权限

已经通过mysql_install_db生成指定的权限如下

![image-20230316114809221](5-mysql二进制和源码编译安装及多实例.assets/image-20230316114809221.png) 

整个目录都设置一下

![image-20230316114953032](5-mysql二进制和源码编译安装及多实例.assets/image-20230316114953032.png)



### ④创建多实例的配置文件

就放到刚才定义的多实例文件夹下各个/etc下

![image-20230316115159108](5-mysql二进制和源码编译安装及多实例.assets/image-20230316115159108.png) 

先改一份，再复制到其他实例路径下

![image-20230316115508449](5-mysql二进制和源码编译安装及多实例.assets/image-20230316115508449.png)

因为是多实例，PORT默认是3306也顺便统一都写上

![image-20230316115801565](5-mysql二进制和源码编译安装及多实例.assets/image-20230316115801565.png)

上面的配置优化成变量的方式，可以吗，哈哈

![image-20230316115632862](5-mysql二进制和源码编译安装及多实例.assets/image-20230316115632862.png)

修改一下各自的端口

![image-20230316120056586](5-mysql二进制和源码编译安装及多实例.assets/image-20230316120056586.png)

![image-20230316120016038](5-mysql二进制和源码编译安装及多实例.assets/image-20230316120016038.png)



至此 二进制文件本来共用的不用动、账号共用不用动、DB文件搞定、配置文件搞定、就剩下启动脚本了。那些socket路径、pid、log都是指定后随着服务启动会自动生成的。



### ⑤启动的方式，采用脚本后台启动

服务脚本是网上找，或者自己写

拖进去即可

![image-20230316120640541](5-mysql二进制和源码编译安装及多实例.assets/image-20230316120640541.png)

这个脚本的路径要修改的，因为它原本是针对源码编译安装的

比如mysqld_safe和mysqld的路径当前所在👇

![image-20230316121039855](5-mysql二进制和源码编译安装及多实例.assets/image-20230316121039855.png)

找不到是因为which是只在PATH路径里搜索的，招不到没事，服务起一下，ps aux可见

![image-20230316121148417](5-mysql二进制和源码编译安装及多实例.assets/image-20230316121148417.png)

![image-20230316121502453](5-mysql二进制和源码编译安装及多实例.assets/image-20230316121502453.png) 

启动后就能看到mysqld的路径

![image-20230316134336619](5-mysql二进制和源码编译安装及多实例.assets/image-20230316134336619.png)

修改的截图

![image-20230316135102121](5-mysql二进制和源码编译安装及多实例.assets/image-20230316135102121.png)

![image-20230316135123909](5-mysql二进制和源码编译安装及多实例.assets/image-20230316135123909.png)

![image-20230316135155283](5-mysql二进制和源码编译安装及多实例.assets/image-20230316135155283.png)

![image-20230316135425910](5-mysql二进制和源码编译安装及多实例.assets/image-20230316135425910.png)

![image-20230316135503965](5-mysql二进制和源码编译安装及多实例.assets/image-20230316135503965.png)



至此就搞完了

![image-20230316135545789](5-mysql二进制和源码编译安装及多实例.assets/image-20230316135545789.png)

3306这个文件夹下的bin下的启动脚本就有了

data下的DB文件也就是数据库也有了

etc下的配置文件也有了

log 、pid 、socket 这些自动生成的

<img src="5-mysql二进制和源码编译安装及多实例.assets/image-20230316135927290.png" alt="image-20230316135927290" style="zoom:50%;" /> 

**启动程序**

![image-20230316140832942](5-mysql二进制和源码编译安装及多实例.assets/image-20230316140832942.png)

![image-20230316140645372](5-mysql二进制和源码编译安装及多实例.assets/image-20230316140645372.png)

![image-20230316141235612](5-mysql二进制和源码编译安装及多实例.assets/image-20230316141235612.png) 

![image-20230316141308338](5-mysql二进制和源码编译安装及多实例.assets/image-20230316141308338.png) 



![image-20230316141452307](5-mysql二进制和源码编译安装及多实例.assets/image-20230316141452307.png)



### ⑥怎么连接

![image-20230316141957358](5-mysql二进制和源码编译安装及多实例.assets/image-20230316141957358.png)

指定各自的socket文件

![image-20230316142031366](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142031366.png)

![image-20230316142146342](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142146342.png)

status看下路径

![image-20230316142227593](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142227593.png)

3307的👇

![image-20230316142247500](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142247500.png)

![image-20230316142316842](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142316842.png)

3308👇

![image-20230316142333893](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142333893.png)

![image-20230316142348019](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142348019.png)



### ⑦关闭数据库

![image-20230316142607288](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142607288.png)

报错了👆

应为服务启动关闭的那个脚本里写了固定的口令，口令是错的

![image-20230316142722001](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142722001.png)

![image-20230316142739815](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142739815.png)

就可以啦👆

通过ss -tlnp看看是不是关掉了3308这个实例

![image-20230316142811867](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142811867.png)



### ⑧补上root的口令

![image-20230316142930098](5-mysql二进制和源码编译安装及多实例.assets/image-20230316142930098.png)

记得改的时候也带上socket参数-S

改完口令，由于原来是空口令所以现在必须带密码才能登入了

![image-20230316143031787](5-mysql二进制和源码编译安装及多实例.assets/image-20230316143031787.png)

![image-20230316143045845](5-mysql二进制和源码编译安装及多实例.assets/image-20230316143045845.png)

把启动关闭脚本里的密码补回去👇，否则关闭服务还得输入密码。

![image-20230316143131497](5-mysql二进制和源码编译安装及多实例.assets/image-20230316143131497.png)



![image-20230316143212006](5-mysql二进制和源码编译安装及多实例.assets/image-20230316143212006.png)



同样修改另外两个实例的密码

![image-20230316143649510](5-mysql二进制和源码编译安装及多实例.assets/image-20230316143649510.png)



