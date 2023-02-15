# 第2节. DNS只缓存服务器实现







下面开始部署DNS服务器

## 主从







### whois

![image-20230214182208109](1-DNS服务简介.assets/image-20230214182208109.png)





![image-20230214182523730](1-DNS服务简介.assets/image-20230214182523730.png)



1、购买域名

2、云上解析即可，此时云上就是你的权威DNS

3、搭建企业内部的dns服务器





使用BIND，其实是一系列的软件。包括bind、bind-utils、bind-chroot

bind-utils应该是客户端工具，比如nslookup、dig、hosts等

bind是服务端提供服务的包

bind-chroot是bind的一个功能,使bind可以在一个chroot的模式下运行.也就是说,bind运行时的/(根)目录,并不是系统真正的/(根)目录,只是系统中的一个子目录而已.这样做的目的是为了提高安全性.因为在chroot的模式下,bind可以访问的范围仅限于这个子目录的范围里,无法进一步提升,进入到系统的其他目录中。bind的默认启动方式就是chroot方式



yum -y install bind

rpm -qi bind

rpm -ql bind

其中/etc/named.conf是主配置文件

/usr/lib/systemd/system/named.service是服务启动的文件

/usr/sbin/named是主程序

/usr/sbin/rndc也是要用到的啥啥啥 梦飒飒

/var/named是数据库文件夹用来存放域名解析的信息

/var/named/named.ca不是证书那套体系里的ca哦，而是传说中的13个根。

/var/log/named.log是日志

![image-20230215092303136](2-DNS只缓存服务器实现.assets/image-20230215092303136.png)





![image-20230215094003129](2-DNS只缓存服务器实现.assets/image-20230215094003129.png)

怎么一下起来这么多53，v4的8个（tcp4个udp4个），v6的8个，

那么问题来了，为什么会是4个udp53呢？

fd好像是socke进程的句柄，可能涉及并发

fd一般就是/dev/fd 文件描述符，这里可能是一个意思？





然后只要启动了bind服务，就默认具备了dns的递归查询功能，因为人家有13个根，所以能够往上游查询，只是效率将不准了，哈哈，也就是说 本地/etc/resolv.conf里写127.0.0.1，然后启动named，本地就能ping通www.xx.com域名了。

![image-20230215095645057](2-DNS只缓存服务器实现.assets/image-20230215095645057.png)



![image-20230215100229096](2-DNS只缓存服务器实现.assets/image-20230215100229096.png)



此时named默认空配置，服务起来后，自己可以解析；但是代理dns，也就是还不能做内网dns

![image-20230215100618112](2-DNS只缓存服务器实现.assets/image-20230215100618112.png)

上图已经关闭了默认的firewalld和selinux

除了nslookup还有host dig都有类似提示

![image-20230215101804646](2-DNS只缓存服务器实现.assets/image-20230215101804646.png)

都是超时，因为udp 53都不通啊，👇

![image-20230215101435445](2-DNS只缓存服务器实现.assets/image-20230215101435445.png)

当然tcp53一样也不通

但是ping ok

防火墙和selinux不止早就关了吗，还有啥？



仔细看前面的ss -tlnup 的截图，上面显示是监听地址为127.0.0.1也就是只侦听在本地环回口，而不是外接的网卡上，所以自然不能提供服务啦

改一下，就是改配置文件

![image-20230215104832919](2-DNS只缓存服务器实现.assets/image-20230215104832919.png)

这样端口就通了，也可以改成localhost--代表当前机器的所有IP，效果是一样一样的，都是监听本地所有ip上

![image-20230215105430945](2-DNS只缓存服务器实现.assets/image-20230215105430945.png) 

<font color=red size=5>localhost是本地所有IP≠any=0.0.0.0/0</font>

本地两个ip一个192.的一个127的，所以就监听在这两个上面了

![image-20230215105931939](2-DNS只缓存服务器实现.assets/image-20230215105931939.png)

![image-20230215104851255](2-DNS只缓存服务器实现.assets/image-20230215104851255.png)

当然tcp，client是不需要的



此时nslookup还是不行

![image-20230215104916037](2-DNS只缓存服务器实现.assets/image-20230215104916037.png)

直接拒绝你了，看来还有地方要改

继续看配置文件，找到关键词 allow-query，改成0.0.0.0/0或者any就行

![image-20230215115030958](2-DNS只缓存服务器实现.assets/image-20230215115030958.png)



![image-20230215115141412](2-DNS只缓存服务器实现.assets/image-20230215115141412.png)

![image-20230215115258295](2-DNS只缓存服务器实现.assets/image-20230215115258295.png)

注意nslookup 不涉及本地缓存哦，上图就说了，named是默认就有缓存功能的。进一步验证，就是讲dns 服务的机器设置成两个网卡，一个网卡沟通内部，一个网卡沟通外网互联网，①第一次client内部dns请求后②关闭server的外网接口，再次请求就会发现依然OK。



![image-20230215121629553](2-DNS只缓存服务器实现.assets/image-20230215121629553.png)



dig 用的不多，看看效果

![image-20230215121743641](2-DNS只缓存服务器实现.assets/image-20230215121743641.png)

这是不能解析的截图



![image-20230215122702839](2-DNS只缓存服务器实现.assets/image-20230215122702839.png)

这个不要深究了，及时ping提示网络不可达其实dns已经由IP了。

正所谓凡是不可毕其尽，谁知道系统怎么提示的，又不是天地法则，都是人为编程。

补上网关

补一个不存的网关，欺骗系统一下，它认为远程的主机是可达的，是有机会访问的

<img src="2-DNS只缓存服务器实现.assets/image-20230215123122947.png" alt="image-20230215123122947" style="zoom:67%;" /> 

![image-20230215154516385](2-DNS只缓存服务器实现.assets/image-20230215154516385.png)

这个时候IP就显示出来了，所以网络不通立马跳出来这个消息，就是网关配有配置，有配置就是尝试通信的。

当然一般dns不是同网段，

如果是同网段+dns在一个子网，解析没问题，网关没配置，一样秒报错 网络不可达。



本来不想细究，但是人家视频里正好演示了，我就顺便截个图咯，本来就没啥意义，只有在处理特殊故障的时候才可能用到这种所谓的底层基本功。





缓存存在域dns server的验证实验截图，人家的 哈哈

dns server下图的路由和网卡：👇

![image-20230215160905407](2-DNS只缓存服务器实现.assets/image-20230215160905407.png)



![image-20230215161024379](2-DNS只缓存服务器实现.assets/image-20230215161024379.png)



关于接口 down的linux 怎么判断是接口不带电了，还是协议down了呢？要知道VMwareworkstaion的VM虚机的网卡断开后，接口DOWN没错，但是ip route show还能看见默认路由的。这就没有数通的产品路由生效要有下一跳可达的前置条件。

![image-20230215161439083](2-DNS只缓存服务器实现.assets/image-20230215161439083.png)

![image-20230215161514298](2-DNS只缓存服务器实现.assets/image-20230215161514298.png)

ifconfig看不出到底是命令down还是没插网线的down，交换机上物理口就有administra down和普通的down-no conn

其实除了 上面的管理员down和没插线的down 还有协议down，普通以太网口这种情况不多，顶多就是交换机的err-disable down，抓哟是ppp 串行线的口的协议down，tunnel口的协议down，这些吧。



ethtool eth1  看看

<img src="2-DNS只缓存服务器实现.assets/image-20230215162014801.png" alt="image-20230215162014801" style="zoom:67%;" /> 

mii-tool比较适合干这活

![image-20230215162117975](2-DNS只缓存服务器实现.assets/image-20230215162117975.png)

**ifconfig eth1 down 等价于网卡禁用**

**VM里断开网络来凝结 等价于把网线**





好~，现在dns server的网关就没了，无法进一步向根询问dns解析了

![image-20230215162647774](2-DNS只缓存服务器实现.assets/image-20230215162647774.png)



结果发现client解析之前解析过的，理应在server上有缓存的，竟然不成功！

![image-20230215162740123](2-DNS只缓存服务器实现.assets/image-20230215162740123.png)



发现自动补了后缀，这是杂肥四

![image-20230215162819831](2-DNS只缓存服务器实现.assets/image-20230215162819831.png)



先不着急删除/etc/resolve.conf里的自动不清的后缀domain

![image-20230215162953769](2-DNS只缓存服务器实现.assets/image-20230215162953769.png)

先去还原dns server外网试试看，恢复dns server的外网后，client  ping 域名 OK👇

![image-20230215163111975](2-DNS只缓存服务器实现.assets/image-20230215163111975.png)



然后再次禁用dns server的外网网卡

![image-20230215163203647](2-DNS只缓存服务器实现.assets/image-20230215163203647.png)

client依然可以ping通

![image-20230215163218127](2-DNS只缓存服务器实现.assets/image-20230215163218127.png)

刚才不通，可能就是server的缓存时间比较短。其实不是是本机有缓存。还需要清除本机缓存--好像没必要，

https://jaminzhang.github.io/dns/flush-dns-cache-in-Linux/

因为centos上好像本地没有dns请求的缓存记录。



dns server服务端清除dns缓存可以的，这样①重启服务②rndc flush； 后client无法解析

![image-20230215165626164](2-DNS只缓存服务器实现.assets/image-20230215165626164.png)

linux 就没有本机的dns 客户端缓存的，除非你是dns server做中继或者做解析的。



上面讲的dns本地是没有解析的，都是问别人得到的，这种就是 dns 只缓存服务器；  纯二道贩子



------------------------------------题外话----------start-------------------------

之前我给开发搭建的NPM缓存服务器就是一样的，NPM代理那里有时间复习下，关键点就是①verdaccio这个软件好像是②npm的代理 client通过它下载的软件网页上是没有记录的，代理服务器自己install的才能上网页，所以就写了个脚本讲代理下载的json文件等 复制到上网页的路径里去就搞定啦~



<img src="2-DNS只缓存服务器实现.assets/image-20230215170421102.png" alt="image-20230215170421102" style="zoom:50%;" /> 



点开都有帮助 使用案例

![image-20230215170438241](2-DNS只缓存服务器实现.assets/image-20230215170438241.png)

1年都没坏，机器上还有一个yum源，也是一周一更，香香的。



------------------------------------题外话----------end-------------------------

下面搭建右下角的本地dns解析条目

![image-20230215171237110](2-DNS只缓存服务器实现.assets/image-20230215171237110.png)











