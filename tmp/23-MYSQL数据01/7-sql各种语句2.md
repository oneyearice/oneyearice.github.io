# 第7节. sql各种语句2



## DML语句



### insert 前后一一对应

![image-20230331103231763](7-sql各种语句2.assets/image-20230331103231763.png)

id默认自增长、gender性别有默认值

![image-20230331103415678](7-sql各种语句2.assets/image-20230331103415678.png)

添加多条记录

![image-20230331103600720](7-sql各种语句2.assets/image-20230331103600720.png)

只要前后的域名和值一一对应就行，顺序不固定

![image-20230331103901387](7-sql各种语句2.assets/image-20230331103901387.png)

字符集，表是集成的库的

![image-20230331112659540](7-sql各种语句2.assets/image-20230331112659540.png)

![image-20230331112932542](7-sql各种语句2.assets/image-20230331112932542.png)

现在的版本都不让你插入不支持的字符。



换个数据库测试一下字符集的支持情况

![image-20230331114754055](7-sql各种语句2.assets/image-20230331114754055.png)

创建表方法1👆，顺便也换种方式创建表，

创建表方法2：以一个查询语句的结果来创建表，

创建表方法3：复制表

```
(2) 通过查询现存表创建；新表会被直接插入查询而来的数据 
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    [(create_definition,...)]	
    [table_options]
    [partition_options]	
    select_statement

(3) 通过复制现存的表的表结构创建，但不复制数据
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name 
    { LIKE old_tbl_name | (LIKE old_tbl_name) }

```

![image-20230331115414874](7-sql各种语句2.assets/image-20230331115414874.png)

但是复制创建表的字符集还是latin是原表的字符集

而新建的表就是集成了数据库的字符集utf8mb4

![image-20230331115549579](7-sql各种语句2.assets/image-20230331115549579.png)

创建方法3，

![image-20230331115857714](7-sql各种语句2.assets/image-20230331115857714.png)

此时依据查询结果来创建表，不涉及原表的默认格式比如字符集，所以表本身还是新建的，字符集集成了数据库的utf8mb4。但是！要注意的是，此时依旧无法在此表中创建中文，因为仔细看上图，虽然ENGINE引擎里是修改了utf8mb4，但是name字段还是latin1_swedish_ci，所以姓名依然不能写中文。

三种方法的创建，只有select 结果创建，是带上了原表内容了

![image-20230331120110511](7-sql各种语句2.assets/image-20230331120110511.png)



##### insert 插入可以不写域名，就是values里必须写全，如下：

![image-20230406092939221](7-sql各种语句2.assets/image-20230406092939221.png)



### 还有一个客户端的字符集的情况

![image-20230406093209688](7-sql各种语句2.assets/image-20230406093209688.png)

客户端的字符集时auto 自动的

制定一下字符集

对比一下制定字符集和不指定字符集的区别

![image-20230406093649870](7-sql各种语句2.assets/image-20230406093649870.png)

![image-20230406093749813](7-sql各种语句2.assets/image-20230406093749813.png)

db并没有区别，然后client characterset客户端字符集同样没啥变化，都是utf8mb4，这不是OK得啊，还指啥指呢。

进入一个DB后，status发现变化了



![image-20230406093853226](7-sql各种语句2.assets/image-20230406093853226.png)

server charactoerset是整个mysql实例用的字符集，所谓实例，就是mysql这个服务，一般多实例就是多监听端口。

db charactoerset是实例下面的某个db使用的字符集

client characterset是客户端使用的字符集

conn characterset望文生义就是链接的字符集，不明觉厉



但是table里的name还是拉丁字符集啊，对不对，我不用继续听你讲， 都知道你的思路是不对的。

![image-20230406093512054](7-sql各种语句2.assets/image-20230406093512054.png)



退出一下，改为不制定字符集的进入方式

![image-20230406094546604](7-sql各种语句2.assets/image-20230406094546604.png)

##### show variables like "%character%"

![image-20230406094646025](7-sql各种语句2.assets/image-20230406094646025.png)



### 总结：字符集定义的地方

1、实例

2、DB

3、table

4、字段

除此之外，还看到了client、conn



##### 改字段的字符集

 ALTER TABLE student3 CHANGE name name VARCHAR(20) CHARACTER SET utf8mb4;

![image-20230406111513480](7-sql各种语句2.assets/image-20230406111513480.png)



![image-20230406111637743](7-sql各种语句2.assets/image-20230406111637743.png)

终于支持中文了，其实一开始创建实例的时候就制定好支持utf8mb4就行了

![image-20230406111752018](7-sql各种语句2.assets/image-20230406111752018.png)



数据库相关的变量

![image-20230505085323141](7-sql各种语句2.assets/image-20230505085323141.png)

修改配置文件

![image-20230505085923199](7-sql各种语句2.assets/image-20230505085923199.png)



![image-20230505085900221](7-sql各种语句2.assets/image-20230505085900221.png)



![image-20230505115313690](7-sql各种语句2.assets/image-20230505115313690.png)

客户端的字符集，上面是命令选项，下面是写入配置文件里

![image-20230505115645072](7-sql各种语句2.assets/image-20230505115645072.png)

![image-20230505120007257](7-sql各种语句2.assets/image-20230505120007257.png)

虽然这个修改不用重启就能看到效果，但是还是要重一下，因为，如果你改的是/etc/my.cnf里的

![image-20230505120120204](7-sql各种语句2.assets/image-20230505120120204.png)

这样是语法错误，重启服务报错的，但是不重启，mysql 进入竟然client字符集是修改成功的。所以奇葩吧。



查看default效果

![image-20230510154915141](7-sql各种语句2.assets/image-20230510154915141.png)







导入一个现成的脚本，生产数据库文件供练习使用

mysql -uroot -pCisc0@123 < hellodb_innodb.sql

除了insert语句还可以用的别的

![image-20230511091003862](7-sql各种语句2.assets/image-20230511091003862.png)



![image-20230511091308903](7-sql各种语句2.assets/image-20230511091308903.png)

StuID有自动增长属性，所以insert的时候不用手动指定了

![image-20230511091616424](7-sql各种语句2.assets/image-20230511091616424.png)

set这种格式用的不多👆这是insert的第二种语法



还有insert第三种语法，也不多用的，了解一下，

适合把一个表中的查询结果批量导入到另一个表种

![image-20230511092103807](7-sql各种语句2.assets/image-20230511092103807.png)





更改的指令

要制定更改的哪条记录里的哪个字段

![image-20230511092401392](7-sql各种语句2.assets/image-20230511092401392.png)

![image-20230511092716335](7-sql各种语句2.assets/image-20230511092716335.png)

![image-20230511092736594](7-sql各种语句2.assets/image-20230511092736594.png)

如果update的时候不加where限制条件，那么所有的记录都被改了，这就是灾难性的操作了，和删库也差不多了

![image-20230511092934944](7-sql各种语句2.assets/image-20230511092934944.png)

为了避免这种情况的发生

![image-20230511093002027](7-sql各种语句2.assets/image-20230511093002027.png)

dummy笨蛋的意思

![image-20230511093214933](7-sql各种语句2.assets/image-20230511093214933.png)

-U是命令里的，也可以写到配置文件里，提到配置文件要知道是  server的配置文件 还是 client 配置文件

这里的mysql -U应该是客户端的命令，所以是client配置文件里修改

![image-20230511093546807](7-sql各种语句2.assets/image-20230511093546807.png)

![image-20230511093538420](7-sql各种语句2.assets/image-20230511093538420.png)

效果OK

![image-20230511093631523](7-sql各种语句2.assets/image-20230511093631523.png)





DELET删除命令

![image-20230511094058531](7-sql各种语句2.assets/image-20230511094058531.png)

![image-20230511094115696](7-sql各种语句2.assets/image-20230511094115696.png)

![image-20230511094211183](7-sql各种语句2.assets/image-20230511094211183.png)

由于开启了安全模式，所以不让清表，👆这是清表，不是删表，删表是DROP

当你的表中记录非常多的时候，清表速度就要快，此时就不能用DELETE，而是用TRUNCATE

![image-20230511094418439](7-sql各种语句2.assets/image-20230511094418439.png)

因为DELETE删表记录的时候，还要记录日志的，所以速度没有TRUNCATE(不生成事务日志)快

事务日志后面讲



DML insert update delete

DDL create drop alter



下面继续学习DCL和DQL

![image-20230511095227010](7-sql各种语句2.assets/image-20230511095227010.png)









---

之前的一个编译报错，版本较新的包，解决思路如下

![image-20230511095524821](7-sql各种语句2.assets/image-20230511095524821.png)

但是rm -f CMakeCache.txt删除后要重新cmake可能。然后yum install libdb-cxx-devel要补一个这个包。要么就是和AMD的CPU有关导致的编译到后面失败了。

![image-20230511095804104](7-sql各种语句2.assets/image-20230511095804104.png)













