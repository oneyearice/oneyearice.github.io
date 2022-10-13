# 第3节. yum工作原理





![img](3-yum工作原理.assets/clip_image002.jpg)

 

![img](3-yum工作原理.assets/clip_image004.jpg)

在已经安装了autofs后，rpm -ql autofs，看下该包里的关键文件

![img](3-yum工作原理.assets/clip_image006.jpg)

以后看到

![img](3-yum工作原理.assets/clip_image007.png)

表示/usr/lib/systemd/system下面的都是服务。

 

##  基于光盘的yum源

<img src="3-yum工作原理.assets/clip_image008.png" alt="img" style="zoom:67%;" />

 

autofs开启后就可以实现自动挂载光盘路径了，号称神奇目录：直接访问/misc下面不存在的文件夹就可以自动挂载。

<img src="3-yum工作原理.assets/clip_image010.jpg" alt="img" style="zoom: 80%;" />

为什么一定要这个目录呢，因为要当yum源必须有一个repodata文件夹。

![img](3-yum工作原理.assets/clip_image012.jpg)

### 当yum源的前提：路径下有repodata

只要看到repodata文件夹，那么其所在的文件夹就是yum源的目录。不要看packages那些。

<img src="3-yum工作原理.assets/image-20220214113445205.png" alt="image-20220214113445205" style="zoom: 50%;" />

### yum repolist的本质 

![img](3-yum工作原理.assets/clip_image013.png)

 下载到哪里呢？通过yum.conf可见该目录

```
[root@centos7 ~]# cat /etc/yum.conf
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
... 省略...
[root@centos7 ~]# cd /var/cache/yum/
[root@centos7 yum]# ll
total 0
[root@centos7 yum]# l.
.  ..
[root@centos7 yum]# yum repolist   👈本质上就是把元数据的down到本地
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirrors.cn99.com
 * extras: ftp.sjtu.edu.cn
 * updates: ftp.sjtu.edu.cn
base                                                                                   | 3.6 kB  00:00:00
extras                                                                                 | 2.9 kB  00:00:00
updates                                                                                | 2.9 kB  00:00:00
(1/4): base/7/x86_64/group_gz                                                          | 153 kB  00:00:00
(2/4): extras/7/x86_64/primary_db                                                      | 243 kB  00:00:00
(3/4): updates/7/x86_64/primary_db                                                     |  13 MB  00:00:00
(4/4): base/7/x86_64/primary_db                                                        | 6.1 MB  00:00:00
repo id                                            repo name                                            status
base/7/x86_64                                      CentOS-7 - Base                                      10,072
extras/7/x86_64                                    CentOS-7 - Extras                                       500
updates/7/x86_64                                   CentOS-7 - Updates                                    3,411
repolist: 13,983
[root@centos7 yum]# ll
total 0
drwxr-xr-x. 3 root root 15 Feb 14 12:06 x86_64
[root@centos7 yum]#

[root@centos7 yum]# ll -R x86_64/
x86_64/:
total 0
drwxr-xr-x. 5 root root 87 Feb 14 12:06 7

x86_64/7:
total 8
drwxr-xr-x. 4 root root 278 Feb 14 12:06 base
drwxr-xr-x. 4 root root 183 Feb 14 12:06 extras
-rw-r--r--. 1 root root  84 Feb 14 12:06 timedhosts
-rw-r--r--. 1 root root 473 Feb 14 12:06 timedhosts.txt
drwxr-xr-x. 4 root root 183 Feb 14 12:06 updates

x86_64/7/base:
total 6376
-rw-r--r--. 1 root root 6351994 Oct 30  2020 6d0c3a488c282fe537794b5946b01e28c7f44db79097bb06826e1c0c88bad5ef-primary.sqlite.bz2
-rw-r--r--. 1 root root  156763 Oct 30  2020 a4e2b46586aa556c3b6f814dad5b16db5a669984d66b68e873586cd7c7253301-c7-x86_64-comps.xml.gz
-rw-r--r--. 1 root root       0 Feb 14 12:06 cachecookie
drwxr-xr-x. 2 root root      31 Feb 14 12:06 gen
-rw-r--r--. 1 root root     546 Feb 14 12:06 mirrorlist.txt
drwxr-xr-x. 2 root root       6 Feb 14 12:06 packages
-rw-r--r--. 1 root root    3736 Oct 30  2020 repomd.xml

x86_64/7/base/gen:
total 30876
-rw-r--r--. 1 root root 31614976 Oct 30  2020 primary_db.sqlite

x86_64/7/base/packages:
total 0

x86_64/7/extras:
total 256
-rw-r--r--. 1 root root      0 Feb 14 12:06 cachecookie
-rw-r--r--. 1 root root 248733 Sep  3 23:22 db1c88508275ffebdc6cd8686da08745d2552e5b219b2e6f4cbde7b8afd3b1a3-primary.sqlite.bz2
drwxr-xr-x. 2 root root     31 Feb 14 12:06 gen
-rw-r--r--. 1 root root    589 Feb 14 12:06 mirrorlist.txt
drwxr-xr-x. 2 root root      6 Feb 14 12:06 packages
-rw-r--r--. 1 root root   2998 Sep  3 23:22 repomd.xml

x86_64/7/extras/gen:
total 1296
-rw-r--r--. 1 root root 1326080 Sep  3 23:22 primary_db.sqlite

x86_64/7/extras/packages:
total 0

x86_64/7/updates:
total 13736
-rw-r--r--. 1 root root 14049533 Feb  9 04:02 c96f20635c7f289398519818a077b294f1855722181378b5105f5ef49f0cf57a-primary.sqlite.bz2
-rw-r--r--. 1 root root        0 Feb 14 12:06 cachecookie
drwxr-xr-x. 2 root root       31 Feb 14 12:06 gen
-rw-r--r--. 1 root root      589 Feb 14 12:06 mirrorlist.txt
drwxr-xr-x. 2 root root        6 Feb 14 12:06 packages
-rw-r--r--. 1 root root     3011 Feb  9 04:02 repomd.xml

x86_64/7/updates/gen:
total 75048
-rw-r--r--. 1 root root 76846080 Feb  9 04:02 primary_db.sqlite

x86_64/7/updates/packages:
total 0


[root@centos7 yum]# yum clean all    👈这下理解到位了，清的哪里，就是这里
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
Cleaning up list of fastest mirrors

[root@centos7 yum]# ls -R x86_64/
x86_64/:
7

x86_64/7:
base  extras  timedhosts  updates

x86_64/7/base:
gen  packages

x86_64/7/base/gen:

x86_64/7/base/packages:

x86_64/7/extras:
gen  packages

x86_64/7/extras/gen:

x86_64/7/extras/packages:

x86_64/7/updates:
gen  packages

x86_64/7/updates/gen:

x86_64/7/updates/packages:

也可以用du -sh /var/cache/yum验证可见大小为0

```

yum clean all，如果你修改yun源后不清理，就是照着就得yum源那会的yum repolist下载下来的元数据缓存去找rpm包的。所以要清一下的。

### yum的问题主要就2个，①路径写错了②缓存没清。你别跟我讲rpm数据库删了。③/etc/yum.repos.d/下面只要有一个repo文件不能用就会影响所有的repo源。



下面是将base源和epel源合在一起讲的： 

![img](3-yum工作原理.assets/clip_image014.png)

![img](3-yum工作原理.assets/clip_image015.png)

 上图👆是epel的alias 快速启用和禁用。主要是考虑安装rpm包的时候，不让他再去找epel，因为base是再本地的，而epel在网上，所以即使你安装base也会去epel找一遍，浪费时间的。不过yum 本地仓库其实是将base和epel都拉下来的，所以实际工作中base和epel都是内网本地的仓库。也无需禁用epel。

![img](3-yum工作原理.assets/clip_image017.jpg)

👆上图是将base源和epel合在一个.repo文件里的，也可以分开来，将epel单独做一个epel.repo文件。

这两个应该是从上往下，能走哪里走哪个。



下图是key的写法

![img](3-yum工作原理.assets/clip_image019.jpg)

 key启用后的第一次安装rpm包，会问你是否导入key，👇下图，

![img](3-yum工作原理.assets/clip_image020.png)

然后后面再安装其他rpm包就不会问你了。





![img](3-yum工作原理.assets/clip_image022.jpg)

![img](3-yum工作原理.assets/clip_image024.jpg)

YUM出问题就两个：①配置文件以及路径问题，②缓存清一下

![img](3-yum工作原理.assets/clip_image026.jpg)

 

 

![img](3-yum工作原理.assets/clip_image028.jpg)

比如要安装sl这个小火车，就要删除yum仓库，清除缓存，安装yum源，epel源，比如centos8

https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.23961b11BCsMUT

yum cleall all

yum makcache

yum updata

https://developer.aliyun.com/mirror/epel

 



### 安装会自动给你安装依赖，那么卸载是否会自动卸载依赖包呢

默认不会自动卸载依赖包的，因为可能别的包也会依赖这些依赖包。

 

yum list可以看到安装历史

![img](3-yum工作原理.assets/clip_image029.png)

实际上yum安装的时候也有自己的log，👇

![img](3-yum工作原理.assets/clip_image031.jpg)

Centos8就是dnf.log

###  yum history

![img](3-yum工作原理.assets/clip_image033.jpg)

**info 11可见当时做的事情，比如command line install mariadb-server。**

![img](3-yum工作原理.assets/clip_image034.png)

把这个事件撤销

undo就是卸载掉

![img](3-yum工作原理.assets/clip_image035.png)

![img](3-yum工作原理.assets/clip_image037.jpg)

这样就卸掉了，然后看下yum事件，卸掉了10个包Altered

![img](3-yum工作原理.assets/clip_image039.jpg)

如果你又不想卸了，还可以yum事件回去

redo就是重装一遍。

![img](3-yum工作原理.assets/clip_image041.jpg)

 

![img](3-yum工作原理.assets/clip_image043.jpg)

 baseurl还有一个mirroslist(这个后面讲)

baseurl可以写多个，如下👇图：

![image-20220214140423396](3-yum工作原理.assets/image-20220214140423396.png) 

写多个baseurl，都生效，

![image-20220214140638214](3-yum工作原理.assets/image-20220214140638214.png)

上图是光盘禁用后，走的网络yum安装的。但是这多个baseurl，优先用谁呢？答案还是在上面的PPT里：failovermethod={roundrobin|priority},cost的值越大优先级越低。



### 然后是yum的baseurl的4种路径： 

![img](3-yum工作原理.assets/clip_image045.jpg)

### yum仓库的内置变量

![img](3-yum工作原理.assets/clip_image047.jpg)

 

![img](3-yum工作原理.assets/clip_image049.jpg)

 

 

![img](3-yum工作原理.assets/clip_image051.jpg)

自己写脚本，里面的yum源可以使用多行重定向来写，👇▲脚本惯用手法-cat多行重定向写文件内容。

![img](3-yum工作原理.assets/clip_image053.jpg)

![img](3-yum工作原理.assets/clip_image055.jpg)

别忘了别名、vim、history格式、PS1、yum源。



 

 

![img](3-yum工作原理.assets/clip_image057.jpg)

 ![image-20220214142412372](3-yum工作原理.assets/image-20220214142412372.png)

yum list就列出了这1w多个包，一共就是1w多行：

![img](3-yum工作原理.assets/clip_image059.jpg)

yum 看到已安装的包是@打头

![img](3-yum工作原理.assets/clip_image061.jpg)

anaconda是操作系统安装向导的时候安装的程序，所以yum list不仅看到哪些已安装，还知道从哪装的，①比如anaconda②还有base仓库③epel等其他仓库。

 

![img](3-yum工作原理.assets/clip_image063.jpg)

这两个包是base仓库创建后，利用yum安装上去的。

 比如，下图👇其实查看已经安装的可以yum list installed更快，必进grep要CPU计算的。

![image-20220214143356426](3-yum工作原理.assets/image-20220214143356426.png)

![image-20220214143439794](3-yum工作原理.assets/image-20220214143439794.png)

![image-20220214143509801](3-yum工作原理.assets/image-20220214143509801.png)



![img](3-yum工作原理.assets/clip_image065.jpg)

也可以用yum search http

![image-20220214143931195](3-yum工作原理.assets/image-20220214143931195.png)

上图，说的是搜索的包名，前提你的知道包名，

包名不知道咋办？以httpd为例，先卸载掉👇 

![img](3-yum工作原理.assets/clip_image067.jpg)

整体卸掉包含之前yum install的所有东西，就需要查案yum history info NUMBER，见到httpd就行。

![img](3-yum工作原理.assets/clip_image069.jpg)

然后undo就行

![img](3-yum工作原理.assets/clip_image070.png)

![img](3-yum工作原理.assets/clip_image072.jpg)

上图提示的是依赖的文件名，而不是包名

找到依赖的文件来自哪个包

![img](3-yum工作原理.assets/clip_image074.jpg)

-qf必须已经安装了的文件名；-pqf后面必须跟包名，package；不对，换方法：

![img](3-yum工作原理.assets/clip_image076.jpg)

![img](3-yum工作原理.assets/clip_image077.png)

查找文件由哪个包提供。这种方法一般也就是用在没有yum源的情况下，自己把依赖准备好，放一起，走哪都yum -y install *rpm就行了。



![img](3-yum工作原理.assets/clip_image078.png)

list查所有的、已经装好的，可用的available-就是未安装的。

yum list 默认就是yum list all

<img src="3-yum工作原理.assets/image-20220214150614946.png" alt="image-20220214150614946" style="zoom:67%;" /> 



 yum repolist是将仓库的元数据down下来，同时给你list列出来，不过是针对启用的仓库，加上all就是关掉的也会列出来：

![img](3-yum工作原理.assets/clip_image079.png)

yum provides 后面可以是某个文件的，也可以是依赖包。相当于既有了-qf的跟文件能力，也有了-qpi的跟包能力。

![img](3-yum工作原理.assets/clip_image081.jpg)

👆上图就是展示了怎么rpm查询/bin/tree来自于哪个包。

![img](3-yum工作原理.assets/clip_image083.jpg)

yum的reinstall和rpm --replaces和--force一样咯。显然不一样，yum的reinstall还带依赖的。

![img](3-yum工作原理.assets/clip_image085.jpg)

红色框框里的常用，其他不怎么用了，使用yum

### yum info 类似于rpm -qi

![img](3-yum工作原理.assets/clip_image087.jpg)

 

![img](3-yum工作原理.assets/clip_image090.jpg)

rpm包也是yum安装的。需要依赖也会自动给你安装的。

 

![img](3-yum工作原理.assets/clip_image088.png)

尽量还是不要升级包。

还有centos6不会升级到7，而是重装成7

 注意update和upgrade的区别就是没区别

  https://cloud.tencent.com/developer/article/1375013?from=article.detail.1604418

<img src="3-yum工作原理.assets/image-20220214152747146.png" alt="image-20220214152747146" style="zoom:67%;" /> 

```
 update If run without any packages, update will update every currently installed
              package.   If  one  or  more packages or package globs are specified, Yum
              will only update the listed packages.  While updating packages, yum  will
              ensure that all dependencies are satisfied. (See Specifying package names
              for more information) If the packages or globs specified match  to  pack‐
              ages which are not currently installed then update will not install them.
              update operates on groups, files, provides and filelists  just  like  the
              "install" command.
              
```





 

 

![img](3-yum工作原理.assets/clip_image092.jpg)

makecahe一般不需要用

因为，第一次使用过yum，就自动makecahe了

![img](3-yum工作原理.assets/clip_image094.jpg)

这样一下yum的缓存也就有了。

 

总结：

![img](3-yum工作原理.assets/clip_image096.jpg)

![img](3-yum工作原理.assets/clip_image098.jpg)

这是查这个包依赖哪些文件，然后这些文件又依赖哪些包--这就需要再次yum deplist xxx了，比如上面的bash

<img src="3-yum工作原理.assets/image-20220214155748455.png" alt="image-20220214155748455" style="zoom:67%;" /> 



 

![img](3-yum工作原理.assets/clip_image099.png)

 

![img](3-yum工作原理.assets/clip_image101.jpg)

 

 
