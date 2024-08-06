# 第3节 Docker实现跨宿主机通信和Docker-compose介绍



# 跨宿主机通信



## 利用桥接实现跨宿主机的容器间通信

![image-20240612172245590](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240612172245590.png)

上图有个点：就是br0和eth0默认是没有桥接在一起的，是需要手动connect的。

上图的veth是画在br0上的，下图的veth是画在宿主机上的和eth0同层级的。两者画法存在冲突其实就是必然有一个是错误的理解。我TM都不用懂什么docker就能判断这里一定是错误的一个。通过查看网络文章以及参考GPT得知，下图画法是错误的，应该将veth画在docker0上才对。

<img src="3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613103425053.png" alt="image-20240613103425053" style="zoom:47%;" />



恢复默认的网卡吧，什么docker100、wordpress_net网桥 都删掉吧

![image-20240612180216068](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240612180216068.png)



## 方法1：手动桥接二层打通



将eth0桥接到docker0

![image-20240613111801875](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613111801875.png)



![image-20240613111908010](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613111908010.png)

也是属于手动的方法，而且会带来eth0与外界通信中断。





1、制作两个测试的容器C1 和 C3 ，要求两个容器的IP不一样，要求两个容器分别在两个宿主机上。所以C1在192.168.126.132上，C3在192.168.126.133上，C2也在133上先于C3启动抢走172.17.0.2这个IP，专业昂C1就是172.17.0.2，C3就是172.17.0.3，ip区分开来了就。

![image-20240613094811826](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613094811826.png)



![image-20240613094827369](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613094827369.png)



2、把物理网卡桥接到br0也就是docker0

然后

```shell
132宿主上
brctl addif docker0 eth0

133宿主上
brctl addif docker0 eth0
```

此时C1和C3就通了

![image-20240613100940986](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613100940986.png)



![image-20240613100952877](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613100952877.png)



但是ssh却断了，初步原因就是因为物理网卡桥接到docker0上了。

估计eth0变成了docker0网络里的ip，docker0是172.17.0.0/16的子网，eth0是192.168.126.132的IP。目前没有找到解决的方法，继续吧。

但是这不是常规手段，仅仅是个测试。





## 方法2：手动打通3层



将两端的ip子网区分开来，走三层互通



这种方法也不太好，需要修改docker0的网段，需要配置路由--2个宿主就要互指路由，10个宿主你岂不是要full mesh就是90条路由了。

![image-20240613112306968](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613112306968.png)

**1、修改图中左侧172.17.0.0为172.27.0.0段先**

<img src="3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613112950229.png" alt="image-20240613112950229" style="zoom:50%;" />

记得重启docker服务





**2、互指路由**



![image-20240613112710496](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613112710496.png)



![image-20240613112750303](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613112750303.png)



**这就通了**

![image-20240613112844101](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613112844101.png)



![image-20240613112853261](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613112853261.png)



![image-20240613114703293](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613114703293.png)

也不用管NAT的事情和iptables的规则，就直接通了。不会说被iptables给挡了，也不存在做什么DNAT。因为docker0的IP本身就是铺在宿主机物理网卡的那一层的。

然后iptables隔离链也是隔离的docker0访问非docker0这个方向的流量，而非docker0访问docker0的是不隔离的，简单来讲就是docker0这些虚拟交换机出去的拒绝，进来的放行的。

![image-20240613115637137](3-Docker实现跨宿主机通信和Docker-compose介绍.assets/image-20240613115637137.png)

看图：docker0  -->  !docker0 也即是从docker0里出去的才会命中，才会走下面的DROP，然后下面的DROP反而是进入docker0的拒绝来着。就是说连起来看就是docker0出去回来的包被DROP了。

所以本段的实验是外界直接进入docker0的流量未曾匹配到这个策略 ( 哪个策略呢，就是调用DOCKER-ISOLATION-STAGE-2的父策略--就是上面的红框的策略 ) 所以不会被DROP。





## 方法3：使用专业的第三方软件解决跨主机通信

主要是K8S里的组件：flannel、calico

也有脱离K8S单独使用的软件。







# Docker Compose

本质上就是docker做了二次封装

比如这么一个场景：多个容器先后启动，每个容器还有很多启动参数，你说有runlike知道当时run了那些选项，但是runlike并不是十分好用，里面很多是默认无需care的选项也一并呈现了，然后多个容器的先后启动也无从得知。所以需要一个文本记录这些启动措施。



这些措施涉及：启动顺序、run的参数、这些参数就多了还涉及 网络、存储等。



compse就是解决方案，类似ansbile



ansbile也分：①如果临时执行 就是一条条命令就完了；②如果稍微复杂的任务就不推荐cli的方式了，而是用playbook来写，完了用ansible-playbook调用就行了。



也好比iptables：①iptables 命令一条条加也是临时的；②也可以写到iptables配置文件里，然后统一重启服务就行，或者iptables-save > iptables.rules ; vim iptables.rules; iptables-restore < iptables.rules



docker也是如此：①docker 命令就是之前的docker run这些命令；②docker compose就是配置文件，类似剧本的功能。



为了实现在单机上的综合的复杂项目，于是产生了docker compose



类似Dockerfile一样，有一个默认文件名，compose的默认文件名就是docker-compose.yml或docker-compose.yaml，只要在当前工作目录下就不用指定。



下面学习如何些docker-compose.yml，本质上就是将docker run的命令写到compose里。



docker-compose不是独立的工具，底层是依赖docker的。

docker-compose就是docker命令的脚本化。

docker-compose是单机的解决方案。单机上跑多个容器需要有编排--限执行谁后执行谁。



https://docs.docker.com/compose/



https://docs.docker.com/compose/compose-file/07-volumes/



docker-compose应用场景：①单机咯②大规模还是K8S。主要还是比如办公网的一些服务：vpn、confluence jira、等就可以用独立的宿主+docker+docker-compose来时间，这样简单方便啊，比如公司的gitlab也可以这样玩，没必要说一定要K8S啊。







