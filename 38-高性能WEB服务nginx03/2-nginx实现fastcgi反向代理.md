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





![image-20240222155113576](2-nginx实现fastcgi反向代理.assets/image-20240222155113576.png)



<img src="2-nginx实现fastcgi反向代理.assets/image-20240222155152395.png" alt="image-20240222155152395" style="zoom:50%;" />







总结下，一两句简单点就是：fastcgi是快速的网关接口，所谓快速就是 类似nginx的master和worker父子进程的一个处理效率；所谓网关接口，就是web服务和php程序代码的一个通信协议。





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







## php的一个参数项

在fastcgi的配置里，也就是php-fpm服务的配置里

<img src="2-nginx实现fastcgi反向代理.assets/image-20240222093917868.png" alt="image-20240222093917868" style="zoom:50%;" /> 



<img src="2-nginx实现fastcgi反向代理.assets/image-20240222105032133.png" alt="image-20240222105032133" style="zoom:50%;" /> 

不过和apache的status冲突了，大家都是根路径带一个/status，不知道访问哪一个了，改之

![image-20240222105339891](2-nginx实现fastcgi反向代理.assets/image-20240222105339891.png)



然后测试fastcgi服务器是否ok的

![image-20240222110236658](2-nginx实现fastcgi反向代理.assets/image-20240222110236658.png)



以上完成了①status页面②ping的页面，这是php-fpm也就是fastcgi的设置，

下面做nginx的配置，将这个两个/fpm_stats   /ping的页面  路由到fastcgi服务上去才行啊



<img src="2-nginx实现fastcgi反向代理.assets/image-20240222111113953.png" alt="image-20240222111113953" style="zoom:40%;" /> 



![image-20240222112225216](2-nginx实现fastcgi反向代理.assets/image-20240222112225216.png)

上图的locaiton 一行  ~*表示正则忽略大小姐，   ^表示主机头后面的部分也就是www.site1.com后面的部分以/开头的。

![image-20240222112243902](2-nginx实现fastcgi反向代理.assets/image-20240222112243902.png)

ping   pong~

![image-20240222112442836](2-nginx实现fastcgi反向代理.assets/image-20240222112442836.png)



xxxxxx/fpm_status?full查看详情👇 所谓详情就是php-fpm服务的所有进程都给你显示出来了

![image-20240222134652486](2-nginx实现fastcgi反向代理.assets/image-20240222134652486.png)







## 将nginx和php-fpm分开来

### 独立安装php-fpm和php-mysqlnd

上面是合在一起的，用的socket文件

①php-fpm的配置，也就是监听

②nginx的fastcgi对接

![image-20240222141457861](2-nginx实现fastcgi反向代理.assets/image-20240222141457861.png)

合在一起，也可以改成listen = 127.0.0.1:9000    以及 nginx的 fastcgi_pass 127.0.0.1:9000



分开来也没啥好说的，无非就是mysql独立、php-fpm独立、nginx独立，用ip port互相连接就行了。

一些信息操作的记录下

安装php-fpm的yum源除了默认EPEL，还可以用REMI仓库，REMI也是依赖EPEL源的。

![image-20240222160027459](2-nginx实现fastcgi反向代理.assets/image-20240222160027459.png)



找到所需的资源

![image-20240222160453770](2-nginx实现fastcgi反向代理.assets/image-20240222160453770.png)



 yum install  https://mirrors.tuna.tsinghua.edu.cn/remi/enterprise/remi-release-9.rpm

或者下载下来后， yum install remi-release-9.rpm

![image-20240222161701659](2-nginx实现fastcgi反向代理.assets/image-20240222161701659.png)



![image-20240222161833538](2-nginx实现fastcgi反向代理.assets/image-20240222161833538.png)



![image-20240222161926677](2-nginx实现fastcgi反向代理.assets/image-20240222161926677.png)



可以用rpm -ql看看installed包里的内容

![image-20240222162010116](2-nginx实现fastcgi反向代理.assets/image-20240222162010116.png)

说白了就是给你安装，也就是自动创建了yum源文件





![image-20240222162821372](2-nginx实现fastcgi反向代理.assets/image-20240222162821372.png)

![image-20240222162912102](2-nginx实现fastcgi反向代理.assets/image-20240222162912102.png)

找到这个版本较高的php-fpm了就





再一个确认下

![image-20240222163242495](2-nginx实现fastcgi反向代理.assets/image-20240222163242495.png)

激活之

![image-20240222163416288](2-nginx实现fastcgi反向代理.assets/image-20240222163416288.png)

发现无所谓了

![image-20240222163435705](2-nginx实现fastcgi反向代理.assets/image-20240222163435705.png)

一样的remi源和remi-safe一样的，估计就是一个更加safe咯。



除了安装这个php-fpm还需要安装这个软件和mysql互通的软件php-mysqlnd

![image-20240222164901871](2-nginx实现fastcgi反向代理.assets/image-20240222164901871.png)



装之  yum -y install php83-php-fpm php83-php-mysqlnd

![image-20240222164948717](2-nginx实现fastcgi反向代理.assets/image-20240222164948717.png)

![image-20240222165133528](2-nginx实现fastcgi反向代理.assets/image-20240222165133528.png)

php-fpm是通过php-pdo或者php-mysqlnd来支持 连接数据库的



### 配置一下php-fpm



需要在php-fpm服务器上创建nginx用户和用户组，创建之前去nginx机器上看下uid，然后php-fpm创建的时候要保持一致。

![image-20240222173552513](2-nginx实现fastcgi反向代理.assets/image-20240222173552513.png)

groupadd -g 989 nginx

useradd -r -u 989 -g nginx -s /sbin/nologin nginx

如果只是有nginx用户名，不保持ID一致会咋样，我怎么感觉没必要id一致呢，试试

![image-20240222173951863](2-nginx实现fastcgi反向代理.assets/image-20240222173951863.png)



![image-20240222174237895](2-nginx实现fastcgi反向代理.assets/image-20240222174237895.png)

上图的注释方式不适合php-fpm的配置文件，需要将#改成;实现注释，否则报错，如下面stauts



![image-20240222175108335](2-nginx实现fastcgi反向代理.assets/image-20240222175108335.png)放行所有人连接过来，其实可以写nginx的IP地址会不会安全点👇，结果写错了应该写130而不是134，[然后这就导致了502报错了见下面的502报错页面](#jump1)

![image-20240222175409115](2-nginx实现fastcgi反向代理.assets/image-20240222175409115.png)

状态页面也改一下：

<img src="2-nginx实现fastcgi反向代理.assets/image-20240222174349075.png" alt="image-20240222174349075" style="zoom:50%;" /> 



改完就启服

![image-20240222175730411](2-nginx实现fastcgi反向代理.assets/image-20240222175730411.png)

这是之前的报错已修正👇

![image-20240222174655498](2-nginx实现fastcgi反向代理.assets/image-20240222174655498.png)

通过status看到具体错误的地方，就是注释#改成;就行了

![image-20240222174925556](2-nginx实现fastcgi反向代理.assets/image-20240222174925556.png)



### 放置好php程序

![image-20240222180339486](2-nginx实现fastcgi反向代理.assets/image-20240222180339486.png)

把项目down下来，直接git clone url就行，网络不给力就down zip吧



![image-20240222180450698](2-nginx实现fastcgi反向代理.assets/image-20240222180450698.png)

修改一下wordpress的配置文件

![image-20240222180658164](2-nginx实现fastcgi反向代理.assets/image-20240222180658164.png)

修改wp-confi.php连接数据的信息

![image-20240222180849302](2-nginx实现fastcgi反向代理.assets/image-20240222180849302.png)

这里的数据库也是独立的

测试下顺便

![image-20240222181155303](2-nginx实现fastcgi反向代理.assets/image-20240222181155303.png)

所以现在

nginx是192.168.126.130

php-fpm是192.168.126.135

mariadb是192.168.126.134

都分开咯



### 修改nginx的fastcgi的配置为远端php-fpm服务器

![image-20240222181724176](2-nginx实现fastcgi反向代理.assets/image-20240222181724176.png)

<span id='jump1'>502报错很可能就是php-fpm里没有放开nginx的访问</span>![image-20240222181818783](2-nginx实现fastcgi反向代理.assets/image-20240222181818783.png)



修改一下

![image-20240222182926997](2-nginx实现fastcgi反向代理.assets/image-20240222182926997.png)

好了就

![image-20240222182906428](2-nginx实现fastcgi反向代理.assets/image-20240222182906428.png)





然后怎么查看php的版本

之前的看法是错误的好像

![image-20240222183248626](2-nginx实现fastcgi反向代理.assets/image-20240222183248626.png)

首先php -v的命令是依赖于php-cli这个包，而这个cli看到的版本其实是php-cli的版本。



status里看不到版本唉

![image-20240222183414785](2-nginx实现fastcgi反向代理.assets/image-20240222183414785.png)

干脆写一个version_look的php页面吧

![image-20240222183544547](2-nginx实现fastcgi反向代理.assets/image-20240222183544547.png)



![image-20240222183627463](2-nginx实现fastcgi反向代理.assets/image-20240222183627463.png)



没问题

![image-20240222183659943](2-nginx实现fastcgi反向代理.assets/image-20240222183659943.png)



再转一个php-cli瞧瞧，看看到底php -v行不行

![image-20240222183750743](2-nginx实现fastcgi反向代理.assets/image-20240222183750743.png)

果然php -v 只是仅仅看php-cli 这个命令函工具的版本的，和php-fpm没有关系



然后这里有一个php-fpm的配置参考

![image-20240222184317167](2-nginx实现fastcgi反向代理.assets/image-20240222184317167.png)



