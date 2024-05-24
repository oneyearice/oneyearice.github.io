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





一些版本需要yum -y install nginx-mod-stream 来手动安装stream模块，当然一般情况/etc/nginx/nginx.conf主配置文件里是包含了这个模块的

<img src="4-nginx四层代理功能和tengine编译安装.assets/image-20240311193546708.png" alt="image-20240311193546708" style="zoom:33%;" />





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



### 安装测下Tengine

下载最新的试试看

```
curl -LO https://tengine.taobao.org/download/tengine-3.1.0.tar.gz
 
tar xvf tengine-3.1.0.tar.gz
 
cd tengine-3.1.0.tar.gz


编译前，需安装依赖

yum install gcc pcre-devel openssl-devel zlib-devel

编译前，需要添加nginx用户
useradd -r -s /sbin/nologin nginx

编译的时候考虑下stream模块是否支持，当然支持了，2015年的nginx-1.9.0支持stream的，tengine-3.1.0都是2023年了，没问题~,其实如果不支持会报错的，同时也可以这么看下是否支持
./configure --help |grep stream   看看是否有--with-stream字眼。有就支持咯

./configure --prefix=/apps/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-pcre \
--with-stream \
--with-stream_ssl_module \
--with-stream_realip_module

 
make && make install

ln -s /apps/nginx/sbin/nginx /usr/bin/nginx

nginx 启动就行，如果报错，就按提示解决报错
```

确认是否支持stream模块👇

![image-20240311104751014](4-nginx四层代理功能和tengine编译安装.assets/image-20240311104751014.png)

然后就ok了

![image-20240311105657369](4-nginx四层代理功能和tengine编译安装.assets/image-20240311105657369.png)



尝试启用concat，但是由于编译的时候没有加上这个模块，所以还是不支持。

![image-20240311110227904](4-nginx四层代理功能和tengine编译安装.assets/image-20240311110227904.png)

结果发现开发版3.1.0不支持唉

![image-20240311110436643](4-nginx四层代理功能和tengine编译安装.assets/image-20240311110436643.png)

换

![image-20240311111026628](4-nginx四层代理功能和tengine编译安装.assets/image-20240311111026628.png)

妈的，2.3.1支持的，他这个分开发版本和稳定版，3.1.0是开发版，还没有详细的明细

![image-20240311111207265](4-nginx四层代理功能和tengine编译安装.assets/image-20240311111207265.png)

就首页有说，难不成大版本1就是稳定版，2就是开发版？



```
重新下载
curl -LO https://tengine.taobao.org/download/tengine-2.3.1.tar.gz
```

![image-20240311111603248](4-nginx四层代理功能和tengine编译安装.assets/image-20240311111603248.png)

操，有个屁



```
再换
curl -LO https://tengine.taobao.org/download/tengine-2.1.2.tar.gz
```

![image-20240311111745525](4-nginx四层代理功能和tengine编译安装.assets/image-20240311111745525.png)

有了，操，但是这个版本没有stream模块，

```
./configure --prefix=/apps/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-pcre \
--with-http_concat_module
```

报错和处理

![image-20240311112720494](4-nginx四层代理功能和tengine编译安装.assets/image-20240311112720494.png)

参考：https://www.linuxquestions.org/questions/slackware-arm-108/gcc-7-x-compile-issue-with-nginx-4175608107/



继续编译，继续报错

![image-20240311113115194](4-nginx四层代理功能和tengine编译安装.assets/image-20240311113115194.png)

处理方法，注释掉这行，

![image-20240311113247197](4-nginx四层代理功能和tengine编译安装.assets/image-20240311113247197.png)

再次编译，再次报错

![image-20240311113321949](4-nginx四层代理功能和tengine编译安装.assets/image-20240311113321949.png)

妈的，所以tenginx的concat到底行不行，为了一个concat，结果高版本的tenginx里没有concat，用低版本里结果stream不支持，而且openssl也要降版本，操。参考：https://blog.csdn.net/qq_39720249/article/details/84655501

![image-20240311113735834](4-nginx四层代理功能和tengine编译安装.assets/image-20240311113735834.png)



```
curl -LO https://www.openssl.org/source/old/1.0.1/openssl-1.0.1u.tar.gz

tar xvf openssl-1.0.1u.tar.gz

cd openssl-1.0.1u

./config --prefix=/apps/nginx/openssl-1.0.1u

make && make install

再重新编译tenginx试试看咯

```

![image-20240311115156438](4-nginx四层代理功能和tengine编译安装.assets/image-20240311115156438.png)

编译报错

![image-20240311115946043](4-nginx四层代理功能和tengine编译安装.assets/image-20240311115946043.png)

![image-20240311120108780](4-nginx四层代理功能和tengine编译安装.assets/image-20240311120108780.png)

给你个openssl目录

![image-20240311120347862](4-nginx四层代理功能和tengine编译安装.assets/image-20240311120347862.png)

![image-20240311120444899](4-nginx四层代理功能和tengine编译安装.assets/image-20240311120444899.png)

ok

![image-20240311120455040](4-nginx四层代理功能和tengine编译安装.assets/image-20240311120455040.png)

继续make

![image-20240311120729649](4-nginx四层代理功能和tengine编译安装.assets/image-20240311120729649.png)

fuck 报错是openssl.o

下错版本呢了，继续

![image-20240311120826356](4-nginx四层代理功能和tengine编译安装.assets/image-20240311120826356.png)



```
curl -LO https://www.openssl.org/source/old/1.0.1/openssl-1.0.1o.tar.gz
cd openssl-1.0.1o
tar xvf openssl-1.0.1o.tar.gz
cd openssl-1.0.1o
rm -rf tengine-2.1.2/openssl   # 删除之前的openssl目录
mv openssl-1.0.1o tengine-2.1.2/openssl

cd tengine-2.1.2

./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-http_concat_module --with-cc-opt="-Wno-error"

make && make install

还是openssl.o的报错，
```

可能是旧版的openssl没有删除导致的。

https://juejin.cn/post/7106429674942627854

按这个来一遍再



![image-20240311202759788](4-nginx四层代理功能和tengine编译安装.assets/image-20240311202759788.png)







```
yum -y install remove nginx
yum -y remove nginx

cd /usr/local/ ; curl -LO https://tengine.taobao.org/download/tengine-2.1.2.tar.gz
tar xvf tengine-2.1.2.tar.gz
cd tengine-2.1.2
ll
./configure --help |grep concat
./configure --help |grep stream
yum remove openssl openssl-devel
cd ..
curl -LO https://www.openssl.org/source/old/1.0.1/openssl-1.0.1o.tar.gz
tar xvf openssl-1.0.1o.tar.gz
cd openssl-1.0.1o
./config --prefix=/opt/ldkjdata/nginx/openssl-1.0.1o
make && make install

这是OK的
```



```
cd ../tengine-2.1.2
pwd
vim src/os/unix/ngx_user.c
./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-openssl=../openssl-1.0.1o --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-http_concat_module --with-cc-opt="-Wno-error"
make -j 2 && make install

最后一步报错openssl的问题还是，反正解决要么正面搞定openssl，要么用3.x.xtenginx安装dso的concat模块。

```



![image-20240311210108349](4-nginx四层代理功能和tengine编译安装.assets/image-20240311210108349.png)





### **方案二：就用最新的然后利用--add-module=结合单独下载模块**





**到这个网站下载concat模块**

https://github.com/alibaba/nginx-http-concat

然后下载最新的tenginx 3.1.0，再编译的时候加上这个模块就行了

```
git clone https://github.com/alibaba/nginx-http-concat.git
git clone https://github.com/vozlt/nginx-module-sts.git

cd tengine-3.1.0

./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=/root/nginx-http-concat

make -j 2 && make install


```



![image-20240312105753287](4-nginx四层代理功能和tengine编译安装.assets/image-20240312105753287.png)



OK鸡巴开了

![image-20240312105720094](4-nginx四层代理功能和tengine编译安装.assets/image-20240312105720094.png)

![image-20240312115230107](4-nginx四层代理功能和tengine编译安装.assets/image-20240312115230107.png)

虽然ok了，但是我有一个疑问啊，就是为什么

1、官网分开发稳定版和稳定版，首页稳定版只显示到2013年，什么鬼

2、开发稳定版里最新的竟然不带concat模块，还需要去11年前的github库里下载，什么鬼







**下载stream模块去试试tengine-2.1.2，不行就算了**

因为编译，所以找了个源文件，

https://github.com/vozlt/nginx-module-stream-sts



还是用tengine-2.1.2来弄试试

```
./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --add-module=/root/nginx-module-sts --with-cc-opt="-Wno-error" --add-module=/root/nginx-module-stream-sts

make dso_install     # 注意这里不直接make

make && make install  # 然后再make 
```

![image-20240312114016086](4-nginx四层代理功能和tengine编译安装.assets/image-20240312114016086.png)

![image-20240312114117855](4-nginx四层代理功能和tengine编译安装.assets/image-20240312114117855.png)

![image-20240312114133551](4-nginx四层代理功能和tengine编译安装.assets/image-20240312114133551.png)

还是一样的报错，不过stream模块好像可以这么加。

![image-20240312114237936](4-nginx四层代理功能和tengine编译安装.assets/image-20240312114237936.png)

openssl，算了不弄了。就上面的方法搞定就行了。





# nginx优化



默认的Linux内核参数考虑的是最通用场景，不符合用于支持高并发访问的Web服务器的 定义，根据业务特点来进行调整，当Nginx作为静态web内容服务器、反向代理或者提供 压缩服务器的服务器时，内核参数的调整都是不同的，此处针对最通用的、使Nginx支持 更多并发请求的TCP网络参数做简单的配置,修改/etc/sysctl.conf来更改内核参数

fs.file-max = 999999

表示单个进程较大可以打开的句柄数

net.ipv4.tcp_tw_reuse = 1

参数设置为 1 ，表示允许将TIME_WAIT状态的socket重新用于新的TCP链接，这对于 服务器来说意义重大，因为总有大量TIME_WAIT状态的链接存在

net.ipv4.tcp_keepalive_time = 600

当keepalive启动时，TCP发送keepalive消息的频度；默认是2小时，将其设置为10分钟， 可更快的清理无效链接

net.ipv4.tcp_fin_timeout = 30

当服务器主动关闭链接时，socket保持在FIN_WAIT_2状态的较大时间



net.ipv4.tcp_max_tw_buckets = 5000
表示操作系统允许TIME_WAIT套接字数量的较大值，如超过此值，TIME_WAIT  套接字将立刻被清除并打印警告信息,默认为8000，过多的TIME_WAIT套接字会使  Web服务器变慢

net.ipv4.ip_local_port_range = 1024 65000    ，比如nginx 往后端发送连接的源随机端口

定义UDP和TCP链接的本地端口的取值范围

net.ipv4.tcp_rmem = 10240 87380 12582912
定义了TCP接受缓存的最小值、默认值、较大值

net.ipv4.tcp_wmem = 10240 87380 12582912
定义TCP发送缓存的最小值、默认值、较大值

net.core.netdev_max_backlog = 8096    # backlog不是log是队列
当网卡接收数据包的速度大于内核处理速度时，会有一个列队保存这些数据包。 这个参数表示该列队的较大值





net.core.rmem_default = 6291456  表示内核套接字接受缓存区默认大小
net.core.wmem_default = 6291456  表示内核套接字发送缓存区默认大小
net.core.rmem_max = 12582912	表示内核套接字接受缓存区最大大小
net.core.wmem_max = 12582912 表示内核套接字发送缓存区最大大小

注意：以上的四个参数，需要根据业务逻辑和实际的硬件成本来综合考虑



net.ipv4.tcp_syncookies = 1
与性能无关。用于解决TCP的SYN攻击

net.ipv4.tcp_max_syn_backlog = 8192
这个参数表示TCP三次握手建立阶段接受SYN请求列队的较大长度，默认1024，将其 设置的大一些可使出现Nginx繁忙来不及accept新连接时，Linux不至于丢失客户端发起 的链接请求

net.ipv4.tcp_tw_recycle = 1
这个参数用于设置启用timewait快速回收

net.core.somaxconn=262114
选项默认值是128，这个参数用于调节系统同时发起的TCP连接数，在高并发的请求中，
默认的值可能会导致链接超时或者重传，因此需要结合高并发请求数来调节此值。

net.ipv4.tcp_max_orphans=262114
选项用于设定系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上。如 果超过这个数字，孤立链接将立即被复位并输出警告信息。这个限制指示为了防止简单的  DOS攻击，不用过分依靠这个限制甚至认为的减小这个值，更多的情况是增加这个值









# zabbix-agent 监控记录



1、主动、被动 

见https://blog.51cto.com/u_15094852/2968778

参考一下https://blog.51cto.com/shone/5333216



然后记录我的关键配置

**ser端**

用被动，也就是 “zabbix 客户端”，键值 这里写的是 ping，其实就是agent上的配置，往下看

![image-20240312152958892](4-nginx四层代理功能和tengine编译安装.assets/image-20240312152958892.png)

上图👆的30s也就是说server端30秒去取一次数据，正因为agent是被动模式，所以server-->agent取才会又30s一次的情况，如果是agent是主动模式server<---agent，此时server就是被动接收，agent发送好像就是5s一次还是100个包一次记不得了。





**agent先安装**

zabbix的安装还是要注意版本的，举例

agent和zabbix版本要统一,否则server端检测项起不来，即使agent那边zabbix-agent -t ping 回车有数值

agent安装走官网zabbix.org下载repo源文件就行：

https://www.zabbix.com/download?zabbix=6.4&os_distribution=centos&os_version=9&components=agent&db=&ws=



**agent端配置**

![image-20240312153215184](4-nginx四层代理功能和tengine编译安装.assets/image-20240312153215184.png)



agent上测试👇，其实agent上最好写成这种，但是server上我不会传参

![image-20240312153839395](4-nginx四层代理功能和tengine编译安装.assets/image-20240312153839395.png)







