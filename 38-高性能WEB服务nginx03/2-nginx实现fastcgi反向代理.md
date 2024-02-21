# 第2节. nginx实现fastcgi反向代理



# 概述

用户还是访问页面，还是http请求，不过请求的内容变成了php文件，也就是程序代码了，是动态资源。

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

是epel源里的

![image-20240220141845092](2-nginx实现fastcgi反向代理.assets/image-20240220141845092.png)



![image-20240220141906346](2-nginx实现fastcgi反向代理.assets/image-20240220141906346.png)



![image-20240220141922449](2-nginx实现fastcgi反向代理.assets/image-20240220141922449.png)

**需要安装php-fpm作为fastcgi的服务器，还需要安装php-mysql用来连接数据库。**

不过现在好像叫php-mysqlnd了，装之

![image-20240220142826383](2-nginx实现fastcgi反向代理.assets/image-20240220142826383.png)





监听的端口没写，而是socket文件，还不是tcp/ip端口。不过由于反代和fastcgi server也就是php-fpm在一个机器上，还想不用改，就用socket文件还不错。

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



关于$fastcgi_script_name具体是啥，可以在log中配置上，然后去log里查看

![image-20240221094301028](2-nginx实现fastcgi反向代理.assets/image-20240221094301028.png)

上图👆fastcgi_pass unix:xxxx是因为php-fpm配置文件里也是这样写的

<img src="2-nginx实现fastcgi反向代理.assets/image-20240221095848859.png" alt="image-20240221095848859" style="zoom:33%;" />

如果上图写成listen = 127.0.0.1:9000那么nginx的配置location里一样要写成fastcgi_pass 127.0.0.1:9000;

log里配置这个$fastcgi_script_name变量

![image-20240221094530021](2-nginx实现fastcgi反向代理.assets/image-20240221094530021.png)





![image-20240221094645223](2-nginx实现fastcgi反向代理.assets/image-20240221094645223.png)

![image-20240221094701140](2-nginx实现fastcgi反向代理.assets/image-20240221094701140.png)

可见fastcgi_script_name为php.index，其实就是用户访问的php程序的名称，通常就是url最后一个字段。

<img src="2-nginx实现fastcgi反向代理.assets/image-20240221100347295.png" alt="image-20240221100347295" style="zoom:33%;" />



![image-20240221095102077](2-nginx实现fastcgi反向代理.assets/image-20240221095102077.png)



![image-20240221095253791](2-nginx实现fastcgi反向代理.assets/image-20240221095253791.png)





# nginx+wordpress

wordpress是php写的，所以nginx也是用fastcgi去对接的，其实wordpress就相当于一个fastcgi服务器了。

![image-20240221115020121](2-nginx实现fastcgi反向代理.assets/image-20240221115020121.png)

![image-20240221115051046](2-nginx实现fastcgi反向代理.assets/image-20240221115051046.png)





### 搞定db

![image-20240221134032774](2-nginx实现fastcgi反向代理.assets/image-20240221134032774.png)

安装这个👆，启动db服务后，创建数据库和用户以及授权👇

![image-20240221134617586](2-nginx实现fastcgi反向代理.assets/image-20240221134617586.png)

确认下

![image-20240221134746128](2-nginx实现fastcgi反向代理.assets/image-20240221134746128.png)

上图/16写错了，改成192.168.%

![image-20240221140024635](2-nginx实现fastcgi反向代理.assets/image-20240221140024635.png)

在远端一台机器上测试联通性，ok👇

![image-20240221140054153](2-nginx实现fastcgi反向代理.assets/image-20240221140054153.png)



wordpress数据库准好了，接下来弄wordpress程序



### 搞定wordpress程序



直接github拉到自己规划的目录里就行

https://github.com/WordPress/WordPress.git

![image-20240221140753723](2-nginx实现fastcgi反向代理.assets/image-20240221140753723.png)

不过wordpress的版本也要php的版本适配的。

github上也提到了推荐的版本

![image-20240221141142507](2-nginx实现fastcgi反向代理.assets/image-20240221141142507.png)

php是7.4以上，mysql是8.0以上

![image-20240221141906357](2-nginx实现fastcgi反向代理.assets/image-20240221141906357.png)

我用的mariadb-server直接yum的，mariadb的版本和mysql有一个平行分叉点是5.5吧，后面mysql又经过了几个5.x的小版本就直接跳到了8，而mariadb则是跳到了10好像。

![image-20240221142354787](2-nginx实现fastcgi反向代理.assets/image-20240221142354787.png)

所以一般来讲，这里可以认为mariadb10.11满足了wordpress的db要求--mysql8.0以上的这个要求。



利用模板创建wordpress配置文件并修改

![image-20240221142918242](2-nginx实现fastcgi反向代理.assets/image-20240221142918242.png)

![image-20240221143149035](2-nginx实现fastcgi反向代理.assets/image-20240221143149035.png)

然后注意下wordpress文件夹的权限，比如某些文件夹的上传权限wp-content这个文件夹就是接收上传的。而权限是开给nginx这个用户的，因为之前针对fastcgi服务也就是php-fpm已经修改为nginx了

![image-20240221143533460](2-nginx实现fastcgi反向代理.assets/image-20240221143533460.png)

略微改一下php的访问路径

![image-20240221144135845](2-nginx实现fastcgi反向代理.assets/image-20240221144135845.png)

注意上图红框里写错了，大小写不对

修改为root /data/php/WordPress;

重启nginx后，发现访问有点问题，👇是空白页，而且是302重定向

![image-20240221145213156](2-nginx实现fastcgi反向代理.assets/image-20240221145213156.png)





重新git clone不修改wp-config.php就是这样👇

浏览器里输入http://www.site1.com/index.php，就会跳到

![image-20240221153937488](2-nginx实现fastcgi反向代理.assets/image-20240221153937488.png)

不过图片之类的不对，就是一些静态页面包裹css，图片之类的，好像没显示出来。

一下就是除了.php以外的文件，应该都属于静态文件

![image-20240221155438103](2-nginx实现fastcgi反向代理.assets/image-20240221155438103.png)



修改wp-config.php继续空白页，继续研究下



打开wp-confi.php里的debug看到502

![image-20240221163435665](2-nginx实现fastcgi反向代理.assets/image-20240221163435665.png)

也去看看错误日志

![image-20240221163405131](2-nginx实现fastcgi反向代理.assets/image-20240221163405131.png)

看到too big header，增加buffer

![image-20240221163829560](2-nginx实现fastcgi反向代理.assets/image-20240221163829560.png)

再次访问，发现原来是存在大量报错导致buffer不够用👇

![image-20240221163803662](2-nginx实现fastcgi反向代理.assets/image-20240221163803662.png)



网上说

https://stackoverflow.com/questions/70040287/php7-4-preg-replace-compilation-failed-unrecognised-compile-time-option-bi

![image-20240221164151250](2-nginx实现fastcgi反向代理.assets/image-20240221164151250.png)



![image-20240221165056229](2-nginx实现fastcgi反向代理.assets/image-20240221165056229.png)

然后我的是rockylinux

![image-20240221165128441](2-nginx实现fastcgi反向代理.assets/image-20240221165128441.png)

没有libpcre2，但是有pcre2

![image-20240221165200392](2-nginx实现fastcgi反向代理.assets/image-20240221165200392.png)

于是果断yum之

![image-20240221165259814](2-nginx实现fastcgi反向代理.assets/image-20240221165259814.png)

再记得重启php-fpm

![image-20240221165322044](2-nginx实现fastcgi反向代理.assets/image-20240221165322044.png)

此时再次浏览器访问http://www.site1.com/index.php就好啦

![image-20240221165446395](2-nginx实现fastcgi反向代理.assets/image-20240221165446395.png)

当然是存在302跳转的。

![image-20240221170048256](2-nginx实现fastcgi反向代理.assets/image-20240221170048256.png)

![image-20240221171446017](2-nginx实现fastcgi反向代理.assets/image-20240221171446017.png)



### 这里可以简单做一个动静分离

![image-20240221171833366](2-nginx实现fastcgi反向代理.assets/image-20240221171833366.png)

改之

![image-20240221172147460](2-nginx实现fastcgi反向代理.assets/image-20240221172147460.png)

这样xx.php的还是走location去 /data/php/WordPress/下找

而不是xx.php的就去/data/site1/wordpressStatics/下找

![image-20240221172459300](2-nginx实现fastcgi反向代理.assets/image-20240221172459300.png)

再次访问http://www.site1.com/index.php看看

没得问题👇

![image-20240221172557802](2-nginx实现fastcgi反向代理.assets/image-20240221172557802.png)



然后优化成访问http://www.site1.com直接就是上图的页面，也就是说index 设置为index.php了。

安排~具体配置如下

![image-20240221174632675](2-nginx实现fastcgi反向代理.assets/image-20240221174632675.png)

![image-20240221174647172](2-nginx实现fastcgi反向代理.assets/image-20240221174647172.png)

尝试在root下新建一个index.html看看，因为index index.php index.html是从左到右先找先得。

![image-20240221174831165](2-nginx实现fastcgi反向代理.assets/image-20240221174831165.png)



![image-20240221174927665](2-nginx实现fastcgi反向代理.assets/image-20240221174927665.png)

好了

![image-20240221175031801](2-nginx实现fastcgi反向代理.assets/image-20240221175031801.png)

这样就实现了输入http://www.site1.com，注意上图我输入的时候最后是没有/的，是nginx自己302补了一个/![image-20240221175204504](2-nginx实现fastcgi反向代理.assets/image-20240221175204504.png)



所以index的php文件要有，才能跳到location 的php里

![image-20240221175356803](2-nginx实现fastcgi反向代理.assets/image-20240221175356803.png)



![image-20240221180120202](2-nginx实现fastcgi反向代理.assets/image-20240221180120202.png)

![image-20240221180143203](2-nginx实现fastcgi反向代理.assets/image-20240221180143203.png)

![image-20240221180148051](2-nginx实现fastcgi反向代理.assets/image-20240221180148051.png)

继续排错吧，哈哈



看图是http://site1.com/wp-admin/的403，我基本就认为是改路径下没有index.php和inde.html

还是因为我做了动静分离，所以这个URL走的是下图的框框里的路径

![image-20240221180433715](2-nginx实现fastcgi反向代理.assets/image-20240221180433715.png)

而该路径下是没有index文件的，补一个就行了，如果坚持要动静分离的话。 其实如果不做动静分离也没这么些毛病。

![image-20240221180449088](2-nginx实现fastcgi反向代理.assets/image-20240221180449088.png)



![image-20240221180619652](2-nginx实现fastcgi反向代理.assets/image-20240221180619652.png)

再次登入就OK啦👇，上图的echo 只是满足一下让他有个跳转到location里。而location里的root /data/php/WordPress/下的wp-admin下是有各种php文件的，关键是有index.php

![image-20240221180613373](2-nginx实现fastcgi反向代理.assets/image-20240221180613373.png)



![image-20240221180818301](2-nginx实现fastcgi反向代理.assets/image-20240221180818301.png)









![image-20240221181506989](2-nginx实现fastcgi反向代理.assets/image-20240221181506989.png)







创建文章，上传图片的时候报错

![image-20240221181845566](2-nginx实现fastcgi反向代理.assets/image-20240221181845566.png)



看error log

![image-20240221182014407](2-nginx实现fastcgi反向代理.assets/image-20240221182014407.png)

所以你认为到底是哪个root呢？哈哈，上图是referrer从http://www.site1.com/wp-admin/post.php?post=8&action=eidt跳过来的，姑且认为在下面的root里面吧

![image-20240221182044739](2-nginx实现fastcgi反向代理.assets/image-20240221182044739.png)

所以去/data/php/WordPress/wp-admin/看看





![image-20240221182532365](2-nginx实现fastcgi反向代理.assets/image-20240221182532365.png)

其实想想也知道，这些多层级的文件夹，肯定不用手动创建的嘛，就是动静分离后一些程序没有生效导致的。



这里手动创建目录，然后接着报错变了



![image-20240221182515920](2-nginx实现fastcgi反向代理.assets/image-20240221182515920.png)

![image-20240221182715501](2-nginx实现fastcgi反向代理.assets/image-20240221182715501.png)

继续报错，可能是php程序没有打赏alt attribute，我猜是动静分离后一些程序没有运行起来。

![image-20240221182701481](2-nginx实现fastcgi反向代理.assets/image-20240221182701481.png)

得，我不弄动静分离了，合起来算了



![image-20240221182806635](2-nginx实现fastcgi反向代理.assets/image-20240221182806635.png)



果然好啦

![image-20240221182845560](2-nginx实现fastcgi反向代理.assets/image-20240221182845560.png)

其实很好理解，你强行做的动静分离，人家一些php没有办法起作用了。













