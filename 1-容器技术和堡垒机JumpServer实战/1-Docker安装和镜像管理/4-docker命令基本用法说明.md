# 第4节 docker命令基本用法说明



# cli分类

![image-20240329141147278](4-docker命令基本用法说明.assets/image-20240329141147278.png)

swarm不行了，被k8s替代了，不学。

管理类👇

![image-20240329141828036](4-docker命令基本用法说明.assets/image-20240329141828036.png)

container 管理容器

image 管理镜像

network 管理网络

volume 管理存储



进一步还可以看docker image 大类下的子命令👇

![image-20240329142036001](4-docker命令基本用法说明.assets/image-20240329142036001.png)



![image-20240329142238407](4-docker命令基本用法说明.assets/image-20240329142238407.png)

但是老司机不爱这种分类的cli，更爱直接点比如

docker image ls 就= docker images

![image-20240329142439034](4-docker命令基本用法说明.assets/image-20240329142439034.png)



docker container ls  = docker ps

![image-20240329142811660](4-docker命令基本用法说明.assets/image-20240329142811660.png)





![image-20240329143553883](4-docker命令基本用法说明.assets/image-20240329143553883.png)

docker info 还可以看到docker compose的版本👇 以及 server 里的容器：

running运行的几个，paused暂停的几个，stopped停止的几个

![image-20240329143618045](4-docker命令基本用法说明.assets/image-20240329143618045.png)

docker 跑一个hello world，

1、要知道docker 跑容器，容器靠镜像，镜像哪里来呢

2、本地找找

3、仓库找找

![image-20240329143910324](4-docker命令基本用法说明.assets/image-20240329143910324.png)

上图就的Hello from Docker！字眼 就说明执行成功了，容器就是运行完就释放掉了，这里又不是当作守护进程来跑的，所以docker ps里看不到了就。



docker pull hello-world 只下载 不run

![image-20240329150130027](4-docker命令基本用法说明.assets/image-20240329150130027.png)



docker run hello-world = docker pull hello-world + docker start hello-world



![image-20240329150719607](4-docker命令基本用法说明.assets/image-20240329150719607.png)

注意知识容器进程退出了，只是程序退出了，这个容器还在磁盘上。

进程没了，磁盘空间照样占着的会，

<img src="4-docker命令基本用法说明.assets/image-20240329151130034.png" alt="image-20240329151130034" style="zoom:70%;" /> 



![image-20240329151310820](4-docker命令基本用法说明.assets/image-20240329151310820.png)

start 起来后运行完了也会退出的

![image-20240329151547964](4-docker命令基本用法说明.assets/image-20240329151547964.png)



![image-20240329160037603](4-docker命令基本用法说明.assets/image-20240329160037603.png)



# 很重要的一个目录-image下载存放目录

![image-20240329152028825](4-docker命令基本用法说明.assets/image-20240329152028825.png)

这个目录是默认存放镜像和容器相关文件的，这个目录没了，所有容器 和 所有镜像就都没了。



速度线索尝试总结，我感觉要单拎出来①★：这个目录必须是一个高速且大的磁盘，又高又大，因为所有的images和containers都默认扔在这里，跑起来，硬盘慢也会拖累容器的运行速度。生产中可以找个目录单独挂一个高速的SSD，没预算就算了哦，一起ssd也是ok的，哈哈，缝缝补补有三年也是ok的



彻底删除容器 

1、卸载软件：apt purge docker-ce  # ubuntu的

centos的“：yum remove docker-ce ? 还是yum history list 找到id后yum history undo xx 卸的还全呢，对吧。

2、删除软件产生的文件：rm -rf /var/lib/docker

学到这里，我又要逼逼两句了，删除A，就是要两步走①删除A②删除A的影响，有首歌叫 你身上的香水味~~，就是人走了，还得通风。HA也是这个道理，不管是线路的HA还是节点的HA，当HA的成员挂了一个，作为HA的底层逻辑同样需要①去掉故障成员②去掉故障成员当前的影响(比如全网ARP，会话缓存、MAC缓存等)③如果故障恢复一样要观察一下是不是真的恢复了稳定了才能将故障点重新加回HA并继续提供服务。



docker info也是可以看到整个默认存放的关键目录的

![image-20240329155611670](4-docker命令基本用法说明.assets/image-20240329155611670.png)

可以换成成别的目录，比如换到一块牛逼plus的单独硬盘上去，或者将整个目录挂过去都是一样的。



搞一个run后不退出的容器

![image-20240329161054740](4-docker命令基本用法说明.assets/image-20240329161054740.png)



<img src="4-docker命令基本用法说明.assets/image-20240329161216695.png" alt="image-20240329161216695" style="zoom:40%;" />







镜像是基于不同的操作系统做出来的，所以docker run也好pull也罢都是要注意的，

![image-20240329171213541](4-docker命令基本用法说明.assets/image-20240329171213541.png)

然后还可以到tag里去看细节

👇看着就是不同系统的镜像，docker pull nginx就是下载linux的且是latest的版本。

![image-20240329171258782](4-docker命令基本用法说明.assets/image-20240329171258782.png)



alpine是一个精简的linux os。是基于某个os制作出来的镜像。区别默认的应该就是deban和ubuntu的os做出来的。这句话怎么了理解哦：

就是docker的分层模型

![image-20240329174910547](4-docker命令基本用法说明.assets/image-20240329174910547.png)

container层就是docker engine了实现了容器的隔离，通过namespace和cgroup两大组件做到了隔离和资源的分配以及限制。

那么要理清楚一个点：就是

![image-20240329175029455](4-docker命令基本用法说明.assets/image-20240329175029455.png)

这些版本代表的意思是啥，对分层理解就会加深一点，

![image-20240329175247257](4-docker命令基本用法说明.assets/image-20240329175247257.png)

根据GPT多说，可以认为她讲的是对的，我们下载的镜像它肯定不是dockerengine ，docker engine是docker server进程，而images名称里的os就是说我整个image利用了底层宿主os的内核(大家都是linux内核啦)，然后我image里还自带一些基于整个内核实现精简os，整个os是半成品os是没有内核的os，GPT称之为用户空间--Alpine linux的用户空间，所以这部分+宿主的内核就构成了完整的os了。



<img src="4-docker命令基本用法说明.assets/image-20240329175853139.png" alt="image-20240329175853139" style="zoom:50%;" />



<img src="4-docker命令基本用法说明.assets/image-20240329175901167.png" alt="image-20240329175901167" style="zoom:50%;" />



<img src="4-docker命令基本用法说明.assets/image-20240329180315384.png" alt="image-20240329180315384" style="zoom:50%;" />



之前学习sendfile这个0复制技术逻辑的时候，还头头是道，其实殊不知当时的整张图就是在OS里聊的，对吧，用户空间，内核空间，这些就是OS啊，唉，蠢笨如我~~~ 

<img src="4-docker命令基本用法说明.assets/image-20240329180704642.png" alt="image-20240329180704642" style="zoom:37%;" />



<img src="4-docker命令基本用法说明.assets/image-20240329180646848.png" alt="image-20240329180646848" style="zoom:37%;" />





<img src="4-docker命令基本用法说明.assets/image-20240329181112552.png" alt="image-20240329181112552" style="zoom:49%;" />



user Space 1 2 3 就是你下载的image 并且运行起来的 容器，这些容器通过 container也就是docker 引擎 (namespace + Cgroup) 来隔离且限制地使用了HOST OS的内核，但是OS本身还有用户空间，这部分userSpace1 2 3 的容器是没有使用的，他们各自用的自家的image里封装好的自带了os一部分的用户空间。



好了反复拉扯应该靠谱了



