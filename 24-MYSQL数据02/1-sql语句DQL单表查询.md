# 第1节. sql语句DQL单表查询



### select的其他作用



#### 打印字符串：

<img src="1-sql语句DQL单表查询.assets/image-20230515165844879.png" alt="image-20230515165844879" style="zoom:50%;" /> 



#### 数字运算：

<img src="1-sql语句DQL单表查询.assets/image-20230515170013908.png" alt="image-20230515170013908" style="zoom:50%;" /> 





select * from table_name  

*表示这个表中所有的字段

### 挑列

<img src="1-sql语句DQL单表查询.assets/image-20230515170439718.png" alt="image-20230515170439718" style="zoom:50%;" /> 

1、挑选的字段，可以自定义顺序，所以设计表的时候，字段的前后顺序并不重要，只要查的时候把它改一下顺序就行了。

2、字段名大小写问题不大，注意规范就好。

<img src="1-sql语句DQL单表查询.assets/image-20230515171025117.png" alt="image-20230515171025117" style="zoom:50%;" /> 



#### 别名-优化显示，配合页面的显示

as可写可不写

<img src="1-sql语句DQL单表查询.assets/image-20230515171430402.png" alt="image-20230515171430402" style="zoom:50%;" /> 



#### 多表查询的时候的优化表名

![image-20230515171733662](1-sql语句DQL单表查询.assets/image-20230515171733662.png)



### 挑行

![image-20230515174327361](1-sql语句DQL单表查询.assets/image-20230515174327361.png)

非标准SQL比如!= 不等于的这种写法，别的数据库里就有可能不支持，mysql里是支持的。

![image-20230515174601221](1-sql语句DQL单表查询.assets/image-20230515174601221.png) 



![image-20230515174715198](1-sql语句DQL单表查询.assets/image-20230515174715198.png)



![image-20230515185430086](1-sql语句DQL单表查询.assets/image-20230515185430086.png)



#### 不等于的写法

![image-20230515185602528](1-sql语句DQL单表查询.assets/image-20230515185602528.png)

注意where里的表达式里，除了数字其他一般都需要加引号。

![image-20230515185745065](1-sql语句DQL单表查询.assets/image-20230515185745065.png)



### 多条件选择

![image-20230515190423184](1-sql语句DQL单表查询.assets/image-20230515190423184.png)

是真不区分大小写啊



### 登入的逻辑，

就是在网页里，输入用户名、密码，最终就表现为一个查询语句，

```sql
select * from students where username='admin' and password='centos';
```

查到了就是存在用户和密码匹配的，如果没查到，就是用户名或者密码不对

![image-20230515191152740](1-sql语句DQL单表查询.assets/image-20230515191152740.png)

![image-20230515191323265](1-sql语句DQL单表查询.assets/image-20230515191323265.png)



![image-20230515192544496](1-sql语句DQL单表查询.assets/image-20230515192544496.png)



### 构建非法输入字符绕过常规select

<img src="1-sql语句DQL单表查询.assets/image-20230515192725145.png" alt="image-20230515192725145" style="zoom:50%;" /> 



利用下面的这个查询or的逻辑，加上面的手法，就可以绕过检查。

<img src="1-sql语句DQL单表查询.assets/image-20230515192841034.png" alt="image-20230515192841034" style="zoom:50%;" /> 

#### 构建奇怪的密码

![image-20230515193129719](1-sql语句DQL单表查询.assets/image-20230515193129719.png)

这就是sql注入，很多年前针对DB的安全攻击。大部分软件JAVA PHP都针对这种有相应的措施。

![image-20230515193625668](1-sql语句DQL单表查询.assets/image-20230515193625668.png)

上图admin\'--  是用户名，密码是一个单引号  \'

<img src="1-sql语句DQL单表查询.assets/image-20230515194335856.png" alt="image-20230515194335856" style="zoom:80%;" />



### LIKE通配符 % _

% 前后都有的，这种是不推荐写的，因为会严重的影响数据库的性能，因为它不能利用索引。利用索引，才能提升性能。

![image-20230515194453821](1-sql语句DQL单表查询.assets/image-20230515194453821.png)

<img src="1-sql语句DQL单表查询.assets/image-20230515194513837.png" alt="image-20230515194513837" style="zoom:50%;" /> 

当数据百万级别的时候，这个前后都有%%的写法就非常不好了！





### RLIKE、REGEXP正则

也是不推荐使用的，影响性能

![image-20230515201741895](1-sql语句DQL单表查询.assets/image-20230515201741895.png)





### 去重-distinct

![image-20230515201838991](1-sql语句DQL单表查询.assets/image-20230515201838991.png)



![image-20230515201943174](1-sql语句DQL单表查询.assets/image-20230515201943174.png)





### 查空置NULL

![image-20230515202238180](1-sql语句DQL单表查询.assets/image-20230515202238180.png)

![image-20230515202315828](1-sql语句DQL单表查询.assets/image-20230515202315828.png)



![image-20230515203014802](1-sql语句DQL单表查询.assets/image-20230515203014802.png)



#### 不为空

![image-20230515203144814](1-sql语句DQL单表查询.assets/image-20230515203144814.png)



### 表中查询统计

分组统计，聚合函数，group和聚合函数通常成对出现

GROUP：根据指定的条件把查询结果进行“分组”以用于做“聚合”运算 avg(), max(), min(), count(), sum()

HAVING: 对分组聚合运算后的结果指定过滤条件

#### 函数在DB中都是要加()的，括号里放的就是函数的要求



##### count()统计非空记录数也就是行数

![image-20230515203519865](1-sql语句DQL单表查询.assets/image-20230515203519865.png)

![image-20230515203858225](1-sql语句DQL单表查询.assets/image-20230515203858225.png)



![image-20230515204024786](1-sql语句DQL单表查询.assets/image-20230515204024786.png)

count(*)，括号里放字段，\*就是所有字段，也就是所有列，不可能所有列都为空

##### 所以也就是表的行数也就是记录数，所以统计表的记录数可以用count(*)还有count(主键)，因为主键也不能为空

![image-20230515204228484](1-sql语句DQL单表查询.assets/image-20230515204228484.png)





#### max() 和 min()以及avg()

![image-20230515204530676](1-sql语句DQL单表查询.assets/image-20230515204530676.png)



### 分类汇总-分组汇总



针对性别分别做平均年龄

![image-20230516104726786](1-sql语句DQL单表查询.assets/image-20230516104726786.png)

但是男女没有标明

做分组分类的时候肯定要带上分组的字段名，比如gender分组的，就要select上gender

![image-20230516104844791](1-sql语句DQL单表查询.assets/image-20230516104844791.png)



group by 要注意下图的效果

如果group by不配合聚合函数，那么就是查的是第一个分组内容的信息，比如 group by gender就是性别分类，结果不配合聚合函数，就是第一个女生的信息和第一个男生的信息

![image-20230516105424977](1-sql语句DQL单表查询.assets/image-20230516105424977.png)

**所以，使用group by分组的时候，前面select后面跟的只有两个东西，一个是group by本身的字段，还一个就是聚合函数**



![image-20230516134324402](1-sql语句DQL单表查询.assets/image-20230516134324402.png)



### where和group by的组合

![image-20230516134719906](1-sql语句DQL单表查询.assets/image-20230516134719906.png)

要么where放前面，过滤之后再分组

要么改where 为having，分组之后再过滤，

![image-20230516135203122](1-sql语句DQL单表查询.assets/image-20230516135203122.png)

as还是比较不错的👆

![image-20230516135242674](1-sql语句DQL单表查询.assets/image-20230516135242674.png)

上面的where也是可以写成having如下👇

![image-20230516151704218](1-sql语句DQL单表查询.assets/image-20230516151704218.png)





### 分组 再 分组

![image-20230516135919751](1-sql语句DQL单表查询.assets/image-20230516135919751.png)

### 对上图进行班级分组后，再对性别进行分组

![image-20230516140522070](1-sql语句DQL单表查询.assets/image-20230516140522070.png)





### ORDER BY排序

升序ASC

降序DESC

针对数字型的有效

![image-20230516141240436](1-sql语句DQL单表查询.assets/image-20230516141240436.png)



![image-20230516141312930](1-sql语句DQL单表查询.assets/image-20230516141312930.png)



升序中，null石排在前面的

![image-20230516141443531](1-sql语句DQL单表查询.assets/image-20230516141443531.png)

倒叙NULL虽然排到后面了，但是整个也倒叙了

![image-20230516144733927](1-sql语句DQL单表查询.assets/image-20230516144733927.png)



![image-20230516144852320](1-sql语句DQL单表查询.assets/image-20230516144852320.png)



###  group by  和 oder by组合使用

按班级分组统计学生年龄，按班级别号正序显示

![image-20230516145759510](1-sql语句DQL单表查询.assets/image-20230516145759510.png)

### 如何去掉null，注意having和order by的前后次序

![image-20230516150733430](1-sql语句DQL单表查询.assets/image-20230516150733430.png)

##### 大概就是：group by 再 having 然后order by，但是having是group by分组汇总后的过滤，就不太好了，

运算减少的思路，就是，先过滤，改成👇

![image-20230516152001762](1-sql语句DQL单表查询.assets/image-20230516152001762.png)



### LIMIT限定记录数(行数)

![image-20230516152205679](1-sql语句DQL单表查询.assets/image-20230516152205679.png)

![image-20230516152308422](1-sql语句DQL单表查询.assets/image-20230516152308422.png)

跳过前2行，取后续的几行👆



### in的语法

![image-20230516152714244](1-sql语句DQL单表查询.assets/image-20230516152714244.png)

等价写法

![image-20230516152801741](1-sql语句DQL单表查询.assets/image-20230516152801741.png)  

