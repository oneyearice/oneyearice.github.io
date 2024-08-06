# 第5节 JumpServer管理数据库资产和批量导入导出





# 1、先准备一个mysql作为后端资产



1、ubuntu

```shell
sed 

apt update
apt -y install mysql-server

vim /etc/mysql/mysql.conf.d/mysqld.cnf
#bind-address		=127.0.0.1
#mysqlx-bind-address	= 127.0.0.1

# 也可以用sed 直接注释掉127.0.0.1的行
sed -i '/127.0.0.1/s/^/#/' /etc/mysql.conf.d/mysqld.cnf

systemctl restart mysql

# 检查端口监听是不是不再是127了
ss -tlnup  # 看到*:3306 就可以了

mysql
mysql>create database wordpress;
mysql>create user wordpress@'192.168.%.%' identified by '123456';
mysql>grant all on wordpress.* to wordpress@'192.168.%.%';

```



2、红帽系统

```shell
sed


yum -y install mysql-server



systemctl enable --now mariadb

mysql
mysql>create database wordpress;
mysql>create user wordpress@'192.168.%.%' identified by '123456';
mysql>grant all on wordpress.* to wordpress@'192.168.%.%';
mysql>create table wordpress.t1(id int);
mysql>exit

```





# 2、jumpserver上配置DB

![image-20240719105731631](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719105731631.png)



![image-20240719105933668](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719105933668.png)



然后授权，可以利旧的

![image-20240719110115611](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110115611.png)

不过要改一下里面的 授权账号

![image-20240719110204516](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110204516.png)

上图👆二选一，要么所有账号，要么精确到wordpress账号



![image-20240719110340600](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110340600.png)



然后用 王麻子测试

![image-20240719110402034](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110402034.png)



![image-20240719110526495](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110526495.png)

不好意思mysql的IP地址写错了。。。

![image-20240719110511419](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110511419.png)



改掉就好了



![image-20240719110634335](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110634335.png)



这里可见一个命名规范👇

![image-20240719110919777](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110919777.png)

去改掉

![image-20240719110939664](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719110939664.png)



因为这个连接jumpserver是一进来就是wordpress这个数据库里的了👆。



![image-20240719111606464](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719111606464.png)



反正就是mariadb的提示符不太友好，CLI界面也就是2222端口连接jumpserver也是一样

![image-20240719113252169](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719113252169.png)



然后已经可以操作了，但是连接性还是显示错误，显然界面刷新或者显示判定有问题，其实是OK的。

![image-20240719113508288](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719113508288.png)



![image-20240719113539459](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719113539459.png)

大概知道了，可能要👆手动点一下三个点里面的  测试 一下就可以了。

看看mysql提示符一样，不咋地👇，不过连接性都是能过够正确显示了。

![image-20240719113832276](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719113832276.png)



![image-20240719113755814](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719113755814.png)







# 资产的批量添加

手动新建一台windows后端资产

然后导出来当作模板

再ctrl+c   ctrl+v 写多个机器

然后上传就行了



就是节点也就是文件夹的id注意下

![image-20240719120018434](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719120018434.png)







# 同步的按钮的作用 看下

1、模板的信息修改

![image-20240719144745743](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719144745743.png)





2、然后调用模板的所有地方不会自动同步的，需要手动的

![image-20240719144844269](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719144844269.png)



![image-20240719144857193](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719144857193.png)





3、修改和同步操作备忘

![image-20240719144940259](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719144940259.png)

此时 资产列表  和 账号列表 里的 用户都不会自动更新的

手动进去模板里进行同步👇

![image-20240719145128048](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719145128048.png)



此时 资产列表 和 账号列表 里就更新了



# 推送账号了解下

### 我推1-自动推送账号一般就是普通账号-看下

使用场景：用在单独下发账号的时候

用root去ansible推的

![image-20240719150021684](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719150021684.png)

这里就是要给独立的推送账号的模块，不过前提是 root 这个特权账号要先配到到推送的机器上👇

![image-20240719150112362](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719150112362.png)





### 我推2-还可以这里推

使用场景：首推这种使用方式，就是从源头就定义了要推下去的。

创建模板的时候就定义号自动推，就是谁用我这个模板，谁就自动推， 谁是谁，谁就是资产咯



![image-20240719150523867](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719150523867.png)





![image-20240719150637308](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719150637308.png)

确认后，等10s钟

![image-20240719150720149](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719150720149.png)

就退下去了，这是模板里自动退了，谁用了这个模板，谁就自动得到了



### 我推3-就是上一篇讲的 资产授权里 也可以推

使用场景：这个就是从碗底吃饭的人爱用的方式了，我上一篇也是这么用的。

![image-20240719150852247](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719150852247.png)

这个上一篇写过了，这里就简单看看上图就行了





### 我推4-账号列表里推

使用场景：也是修改的时候推的，就是不是一个从无到有的过程，而是不走模板，直接创建账号的时候，关联资产了，确认了，然后在外面的界面里也可以推



![image-20240719151752317](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719151752317.png)



![image-20240719151700646](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719151700646.png)



![image-20240719151816105](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719151816105.png)



![image-20240719151835170](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719151835170.png)

10s不到，也推下来了

![image-20240719151912218](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719151912218.png)







# 网域列表



![image-20240719160807216](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719160807216.png)



就是你的资产在互联网上，很多个地方，你不希望搭建jumpserver 1 2 3 那么多台，于是可以直接利用网域网关的逻辑去实现一个jumpserver搞定多地的资产跳登。

https://docs.jumpserver.org/zh/v4/guide/admin/asset/domain_list/?h=%E7%BD%91%E5%9F%9F

好像也没说啥，自己走一遍看看玩



## 一 、弄网域



### 1、创建网域

![image-20240719161606930](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719161606930.png)



![image-20240719161643941](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719161643941.png)







### 2、创建网关列表

点击进到上面创建的网域里，创建网关列表



找了一个我的QQ云上的机器的公网IP

![image-20240719162640465](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719162640465.png)



![image-20240719162603983](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719162603983.png)

此时netArea001这个网域里就有个一个网关，

1、当然这里可以创建多个GW，所以才叫网关列表，好比腾讯云这个网域里自然有多个VPC，一个VPC一个GW接入；

2、然后将需要跳板连接的机器也就是资产 去 "资产列表" 里创建好，

3、最后关联到这些资产到对应的网关里就行了



好实操开始





## 二、补资产



### 1、创建文件夹归类

<img src="5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719163409915.png" alt="image-20240719163409915" style="zoom:50%;" />



### 2、创建资产

![image-20240719163514188](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719163514188.png)



这里变通下，就用网关本身的内网IP充当后端资产就行咯👇

![image-20240719163631004](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719163631004.png)

上图👆的账号就新增一个root测试。



## 三、回到网域里

### 1、关联资产

![image-20240719163757368](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719163757368.png)



![image-20240719163834582](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719163834582.png)





👆上图连接性错误，也好理解，是QQ云上的内网IP，资产那边怎么可能是OK呢。



测试下，

![image-20240719164042316](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719164042316.png)

也不知道是通还是不通。。。因为如果走GW，就会通的。。。





## 四、资产授权



![image-20240719164314891](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719164314891.png)





## 五、测试-OK~



![image-20240719164406157](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719164406157.png)



看图👇是说gw不可用

![image-20240719164538544](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719164538544.png)



报错鸟，我的是2208，GW的SSH那里改下👇

![image-20240719164801711](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719164801711.png)



顺便测一下GW

![image-20240719164913298](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719164913298.png)





再次测试，看图👇是说gw dial 去连接资产失败

![image-20240719165024228](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719165024228.png)

哦，因为GW的内网IP，实验代替了后端资产，所以一样要改SSH为2208，👇改掉

![image-20240719165157924](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719165157924.png)





搞定~~

![image-20240719165306760](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719165306760.png)



掌声在哪里~~



不过连接性 还是显示不对，不管了

![image-20240719165450181](5-JumpServer管理数据库资产和批量导入导出.assets/image-20240719165450181.png)



跳板机的登入都好了，显示问题，或者这里的联通测试 没有走GW去测 不管了。

















