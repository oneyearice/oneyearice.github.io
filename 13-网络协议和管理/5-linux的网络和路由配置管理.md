# 第5节. linux的网络和路由配置管理



## 修改网卡名-方法1-网卡驱动模块

研究下为什么是eth1，怎么改，这是centos6的：

![image-20220315091523380](5-linux的网络和路由配置管理.assets/image-20220315091523380.png)

![image-20220315092958594](5-linux的网络和路由配置管理.assets/image-20220315092958594.png) 

![image-20220315093035859](5-linux的网络和路由配置管理.assets/image-20220315093035859.png)

可见eth0已经被某个mac占用，所以现在看到的就是eth1了。

删除并修改

![image-20220315093143457](5-linux的网络和路由配置管理.assets/image-20220315093143457.png)

修改后，重启网络服务是不行的

![image-20220315093236310](5-linux的网络和路由配置管理.assets/image-20220315093236310.png)

要修改eth1这个名称，需要卸载网卡驱动，并重新加载驱动。

mii-tool 查看

![image-20220315093339559](5-linux的网络和路由配置管理.assets/image-20220315093339559.png) 

查看网卡驱动

![image-20220315093551952](5-linux的网络和路由配置管理.assets/image-20220315093551952.png) 



![image-20220315093644515](5-linux的网络和路由配置管理.assets/image-20220315093644515.png) 

👇找到了驱动模块

![image-20220315093717338](5-linux的网络和路由配置管理.assets/image-20220315093717338.png) 



lsmod 是找到加载的所有驱动模块

![image-20220315093808609](5-linux的网络和路由配置管理.assets/image-20220315093808609.png) 

卸载模块用rmmod或者modprobe -r

![image-20220315094152620](5-linux的网络和路由配置管理.assets/image-20220315094152620.png) 

网卡模块卸了也就是网卡驱动卸了。

此时网卡就自然看不见了

![image-20220315094228945](5-linux的网络和路由配置管理.assets/image-20220315094228945.png) 



然后再重新加载模块(驱动)：

![image-20220315094330745](5-linux的网络和路由配置管理.assets/image-20220315094330745.png) 

此时就改为eth0了

![image-20220315094357316](5-linux的网络和路由配置管理.assets/image-20220315094357316.png) 



## 方法2-ip子命令

ip 子命令，在centos6上tab不出来，centos7可以

![image-20220315095018759](5-linux的网络和路由配置管理.assets/image-20220315095018759.png) 

centos6要额外安装软件包才能支持tab键补全





## 网卡配置

1、setup 进去选-选-选 等价于system-config-network-tui  # 这两种方法就是算了，而且在centos7里setup里也没有网络配置选项了。

2、重点看命令和配置文件



![image-20220315095711283](5-linux的网络和路由配置管理.assets/image-20220315095711283.png) 

ifconfig过时了，擦

主要是因为net-tools这个工具包过时了，所以包里的很多比如ifconfig、netstat都过时了。

![image-20220315095807943](5-linux的网络和路由配置管理.assets/image-20220315095807943.png) 

![image-20220315095918949](5-linux的网络和路由配置管理.assets/image-20220315095918949.png) 



![image-20220315100004582](5-linux的网络和路由配置管理.assets/image-20220315100004582.png) 



推荐你用iproute这个包

![image-20220315100230039](5-linux的网络和路由配置管理.assets/image-20220315100230039.png)







ifconfig直接回车看的是激活状态的网卡

禁用和激活

![image-20220315100631020](5-linux的网络和路由配置管理.assets/image-20220315100631020.png) 

ifconfig eth1 down 禁用eth1网卡

ifconfig 就看不到eht1，ifconfig -a 可以看到eth1但是IP没了，ip a就看的更清除

![image-20220315100750969](5-linux的网络和路由配置管理.assets/image-20220315100750969.png)

ifconfig eth1 up启用

![image-20220315100818973](5-linux的网络和路由配置管理.assets/image-20220315100818973.png)



禁用网卡还可以ifdown eth1

![image-20220315100908111](5-linux的网络和路由配置管理.assets/image-20220315100908111.png) 

不过这种down和刚才的ifconfig eth1 down又不同了

![image-20220315100943876](5-linux的网络和路由配置管理.assets/image-20220315100943876.png)

这个ifdown 后，ifconfig确实可以看到的，但是没有地址。

这个ifdown属于网络层的down，IP没了但是数据链路层是通着的。

对比ifconfig属于数据链路层的down。

所以ip a看就会发现是UP的，所以ip a看的就是L2链路层咯。

![image-20220315101114368](5-linux的网络和路由配置管理.assets/image-20220315101114368.png)

```shell
1、ifdown\ifup是L3层的up\down， 这个ifdown后ip a看还是UP的

2、ifconfig down\ifconfig up是L2层的up\down，这个ifconfig down后ip a看就是DOWN的

3、ip a看的是L2的up\down

4、还一种是物理层的down，就是拔网线了
```







![image-20220315104251818](5-linux的网络和路由配置管理.assets/image-20220315104251818.png)

ifup 要起来还需要依赖一些网络配置文件，所以ip a还是看不到地址

![image-20220315104343790](5-linux的网络和路由配置管理.assets/image-20220315104343790.png)

没关系，在centos6上用service NetworkManager restart就可以了。

![image-20220315104435149](5-linux的网络和路由配置管理.assets/image-20220315104435149.png)





把网线的演示

![image-20220315104548951](5-linux的网络和路由配置管理.assets/image-20220315104548951.png) 

这就类似拔网线了

1、ifconfig看就没有地址了

![image-20220315104719668](5-linux的网络和路由配置管理.assets/image-20220315104719668.png) 



2、ip a看就是down

![image-20220315104752991](5-linux的网络和路由配置管理.assets/image-20220315104752991.png) 

此时上图👆就能判断是网线拔了，而不是其他的，因为有关键词NO-CARRIER。

对不ifconfig eth1 down的描述信息

<img src="5-linux的网络和路由配置管理.assets/image-20220315105913650.png" alt="image-20220315105913650" style="zoom:50%;" /> 





## 配地址-临时

![image-20220315110417795](5-linux的网络和路由配置管理.assets/image-20220315110417795.png) 



## 清地址-临时

![image-20220315110512788](5-linux的网络和路由配置管理.assets/image-20220315110512788.png)



## 增加地址-临时

就是huawei里的sub地址咯，不是sub是思科的second ip，还一种是子接口，子接口在linux里是一种别名

```
ifconfig eth1:123 1.1.1.1/24
ifconfig eth1:321 2.2.2.2/24
```

ifconfig可见

![image-20220315110838635](5-linux的网络和路由配置管理.assets/image-20220315110838635.png) 

这里不涉及子接口的vlan id封装解封装，直接就能和外界同样的IP进行通信，所以我感觉更像是second ip而不是子接口。



### 删除linux的子接口

ifconfig eth1:123 down

![image-20220315111409880](5-linux的网络和路由配置管理.assets/image-20220315111409880.png)

这个down掉就是删除子接口了，这里就称之为linux的子接口咯，虽然它没有vlan的概念。

![image-20220315111524070](5-linux的网络和路由配置管理.assets/image-20220315111524070.png) 





![image-20220315111627465](5-linux的网络和路由配置管理.assets/image-20220315111627465.png)



## 混杂模式

抓包的时候就用的混杂模式

混杂--不管这个数据是否给我的，我都收。

https://cloud.tencent.com/developer/article/1439013







## mii-tool 工具

![image-20220315121037577](5-linux的网络和路由配置管理.assets/image-20220315121037577.png)

上图说明

capabilities 是支持的能力比如FD就是full duplex；HD就是hafl duplex

第一行就是协商后的当前工作模式是千兆全双工



## ethtool 工具

![image-20220315121215498](5-linux的网络和路由配置管理.assets/image-20220315121215498.png)

网线8根线，如果断了一根运气好，还能用就是100M，👆这里就可以检查到。



ehtool可以修改网卡工作模式，一般不改

![image-20220315121747573](5-linux的网络和路由配置管理.assets/image-20220315121747573.png)



CMD的grep

![image-20220315122205793](5-linux的网络和路由配置管理.assets/image-20220315122205793.png) 

例子，查看多少人连到我的VNC上

![image-20220315122317321](5-linux的网络和路由配置管理.assets/image-20220315122317321.png)

![image-20220315122337240](5-linux的网络和路由配置管理.assets/image-20220315122337240.png)

第三列就是client IP地址



### 常见服务端口

cat /etc/services

![image-20220315122952986](5-linux的网络和路由配置管理.assets/image-20220315122952986.png)



![image-20220315135033610](5-linux的网络和路由配置管理.assets/image-20220315135033610.png)

一般客户端电脑不会使用1W6+个端口，但是如果你这台linux或啥系统的电脑是作为代理上网，比如SNAT，那么1W6+也不是没有可能，因为1台内网的PC大概10连接，1000台PC对吧，再加上手机，还是有可能让你的这台NAT服务器的端口超出1w6+这个默认值的。

![image-20220315135820194](5-linux的网络和路由配置管理.assets/image-20220315135820194.png)

如果要当代理，这个端口就要调大👆



## 混杂端口的说明

![image-20220315140731029](5-linux的网络和路由配置管理.assets/image-20220315140731029.png)

数据包的序列号，并非0，0是相对编号，应该是初始ISN，好久不碰了有点忘了都，而是下面的97fa6e02



## tcpdump的举例-整理稍后

<img src="5-linux的网络和路由配置管理.assets/image-20220315153047080.png" alt="image-20220315153047080" style="zoom:80%;" />  

![image-20220315152819339](5-linux的网络和路由配置管理.assets/image-20220315152819339.png)

上图👆是drop后的一个抓包结果，[S]是不断的SYN包，是重传机制导致的。

其实更多的时候，我一般就是抓个端口然后|grep 哈哈，是不是没想到，哈哈



![image-20220315153635512](5-linux的网络和路由配置管理.assets/image-20220315153635512.png)

![image-20220315154709605](5-linux的网络和路由配置管理.assets/image-20220315154709605.png) 

|前导码+开始符|DA|SA|TYPLE/LENGTH|DATA|FCS|   一共72B的最小值。

ping 默认就是64也即是没有算开头的8B。

附上之前的一个分片解析图

![PING包分片的计算](5-linux的网络和路由配置管理.assets/PING包分片的计算.png)

## TCP超时重传

![image-20220315152643847](5-linux的网络和路由配置管理.assets/image-20220315152643847.png)





## TCP的拥塞控制

四个机制

慢启动、拥塞避免、快速重传、快速恢复

![image-20220315153411212](5-linux的网络和路由配置管理.assets/image-20220315153411212.png) 

![image-20220315153354410](5-linux的网络和路由配置管理.assets/image-20220315153354410.png)







## 并发ping-首推方法

![image-20220315155919075](5-linux的网络和路由配置管理.assets/image-20220315155919075.png)

```
seq 1 255 |xargs -i -P 0 bash -c 'ping -w 2 192.168.25.{} &> /dev/null && echo 192.168.25.{} icmp allowed'
```

还有awk的并发ping，想不起来了，没影响了。

![image-20220315162214235](5-linux的网络和路由配置管理.assets/image-20220315162214235.png)

这个也快

```
ls --hide=proc | xargs -i -P 0 find /{} -name "sshd"
```





## awk的举例

![image-20220315160320544](5-linux的网络和路由配置管理.assets/image-20220315160320544.png)

```
tcpdump -i eth0 -tnn dst port 443 -c 100 |awk -F "." '{print $1"."$2"."$3"."$4}'|sort|uniq -c|sort -rn|head -n20
```

![image-20220315160345007](5-linux的网络和路由配置管理.assets/image-20220315160345007.png)

我之前说过awk和xargs一样快，ping的处理上我怎么不记得有做过呢，肯定是有过的，现在想来是写个for循环awk调用系统命令，但有这么复杂吗，当然其实不复杂，我是说之前的awk实现批量ping 是怎么弄的？



还有好多工具类的使用iperf 、curl、F12等等，这些计算了，不放到这里了。





## 广播的情况

![image-20220315171023241](5-linux的网络和路由配置管理.assets/image-20220315171023241.png)

👆ping 255广播只有一个37.2响应，因为linux默认是不响应广播ping的，需要开启

![image-20220315171142718](5-linux的网络和路由配置管理.assets/image-20220315171142718.png) 

1就是忽略，就是不回应，改为0

![image-20220315171210639](5-linux的网络和路由配置管理.assets/image-20220315171210639.png)

centos7上一样

![image-20220315171234057](5-linux的网络和路由配置管理.assets/image-20220315171234057.png)

![image-20220315171258071](5-linux的网络和路由配置管理.assets/image-20220315171258071.png)

从默认的TTL上可以判断37.2的ttl是128，是台windows机器；37.7和37.6是linux机器。



## loopback

略，注意下图

![image-20220315171942724](5-linux的网络和路由配置管理.assets/image-20220315171942724.png)

认为改成6.6.6.6/24 ，这样这个段都处于loop，并不是说必须是127。



![image-20220315172633721](5-linux的网络和路由配置管理.assets/image-20220315172633721.png)

这里也可以写IP地址，不一定写lo





## 网卡配置文件

![image-20220315174506017](5-linux的网络和路由配置管理.assets/image-20220315174506017.png) 



name将来就表现为GUI图形界面里的eth0

![image-20220315174438843](5-linux的网络和路由配置管理.assets/image-20220315174438843.png) 

 

改的玩下，

![image-20220315174646633](5-linux的网络和路由配置管理.assets/image-20220315174646633.png) 

BOOTPROTO=none也行都是静态手动配置，BOOTPROTO=dhcp就是动态

1、注意一旦写了dhcp，后买手动配置的IP地址，DNS就会被覆盖了；

2、文件其实就是脚本用的，前面其实就是变量，变量是区分大小写的，然后=左右不能带空格；

这块书上讲的更全，所谓全其实也就是下发明细和下发默认的一些优先级问题。

还能精简成如下3行，再补一个GW和DNS1

![image-20220315175527530](5-linux的网络和路由配置管理.assets/image-20220315175527530.png) 

改完配置文件后，一般不会立即生效，有时候会立即生效，那是因为NetworkManager服务，不过这个服务不是时刻都能立即发现你修改了文件然后使之生效的。而且这个服务一般也不用都是关掉的。最小化安装好像也是没有的。说反了，一般是我们只用network，但是rocky-linux和centos7最小化安装后好像rocky-linux是没有network服务只有NetworkManager，然后centos7是networkfail的NetworkManager是active的。好奇怪~没事停用禁用NetworkManager后就可以启用network服务了。

![image-20220315181200508](5-linux的网络和路由配置管理.assets/image-20220315181200508.png)

看来rocky-linux不喜欢这个。centos8一样，







![image-20220315174557612](5-linux的网络和路由配置管理.assets/image-20220315174557612.png)





## linux开启路由功能

注意临时修改是用的echo，vim是不能编辑内存里的文件的

![image-20220315181957775](5-linux的网络和路由配置管理.assets/image-20220315181957775.png) 

vim是改磁盘文件的，不能改内存的数据。





## mtr的使用







## frr可以弄一下

<img src="5-linux的网络和路由配置管理.assets/image-20220315182934167.png" alt="image-20220315182934167" style="zoom:80%;" /> 

frr替代quagga了好像



