# 第3节. nginx反向代理实现负载均衡及调度方法

# nginx动态资源缓存

接上文，fastcgi的缓存利用和之前讲的基本一样

之前的缓存是nginx针对静态页面的缓存👇

![image-20240223102011237](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223102011237.png)

现在要梳理的是nginx针对动态页面方面的fastcgi的缓存支持

![image-20240223102339002](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223102339002.png)



对比下

![image-20240223102255395](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223102255395.png)

①fastcgi_cache zone |off     # 调用keys_zone=xxx起的名字xxx:20m 

②fastcgi_cache_key string  # 就是哪些部分作为哈希缓存，就是所谓的key string

③fastcgi_cache_methods GET | HEAD | POST ...;    # 哪些方式定义缓存

④fastcgi_keep_conn on | off ;   # 收到后端realserver后是否立即关闭连接，推荐启用长连接

⑤fastcgi_cache_valid [CODE ... ]time;  # 不同的响应码的缓存时长



## fastcgi缓存的具体配置



主配配置文件里修改一下👇

![image-20240223111040548](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223111040548.png)

然后调用这个定义的keys_zone:fcgicache

![image-20240223111650805](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223111650805.png)

对比测试：未开启的效率

![image-20240223113215761](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223113215761.png)

对比测试👇

![image-20240223113247955](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223113247955.png)

响应速度明显提升

![image-20240223113311323](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223113311323.png)



这是缓存信息👇

![image-20240223115703750](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223115703750.png)





这就是缓存的效果，速度大大提升，问题--动态页面静态化了；什么意思，就是命名时php程序的访问，结果变成了缓存的静态资源应答，页面的更新明显时滞后的，所以使用的场景一般就是博客文章、论坛这些不要实时的业务了。





另外，上面时php程序的服务，用fastcgi对接；

如果是java程序，比如tomcat的，就直接nginx代理就行就是http协议i对接。

如果是python程序，比如django框架，就需要用uwsgi_params，其实nginx也是交给类似uwsgi的服务，nginx把请求交给该服务，然后该uwsgi服务再转给python程序。就类似nginx发给php-fpm，php-fpm再转给wordpress这个php代码一样。

![image-20240223132630845](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223132630845.png)





# 调度功能

终于学到这一章节啦，调度才是网工普遍关心的点，也是从F5联系过来的吧。



upstream模块，可以把后端realserver分组起名，nginx就往这些分组上调度，每个组里一般就有好多服务器，此时就不再固定的只转发到一台机器了。



配置逻辑上 👇下图是一个tcp 反代，配置在stream层级下的，如果是http反代是写到http模块里的。

<img src="3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223140525519.png" alt="image-20240223140525519" style="zoom:50%;" /> 

①先用upstream定义出一组服务器定义出来，后端serverS的ip+port

②然后把这些ip和端口，起一个统一的名字

③接着往这些分组上调度，用proxy_pass调度到upstream定义的分组名称上

④stream的上下文是main，换句话讲 stream是配置在全局最顶层的；

⑤upstream是配置再stream或http下的。

![image-20240223144203114](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223144203114.png)

![image-20240223143637539](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223143637539.png)

![image-20240223143621402](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223143621402.png)



一些参数看看

![image-20240223142147918](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223142147918.png)





### 先配置http的反代调度

简单讲就是proxy_pass不再指向某个固定url，而是一个定义好的组名，组内有N个url。



一般来讲 配置上，一个是主配指文件/etc/nginx/nginx.conf 和 其他子配置文件/etc/nginx/conf.d/xxx.conf。 前置放http，后者放独立的server虚拟主机。



所以按照这个思路，我就把upstream要去主配置文件里配置，而不是折腾到子配置文件里了对吧，否则冲突了就。



准备两个后端服务器，并配置好测试页面

![image-20240223145936027](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223145936027.png)

![image-20240223150033819](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223150033819.png)

然后在nginx的主配置文件里的http层级下，定义好虚拟主机组

<img src="3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223154404530.png" alt="image-20240223154404530" style="zoom:50%;" />



<img src="3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223155004585.png" alt="image-20240223155004585" style="zoom:33%;" />



然后在子配置文件里，配置调度

![image-20240223160631933](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223160631933.png)

上图的index.php要删掉，否则还是访问到之前实验的php文件了。

重启服务，测试

![image-20240223161534965](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223161534965.png)

发现没有轮询，memebers，而且即使192.168.126.134 httpd服务down掉了，还是显示134，说明缓存起作用咯。

<img src="3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223161737898.png" alt="image-20240223161737898" style="zoom:50%;" />

确实开启了缓存

![image-20240223161816534](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223161816534.png)



看下缓存文件，



此时2分钟时间到了，缓存失效了，再次curl发现轮询调度到另一台机器了也就是135

![image-20240223161943055](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223161943055.png)

再次开启134的httpd，这样134和135的apache服务都开着了，

然后测试如下图

![image-20240223162127631](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223162127631.png)

发现轮询是存在的。



干脆关闭缓存

![image-20240223162251255](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223162251255.png)

再次测试轮询

![image-20240223162315580](3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223162315580.png)

也是ok的。上面已经实现了自动调度了，



### 再看下调度的算法的细节

首先，默认就是wrr调度机制

wrr就是Nginx 调度算法中的 "WRR" 全称是 "Weighted Round Robin"，即加权轮询算法。

测试，通过配置权重，观察curl的返回站点的页面，看调度比例是否改变。

配置之前，先看下上图默认wrr 大家一样的权重，调度基本是均匀的。

<img src="3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223164521770.png" alt="image-20240223164521770" style="zoom:50%;" />



测试发现还真是3:1的调度比例

<img src="3-nginx反向代理实现负载均衡及调度方法.assets/image-20240223164617408.png" alt="image-20240223164617408" style="zoom:50%;" />









