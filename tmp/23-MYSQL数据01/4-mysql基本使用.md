# 第4节. mysql基本使用



![ ](4-mysql基本使用.assets/image-20230314134117979.png) 

空的意思是用户名随便写，上图👆框错了哦，user为空的是最后两行，PUBLIC本身就是用户名

![image-20230314134302856](4-mysql基本使用.assets/image-20230314134302856.png)

空用户进来的权限👆



### 进行安全加固

![image-20230314140536805](4-mysql基本使用.assets/image-20230314140536805.png)

这是我之前用直接yum的mariadb，是由安全加固脚本的

![image-20230314140650127](4-mysql基本使用.assets/image-20230314140650127.png) 

但是新的mariadb版本改动了一下，👇

![image-20230314141158447](4-mysql基本使用.assets/image-20230314141158447.png)

https://mariadb.com/kb/en/mysql_secure_installation/

结合文章，看来软连接就是过渡一下用的，现在的新版本呢都没有mysql_secue_installation这个软连接文件了。

https://mariadb.com/kb/en/authentication-from-mariadb-104/

文中说“Remember, the best way to keep your password safe is not to have one!”

服了U~

回到这里学习的安全加固脚本：

![image-20230314142442928](4-mysql基本使用.assets/image-20230314142442928.png) 

跑一下



10.11版的情况

![image-20230314142819734](4-mysql基本使用.assets/image-20230314142819734.png)

![image-20230314142839088](4-mysql基本使用.assets/image-20230314142839088.png)

视频中的低版本👇，主要区别是swith to unix_socket authentication吧，反正上面走完root密码是不生效的，应为有插件要改一下。emm不过，人家官网说了没有密码就是最安全的密码。

![image-20230314142932705](4-mysql基本使用.assets/image-20230314142932705.png)

脚本到这里已经起作用了，要密码登入了，当然我说的是老版本

![image-20230314143952743](4-mysql基本使用.assets/image-20230314143952743.png)

![image-20230314143818128](4-mysql基本使用.assets/image-20230314143818128.png)

这个centos7.localdomain是禁止root远程登入干掉的，因为它担心有人使用域名解析将cento7.localdomain解析成远端mysql的IP地址，从而使用这个hosts进来。



回到我的新版mariadb

![image-20230314144410834](4-mysql基本使用.assets/image-20230314144410834.png)



![image-20230314144541574](4-mysql基本使用.assets/image-20230314144541574.png)

密码已经改好了，如果非要使用root密码生效，则需要

https://blog.csdn.net/tiny_du/article/details/123924376

https://www.orcy.net.cn/1410.html

修改global_priv里的unix_socket

> ALTER USER root@localhost IDENTIFIED VIA mysql_native_password USING PASSWORD("你的密码");

文中的这个方法，记不清我之前是不是直接修改了global_priv里的内容。

![image-20230314151429167](4-mysql基本使用.assets/image-20230314151429167.png)

```shell
ALTER USER root@localhost IDENTIFIED VIA mysql_native_password USING PASSWORD("xxxxx");
```

然后再看

![image-20230314153302995](4-mysql基本使用.assets/image-20230314153302995.png)

![image-20230314153504397](4-mysql基本使用.assets/image-20230314153504397.png)

我再找找文章

https://blog.whsir.com/post-6795.html

这个靠谱，在配置文件中禁用unix_socket

不过是不是可以直接修改gloabl_priv里的参数的，好像没有唉，👇官网也是这个方法：

https://mariadb.com/kb/en/authentication-plugin-unix-socket/



然后\G的排版看下

![image-20230314155306120](4-mysql基本使用.assets/image-20230314155306120.png)



-uroot不写也行，默认就是root，你换linux的用户，也一样，Pia!(ｏ ‵-′)ノ”(ノ﹏<。)

![image-20230314160829888](4-mysql基本使用.assets/image-20230314160829888.png) 

一样个毛👇

![image-20230314161017586](4-mysql基本使用.assets/image-20230314161017586.png)

Pia!(ｏ ‵-′)ノ”(ノ﹏<。)

![image-20230314161208874](4-mysql基本使用.assets/image-20230314161208874.png) 





mysql -uroot -pxxx -h a.b.c.d   这个是完全命令

![image-20230314161410352](4-mysql基本使用.assets/image-20230314161410352.png) 

注意-h 127.0.0.1和-h localhost不一样，前者是走的tcp/ip socket，后者是走的unix_socket文件socket

![image-20230314161717077](4-mysql基本使用.assets/image-20230314161717077.png)



![image-20230314161630971](4-mysql基本使用.assets/image-20230314161630971.png)

下图虽然是本地就是192.168.126.129自己登自己，但是还是判定为root@django001，解析成了hostname所以不对了。

![image-20230314161903110](4-mysql基本使用.assets/image-20230314161903110.png)

尝试改host为192.168.126.129看看，果然被反向解析成了hostname了

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.126.129' IDENTIFIED BY 'Cisc0@123' WITH GRANT OPTION;	
```

![image-20230314164635872](4-mysql基本使用.assets/image-20230314164635872.png)

然后将192.168.126.129改成hostname也就是django001看看，不行，就是IP，虽然显示的是root@django001，但是只是反向解析的结果，只认IP。

![image-20230314171243990](4-mysql基本使用.assets/image-20230314171243990.png)





报错的情况：update不行？

![image-20230314164857474](4-mysql基本使用.assets/image-20230314164857474.png)



![image-20230314165818586](4-mysql基本使用.assets/image-20230314165818586.png)



https://stackoverflow.com/questions/64841185/error-1356-hy000-view-mysql-user-references-invalid-tables-or-columns-o

```
ALTER USER root@localhost IDENTIFIED VIA mysql_native_password USING PASSWORD("xxxxx");
或者使用SETPASSWORD

```

https://mariadb.com/kb/en/set-password/

## Example

For example, if you had an entry with User and Host column values of '`bob`' and '`%.loc.gov`', you would write the statement like this:

```
SET PASSWORD FOR 'bob'@'%.loc.gov' = PASSWORD('newpass');
```

If you want to delete a password for a user, you would do:

```
SET PASSWORD FOR 'bob'@localhost = PASSWORD("");
```





查看的细节

![image-20230314172522559](4-mysql基本使用.assets/image-20230314172522559.png)

新版本就是-server，-client这种

<img src="4-mysql基本使用.assets/image-20230314172611312.png" alt="image-20230314172611312" style="zoom:60%;" /> 



老版本就是

<img src="4-mysql基本使用.assets/image-20230314172809902.png" alt="image-20230314172809902" style="zoom:50%;" /> 

<img src="4-mysql基本使用.assets/image-20230314172828907.png" alt="image-20230314172828907" style="zoom:50%;" /> 

这个就是client





mysql-client里的mysqldump很关键

<img src="4-mysql基本使用.assets/image-20230314173120892.png" alt="image-20230314173120892" style="zoom:50%;" /> 



 

### mysqladmin的用法

![image-20230314174240949](4-mysql基本使用.assets/image-20230314174240949.png)

-？等价于--help

![image-20230314174312496](4-mysql基本使用.assets/image-20230314174312496.png)

检测mysql服务是否正常

![image-20230314174429328](4-mysql基本使用.assets/image-20230314174429328.png)

 远程测试要密码和端口：

![image-20230314175612081](4-mysql基本使用.assets/image-20230314175612081.png) 

超时时间要设置下的

![image-20230314180125738](4-mysql基本使用.assets/image-20230314180125738.png)

![image-20230314180324026](4-mysql基本使用.assets/image-20230314180324026.png)

实测是2m的超时时间

调整探测时间上限方法就是--connect-timeout

https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html

![image-20230314180735117](4-mysql基本使用.assets/image-20230314180735117.png)

但是密码要想办法隐藏好，否则监控存在密码泄露风险。





##### 关闭服务：

![image-20230314175904203](4-mysql基本使用.assets/image-20230314175904203.png)

数据的进程，一定不要用kill来杀，一定不要！！！，杀完就可能起不来了。



##### 改口令

![image-20230314181101517](4-mysql基本使用.assets/image-20230314181101517.png)

![image-20230314181533280](4-mysql基本使用.assets/image-20230314181533280.png)

 

![image-20230314181351245](4-mysql基本使用.assets/image-20230314181351245.png) 

以什么身份user@host 连进去的，就改的谁的密码。



##### 还可以直接创建数据库

![image-20230314181703811](4-mysql基本使用.assets/image-20230314181703811.png)



![image-20230314182005913](4-mysql基本使用.assets/image-20230314182005913.png)



##### 不进入交互式的cli方法

```shell
mysql -e "show databases"	
```

![image-20230314182403116](4-mysql基本使用.assets/image-20230314182403116.png) 



![image-20230314182948403](4-mysql基本使用.assets/image-20230314182948403.png)

mysql进入是交互模式，也就是有标准输入的，所以重定向文件内容进去就行了

![image-20230314185120520](4-mysql基本使用.assets/image-20230314185120520.png)

表格式没了

![image-20230314185238390](4-mysql基本使用.assets/image-20230314185238390.png)







![image-20230314185515661](4-mysql基本使用.assets/image-20230314185515661.png)

![image-20230314185544748](4-mysql基本使用.assets/image-20230314185544748.png)

mysqld_safe 是mysqld的父进程，上图是通过ps auxf看的

![image-20230314185911485](4-mysql基本使用.assets/image-20230314185911485.png)

pstree -p可见

![image-20230314185946981](4-mysql基本使用.assets/image-20230314185946981.png)



新版本没有这个safe父进程了

![image-20230314185738866](4-mysql基本使用.assets/image-20230314185738866.png)



进程是以mysql这个用户运行的。

而该用户是安装程序的时候脚本创建的。

![image-20230314190640815](4-mysql基本使用.assets/image-20230314190640815.png)

![image-20230314190412995](4-mysql基本使用.assets/image-20230314190412995.png)



后面源码编译，二进制安装的时候，需要手动创建用户

#### 多实例，也可以同一个版本来做

<img src="4-mysql基本使用.assets/image-20230314191007434.png" alt="image-20230314191007434" style="zoom:50%;" /> rpm -ql MariaDB-server可见

端口错开，用户透明，测试环境用







### 用户账号

![image-20230314191725785](4-mysql基本使用.assets/image-20230314191725785.png)

_就是等价于regex里的?，可能吧

疑问：172.16.%.%，%匹配任意长度字符，那为啥不写成172.16.%











![image-20230314192246377](4-mysql基本使用.assets/image-20230314192246377.png)



![image-20230314192234857](4-mysql基本使用.assets/image-20230314192234857.png)



![image-20230314192448853](4-mysql基本使用.assets/image-20230314192448853.png)



![image-20230314192426086](4-mysql基本使用.assets/image-20230314192426086.png)





### 修改mysql交互模式的提示符

man手册里看

![image-20230314192650555](4-mysql基本使用.assets/image-20230314192650555.png)

![image-20230314192718294](4-mysql基本使用.assets/image-20230314192718294.png)

![image-20230314192745042](4-mysql基本使用.assets/image-20230314192745042.png)

![image-20230314192831766](4-mysql基本使用.assets/image-20230314192831766.png)



![image-20230314192940298](4-mysql基本使用.assets/image-20230314192940298.png)



![image-20230314193048168](4-mysql基本使用.assets/image-20230314193048168.png)

进到/etc/my.conf.d下面，可以看到client.cnf和mysql-clients.conf等文件

打开mysql-clients.conf进行编辑

![image-20230314193331880](4-mysql基本使用.assets/image-20230314193331880.png)

生效👇

![image-20230314193350538](4-mysql基本使用.assets/image-20230314193350538.png)





![image-20230314193532391](4-mysql基本使用.assets/image-20230314193532391.png)

数据库的路径，可以改成LVM里，优点就是可以伸缩空间。

上图还有socket文件的路径，就是mysql -uroot -pxxx 回车走的文件socket

![image-20230314195453405](4-mysql基本使用.assets/image-20230314195453405.png)





<img src="4-mysql基本使用.assets/image-20230314193942323.png" alt="image-20230314193942323" style="zoom:50%;" /> 

这是日志，进程ID文件



![image-20230314194233377](4-mysql基本使用.assets/image-20230314194233377.png)



### 自动补齐

![image-20230314194207368](4-mysql基本使用.assets/image-20230314194207368.png)



看到没有带任何默认值👇

![image-20230314194456506](4-mysql基本使用.assets/image-20230314194456506.png)



修改一个选项，然后看下

![image-20230314194637278](4-mysql基本使用.assets/image-20230314194637278.png)



![image-20230314194832380](4-mysql基本使用.assets/image-20230314194832380.png)

说明还是可以补的，好吧，

<img src="4-mysql基本使用.assets/image-20230314195013378.png" alt="image-20230314195013378" style="zoom:50%;" /> 



![image-20230314195338027](4-mysql基本使用.assets/image-20230314195338027.png)





如果源码安装，将来上文提到的路径，就需要自己定义了，包括

1、socket路径

2、数据库路径

3、log日志路径

等



##### 直接连进数据库

![image-20230314195806366](4-mysql基本使用.assets/image-20230314195806366.png) 

-D 可省略

![image-20230314195838877](4-mysql基本使用.assets/image-20230314195838877.png)



-C 压缩可节省带宽

<img src="4-mysql基本使用.assets/image-20230314195911236.png" alt="image-20230314195911236" style="zoom:80%;" /> 

-e 上文讲过了

-V 版本

-v 明细

![image-20230314195936932](4-mysql基本使用.assets/image-20230314195936932.png)









![image-20230314200008497](4-mysql基本使用.assets/image-20230314200008497.png)



上文也做个实验了，127.0.0.1不是走的unix sock而是走的TCP/IP







##### 查看当前用户👇

![image-20230314200237299](4-mysql基本使用.assets/image-20230314200237299.png)





##### 查看当前版本

注意哦，我这是自己的10.11和视频的5.5混着截图了，意思到了就行

![image-20230314200415679](4-mysql基本使用.assets/image-20230314200415679.png)





### 大小写-库名、表名区分大小写，命令和字段不区分大小写



##### 命令不区分大小写

<img src="4-mysql基本使用.assets/image-20230314200513818.png" alt="image-20230314200513818" style="zoom:50%;" /> 



##### 数据名称区分大小写

<img src="4-mysql基本使用.assets/image-20230314200545807.png" alt="image-20230314200545807" style="zoom:50%;" /> 



##### 表名大小写敏感

##### ![image-20230314200651456](4-mysql基本使用.assets/image-20230314200651456.png)



##### 表里字段也就是列属性不区分大小写 

![image-20230314200814454](4-mysql基本使用.assets/image-20230314200814454.png) 

推荐SELECT这种命令也是大写



##### 删库的方式又增加了

![image-20230314203014630](4-mysql基本使用.assets/image-20230314203014630.png)







### 配置文件也可以合在一起

通过名称区分

![image-20230314203323429](4-mysql基本使用.assets/image-20230314203323429.png) 

这些配置内容的格式就是上图说的

parameter = value

而value的启用和禁用，又可以有1 ON TRUE等写法如上图所示。

如果配置文件里是不区分_和-的

![image-20230314203659023](4-mysql基本使用.assets/image-20230314203659023.png) 



##### 配置文件路径

![image-20230314203748835](4-mysql基本使用.assets/image-20230314203748835.png) 

优先级上图的从上往下，但是不同的安装方式，优先级的结论不同

不要可以记这些，就是实际使用，配置文件就放一个地方，避免混淆。





#### 维护模式

![image-20230314203956708](4-mysql基本使用.assets/image-20230314203956708.png) 

上图=1可以省略不写，就是=1了

skip-networking=1就关闭了3306，相当于维护模式，外界就连不进来了。



vim /etc/my.conf

![image-20230314204221530](4-mysql基本使用.assets/image-20230314204221530.png) 

![image-20230314204242911](4-mysql基本使用.assets/image-20230314204242911.png)

再看下端口号就不在了

![image-20230314204304435](4-mysql基本使用.assets/image-20230314204304435.png)

但是自己可以连

![image-20230314204321728](4-mysql基本使用.assets/image-20230314204321728.png)

因为走的是unix_socket

![image-20230314204344349](4-mysql基本使用.assets/image-20230314204344349.png)



##### 配一个配置文件路径

![image-20230314204410009](4-mysql基本使用.assets/image-20230314204410009.png)

/etc/mysql这个路劲是默认没有的，没有就创建一个

![image-20230314204443408](4-mysql基本使用.assets/image-20230314204443408.png) 

![image-20230314204501710](4-mysql基本使用.assets/image-20230314204501710.png) 

和刚才的配置相反了

重启服务看下

![image-20230314204525161](4-mysql基本使用.assets/image-20230314204525161.png)

说明/etc/my.cnf是优于/etc/mysql/my.cnf的







### 二进制安装

![image-20230314204744080](4-mysql基本使用.assets/image-20230314204744080.png)

二进制安装，配置文件/etc下的XX、用户账号、DB文件 都没有



![image-20230314204859966](4-mysql基本使用.assets/image-20230314204859966.png)

指定了家目录，就是用来放DB文件的，而/data将来推荐做成LVM

同样的所有者和所有组参考原来的DB文件设置下

![image-20230314205037791](4-mysql基本使用.assets/image-20230314205037791.png)





![image-20230314205307798](4-mysql基本使用.assets/image-20230314205307798.png) 



![image-20230314205251335](4-mysql基本使用.assets/image-20230314205251335.png) 

文件名有linux字样的的就是二进制的包，大小也不一样，二进制的编译后的肯定大很多。



##### 注意，有个安装路径是固定的，人家编译成二进制的时候就写死了

![image-20230314205411284](4-mysql基本使用.assets/image-20230314205411284.png) 

/usr/local要手动补上的，当初人家configure 编译的时候指定的类路径。<img src="4-mysql基本使用.assets/image-20230314205550655.png" alt="image-20230314205550655" style="zoom:30%;" />  

**/usr/local/mysql**就是当初人家编译的时候指定的路径

这里就是一些mysql的程序



#### 配置文件

![image-20230314205831836](4-mysql基本使用.assets/image-20230314205831836.png)



![image-20230314205944787](4-mysql基本使用.assets/image-20230314205944787.png)

二进制安装后有一个模板my-large.cnf复制过来用，然后为了不和/etc/my.cnf冲突，就另起炉灶放到新建的/etc/mysql文件夹下

![image-20230314210101278](4-mysql基本使用.assets/image-20230314210101278.png)



这就是计划数据的路径，下两行再说是优化的



![image-20230314210153486](4-mysql基本使用.assets/image-20230314210153486.png) 

二进制安装后DB文件时空的，需要生成出来，mysql、test、performance_scheme这些

yum安装其实也是依赖这个脚本生成的



![image-20230314210336350](4-mysql基本使用.assets/image-20230314210336350.png) 

centos7上也能用这个service xxx start，可以参考现在的做成systemctl start xxx的service文件。yum安装的mysql里service文件写法看看。



##### PATH变量![image-20230314210512561](4-mysql基本使用.assets/image-20230314210512561.png)



##### 初始化

![image-20230314210536985](4-mysql基本使用.assets/image-20230314210536985.png) 











### 源码包编译安装

和二进制差别不大，就是多一步编译

![image-20230314210642706](4-mysql基本使用.assets/image-20230314210642706.png) 

tar解压

cmake编译安装，以前我们都是用configure来弄，这里推荐用cmake，原来的configure make makeinstall也能用，不推荐

![image-20230314210834843](4-mysql基本使用.assets/image-20230314210834843.png) 

支持很多存储引擎，不代表就用得到。



![image-20230314211122053](4-mysql基本使用.assets/image-20230314211122053.png)











