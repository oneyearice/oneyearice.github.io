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
