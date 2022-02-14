# 第4节. 实现yum源仓库和使用yum管理软件以及dnf



定下来学某个东西的时候，就要摸清楚来龙去脉，这样比较对得起人这种动物它这个脑部、神经、心跳节奏的规律。否则不舒服的，我在试图理解底层心理学？显然这是不对的，感觉对了就好。



## 配置内网的YUM源

### yum -y isntall httpd

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image002.jpg)

安装httpd服务，通过光盘镜像安装就行（各种rpm包，和yum仓库-repodata元数据）

通过rpm -ql 查看安装的文件

### systemctl start httpd

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image004.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image006.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image008.jpg)

### 目录结构

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image010.jpg)

所以就是/var/www/html/centos/7/os/x86_64

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image012.jpg)

 <img src="4-实现yum源仓库和使用yum管理软件以及dnf.assets/image-20220214173023977.png" alt="image-20220214173023977" style="zoom:67%;" />  



![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image014.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image016.jpg)

 

把centos7的安装光盘整个目录复制过去，也可以挂在光盘的。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image018.jpg)

再接一个CENTOS6的

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image020.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image022.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image024.jpg)

### 把对应的光盘挂过去-工作中复制过去

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image026.jpg)

临时挂用命令mount就行

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image028.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image030.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image032.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image034.jpg)

centos6的也有了

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image036.jpg)

 yum源就OK了。

### 服务端就配置好了，下面开始配置CLIENT

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image038.jpg)

只要不是repo后缀的就不会干扰repo源。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image040.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image042.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image044.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image046.jpg)

 

验证可以不写，写的话也很简单

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image048.jpg)

这就是校验文件，赋值一下路径

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image050.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image052.jpg)

搞定，当然上面的6也可以用$releasever替代他

 

 

测试下

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image054.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image056.jpg)

这里注意下，6713个包的解释：我们就挂了一张盘，但是6K个包是两张盘的总量，可能就是第一张盘里有两张盘的包总量信息。然后一些偏门的包在第二张盘里，你是安装不了的。

 

以上就是基于HTTP协议的yum源



### 如果没有光驱-显然基本都没有，就用iso文件去挂

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image058.jpg)

光盘可以挂，ISO文件也可以挂

然后web看到就成功了

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image060.jpg)

但是前置条件别忘了：selinux和防火墙

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image062.jpg)

 

 

还有客户端

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image064.jpg)

 

 

其他补充

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image066.jpg)

## mirrorlist是把多个路径写到一个文件里

 

现在这个机器可以指两个源，准备添加一个网上的阿里源：

复制远端路径

![image-20220214175841504](4-实现yum源仓库和使用yum管理软件以及dnf.assets/image-20220214175841504.png) 



![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image070.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image072.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image074.jpg)

把上面的txt路径一复制，贴到下面去就行了。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image076.jpg)

搞定了

 不信可以看看centos默认的yum源写法，他也是用的mirrorlist的。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image078.jpg)

 

 

 

## 包组的管理用group

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image080.jpg)

group分为环境组和普通组，环境组就是安装系统的时候让你选择的包--老王使用的事GNOME Destop，刘遄使用的是Server with GUI，我使用的是Minimal Install。

你选择某一种比如Serer with GUI就会对应地安装很多包。

![image-20220214180924589](4-实现yum源仓库和使用yum管理软件以及dnf.assets/image-20220214180924589.png) 

### 开发包组里的包

"Development Tools"注意引号，否则当作两个包组了。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image082.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image084.jpg)

 强制组、可选组

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image086.jpg)

不是随着group包组安装的，是随系统安装的。

不同符号不同状态

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image088.jpg)

groupinstall可以连起来写

工作中一般不用包组group，都是需要什么安装什么，group里的东西可能太多了。



### yum 的脚本安装可以加上-q

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image090.jpg)

 

 --disablerepo和--enablerepo没有试验成功，不知道咋弄的，不过我倒是可以用sed 来开关。恩，也不常用。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image092.jpg)

## 创建自定义的仓库createrepo-其实就是为一堆rpm包生成元数据。

**仓库虽然说要rpm包和元数据以及公钥，但实际上站在机器角度，仓库其实就只要一个repodata文件夹和里面的元数据。**

自己研发的rpm包，或者网上找的单个rpm包，没有现成的仓库，或是就算你down的是常规的rpm包，但是仓库没down，也可以自己生成。这个题外话参考别人的博客https://www.jianshu.com/p/3b669bcebfb6

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image094.jpg)

 👆这样仓库的元数据就有了。

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image096.jpg)

![image-20220214204839607](4-实现yum源仓库和使用yum管理软件以及dnf.assets/image-20220214204839607.png) 

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image100.jpg)

 

但是有个问题

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image102.jpg)

httpd要开启443

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image104.jpg)

仓库路径得找repodata的路径。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image106.jpg)

如图是在server下的，而其他的分别是集群、集群存储、server、虚拟化的repodata

多个repodata就是多个yum源。如果你不需要就配一个yum源就行了。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image108.jpg)

上图👆两个点①cat 重定向写法ctrl + d安全退出才行②baseurl的路径是repodate的pwd路径。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image110.jpg)

 ![image-20220214205730555](4-实现yum源仓库和使用yum管理软件以及dnf.assets/image-20220214205730555-16448434510601.png)

 

## DNF

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image112.jpg)

 dnf比yum的优势据说就是速度快

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image114.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image116.jpg)

推荐yum install *.rpm因为包可能不全，还需要依赖。

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image118.jpg)

这样就可以用dnf了，

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image120.jpg)

dnf的使用和yum一样

仓库配置也是一样yum的配置文件。

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image122.jpg)

比较下dnf的性能

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image124.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image126.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image128.jpg)

 

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image130.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image132.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image134.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image136.jpg)

= = 效果不行啊，哈哈

 

```
time dnf reinstall httpd -y
time yum reinstall httpd -y
这个可以看到dnf快些，可能。

```

 

### yum其实是python写的程序

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image138.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image140.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image142.jpg)

yum底层就是依赖于rpm的

 

## 小软件编译举例

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image144.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image146.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image148.jpg)

上图gcc hello.c 回车，会自动生成一个a.out。注意上图不是一个命令哦。

指定一个名称-o

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image150.jpg)

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image152.jpg)

 

### 将来要编译的软件必然不是一个c文件咯：

看看这个httpd的源码有多少个c

![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image154.jpg)



![img](4-实现yum源仓库和使用yum管理软件以及dnf.assets/clip_image156.jpg)

解开的文件里有284个c文件

gcc 能搞定吗？还有编译先后顺序的。所以就不是gcc这种方式了。

那是项目了，项目管理工具了，不同的语言编译工具不一样。JAVA用maven。C语言里make

 

 



 

 
