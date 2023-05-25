# 第2节. sql语句DQL多表查询和视图



两张、三张，四张，都是从两张开始的，

两张表查询，应该就是先讨论两张表的组合起来。

## JOIN

两种形式的组合

纵向和横向合并



### 纵向union合并

就像两个文件内容cat file1 file2 追加在一起一样，但是db.tables纵向合并需要title对齐

![image-20230517103239189](2-sql语句DQL多表查询和视图.assets/image-20230517103239189.png)

两张表的TID和stuid不一样没关系，也可以合，只要内容相匹配就行。

### union合并

![image-20230517103444525](2-sql语句DQL多表查询和视图.assets/image-20230517103444525.png)

错位合起来，没有意义了就

![image-20230517103646044](2-sql语句DQL多表查询和视图.assets/image-20230517103646044.png)



### union的去重功能的开关

通过两张一样的表来测试

![image-20230517104058223](2-sql语句DQL多表查询和视图.assets/image-20230517104058223.png)





![image-20230517104357060](2-sql语句DQL多表查询和视图.assets/image-20230517104357060.png)



### 去重方法2：distinct

![image-20230517104618445](2-sql语句DQL多表查询和视图.assets/image-20230517104618445.png)





### 横向合并的逻辑较为复杂

联想paste file1 file2这个shell命令，虽然关系不大



### 横向1：cross交叉连接,也就是笛卡尔乘积组合，就是10条*20条这种200条结果的组合

![image-20230517110119043](2-sql语句DQL多表查询和视图.assets/image-20230517110119043.png)

![image-20230517110343572](2-sql语句DQL多表查询和视图.assets/image-20230517110343572.png)

好像A CROSS B 和B CROSS A 其实是一样的

cross一般用的少的，一般是找到相关联的信息

![image-20230517133430647](2-sql语句DQL多表查询和视图.assets/image-20230517133430647.png)

### 内连接-inner join

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517133526231.png" alt="image-20230517133526231" style="zoom:50%;" /> 



![image-20230517133824655](2-sql语句DQL多表查询和视图.assets/image-20230517133824655.png)

限制制定列，需要标明哪个表

![image-20230517135347179](2-sql语句DQL多表查询和视图.assets/image-20230517135347179.png)

别名简化下输入

![image-20230517143231473](2-sql语句DQL多表查询和视图.assets/image-20230517143231473.png)



![image-20230517145222967](2-sql语句DQL多表查询和视图.assets/image-20230517145222967.png)





### 左外连接-left outer join, outer可不写

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517150535217.png" alt="image-20230517150535217" style="zoom:50%;" /> 

就是A的全要，并且根据A关联B里的信息都捞出来。

观察下图，理解left out join的意思👇

![image-20230517150936457](2-sql语句DQL多表查询和视图.assets/image-20230517150936457.png)

写命令的时候 left join的左边的表就是全部要，右边的表就是写关联过去的交集。

![image-20230517151418481](2-sql语句DQL多表查询和视图.assets/image-20230517151418481.png)



### 右外连接

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517151616756.png" alt="image-20230517151616756" style="zoom:50%;" /> 

![image-20230517151605220](2-sql语句DQL多表查询和视图.assets/image-20230517151605220.png)



as真的哪哪都可以加哦，优化下表头便于理解：

![image-20230517152254164](2-sql语句DQL多表查询和视图.assets/image-20230517152254164.png)



### 对查询出来的inner join 、left join、right join进一步做过滤

表的别名和字段的别名，表的别名定义了，调用就要用别名，字段的别名只是一次性有效，调用还得用原名

![image-20230517152740042](2-sql语句DQL多表查询和视图.assets/image-20230517152740042.png)





### left join 去掉交集

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517153836673.png" alt="image-20230517153836673" style="zoom:50%;" /> 

![image-20230517154128390](2-sql语句DQL多表查询和视图.assets/image-20230517154128390.png)

上图👆是A left join B，下图👇是A left join B and xxx is null：

![image-20230517154136754](2-sql语句DQL多表查询和视图.assets/image-20230517154136754.png)

👆图写错了，不应该是and而是having或者where

![image-20230517160312060](2-sql语句DQL多表查询和视图.assets/image-20230517160312060.png)

![image-20230517160322135](2-sql语句DQL多表查询和视图.assets/image-20230517160322135.png)

### right join 去掉交集

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517160927670.png" alt="image-20230517160927670" style="zoom:50%;" /> 

![image-20230517161243949](2-sql语句DQL多表查询和视图.assets/image-20230517161243949.png)



### 全连接 full outer join

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517161600787.png" alt="image-20230517161600787" style="zoom:50%;" /> 



![image-20230517161547386](2-sql语句DQL多表查询和视图.assets/image-20230517161547386.png)



![image-20230517161637568](2-sql语句DQL多表查询和视图.assets/image-20230517161637568.png)



理论上是full join，可惜mysql和mariadb不支持所谓的full

![image-20230517161815127](2-sql语句DQL多表查询和视图.assets/image-20230517161815127.png)



换种写法

![image-20230517162437787](2-sql语句DQL多表查询和视图.assets/image-20230517162437787.png)

排版换行下

![image-20230517162519555](2-sql语句DQL多表查询和视图.assets/image-20230517162519555.png)



### full join去除交集

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517164227007.png" alt="image-20230517164227007" style="zoom:50%;" /> 

如果👇这样写是没有用的

![image-20230517164150550](2-sql语句DQL多表查询和视图.assets/image-20230517164150550.png)

### 要用到子查询：select的结果嵌入另一个select语句或者DML语句。

比如：查询年龄大于平均年龄的学生，

![image-20230517164405204](2-sql语句DQL多表查询和视图.assets/image-20230517164405204.png)



把老师的平均年龄 覆盖到 20号学生的年龄

![image-20230517165253526](2-sql语句DQL多表查询和视图.assets/image-20230517165253526.png)



然后就做一下FULL JOIN 去除交集

![image-20230517172000852](2-sql语句DQL多表查询和视图.assets/image-20230517172000852.png)

所以这个name，age，gender的冲突是和tearcher表里的冲突了

![image-20230517172232575](2-sql语句DQL多表查询和视图.assets/image-20230517172232575.png)

### 至此，就得到了FULL JOIN 的去除交集的写法👆



现在回头来，👇这些都是存在冲突隐患的

![image-20230517172411983](2-sql语句DQL多表查询和视图.assets/image-20230517172411983.png)







案例1：

![image-20230517181026794](2-sql语句DQL多表查询和视图.assets/image-20230517181026794.png)

需求：查询每个员工的姓名和 上级负责人的姓名

如果是两张表，就可以这么做

<img src="2-sql语句DQL多表查询和视图.assets/image-20230517182759364.png" alt="image-20230517182759364" style="zoom:50%;" /> 

### 这样就学到了自连接👇

![image-20230517183133775](2-sql语句DQL多表查询和视图.assets/image-20230517183133775.png)



![image-20230517183257675](2-sql语句DQL多表查询和视图.assets/image-20230517183257675.png)

![image-20230525142712060](2-sql语句DQL多表查询和视图.assets/image-20230525142712060.png)



### 涉及三张表的一个关联操作

表1，得分表

<img src="2-sql语句DQL多表查询和视图.assets/image-20230525151800097.png" alt="image-20230525151800097" style="zoom:50%;" /> 



表2，学生表

![image-20230525151831280](2-sql语句DQL多表查询和视图.assets/image-20230525151831280.png) 



表3，课程表

<img src="2-sql语句DQL多表查询和视图.assets/image-20230525151852533.png" alt="image-20230525151852533" style="zoom:50%;" /> 





![image-20230525152439551](2-sql语句DQL多表查询和视图.assets/image-20230525152439551.png)

挑选一些字段

![image-20230525152701252](2-sql语句DQL多表查询和视图.assets/image-20230525152701252.png)



![image-20230525154037109](2-sql语句DQL多表查询和视图.assets/image-20230525154037109.png)



### select语句流程图

select的处理次序，看图可以知道select columns这个语句是在from xxx 之后的，所以也能解释from tables as t的t可以在select t.name1里进行调用的原因了。

![image-20230525155959973](2-sql语句DQL多表查询和视图.assets/image-20230525155959973.png)



### VIEW试图

视图，在CCNA-SEC里见过，在BIND里见过，现在又在MYSQL里看到视图这个概念。反正不一样，知道一下就好了。



mysql里的VIEW长的像表，但其实是虚拟出来的表，不是真实存在的。



1、作用有点像linux的alias别名，固化查询语句，下次直接调用，省的再次输入长长的SQL CLI。

2、隐藏了真实的表结构。



create databases

create tables

create view

注意view和tables长的基本一样，所以命名的时候要认为区分注意规范



![image-20230525162927601](2-sql语句DQL多表查询和视图.assets/image-20230525162927601.png)

这样就创建了一个视图，所以和alias别名不一样，alias是将cli简化，而view是将cli的结果固化，相当于多了一个表，只不过是虚拟的表。



![image-20230525163205690](2-sql语句DQL多表查询和视图.assets/image-20230525163205690.png)

<img src="2-sql语句DQL多表查询和视图.assets/image-20230525163434726.png" alt="image-20230525163434726" style="zoom:50%;" /> 



### 如何区分这是一个真实的表table还是一个视图view呢

![image-20230525164011051](2-sql语句DQL多表查询和视图.assets/image-20230525164011051.png)

like是通配符，rike是正则，但是show table status里不支持rlike

<img src="2-sql语句DQL多表查询和视图.assets/image-20230525165849326.png" alt="image-20230525165849326" style="zoom:50%;" /> 



试图的tables status一眼看过去都是NULL，只有comment表达一些这个是VIEW

<img src="2-sql语句DQL多表查询和视图.assets/image-20230525165955837.png" alt="image-20230525165955837" style="zoom:50%;" /> 

真正的表里是由引擎、不能把、字符集各种设定清空的。而且Comment是空。

<img src="2-sql语句DQL多表查询和视图.assets/image-20230525170031168.png" alt="image-20230525170031168" style="zoom:50%;" /> 



### 这个view是动态的，修改视图里的源表格看看效果

![image-20230525173317122](2-sql语句DQL多表查询和视图.assets/image-20230525173317122.png)

删除学生，在看看view里的

![image-20230525173348392](2-sql语句DQL多表查询和视图.assets/image-20230525173348392.png)

是动态的，这里为什么还有NULL，因为VIEW生成的原SQL是left join的。

![image-20230525173731199](2-sql语句DQL多表查询和视图.assets/image-20230525173731199.png)



### view可以插入如何理解

插入50岁的可以，因为创建视图的时候就是age>30

![image-20230525174248033](2-sql语句DQL多表查询和视图.assets/image-20230525174248033.png)

创建20岁的就不行了，因为你这个操作是对view的insert但实际上是修改了原表，而且view本来就是过滤了age>30的，所以就有一个现象：你insert了view，但是view里却看不到👇

##### 所以实际工作中，不推荐用VIEW，因为存在这样的情况。

![image-20230525174455396](2-sql语句DQL多表查询和视图.assets/image-20230525174455396.png)

对视图的操作会影响原表

![image-20230525174547142](2-sql语句DQL多表查询和视图.assets/image-20230525174547142.png)

### 如果是三张表的join呢，，此时对VIEW的操作，又是对哪张表进行的呢？测试下，拿之前的那个view_test来测

![image-20230525174817840](2-sql语句DQL多表查询和视图.assets/image-20230525174817840.png)

哈哈，不让~

我感觉就不应该对VIEW进行操作！

而且也不建议使用VIEW



drop view xxx;删除视图

![image-20230525175716101](2-sql语句DQL多表查询和视图.assets/image-20230525175716101.png)







