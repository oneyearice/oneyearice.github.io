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

2、DB3

3、table

4、字段

除此之外，还看到了client、conn



##### 改字段的字符集

 ALTER TABLE student3 CHANGE name name VARCHAR(20) CHARACTER SET
 utf8mb4;

![image-20230406111513480](7-sql各种语句2.assets/image-20230406111513480.png)



![image-20230406111637743](7-sql各种语句2.assets/image-20230406111637743.png)

终于支持中文了，其实一开始创建实例的时候就制定好支持utf8mb4就行了

![image-20230406111752018](7-sql各种语句2.assets/image-20230406111752018.png)







