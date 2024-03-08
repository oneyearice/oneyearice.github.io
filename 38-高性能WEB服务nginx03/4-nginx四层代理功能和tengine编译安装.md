# 第4节. nginx四层代理功能和tengine编译安装



# 非常经典的一致性hash算法

①背景问题产生的原因：接上篇，上篇交代了该算法产生的背景，是缓存穿透。所以才有了这么个   "一致性hash算法"

https://www.xiaolincoding.com/os/8_network_system/hash.html#%E5%A6%82%E4%BD%95%E5%88%86%E9%85%8D%E8%AF%B7%E6%B1%82

可以看看这篇介绍，别人写的👆

简单总结下

②为了解决上述问题(①里介绍)，于是做了如下优化

```
hash (/a.html) = 128bit % 2^32 = 0 ~ 2^32-1     # 一个url就是某一个值，无数个类似的值就组成了一个0~2^32范围的区间,这区间他们说看成一个环，好，先这么着，依你~ 环你大爷，为啥环？先理解只是不吵，我估计就是取模的数据模型可能就是不断的各种被除数都是逃不出这个余数的圈圈咯。

varnish 1 hash(192.168.126.101) % 2^32   # varnish缓存的ip也一样落在上面的0~2^32区间里
varnish 2    # 这个节点权重是2，也是生成2次随机数👇，这两个随机数也落在区间里的2个位置。
				hash(192.168.126.102 + random_1) % 2^32
				hash(192.168.126.102 + random_2) % 2^32
varnish 3	# 这个节点权重是3，也是生成3次随机数👇，这两个随机数也落在区间里的3个位置。
				hash(192.168.126.103 + random_1) % 2^32
				hash(192.168.126.103 + random_2) % 2^32
				hash(192.168.126.103 + random_3) % 2^32
```

一句话总结：调度的节点 落在 区间里；请求的url也落在区间里。把区间看成换于是就是这样了👇

大概意思就是黑圈是url，绿框就是varnish节点，都是通过哈希取模的算法落进去的。

![image-20240305093709350](4-nginx四层代理功能和tengine编译安装.assets/image-20240305093709350.png)

然后调度的时候就是，顺时针，某个黑圈就是url咯，于是这个url的请求就调度到顺时针最近的一个绿框也就是varnish。



③虽然优化，但是还是存在问题：

调度上存在不均衡的情况，而且是经常性的。称之为"倾斜"，下图就调度倾斜的厉害咯👇

![image-20240305095803742](4-nginx四层代理功能和tengine编译安装.assets/image-20240305095803742.png)

好，问题就是后端5个节点如何均衡调度呢？

解决方法就是虚拟节点，大量的虚拟节点充斥这哈希环，于是就错落在了url落脚点周围了，就均衡了。

<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240305101415867.png" alt="image-20240305101415867" style="zoom:33%;" />

那如何生成虚拟节点呢，答案就是 利用权重

```
varnish 1 hash(192.168.126.101) % 2^32   # varnish缓存的ip也一样落在上面的0~2^32区间里
varnish 2    # 这个节点权重是2，也是生成2次随机数👇，这两个随机数也落在区间里的2个位置。
				hash(192.168.126.102 + random_1) % 2^32
				hash(192.168.126.102 + random_2) % 2^32
varnish 3	# 这个节点权重是3，也是生成3次随机数👇，这两个随机数也落在区间里的3个位置。
				hash(192.168.126.103 + random_1) % 2^32
				hash(192.168.126.103 + random_2) % 2^32
				hash(192.168.126.103 + random_3) % 2^32

本来 1 2 3 就是权重，就是生成几个落脚点，现在改成1000 2000 3000，这样权重的调度比例还是1：2：3没变，就是3个varnish缓存节点的调度比重不变，但是1000 2000 3000，就是分别会有1000个随机数来生成1000个哈希环上的落脚点。
hash(192.168.126.102 + random_1) % 2^32
hash(192.168.126.102 + random_2) % 2^32
... ...
hash(192.168.126.102 + random_1000) % 2^32

这样就实现了虚拟节点。1000个落脚点，其实都是varnish的节点映射。
```



<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240307100713570.png" alt="image-20240307100713570" style="zoom:50%;" />



<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240307100717370.png" alt="image-20240307100717370" style="zoom:50%;" />





讲了这么多，实现起来，一个单词consistent就搞定了。

```shell
hash $request_uri consistent;  
```



对比下lvs的调度算法：

静态和动态

静态4种，rr，wr，sh，dh

![image-20240307115633000](4-nginx四层代理功能和tengine编译安装.assets/image-20240307115633000.png)

![image-20240307115640506](4-nginx四层代理功能和tengine编译安装.assets/image-20240307115640506.png)

静态的4种，nginx都是支持的，wrr默认的也可以写权重也就是等价于LVS的RR和WRR

然后nginx的hash 写变量 hash $remote_addr; ip_hash就是SH，hash $requst_uri;就是目标地址不过是url了不是DH，不过意思是一样的，都是基于目标调度--就是什么目标调度到什么机器上。



6种动态：LC、

最少连接LC，这个nginx也有 leat_conn，而且nginx加个weight值也就是wlc了。

![image-20240307132331915](4-nginx四层代理功能和tengine编译安装.assets/image-20240307132331915.png)



最少加权连接WLC，上一行已经说过了，nginx也是有的



![image-20240307133514639](4-nginx四层代理功能和tengine编译安装.assets/image-20240307133514639.png)



以及都是基于连接数和负载的一个调度，不过LVS都是基于四层的，并不涉及应用层，而nginx可以。

​     <img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240307133108326.png" alt="image-20240307133108326" style="zoom:40%;" /> 

​      <img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240307133350615.png" alt="image-20240307133350615" style="zoom:40%;" />

 

![image-20240307133443146](4-nginx四层代理功能和tengine编译安装.assets/image-20240307133443146.png) 







# nginx做四层调度

之前http应用层调度用的是ngx_http_upstream-module，现在做L4的调度，用的是ngx_stream_core_module模块。



所以http的反代-调度是写在http语句块下的，

而，L4的反代-调度是写在stream语句块下的



![image-20240307152126668](4-nginx四层代理功能和tengine编译安装.assets/image-20240307152126668.png)

1.9才开始支持L4的反代

![image-20240307152152554](4-nginx四层代理功能和tengine编译安装.assets/image-20240307152152554.png)





![image-20240307152827346](4-nginx四层代理功能和tengine编译安装.assets/image-20240307152827346.png)



点击去

![image-20240307152929978](4-nginx四层代理功能和tengine编译安装.assets/image-20240307152929978.png)





就是2015年4月29 开始支持的咯





## 开始实验-tcp代理db



**1、搞两个 mysql做realserver**



创建数据库便于nginx反代调度后 client测试 知道连的哪个realserver。

![image-20240307155751801](4-nginx四层代理功能和tengine编译安装.assets/image-20240307155751801.png)



![image-20240307154530057](4-nginx四层代理功能和tengine编译安装.assets/image-20240307154530057.png)

<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240307155653156.png" alt="image-20240307155653156" style="zoom: 33%;" />





**2、nginx反代配置基于tcp协议也就是L4的调度**

![image-20240307162412732](4-nginx四层代理功能和tengine编译安装.assets/image-20240307162412732.png)

stream语句块是写在main下的，也就是和http



就写到主配指文件里，放到最下面，和http平级，自然就是main层级了。

![image-20240307164636635](4-nginx四层代理功能和tengine编译安装.assets/image-20240307164636635.png)

重启nginx就会看到本地就会监听3306，这样才能反代哦

这里也可以写成0.0.0.0:3306

![image-20240307164917175](4-nginx四层代理功能和tengine编译安装.assets/image-20240307164917175.png)

顺手测下端口连通OK的👇

![image-20240307164954278](4-nginx四层代理功能和tengine编译安装.assets/image-20240307164954278.png)



这就好了，client测试

![image-20240307165952532](4-nginx四层代理功能和tengine编译安装.assets/image-20240307165952532.png)

发现调度到了132这个DB上了

多试几次也会调度到133，因为是least_conn，就两台realserver都没有连接的。

![image-20240307170040323](4-nginx四层代理功能和tengine编译安装.assets/image-20240307170040323.png)



试试权重

![image-20240307171001677](4-nginx四层代理功能和tengine编译安装.assets/image-20240307171001677.png)

测试OK👇

![image-20240307170939556](4-nginx四层代理功能和tengine编译安装.assets/image-20240307170939556.png)





## 开始实验-tcp代理redis



**1、安装redis，并修改监听所有端口**

这里简单用下redis，后面单独开章节

在后端realserver，两台 上都安装redis，当然得有epel源。

```
yum -y install redis
```

以前看到yum redis会有依赖包，现在看不到了

![image-20240308152903495](4-nginx四层代理功能和tengine编译安装.assets/image-20240308152903495.png)

![image-20240308152749491](4-nginx四层代理功能和tengine编译安装.assets/image-20240308152749491.png)







![image-20240308145330202](4-nginx四层代理功能和tengine编译安装.assets/image-20240308145330202.png)

修改配置文件监听所有端口，👇删掉也一样的效果

![image-20240308145500370](4-nginx四层代理功能和tengine编译安装.assets/image-20240308145500370.png)

![image-20240308145545096](4-nginx四层代理功能和tengine编译安装.assets/image-20240308145545096.png)



redis集群搭建后面再弄。



**测试下，本地测试就行**
![image-20240308150103650](4-nginx四层代理功能和tengine编译安装.assets/image-20240308150103650.png)

可以看到redis cli里自带帮助信息的。

最简单的就是设置key value，键值对。

写，读👇

![image-20240308150311795](4-nginx四层代理功能和tengine编译安装.assets/image-20240308150311795.png)

这是134的一个redis的键值对👆，

再设置133的redis里的一个key value，区分开来方便测试效果

发现默认要本地配置，远端默认还不行

![image-20240308150602510](4-nginx四层代理功能和tengine编译安装.assets/image-20240308150602510.png)

改本地配置👇

![image-20240308150529912](4-nginx四层代理功能和tengine编译安装.assets/image-20240308150529912.png)



或者取消proctected mode

![image-20240308150752029](4-nginx四层代理功能和tengine编译安装.assets/image-20240308150752029.png)

![image-20240308150821575](4-nginx四层代理功能和tengine编译安装.assets/image-20240308150821575.png)

上图取消肯定不安全咯，推荐去配置文件里写明bind ip就行了--就是监听自己的IP就行了。![image-20240308153519706](4-nginx四层代理功能和tengine编译安装.assets/image-20240308153519706.png)





**3、配置nginx 反代 到redis去**
![image-20240308151337392](4-nginx四层代理功能和tengine编译安装.assets/image-20240308151337392.png)



![image-20240308151646616](4-nginx四层代理功能和tengine编译安装.assets/image-20240308151646616.png)

但是发现默认说好的1:1的调度，默认权重=1嘛。但是没有看到134的redis的key value出现

调整调度测试下👇

![image-20240308151853583](4-nginx四层代理功能和tengine编译安装.assets/image-20240308151853583.png)

继续观察

![image-20240308152502723](4-nginx四层代理功能和tengine编译安装.assets/image-20240308152502723.png)

![image-20240308152448984](4-nginx四层代理功能和tengine编译安装.assets/image-20240308152448984.png)

👆说好的1:3也没看到。



要exit退出测试才能看到调度效果，继续测试

调度也不是立马生效的好像

<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240308153800514.png" alt="image-20240308153800514" style="zoom:44%;" />



要退出才能看到1:3权重效果

<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240308154036850.png" alt="image-20240308154036850" style="zoom:44%;" />





<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240308154426204.png" alt="image-20240308154426204" style="zoom:44%;" />



然后redis是比member cache强，Memcache是基于内存的，redis是可以内存也可以放到磁盘上。



注意也可以用UDP。

![image-20240308160650936](4-nginx四层代理功能和tengine编译安装.assets/image-20240308160650936.png)



<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240308160858732.png" alt="image-20240308160858732" style="zoom:40%;" />

超时间也关注下



<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240308161820725.png" alt="image-20240308161820725" style="zoom:40%;" />

consistent就是前文讲的哈希一致性算法，不过这里是针对的源IP计算的咯，所以再来看这图

![image-20240308161924284](4-nginx四层代理功能和tengine编译安装.assets/image-20240308161924284.png)

这里的node--也就是后端realservers以及键--就是hash $xxx，xxx是requset_uri还是remote_addr基于你怎么写。所以均匀的调度还真可能就是依赖这个consistent关键词了，呵呵。



### nginx 不能这里用下划线_低版本？

![image-20240308162649111](4-nginx四层代理功能和tengine编译安装.assets/image-20240308162649111.png)

![image-20240308162709503](4-nginx四层代理功能和tengine编译安装.assets/image-20240308162709503.png)

调用的地方自然也不能咯



**如果用下划线会有问题 据说**

测试👇

![image-20240308162935020](4-nginx四层代理功能和tengine编译安装.assets/image-20240308162935020.png)

修改为下划线

![image-20240308163202116](4-nginx四层代理功能和tengine编译安装.assets/image-20240308163202116.png)

也没问题，高版本修复了可能👆。 估计就是低版本里的_下划线，代码里没有匹配好比如regex写漏了？



# nginx的二次开发的版本



https://tengine.taobao.org/

![image-20240308163738823](4-nginx四层代理功能和tengine编译安装.assets/image-20240308163738823.png)



比如 ngx_http_concat_module模块的效率

https://tengine.taobao.org/document_cn/http_concat_cn.html

该模块类似于apache中的mod_concat模块，用于合并多个文件在一个响应报文中。









