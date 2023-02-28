# 第7节. 实现internet架构和DNS服务



### 拓扑



<img src="7-实现internet架构和DNS服务.assets/image-20230228141956251.png" alt="image-20230228141956251" style="zoom:50%;" /> 





搭建权威dns的主从



#### 67作为httpd server，提供一个测试页

<img src="7-实现internet架构和DNS服务.assets/image-20230228142413178.png" alt="image-20230228142413178" style="zoom:60%;" /> 

<img src="7-实现internet架构和DNS服务.assets/image-20230228142345903.png" alt="image-20230228142345903" style="zoom:50%;" /> 

![image-20230228142438875](7-实现internet架构和DNS服务.assets/image-20230228142438875.png) 

![image-20230228142451987](7-实现internet架构和DNS服务.assets/image-20230228142451987.png) 





![image-20230228143141930](7-实现internet架构和DNS服务.assets/image-20230228143141930.png)

测试OK

![image-20230228143218486](7-实现internet架构和DNS服务.assets/image-20230228143218486.png) 

---

#### 







### 47上安装bind作为主MASTER

![image-20230228143408311](7-实现internet架构和DNS服务.assets/image-20230228143408311.png)

注释掉listen-和allow-

![image-20230228143535582](7-实现internet架构和DNS服务.assets/image-20230228143535582.png)

添加只允许从服务IP来抓取信息（-t axfr）

![image-20230228143636066](7-实现internet架构和DNS服务.assets/image-20230228143636066.png) 



![image-20230228144041934](7-实现internet架构和DNS服务.assets/image-20230228144041934.png) 





![image-20230228144220210](7-实现internet架构和DNS服务.assets/image-20230228144220210.png) 



![image-20230228144302726](7-实现internet架构和DNS服务.assets/image-20230228144302726.png) 



![image-20230228144326339](7-实现internet架构和DNS服务.assets/image-20230228144326339.png) 





### 57上配置从SLAVE

同样yum -y install bind

![image-20230228144559902](7-实现internet架构和DNS服务.assets/image-20230228144559902.png)

两行注释

一行拒绝所有

![image-20230228144754360](7-实现internet架构和DNS服务.assets/image-20230228144754360.png) 

![image-20230228144822002](7-实现internet架构和DNS服务.assets/image-20230228144822002.png) 

zone数据库文件从上面就无需创建了，重启同步过来就行了。





### 测试主从解析情况

client的dns指一下主从

 ![image-20230228145008642](7-实现internet架构和DNS服务.assets/image-20230228145008642.png)

看下是否生效--也就是/etc/resovle.conf里自动写进入了

![image-20230228145052726](7-实现internet架构和DNS服务.assets/image-20230228145052726.png) 

测试解析OK

![image-20230228145119417](7-实现internet架构和DNS服务.assets/image-20230228145119417.png) 

测试主从复制

![image-20230228145221562](7-实现internet架构和DNS服务.assets/image-20230228145221562.png) 

![image-20230228145229207](7-实现internet架构和DNS服务.assets/image-20230228145229207.png) 

slave上文件日期已经变了

![image-20230228145242627](7-实现internet架构和DNS服务.assets/image-20230228145242627.png) 



 ![image-20230228145316126](7-实现internet架构和DNS服务.assets/image-20230228145316126.png)



### 37配置成.com域

![image-20230228145733130](7-实现internet架构和DNS服务.assets/image-20230228145733130.png) 

属于父域的知识应用

![image-20230228145906318](7-实现internet架构和DNS服务.assets/image-20230228145906318.png) 



在zone数据库里配置子域

![image-20230228150404022](7-实现internet架构和DNS服务.assets/image-20230228150404022.png) 

![image-20230228150159491](7-实现internet架构和DNS服务.assets/image-20230228150159491.png) 

magedu是com的子域，被委派给了ns2和ns3，分别对应两个ip地址。

![image-20230228150418555](7-实现internet架构和DNS服务.assets/image-20230228150418555.png) 



测试下，委派成功👇

![image-20230228150643276](7-实现internet架构和DNS服务.assets/image-20230228150643276.png)





### 27作为根

![image-20230228151021401](7-实现internet架构和DNS服务.assets/image-20230228151021401.png) 

这里是自己实验里自建根，所以原来的根zone改一下

![image-20230228151208979](7-实现internet架构和DNS服务.assets/image-20230228151208979.png) 



![image-20230228152628947](7-实现internet架构和DNS服务.assets/image-20230228152628947.png)

![image-20230228152646572](7-实现internet架构和DNS服务.assets/image-20230228152646572.png)

chgrp named /var/named/root.zone

chmod 640 /var/named/root.zone



测试

![image-20230228153113436](7-实现internet架构和DNS服务.assets/image-20230228153113436.png) 



### 17作为forwardDNS

![image-20230228162519333](7-实现internet架构和DNS服务.assets/image-20230228162519333.png) 

修改默认的13个根的zone数据库文件

![image-20230228162605267](7-实现internet架构和DNS服务.assets/image-20230228162605267.png)

![image-20230228162750737](7-实现internet架构和DNS服务.assets/image-20230228162750737.png) 

只需要写一行：27就是实验的根

![image-20230228163059235](7-实现internet架构和DNS服务.assets/image-20230228163059235.png)

![image-20230228163134061](7-实现internet架构和DNS服务.assets/image-20230228163134061.png) 

测试下，失败：

![image-20230228163152454](7-实现internet架构和DNS服务.assets/image-20230228163152454.png)



关闭安全选项

<img src="7-实现internet架构和DNS服务.assets/image-20230228163231496.png" alt="image-20230228163231496" style="zoom:50%;" /> 

![image-20230228163247310](7-实现internet架构和DNS服务.assets/image-20230228163247310.png) 

然后就OK了

![image-20230228163302868](7-实现internet架构和DNS服务.assets/image-20230228163302868.png) 







### 7作为LOCALDNS

![image-20230228163747582](7-实现internet架构和DNS服务.assets/image-20230228163747582.png) 

![image-20230228163820443](7-实现internet架构和DNS服务.assets/image-20230228163820443.png) 

安全关了

![image-20230228163929780](7-实现internet架构和DNS服务.assets/image-20230228163929780.png) 



测试OK👇

 ![image-20230228163950356](7-实现internet架构和DNS服务.assets/image-20230228163950356.png)

测试curl之前改一下默认的dns

![image-20230228164055111](7-实现internet架构和DNS服务.assets/image-20230228164055111.png) 

![image-20230228164112127](7-实现internet架构和DNS服务.assets/image-20230228164112127.png) 

![image-20230228164122263](7-实现internet架构和DNS服务.assets/image-20230228164122263.png)

OK



