# 第8节 Docker的默认网络和容器间通信



### 网络简述

容器启动都会获取docker0一个网段的IP，默认都是172.17.0.0/16

![image-20240528133910440](8-Docker的默认网络和容器间通信.assets/image-20240528133910440.png)



同主机下不同容器互访，因为IP是动态获取的，存在变动的可能

不同主机下容器之间互访，首当其冲的就是IP段默认一样，出都出不来。听不懂啊，什么叫出不来，就是路由啊，同网段不会找GW，不走GW怎么出本地局域网呢。



启动一个容器，就会生成一个虚拟网卡，该网卡在宿主机表现为vethxxxx，在容器里表现为eth0#ifxxx;  一体两面 就这么理解就行了，官方称之为veth pair。

![image-20240528141955478](8-Docker的默认网络和容器间通信.assets/image-20240528141955478.png)



<img src="8-Docker的默认网络和容器间通信.assets/image-20240528142927753.png" alt="image-20240528142927753" style="zoom:50%;" />



docker0相当于一个交换机，eth0--veth   都桥接上去。这样容器之间沟通，走docker0，容器出宿主的流量也走docker0不过还得过一层SNAT出去。





### 确认某个veth虚拟网卡所属容器

**1、进入容器**

看编号对应关系就行了，宿主上看9对应到容器的8，就是编号对应的。

![image-20240528152533265](8-Docker的默认网络和容器间通信.assets/image-20240528152533265.png)

也要注意这个veth-pair，一体两面的 MAC 是不一样的；

然后docker0和veth是桥接关系。

![image-20240528152829486](8-Docker的默认网络和容器间通信.assets/image-20240528152829486.png)





**2、不进容器**

但要注意：会导致 进入多层 bash，如果操作不当

```shell
docker inspect --format '{{.State.Pid}}' <container_id>
nsenter -t <PID> -n ip link
```

![image-20240528153501437](8-Docker的默认网络和容器间通信.assets/image-20240528153501437.png)

所以这个方法

<img src="8-Docker的默认网络和容器间通信.assets/image-20240528153536947.png" alt="image-20240528153536947" style="zoom: 44%;" />

其实就是进入了命名空间了



**其他工具看看docker0挂了几个容器或者几个veth**



这个可以直观看到docker里挂的容器的NAME和容器里的IP和MAC，还不错👇

![image-20240528155538716](8-Docker的默认网络和容器间通信.assets/image-20240528155538716.png)



这个就是个排版看的舒服而已，ip a 看到的docker0自然是桥接到所有的vethxxx的。

![image-20240528155915368](8-Docker的默认网络和容器间通信.assets/image-20240528155915368.png)



![image-20240528160430743](8-Docker的默认网络和容器间通信.assets/image-20240528160430743.png)





同宿主容器之间的ping就是默认通的👇

![image-20240528160748969](8-Docker的默认网络和容器间通信.assets/image-20240528160748969.png)



# 解决容器IP不固定的问题

**背景：**

1、容器run起来了，IP不确定，run起来了IP是不是也会受dhcp的释放周期影响的咯。

2、如果你公司内网有172.17段，那么容器如果要和这个段互通，就不行了。



**需求：**

解决这个不固定的问题咯，然后就可以更好的自动化咯。以及更换容器的网段。



**落实：**



## 修改docker0的dhcp的地址段



方法1：

```shell
vim /etc/docker/daemon.json
{
	"bip": "172.19.0.0/16"  # 这里写错啦，这写host和掩码而不是子网，改成0.1就行
}
```

![image-20240530150122996](8-Docker的默认网络和容器间通信.assets/image-20240530150122996.png)



改完以后，哪怕删掉重启电脑，这个ip就是你改后的样子了，不会恢复到出厂默认值的。

可能猜测是，考虑到以前已经有容器在了，不管是up还是exited的，哪怕你删除了，也给你留存上一次修改的记录。

如果你想恢复出厂IP，那么就手动写成172.17.0.1/16，重启docker，顶多在删掉配置...不过无所谓了。







方法2：

```shell
vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --bip=172.19.100.0/24
```

![image-20240530134834951](8-Docker的默认网络和容器间通信.assets/image-20240530134834951.png)

方法2没有成功

![image-20240530151945083](8-Docker的默认网络和容器间通信.assets/image-20240530151945083.png)







## 桥接到自定义的网桥

默认是docker的内外网卡都是桥接到docker0这个虚拟网桥的，现在改掉。



### 1、首先创建网桥

```shell
yum -y install bridge-utils
brctl addbr br0
ip a a 172.20.100.1/24 dev br0
brctl show

```

![image-20240530164736771](8-Docker的默认网络和容器间通信.assets/image-20240530164736771.png)

发现docker100还没有IP地址，然后docker0和docker100都是UP的。



![image-20240530165028628](8-Docker的默认网络和容器间通信.assets/image-20240530165028628.png)



### 2、修改服务启动文件

```shell
vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -b docker 100

```



### 3、重载和重启

```
systemctl daemon-reload
systemctl restart docker
```



### 4、查看确认

```shell
ps -ef |grep dockerd

docker run --rm busybox hostname -i
```



![image-20240530165826921](8-Docker的默认网络和容器间通信.assets/image-20240530165826921.png)



![image-20240530170035918](8-Docker的默认网络和容器间通信.assets/image-20240530170035918.png)



再创建容器，就会桥接到docker100上了

![image-20240530170154779](8-Docker的默认网络和容器间通信.assets/image-20240530170154779.png)



![image-20240530170329555](8-Docker的默认网络和容器间通信.assets/image-20240530170329555.png)



**在检查下容器里面是否可以ping通外面**
![image-20240530170805223](8-Docker的默认网络和容器间通信.assets/image-20240530170805223.png)

可以的，通过抓docker100的包也可以看到

![image-20240530170824991](8-Docker的默认网络和容器间通信.assets/image-20240530170824991.png)

尝试抓容器的宿主虚拟接口的包

先定位这个容器id的内外IP是多少，当然我们通过上面抓包已经知道是172.20.100.2了。

![image-20240530171202335](8-Docker的默认网络和容器间通信.assets/image-20240530171202335.png)

所以这一次通过这个命令docker network inspect得知什么容器的里面的ip是什么后

再检查这个容器里的接口名称

### 如何知道exec 容器的接口名称

1、比如我要使用这个bd98e2b01c59容器来做抓包

就可以通过exec -it xxxx  ip a 知道接口对，就知道抓外面的哪个接口的包了

![image-20240530171526764](8-Docker的默认网络和容器间通信.assets/image-20240530171526764.png)



2、然后还可以通过，

docker network inspect 你要操作容器的id去确认待会抓包看到的内网IP是啥

![image-20240530171357089](8-Docker的默认网络和容器间通信.assets/image-20240530171357089.png)



3、然后开始tcpdump 那个接口

![image-20240530172016350](8-Docker的默认网络和容器间通信.assets/image-20240530172016350.png)



![image-20240530172143088](8-Docker的默认网络和容器间通信.assets/image-20240530172143088.png)



然后就看到包了

![image-20240530172156224](8-Docker的默认网络和容器间通信.assets/image-20240530172156224.png)



此时修了容器里的ip，也能够给通外面来，说明你修改了docker100为所有容器桥接的虚拟网桥，那么SNAT是自动给你做了的，无需配置的

可以通过iptables -vnL -t nat确认的

![image-20240530172500497](8-Docker的默认网络和容器间通信.assets/image-20240530172500497.png)













# 工作案例-linux的ping测

linux常ping脚本，



**方法1：**

```shell
ping 10.100.8.29 | awk '{ print $0 " " strftime("%F %H:%M:%S",systime()) }'  >> test.ping
```

![image-20240528175906802](8-Docker的默认网络和容器间通信.assets/image-20240528175906802.png)

注意这种直接 >> 重定向到 文件，大概是60个ping包一起写入文件的。不是每个包写一个的，相当于默认的有一个I/O优化吧



**方法2：**

```shell
ping 223.5.5.5 |& while read -r line;do echo "$(date) $line";done >> ping.log

ping 223.5.5.5 |& while read -r line;do echo "$(date) $line" >> /tmp/123.ping;done
```

![image-20240528180042025](8-Docker的默认网络和容器间通信.assets/image-20240528180042025.png)

这种方式写入文件很及时，一个ping包就是一个。

时间格式优化下

```
ping 223.5.5.5 |& while read -r line;do echo "$(date +%F_%H:%M:%S) $line" >> /tmp/ping.test ;done
```

![image-20240528180530420](8-Docker的默认网络和容器间通信.assets/image-20240528180530420.png)





然后改成脚本，做成服务，可以实现异常停止后的服务自动起来，也包括重启机器自动起来。

![image-20240528181218892](8-Docker的默认网络和容器间通信.assets/image-20240528181218892.png)



![image-20240528181230987](8-Docker的默认网络和容器间通信.assets/image-20240528181230987.png)



**做成服务**



```shell
vim /etc/systemd/system/pingtest.service
```



```shell
[Unit]
Description=Ping Test Service
After=network.target

[Service]
User=root
Type=simple
ExecStart=/path/to/pingtest.sh 8.8.8.8
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```



```shell
systemctl daemon-reload
```



```shell
systemctl start pingtest.service
systemctl enable pingtest.service
```



```shell
systemctl status pingtest.service
```



**测试kill 后自动起来**

找到main pid

![image-20240528182214861](8-Docker的默认网络和容器间通信.assets/image-20240528182214861.png)



kill -9会自动起来

![image-20240528182515101](8-Docker的默认网络和容器间通信.assets/image-20240528182515101.png)



kill 也会起来

![image-20240528182823247](8-Docker的默认网络和容器间通信.assets/image-20240528182823247.png)



stop纯人工停止自然不会起来咯，否则你怎么停服务~

![image-20240528183043114](8-Docker的默认网络和容器间通信.assets/image-20240528183043114.png)

注意status里可以看到一些服务器启动日志的含时间。



系统message也可以看

![image-20240528183549863](8-Docker的默认网络和容器间通信.assets/image-20240528183549863.png)





以上就是简单的一个做法，或者不写服务用screen去做也不错，

不过正规还是要上监控的，比如用zabbix-agent去实现，这个前文也有。







# 案例2，如何做个远端节点的PING测监控



其实和斗数一样，也是看三方四正



举例，你要监控从上海到洛杉矶的云主机的ping测。就需要出5张图，也是一个三方四正

①公司zabbix-----洛杉矶节点

②proxy1代理----到洛杉矶节点

③proxy2代理----到洛杉矶节点

④公司zabbix----到proxy1节点

⑤公司zabbix----到proxy2节点



**看图**

![image-20240530115548153](8-Docker的默认网络和容器间通信.assets/image-20240530115548153.png)



有了这个总结，以后做监控，基本上如果重要的节点，就可以一步到位全面监控住。







### 案例-ipsecvpn

云上的VPN的网关其实是PAT出来的，要IDC的SSG开启NAT-T来解封的。



https://cshihong.github.io/2019/04/17/IPSec%20VPN%E7%9A%84NAT%E7%A9%BF%E8%B6%8A-NAT-T-%E5%8E%9F%E7%90%86/

文章中的这个传输模式同样不能转换端口的

![image-20240530184126020](8-Docker的默认网络和容器间通信.assets/image-20240530184126020.png)

