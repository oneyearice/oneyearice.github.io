# 第1节. 文本处理三剑客2_sed



## sed内置行为

- Stream EDitor, 行编辑器
- sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（ pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。然后读入下行，执行下一个循环。如果没有使诸如‘D’ 的特殊命令，那会在两个循环之间清空模式空间，但不会清空保留空间。这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。
- 功能：主要用来自动编辑一个或多个文件,简化对文件的反复操作,编写转换程序等
- 参考： http://www.gnu.org/software/sed/manual/sed.html  



sed就是用来解决我多地方dns文件的最佳实践，好好学。

sed命令内置特性，内置循环，内置一行行，和grep也是一样一行一行处理

内存空间在这里就叫模式空间

![img](1-文本处理三剑客2_sed.assets/clip_image004.jpg)

<img src="1-文本处理三剑客2_sed.assets/clip_image006.jpg" alt="img" style="zoom: 50%;" />

一行处理完 print，删除，第二行读进来，处理，print，删除，第三行继续



## sed 语法



![img](1-文本处理三剑客2_sed.assets/clip_image008.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image010.jpg)

 

 

![img](1-文本处理三剑客2_sed.assets/clip_image012.jpg)

地址+命令， 地址就是哪行，

![img](1-文本处理三剑客2_sed.assets/clip_image014.jpg)

 **sed '' passwd**   👈这个就是内置行为的证明了，就是每行打印一遍

下面是脱裤子放屁的命令，是不是很吊~👇

![img](1-文本处理三剑客2_sed.assets/clip_image016.jpg)

### 2P- 只打印第2行

![img](1-文本处理三剑客2_sed.assets/clip_image018.jpg)

 sed是读取标准输入处理的👇，有这玩意--标准输入，就可以利用管道；没有就利用xargs一样传！

![img](1-文本处理三剑客2_sed.assets/clip_image020.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image022.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image024.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image026.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image028.jpg)

### 最后一行  

tail -n1就行，sed 10 | tail -1或seq 10 | sed -n '$p'



在最后一行添加文本

sed '$ a xxx' /etc/passwd

sed '10 a xxx' /etc/passwd 就是在第10行下面插入，$在这个位置就是最后一行的行号

sed '/nginx/s@xxx@zzz@' /etc/passwd 这就是查找nginx关键字的那一行进行替换，不同于上面的 第几行 定位。

![image-20230727095638576](1-文本处理三剑客2_sed.assets/image-20230727095638576.png)

![image-20230727095715976](1-文本处理三剑客2_sed.assets/image-20230727095715976.png)

![image-20230727095733644](1-文本处理三剑客2_sed.assets/image-20230727095733644.png)



下图最后一样是cli，上面的内容是cli的结果

![image-20230727095909851](1-文本处理三剑客2_sed.assets/image-20230727095909851.png)

![image-20230727100209948](1-文本处理三剑客2_sed.assets/image-20230727100209948.png)





### 正则匹配

<img src="1-文本处理三剑客2_sed.assets/clip_image030.jpg" alt="img" style="zoom:33%;" />

![img](1-文本处理三剑客2_sed.assets/clip_image032.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image034.jpg)

基本上还是grep的等价命令。还没有体现sed的自己的活。

###  行的范围

<img src="1-文本处理三剑客2_sed.assets/clip_image036.jpg" alt="img" style="zoom: 33%;" />

![img](1-文本处理三剑客2_sed.assets/clip_image038.jpg)

显示3-5行

![img](1-文本处理三剑客2_sed.assets/clip_image040.jpg)

👆上图可以用来过滤出你想要的日志--几点几分到几点几分。



### 步进行-实现打印奇偶行

<img src="1-文本处理三剑客2_sed.assets/image-20220211113740326.png" alt="image-20220211113740326" style="zoom:50%;" />



![img](1-文本处理三剑客2_sed.assets/clip_image045.png)

 

![img](1-文本处理三剑客2_sed.assets/clip_image046.png)

 

![img](1-文本处理三剑客2_sed.assets/clip_image048.jpg)

 

## sed编辑命令-这个只是print的时候修改下，不修改原文件

<img src="1-文本处理三剑客2_sed.assets/image-20220211114150887.png" alt="image-20220211114150887" style="zoom: 50%;" />

![img](1-文本处理三剑客2_sed.assets/clip_image052.jpg)

d 删除模式空间匹配的行，并立即启用下一轮循环

![img](1-文本处理三剑客2_sed.assets/clip_image056.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image058.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image060.jpg)

两个sed👆其实可以合成一个sed来做👇，两次操作合并▲

![img](1-文本处理三剑客2_sed.assets/clip_image062.jpg)

但是要注意下这个是不是真的就是两个sed合在一起，看下图👇就知道明显不是。

我觉得正确解释就是，针对处理内容，进行分号前后的两个命令的执行。这也是还原了最基本的逻辑。①👆针对/etc/fstab执行去注释;和去空行的动作②👇针对/etc/passwd执行了打印10行和打印20行的动作。你看有的就是两个sed合并，有的就不是，哈哈。神奇嘛，其实不神奇，就是一个常规操作在不同场景(一个描述在不同语境)里的不同解释。

一个是d删除操作，一个是p选择操作；删除自然两个动作可以用|管道符两个sed作为前后传参，说~哦，我一个文件删除这个再删除那个；而挑选的动作就是我针对这个文件挑选这个，再挑选那个。一个圆圈挖掉两个洞和一个圆圈取出两个洞，都是针对圈圈这个实体，但是挖掉两个洞和先挖一个后的结果作为后挖动作的输入是一致的；而一个圆圈取出两个洞，就不能说我取出的那个洞作为后取得输入参数，这就是逻辑上好玩地方，我希望我自己能把这些看似不好理解，但是实际是一会事得东西啊，有时间又机会琢磨透，世人常说转牛角尖，我很小的时候就想过很小很小10岁，小学吧好像，就说钻出来不就行了，哈哈，其实底层知识逻辑就是哲学了，换了个名字，世人就认了，世人是愚昧的。但是这样的人当前社会很难成功，因为给他的时间不够，他也要玩啊，哈哈哈哈哈~~~~~

![image-20220930134559276](1-文本处理三剑客2_sed.assets/image-20220930134559276.png)

问：下面的[\\]是啥东东，答：答NM，是转义，举例

![image-20230727163308500](1-文本处理三剑客2_sed.assets/image-20230727163308500.png)<img src="1-文本处理三剑客2_sed.assets/clip_image064.jpg" alt="img" style="zoom: 67%;" />

![img](1-文本处理三剑客2_sed.assets/clip_image066.jpg)



同样对原文件未做修改

如果想要改

## sed修改原文件-i，为了安全推荐-i.bak

<img src="1-文本处理三剑客2_sed.assets/clip_image068.jpg" alt="img" style="zoom:50%;" />

![img](1-文本处理三剑客2_sed.assets/clip_image070.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image072.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image074.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image076.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image078.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image080.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image082.jpg)

 👆/^Listen/i    listen 8080，这里你几个空格都没用，如果你要插入的字符前面带空额，就需要上图的

​    /^Listen/i\   listen 8080，转义下

![img](1-文本处理三剑客2_sed.assets/clip_image084.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image086.jpg)

 **注意c是找到行后，整行替换**

![img](1-文本处理三剑客2_sed.assets/clip_image088.jpg)



<img src="1-文本处理三剑客2_sed.assets/clip_image090.jpg" alt="img" style="zoom:50%;" />

 

![img](1-文本处理三剑客2_sed.assets/clip_image092.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image094.jpg)

 

##  sed另存为，之前是修改原文件或是备份，这个直接是另存为

![img](1-文本处理三剑客2_sed.assets/clip_image096.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image098.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image100.jpg)

 👆注意d;w用;分开来，两次操作合并▲

<img src="1-文本处理三剑客2_sed.assets/clip_image102.jpg" alt="img" style="zoom: 50%;" />

 

![img](1-文本处理三剑客2_sed.assets/clip_image104.jpg)

a是追加文本，r是追加整个文件的内容，注意sed -r和上图的r不是一个东西哦。

 **其实就是位置+动作，位置就是/正则或者行号之类/ ，都工作就是这里的a也好c也好。**



### 包含root行的行号

![img](1-文本处理三剑客2_sed.assets/clip_image106.jpg)

 

 

<img src="1-文本处理三剑客2_sed.assets/clip_image108.jpg" alt="img" style="zoom:50%;" />

 

![img](1-文本处理三剑客2_sed.assets/clip_image110.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image112.jpg)

 

## 查找替换

![img](1-文本处理三剑客2_sed.assets/clip_image114.jpg)

这里和vim里面很像

![img](1-文本处理三剑客2_sed.assets/clip_image116.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image118.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image120.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image122.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image124.jpg)

### sed -e 的用法-等价于上面的两次操作合并▲

![img](1-文本处理三剑客2_sed.assets/clip_image126.jpg)

###  例子，找出IP地址

![img](1-文本处理三剑客2_sed.assets/clip_image128.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image130.jpg)

 sed -r 是表示后面使用扩展正则，而sed /基本正则/    ▲正则

![img](1-文本处理三剑客2_sed.assets/clip_image132.jpg)

这里用到了分组()的概念

![img](1-文本处理三剑客2_sed.assets/clip_image134.jpg)

这个和py里的format字符串格式化一样。再合并一下整成一个sed命令👇

![img](1-文本处理三剑客2_sed.assets/clip_image136.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image138.jpg)

这个其实就等价于py里的rematch，而不是refind，而refind不用写全。rematch还需要**写全**。确实要用正则匹配全了，证明：

![image-20220211140059067](1-文本处理三剑客2_sed.assets/image-20220211140059067.png)

优化下

![image-20220211140609198](1-文本处理三剑客2_sed.assets/image-20220211140609198.png)

牛逼的是👆(())这种写法它能识别好。

 

### 例子，取消注释

![img](1-文本处理三剑客2_sed.assets/clip_image139.png)

先定位位置

![img](1-文本处理三剑客2_sed.assets/clip_image140.png)

上图是系统判定为/<词首锚定了。  # 这个说发也是不对！sed 的-r没有单词锚定的/<  />的用法，至少查找的//是占用了的。不行你猫猫看。

![img](1-文本处理三剑客2_sed.assets/clip_image142.jpg)

 批量取消注释▲，这个vim里也有操作比如ctrl+v  ,I,  # 两下esc注意是vim不是vi。

![img](1-文本处理三剑客2_sed.assets/clip_image144.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image146.jpg)

 

###  例子，sed实现dirname和basename

![img](1-文本处理三剑客2_sed.assets/clip_image148.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image150.jpg)

 

###  例子，修改网卡名称为eth0，可能不对

![img](1-文本处理三剑客2_sed.assets/clip_image152.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image154.jpg)

思考这么对不对

**sed -rn '/ .\*linux/s/$/net.ifnames=0$/' /boot/grub2/grub.cfg**

**上面的显然不对，而且还多了一个$**

![img](1-文本处理三剑客2_sed.assets/clip_image156.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image158.jpg)

上图infname写错了改成ifname

![img](1-文本处理三剑客2_sed.assets/clip_image160.jpg)

**&就表示前面搜索出来的内容。注意下面的p参数拿掉，不然会将目标行复制2遍。**

![img](1-文本处理三剑客2_sed.assets/clip_image162.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image164.jpg)

 

```
成功案例
[root@centos7 ~]# sed -rn 's/^[[:space:]]+linux16.*/& net.ifnames=0/p' /boot/grub2/grub.cfg
        linux16 /vmlinuz-3.10.0-1160.el7.x86_64 root=UUID=db575dcc-512a-4240-ab49-f3d41bc3e372 ro crashkernel=auto rhgb quiet LANG=en_US.UTF-8 net.ifnames=0
        linux16 /vmlinuz-0-rescue-8173b565dc324c7180303567796b941c root=UUID=db575dcc-512a-4240-ab49-f3d41bc3e372 ro crashkernel=auto rhgb quiet net.ifnames=0
[root@centos7 ~]#
[root@centos7 ~]# sed -ri.org 's/^[[:space:]]+linux16.*/& net.ifnames=0/' /boot/grub2/grub.cfg

[root@centos7 ~]# ll /boot/grub2/grub*
-rw-r--r--. 1 root root 4267 Feb 11 14:47 /boot/grub2/grub.cfg
-rw-r--r--. 1 root root 4239 Jan  5 17:45 /boot/grub2/grub.cfg.bak
-rw-r--r--. 1 root root 4239 Jan  5 17:45 /boot/grub2/grub.cfg.org
-rw-r--r--. 1 root root 1024 Jan  5 17:45 /boot/grub2/grubenv
[root@centos7 ~]# cat /boot/grub2/grub.cfg |grep ifname
        linux16 /vmlinuz-3.10.0-1160.el7.x86_64 root=UUID=db575dcc-512a-4240-ab49-f3d41bc3e372 ro crashkernel=auto rhgb quiet LANG=en_US.UTF-8 net.ifnames=0
        linux16 /vmlinuz-0-rescue-8173b565dc324c7180303567796b941c root=UUID=db575dcc-512a-4240-ab49-f3d41bc3e372 ro crashkernel=auto rhgb quiet net.ifnames=0

重启后
[root@centos7 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:b9:89:eb brd ff:ff:ff:ff:ff:ff
    inet 192.168.25.44/24 brd 192.168.25.255 scope global noprefixroute dynamic eth0
       valid_lft 64705sec preferred_lft 64705sec
    inet6 fe80::4efd:3be2:da5a:12cb/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
[root@centos7 ~]#
[root@centos7 ~]#
[root@centos7 ~]#

```

然后再补一个这个：也不知道是不是上面需求擦头必须的：

![img](1-文本处理三剑客2_sed.assets/clip_image166.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image168.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image170.jpg)

或者

![img](1-文本处理三剑客2_sed.assets/clip_image172.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image174.jpg)

 

```
[root@centos7 ~]# cat /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
[root@centos7 ~]#
[root@centos7 ~]#
[root@centos7 ~]# sed -rn 's/(.*CMD.*)"$/\1 net.ifnames=0"/p' /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet net.ifnames=0"

这样👇更好，我找到不直接s///，而是先//再s///也就是/zz/s#xx#yy#m，因为/CMD/是找出这一行然后再查找部分字符进行替换，而s///可能就匹配的过多，理由不充分哈哈~还没没找到
[root@centos7 ~]# sed -rn '/CMD/s/"$/net.ifnames=0"/p' /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quietnet.ifnames=0"
[root@centos7 ~]#
👇如果是直接s@@@，会全部行都直接换了 “部分” 字符比如s@CMD@zzz@，
[root@centos7 ~]# sed -rn 's/"$/net.ifnames=0"/p' /etc/default/grub
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)net.ifnames=0"
GRUB_TERMINAL_OUTPUT="consolenet.ifnames=0"
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quietnet.ifnames=0"
GRUB_DISABLE_RECOVERY="truenet.ifnames=0"
[root@centos7 ~]# sed -rn 's/CMD/net.ifnames=0"/p' /etc/default/grub
GRUB_net.ifnames=0"LINE_LINUX="crashkernel=auto rhgb quiet"
[root@centos7 ~]#
[root@centos7 ~]# sed -rn 's/CMD//p' /etc/default/grub
GRUB_LINE_LINUX="crashkernel=auto rhgb quiet"
[root@centos7 ~]# sed -rn 's/CMD/&/p' /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet"
[root@centos7 ~]#

s///是基于行去进行字符替换，//是找到这一行。
```

![img](1-文本处理三剑客2_sed.assets/clip_image176.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image178.jpg)

不加g，就是只处理每行第一个命中的，g加上就是行内所有都替换。

![img](1-文本处理三剑客2_sed.assets/clip_image180.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image182.jpg)

 

\---------------

###  sed 中的变量要注意双引号的基本常识

![img](1-文本处理三剑客2_sed.assets/clip_image184.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image186.jpg)

### sed自己可以👇这样，在单引号里使用变量

![img](1-文本处理三剑客2_sed.assets/clip_image188.jpg)

这是sed的自己的用法，比较少见。



存在这种情况  'xx'''$var'''xx"xxx'，这样可以使用变量了，然后可不可以这样

![img](1-文本处理三剑客2_sed.assets/clip_image190.jpg)

"xx$varxx\\"xx"   使用转义可以不，可以的吧，可以的

所以要啥自行车~，注意哦，下图三引号不是在sed里用的，是我在echo玩的，别搞混了，本段讨论的是sed里如何使用变量以及转义的双引号，哈哈。 ![image-20220211151946285](1-文本处理三剑客2_sed.assets/image-20220211151946285.png)

 然后上图的另一个点就是和sed一样，也支持单引号里表达变量

![image-20220211152306719](1-文本处理三剑客2_sed.assets/image-20220211152306719.png)



 

##  sed高级命令-多了一个空间

**体现在sed内置的行为就是一个模式空间，高级就高级在多了一个保持空间**

 hold空间就是临时存一下

![image-20220211152542793](1-文本处理三剑客2_sed.assets/image-20220211152542793.png)



sed的强大之处还体现在高级命令及其保持空间。

![img](1-文本处理三剑客2_sed.assets/clip_image194.jpg)

 

![img](1-文本处理三剑客2_sed.assets/clip_image195.png)

hold住保持一会，待会取回来用。

![img](1-文本处理三剑客2_sed.assets/clip_image197.jpg)

模式空间里是可以放好几行的，D是删除一行，后续行就不删了。

 

### sed高级用法的的例子

<img src="1-文本处理三剑客2_sed.assets/clip_image199.jpg" alt="img" style="zoom: 33%;" />

###  							分析sed -n 'n;p' FILE

​					①初始

<img src="1-文本处理三剑客2_sed.assets/clip_image201.jpg" alt="img" style="zoom: 33%;" />



​							②打印第2行

<img src="1-文本处理三剑客2_sed.assets/clip_image203.jpg" alt="img" style="zoom:33%;" />

​							③读入第3行

<img src="1-文本处理三剑客2_sed.assets/clip_image205.jpg" alt="img" style="zoom:33%;" />

​							④同理

<img src="1-文本处理三剑客2_sed.assets/clip_image207.jpg" alt="img" style="zoom:33%;" />

​						⑤试试：

<img src="1-文本处理三剑客2_sed.assets/clip_image209.jpg" alt="img" style="zoom:50%;" />

 

### **看sed '1!G;h;$!d' FILE**

不是第一行就G，不是最后一样就删除，中间有G和h追加和覆盖操作，涉及两个模式空间

<img src="1-文本处理三剑客2_sed.assets/clip_image211.jpg" alt="img" style="zoom:50%;" />

 

 

<img src="1-文本处理三剑客2_sed.assets/clip_image213.jpg" alt="img" style="zoom:50%;" />

真要倒着写，tac就行了，

![img](1-文本处理三剑客2_sed.assets/clip_image215.jpg)

 ![image-20220211154217627](1-文本处理三剑客2_sed.assets/image-20220211154217627.png)

 

#### 例子sed 'N;D' FILE

![img](1-文本处理三剑客2_sed.assets/clip_image217.jpg)

![img](1-文本处理三剑客2_sed.assets/clip_image219.jpg)

 

<img src="1-文本处理三剑客2_sed.assets/image-20220211154606075.png" alt="image-20220211154606075" style="zoom: 50%;" />



 

 

 补充



linux每行结尾只有换行“\n”，

Windows每行结尾是换行+回车“\r\n”

Mac OS 为 “\r”。

用dos2unix file 活unix2dos file命令直接转换，有时候是最小化安装，所以vim里的方法也是要会的。

利用Linux下的vim，去除^M，去之前file看一下

![image-20220221113730252](1-文本处理三剑客2_sed.assets/image-20220221113730252.png)

vi xxx

然后
:set ff

用于查看当前文件是dos格式还是unix格式，显示如下：

![image-20220221113539901](1-文本处理三剑客2_sed.assets/image-20220221113539901.png)

切换为unix格式，然后保存即可：

:set ff=unix      #👈转换为unix格式
:wq

![image-20220221113637003](1-文本处理三剑客2_sed.assets/image-20220221113637003.png)

![image-20220221113651671](1-文本处理三剑客2_sed.assets/image-20220221113651671.png)

如果上图的格式不对，直接./xxx.py是找不到文件的，只能用python xxx变通运行，这里改成unix换行符后，就可以直接./xxx.py运行了。

![image-20220221114104635](1-文本处理三剑客2_sed.assets/image-20220221114104635.png)

