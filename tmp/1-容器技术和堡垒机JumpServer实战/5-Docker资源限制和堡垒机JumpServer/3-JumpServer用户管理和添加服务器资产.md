# 第3节 JumpServer用户管理和添加服务器资产



呃呃呃，我的目的就是减少沙箱这种收费软件的LIC的使用，而期待Jumpserver来做安全审计的远程操作。



然后邮箱是要的，因为一般涉及告警发送，钉钉后面再看，api也要的。



架构上，肯定不会说先拨vpn再连jumpserver，直接公网加端口怼出去，反正有审计，关键还要做风险操作的及时告警，这个学完后自己折腾下。



# 邮箱设置



![image-20240716101218814](3-JumpServer用户管理和添加服务器资产.assets/image-20240716101218814.png)

其实也没啥好说的，就是常规配置，倒是这里有一个extmail的常规操作

我的extmail邮箱满了，web点击要么都是无标题，要么就直接页面报错了，所以需要清空一下邮件

1、删除收件箱、Junk(垃圾邮件)、Trash(垃圾箱)的邮件

2、删除用户Maildir下的extmail-curcache.db文件

![image-20240716101516117](3-JumpServer用户管理和添加服务器资产.assets/image-20240716101516117.png)

然后刷新web就好了

<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240716101544265.png" alt="image-20240716101544265" style="zoom:50%;" />



然后jumpserver的测试一下啊邮件就发过来了👆

![image-20240716101627428](3-JumpServer用户管理和添加服务器资产.assets/image-20240716101627428.png)





## 这里如果配置SSL 465发送邮箱会报错

<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240716102137298.png" alt="image-20240716102137298" style="zoom:50%;" />







<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240716102146638.png" alt="image-20240716102146638" style="zoom:40%;" />





### 你还在为^H而困恼嘛

```shell
stty erase ^H
```

关于容器里(其实其他地方也可能遇到)较多遇到Backspace往前删除，删不掉的清空，这个时候简单操作一下就行了

![recording](3-JumpServer用户管理和添加服务器资产.assets/recording1.gif)





其实backspace就是^H，所以最原初的你敲backspace，屏幕上显示的就是^H，所以需要优化下👆

<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240716104142150.png" alt="image-20240716104142150" style="zoom:50%;" />



### 其实正确的方法是安装bash

因为你就算解决backspace，你的上下左右键还是没办法弄，而切换到bash就都有了，如果没有bash就安装一下



<img src="3-JumpServer用户管理和添加服务器资产.assets/recording2.gif" alt="recording2" style="zoom:50%;" />



然后继续解决邮件465SSL发送报错DH长度的问题

![image-20240716113820450](3-JumpServer用户管理和添加服务器资产.assets/image-20240716113820450.png)



查文档

1、这是python 的处理方式

https://blog.csdn.net/weixin_47383889/article/details/125019751

2、这是openssl，linux里的处理方式

https://www.cnblogs.com/testzcy/p/17425364.html



3、这个好像才是正解

https://stackoverflow.com/questions/61626206/what-could-cause-dh-key-too-small-error



4、都不对，jumpserver的邮件发送好像是django里的模块实现，所以要从这里入手







进入容器的时候用的是sh，所以还得切bash，没有就安装

然后安装vim

```shell
sed -i 's|http://deb.debian.org/debian|https://mirrors.tuna.tsinghua.edu.cn/debian|g' /etc/apt/sources.list.d/debian.sources
sed -i 's|http://deb.debian.org/debian-security|https://mirrors.tuna.tsinghua.edu.cn/debian-security|g' /etc/apt/sources.list.d/debian.sources


apt update
apt install vim
```



4、都不对，jumpserver的邮件发送好像是django里的模块实现，所以要从这里入手

https://kb.fit2cloud.com/?p=77

![image-20240716120722619](3-JumpServer用户管理和添加服务器资产.assets/image-20240716120722619.png)         



再安装postfix

```shell
yum -y install postfix
systemctl enable postfix
systemctl start postfix
```

再设置s-nail

```
set smtp=smtps://smtp.gmail.com:465  # 这行删掉就是25的smtp了
set smtp-auth=login
set smtp-auth-user=your-email@gmail.com
set smtp-auth-password=your-password
set from=your-email@gmail.com
set ssl-verify=ignore

```

![image-20240716140755234](3-JumpServer用户管理和添加服务器资产.assets/image-20240716140755234.png)



<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240716140947342.png" alt="image-20240716140947342" style="zoom:50%;" />





s-nail好像要配置一个真实的邮箱用户和密码了，但是老早的centos的postfix+mailx哪里就不需要，直接匿名发送了就。



上面就测试了jumpserver所在宿主是可以465发送邮件的，问题还是在jumpserver容器里的DH问题



jms_all容器里测试

使用mailutils提供mail命令好比s-nail，ssmtp提供postfix的邮件client端但是比postfix轻量级。

```shell
apt install ssmtp

apt install mailutils

----------------

root@f8fcc4d9b3a8:/opt# cat /etc/ssmtp/ssmtp.conf  |grep -Ev '^(#|$)'
root=postmaster
mailhub=mail.iwgame.com:25
hostname=f8fcc4d9b3a8
AuthUser=shenyiming@iwgame.com
AuthPass=Cisc0@123
UseTLS=NO
UseSTARTTLS=NO
FromLineOverride=YES

---------------

root@f8fcc4d9b3a8:/opt# cat /etc/ssmtp/revaliases
root:shenyiming@iwgame.com:mail.iwgame.com
ubuntu:shenyiming@iwgame.com:mail.iwgame.com


```





![image-20240716145707445](3-JumpServer用户管理和添加服务器资产.assets/image-20240716145707445.png)

OK的，所以定是jumpserver使用邮件发送模块里的DH的问题，可能就是django里发送的

不搞了，一键安装方式v4.0.0的版本也一样不行

docker安装的版本是

![image-20240716152520535](3-JumpServer用户管理和添加服务器资产.assets/image-20240716152520535.png)



一键脚本安装的版本是

![image-20240716152538766](3-JumpServer用户管理和添加服务器资产.assets/image-20240716152538766.png)

不搞了，一键安装方式v4.0.0的版本也一样不行

![image-20240716152556521](3-JumpServer用户管理和添加服务器资产.assets/image-20240716152556521.png)









# 操作

当前用户是admin登入的，

<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240716153707103.png" alt="image-20240716153707103" style="zoom:50%;" />



该角色身兼三职①管理员，操作jumpserver，②审计员，安全log查看的人，③使用jumpserver提供的跳转功能干活的，走工作台了就。

![image-20240716153645253](3-JumpServer用户管理和添加服务器资产.assets/image-20240716153645253.png)





## 创建账号

管理员走控制台

1、分组，开发，测试、运维等

![image-20240716154335169](3-JumpServer用户管理和添加服务器资产.assets/image-20240716154335169.png)



2、建用户

**新版本反而没有钉钉认证了，老版本有的**

![image-20240716154857407](3-JumpServer用户管理和添加服务器资产.assets/image-20240716154857407.png)





![image-20240716154923370](3-JumpServer用户管理和添加服务器资产.assets/image-20240716154923370.png)

这个密码选项图中的也能用，不过发给用户那边的链接其实就是127.0.0.1的地址，因为是容器里的嘛，也没有优化好，所以不推荐用这种方式。

<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240717095837891.png" alt="image-20240717095837891" style="zoom:50%;" />



然后测试了重置密码(会有邮件的交互)也都是ok的，就是同样要注意127.0.0.1改为真实的宿主IP的问题，要是能找到容器里的这个127.0.0.1的设置就好了。



不过还是不推荐邮件参与进来，直接给一个默认密码，然后让用户首次登入修改就很好。

![image-20240717100643620](3-JumpServer用户管理和添加服务器资产.assets/image-20240717100643620.png)







![image-20240717101237942](3-JumpServer用户管理和添加服务器资产.assets/image-20240717101237942.png)





![image-20240717101422416](3-JumpServer用户管理和添加服务器资产.assets/image-20240717101422416.png)



注意点编辑进去的系统角色显示有问题，记住你选啥，就是啥，外面确认是对的就行

![image-20240717101735451](3-JumpServer用户管理和添加服务器资产.assets/image-20240717101735451.png)









先看看审计员的界面

![image-20240717102048844](3-JumpServer用户管理和添加服务器资产.assets/image-20240717102048844.png)





<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240717102033518.png" alt="image-20240717102033518" style="zoom:50%;" />





![image-20240717102219769](3-JumpServer用户管理和添加服务器资产.assets/image-20240717102219769.png)



也没啥好看的，就是一堆日志，不过也有工作台就是没有配置任何机器给他用。主要还是看日志。



此时再开一个浏览使用别的账号登入一下，啥，为啥要别的浏览器，因为有cookie啊，chrome用的admin，chrom无痕用的审计员账号，所以再开一个firefox登入一个普通用户。

此时审计员可以看到记录，不管是在线记录，还是登入记录

![image-20240717103143111](3-JumpServer用户管理和添加服务器资产.assets/image-20240717103143111.png)



![image-20240717103206558](3-JumpServer用户管理和添加服务器资产.assets/image-20240717103206558.png)



连管理员的操作日志都能看到

![image-20240717103224947](3-JumpServer用户管理和添加服务器资产.assets/image-20240717103224947.png)



审计专员还是挺牛逼的，哈哈哈





普通人员只有工作台

![image-20240717103408843](3-JumpServer用户管理和添加服务器资产.assets/image-20240717103408843.png)









### 下面就开始添加资产

这里有一个底层做事情的逻辑

1、用户---通过跳板机---链接业务机器

​		用户通过账号进入跳板机，而跳板机分配该用户可以链接哪些机器

​		所以跳板机就需要事先连接好后端的很多机器

2、ansible就是如此：ansible要管理机器ABC，就要配置一个主机清单文件，然会为了方便就要做key验证。

3、jumpserver亦是如此：把后端需要远程访问的机器配置到jumpserver资产里，后面分配给不同的用户。

​			这些jumpserver关联的后端资产可以有(一般就是linux和windows)：服务器(linux\windows\Unix)、数据库、网络设备、应用、K8S。

​			涉及到另外的账号概念：早期jumpserver称之为，现在统一叫系统用户里的特权和普通👇



### jumpserver里的三种用户

前面梳理了  管理员、审计员、用户，现在



关于账号的理解，这里有一个宇宙法则：谁的，什么系统的账号，就TMD从这个角度去梳理就一目了然。就问你屌不屌~一念之间，天地翻转，地天泰啦~



1、登录用户

​		① 管理员 ②审计员 ③用户

​		说白了就是jumpserver自身的账号。



2、系统用户 里的 特权用户 ，早期jumpsever版本叫 管理用户

​		对后端服务器具有管理权限的账号root或administrator以及sudo ALL权限的用户，用于管理后端服务器

​		说白了就是后端的root账号



3、系统用户 里的 普通用户，早期jumpserver版本叫 系统用户

​		给登录用户使用ssh连接后端服务时对应的系统用户，一般是后端服务器的普通的系统用户账号

​		说白了就是后端的普通账号





<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240717135141735.png" alt="image-20240717135141735" style="zoom:50%;" />



jumpserver内置了ansible，拿后端的root账号是可以批量去后端机器上创建账号的

然后用户通过jumpsever的账号 登录 jumpserver后，就可以使用批量的各个账号去登入 后端机器了。

xiaoming 图中的xiaoming哦，看图👆，通过xiaoming账号登入jumpserver，jumpserver通过不同机器的各自的root账号+ansible批量创建下发后端机器的所需用户ID，就是哪些linux机器的用户，windows的也有ansible？回头看看



![image-20240717145121122](3-JumpServer用户管理和添加服务器资产.assets/image-20240717145121122.png)

用户管理：就是jumpserver自己的登入账号

账号管理：就是后端服务器的账号，root和master以及其他的



新版本，root还是普通就一个选项就搞定了👇

![image-20240717150215590](3-JumpServer用户管理和添加服务器资产.assets/image-20240717150215590.png)



往前的版本如下👇就是系统用户里的 普通 和 特权 之分，

![image-20240717145942547](3-JumpServer用户管理和添加服务器资产.assets/image-20240717145942547.png)



### 开始创建账号

1、直接创建账号是要有前置条件的，需要关联资产也就是这个账号登入的后端机器，所以要去新建资产

2、可以先新建账号模板，将来创建账号的时候调用就行了

3、新建资产-也就是后端的机器



实验开始

先准备两台机器作为跳板机的资产

就133 和 134吧，

![image-20240717161051133](3-JumpServer用户管理和添加服务器资产.assets/image-20240717161051133.png)





1、**先创建账号模板，**不同后端机器使用不同的管理员账号，一般来讲偷懒就是dev环境就用一个哈哈。

![image-20240717161345016](3-JumpServer用户管理和添加服务器资产.assets/image-20240717161345016.png)

图中的自动推送，肯定是在这个图中是不能选的啦，因为推送本质上jumpserver应该就是用的ansible，所以要先有root权限才能推的，或者做了key登入理论上分析这样的没错的。



顺便看看自动推送

![image-20240717162047712](3-JumpServer用户管理和添加服务器资产.assets/image-20240717162047712.png)



![image-20240717162120130](3-JumpServer用户管理和添加服务器资产.assets/image-20240717162120130.png)



然后看

![image-20240717162132136](3-JumpServer用户管理和添加服务器资产.assets/image-20240717162132136.png)

这里并没有，所以相关性有待继续学习



**2、然后新建账号推送**

![image-20240717162829603](3-JumpServer用户管理和添加服务器资产.assets/image-20240717162829603.png)

图中的无需再次输入密码有待验证，因为少了一个老版本里的托管密码按钮。



老版本的逻辑也是类似，就是一个密码的自动生成，自动托管，什么意思，就是上图红色字表达的，你不用二次输入密码，不管是不是图中的按钮具体意思，总之这里的开发意图都是如此。因为我也是没有学完，这里直接下结论也是为了更好的梳理，要敢于下结论，不管他的具体作用是什么，一定是为了什么。要有这个态度和眼光，就好比机房精密空调，不管怎么个精密，一定是机房的需求--送风一定要到位，到什么位置，到机柜底部送来冷风，对吧。从最终结果同样也是最初需求出发，准没错。咦~怎么有点像 我不管过程，我就要结果，我日TMD。。。



再来一个账号推送，这个是测试账号

![image-20240717163725933](3-JumpServer用户管理和添加服务器资产.assets/image-20240717163725933.png)



![image-20240717163740112](3-JumpServer用户管理和添加服务器资产.assets/image-20240717163740112.png)





1和2 两步动作，相当于 普通账号有了，特权账号也有了，下面就开始关联后端资产了。

所以前文的操作，就是 新建账号模板-且都是特权；新建账号推送-且都是普通。思路倒是OK的，也是从需求出发的，特权的用来连接后端机器，普通账号就是推送过去，挺好的，就不知道界面里的 账号模板里的 普通账号 自动推送怎么玩的，估计也是类似的逻辑吧。



**3、新建资产**

两台机器 192.168.126.133和134，分别作为开发环境 和 测试环境的机器

我再搞一个windows吧，恩，放到实验最后工作案例吧，也是我的初衷



![image-20240717164738263](3-JumpServer用户管理和添加服务器资产.assets/image-20240717164738263.png)



然后就搞两个文件夹出来

<img src="3-JumpServer用户管理和添加服务器资产.assets/image-20240717164809530.png" alt="image-20240717164809530" style="zoom:50%;" />





![image-20240717165938798](3-JumpServer用户管理和添加服务器资产.assets/image-20240717165938798.png)



 好

![image-20240717171216163](3-JumpServer用户管理和添加服务器资产.assets/image-20240717171216163.png)



![image-20240717171816030](3-JumpServer用户管理和添加服务器资产.assets/image-20240717171816030.png)

能推送啥哦，不就是后端机器的普通账号咯。总感觉该产品的这里逻辑设计还不够简单。

选择立即推送后的界面无明显变化

![image-20240717172451609](3-JumpServer用户管理和添加服务器资产.assets/image-20240717172451609.png)

联想到要推送的就是普通账号，于是去 自动化-账号推送里看看，应该是后面要补充下资产来明确往什么机器推什么帐号了。



![image-20240717172614807](3-JumpServer用户管理和添加服务器资产.assets/image-20240717172614807.png)





下一篇继续~~休息下







