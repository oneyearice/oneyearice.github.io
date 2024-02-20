# 第2节. nginx实现fastcgi反向代理



# 概述

用户还是访问页面，还是http请求，不过请求的内容变成了php文件。

![image-20240220140451626](2-nginx实现fastcgi反向代理.assets/image-20240220140451626.png)

现在nginx反代收到这个php的http请求，本身是处理不了的，需要通过fastcgi协议把php请求发送给后端的php服务器，类似fpm-php服务器，默认监听9000端口。



这种就是异构了👇

<img src="2-nginx实现fastcgi反向代理.assets/image-20240220140322768.png" alt="image-20240220140322768" style="zoom:33%;" />





搭建一个fastcgi的real server

1、物理上可以将反代nginx和real server放在一台服务器上；

2、也可以分开放。



# 实验：反代+real server放在一台机器上



先搞一个fastcgi的服务器：

![image-20240220141640646](2-nginx实现fastcgi反向代理.assets/image-20240220141640646.png)



![image-20240220141716757](2-nginx实现fastcgi反向代理.assets/image-20240220141716757.png)

也是epel源里的

![image-20240220141845092](2-nginx实现fastcgi反向代理.assets/image-20240220141845092.png)



![image-20240220141906346](2-nginx实现fastcgi反向代理.assets/image-20240220141906346.png)



![image-20240220141922449](2-nginx实现fastcgi反向代理.assets/image-20240220141922449.png)

**需要安装php-fpm作为fastcgi的服务器，还需要安装php-mysql用来连接数据库。**

不过现在好像叫php-mysqlnd了，装之

![image-20240220142826383](2-nginx实现fastcgi反向代理.assets/image-20240220142826383.png)





监听的端口没写，而是监听用的是socket还不是tcp/ip端口。不过由于反代和fastcgi server也就是php-fpm在一个机器上，还想不用改，就用socket文件还不错。

![image-20240220143900756](2-nginx实现fastcgi反向代理.assets/image-20240220143900756.png)

然后listen.allowed_cliants 就是127本地来连接。

需要修改的是user = apache 改为user = nginx，因为php-fpm这里配合的是nginx不是apache。

要用nginx作为服务的启动账户，也会开启类似httpd的worker进程，master进程。这里nginx就是work进程的用户身份。

不改就会存在问题，比如nginx传一个文件过来，php用其他的账号，可能就会存在权限不够的情况。

![image-20240220145028937](2-nginx实现fastcgi反向代理.assets/image-20240220145028937.png)

修改user 和 group 为nginx

![image-20240220145136534](2-nginx实现fastcgi反向代理.assets/image-20240220145136534.png)



重启服务后，work进程的用户就是nginx了，当然master进程还的是root

![image-20240220145219418](2-nginx实现fastcgi反向代理.assets/image-20240220145219418.png)



由于是socket文件，所以ss -tlnup 就没必要看监听端口了

然后php程序放在哪？nginx的站点root路径？往下看，可以不在一起的

![image-20240220151215703](2-nginx实现fastcgi反向代理.assets/image-20240220151215703.png)

由于上图的虚拟主机没有配置root，所以报错

![image-20240220151238805](2-nginx实现fastcgi反向代理.assets/image-20240220151238805.png)



补上root

![image-20240220151315349](2-nginx实现fastcgi反向代理.assets/image-20240220151315349.png)

ok

![image-20240220151332761](2-nginx实现fastcgi反向代理.assets/image-20240220151332761.png)

将php程序放到别的地方，

![image-20240220171159818](2-nginx实现fastcgi反向代理.assets/image-20240220171159818.png)

下面要让nginx收到http请求访问php程序的报文后，nginx要知道去找php-fpm服务，然后该服务(从socket文件或者9000端口收到后)还得知道去哪找php程序，也就是上图的路径。



1、首先nginx反代上要有fastcgi模块：ngx_http_fastcgi_module

2、然后要配置fastcgi服务器的地址：fastcgi_pass address；fastcgi和反代在一起就指向本机，不过是127.0.0.1还是本机的IP地址需要明确一下。

3、指定类似index.html的php资源，一般就是index.php，指定cli为：fastcgi_index index.php;

4、fastcgi和nginx之间的数据交换，需要用一些参数，这些参数是固定格式的，引用就行：fastcgi_param parameter value [if_not_empty];   所以看看fastcgi_params参数👇，直接引用就行：

![image-20240220180724576](2-nginx实现fastcgi反向代理.assets/image-20240220180724576.png)

参数的使用是需要在配置文件里引用这个文件的，具体就是location里配置 include fastcgi_params;

![image-20240220181437433](2-nginx实现fastcgi反向代理.assets/image-20240220181437433.png)



fastcgi_params里就是一个文件，里面内容上面也讲了。

![image-20240220181614116](2-nginx实现fastcgi反向代理.assets/image-20240220181614116.png)

然后fastcgi_param看起来就是自己定义的一些其他参数咯，比如

![image-20240220181859443](2-nginx实现fastcgi反向代理.assets/image-20240220181859443.png)

SCRIPT_FAILENAME脚本文件名，$document_root就是 root /xx/xx定义的根路径；而fastcgi_script_name就是脚本名称。

上图也写明了，只有访问 ~* \.php$的请求才发送到fastcgi_param参数指定的路径/data/php下的$fastcgi_script_name这个文件。



















