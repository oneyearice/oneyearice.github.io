# 第7节. 基于主机头的多虚拟主机和实现HTTPS加密



书接上回，虚拟主机的实现继续，上一篇实现了基于dstIp，基于dstPort

## 虚拟主机-基于主机头head里的host字段实现负载均衡

原理就是client发送http报文请求的时候，http头部里就携带了访问网站的FQDN



抓包是看不到https的head的，TCP以上都是加密的。

![image-20231010161941842](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010161941842.png)



<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010162139465.png" alt="image-20231010162139465" style="zoom:50%;" />

HTTP不加密可以抓到head内容

![image-20231010162417207](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010162417207.png)



**实验**

👇就是统一监听80，80本来在/etc/httpd/conf/httpd.conf这个主配置文件里就启用了，

然后documentroot及其all granted授权，documentroot和directory是一对，根和授权。

最后就是servername转发依据和LOG

![image-20231010163336943](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010163336943.png)

搞定👆



疑问：为什么cstie没写servername，结果转发到www.a.com了

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010163715609.png" alt="image-20231010163715609" style="zoom:50%;" />



估计就是 不写servername，然后用的就是第一个 从上往下找 <img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010164647324.png" alt="image-20231010164647324" style="zoom:50%;" />

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010164702128.png" alt="image-20231010164702128" style="zoom:50%;" /> 



![image-20231010165801318](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010165801318.png)

![image-20231010165841739](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010165841739.png)

通过官网查看就会自然知道现在servername还不够，还需要知道serveralais

![image-20231010171040782](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171040782.png)



![image-20231010171054597](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171054597.png)



<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171156199.png" alt="image-20231010171156199" style="zoom:33%;" />



<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171227653.png" alt="image-20231010171227653" style="zoom:33%;" />







看个现象

![image-20231010171802366](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171802366.png)![image-20231010171828433](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171828433.png)

问，为什么192.168.126.130的访问变成了www.a.com了

答，因为www.a.com在前面

![image-20231010172237006](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010172237006.png)

问，为什么不是原来的 非虚拟主机的页面了，答：从结果判断，就是虚拟主机的转发配置抢先了

![image-20231010172331424](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010172331424.png)

![image-20231010172307449](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010172307449.png)

其实就是虚拟主机的80和非虚拟主机的80冲突了，结果就是虚拟主机胜利

![image-20231010173211780](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010173211780.png)

80变成www.c.com排在第一个，所以和

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010173242299.png" alt="image-20231010173242299" style="zoom:50%;" /> 

冲突了，存在优先级了就。





**如果ip+port+servername一起用会如何**

![image-20231010175649095](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010175649095.png)

![image-20231010175657444](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010175657444.png)

**说明端口 port 优先> servername**

![image-20231010180239229](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010180239229.png)



![image-20231010180601048](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010180601048.png)





## 压缩技术，用的也非常多

server在页面发送到网络上之前，做一下压缩，节省带宽用。穿越网络，到达彼岸用户手上后，浏览器会自动解压出来后进行页面展示。   --- 属于典型的时间换空间的打法，时间--CPU压缩耗时，空间--磁盘和带宽的空间节省。    --- 什么时候用什么换什么，有时候也会用空间换时间，具体就看性价比，怎么划算怎么来。   --- 再一个压缩技术，消耗的是两头的CPU：server压，client解压，用户的资源消耗的提升也是一种拉动内需的手段。

![image-20231011092638301](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011092638301.png)

上图👆SetOutputFilter DEFLATE不写也行。   AddOutputFilterByTyep DEFALE XXX这是要压缩的文件类型，一般来讲视频压缩效果不太理想(不过具体也看如果视频本身的画面单一，可能压缩比也高的)。

使用压缩技术，server端依赖的是deflate_module，client依靠浏览器

![image-20231011092549354](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011092549354.png)



![image-20231011144338667](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011144338667.png)

压缩级别的指定👆



### 下面实验

首先找一个大文件，用来做页面

![image-20231011095557646](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011095557646.png)

![image-20231011100905737](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011100905737.png)

是的，关键是curl www.a.com 回车后直接默认就是访问的www.a.com/m.txt，奇怪了不应该是www.a.com/index.html嘛

上图403的原因：正式因为访问www.a.com默认就访问了m.txt，而m.txt没有给r权限，所以就403 Forbidden了。  至于为啥不是index.html就不清楚了。

**curl 看server有没有压缩，要用--compressed选项才能确定**

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011102441247.png" alt="image-20231011102441247" style="zoom:50%;" /> 

![image-20231011105617540](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011105617540.png)

好上图就说明没有压缩，原文件多大，curl 就下载了多大---conntent-length

![image-20231011104859460](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011104859460.png)



开启压缩

![image-20231011105127574](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011105127574.png)

重启服务

浏览器可以看到压缩了👇

![image-20231011103946353](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011103946353.png)



curl压缩要用选项

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011105517018.png" alt="image-20231011105517018" style="zoom:50%;" /> 



压缩的意义在于 money的节省，举个例子，一个图片没压缩钱是4MB，压缩后400KB，结果忘记开启压缩了，结果对外服务大量用户都是下载的4MB，流量哗哗的就出去了，都是钱啊。





# HTTPS



## 逻辑

![image-20231011114131993](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011114131993.png)



## 实现



### 1、颁发证书的两种方式

**自签名证书**，自己给自己颁发，就是私网里自己做一个CA证书颁发机器来实现内部的ssl证书颁发，不过这个互联网上是不认得。

**申请购买商业SSL证书**正儿八经去相关机构比如亚信、godaddy去申请SSL证书。

**申请免费SSL证书**这个互联网也认

凡是互联网认得，不管是买的还是免费的，关键点在于"受信任的根证书颁发机构"这一随OS安装就存在于电脑上的，手机同理。

![image-20231011115100992](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011115100992.png)



![image-20231011115149963](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011115149963.png)

这些证书就在了，也就是得到这些著名的CA的公钥了。



所以这些CA的颁发的SSL证书--也就是用他们的私钥加密来文件(这个文件往往就是网站的公钥)--也就是Sca(Psite)，PC-client就可以用已安装的公钥解密了，所以就可以解开了

![image-20231011115517928](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011115517928.png)





### 2、加载httpd的加密模块



<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011115923511.png" alt="image-20231011115923511" style="zoom:50%;" /> 



![image-20231011120057193](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011120057193.png)

上图可知：本身105个模块加载了93，12个没有加载，然后105也没有ssl模块，所以ssl模块需要另外安装。安装就用yum install mod_ssl

![image-20231011120237315](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011120237315.png)

![image-20231011120247561](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011120247561.png)

yum 安装后，就有这个ssl模块了，而且也加载了

![image-20231011144539907](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011144539907.png)

👆为什么yum后就加载了，其实yum安装本质上也是添加了配置文件，做了加载的配置👇

![image-20231011144846633](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011144846633.png)



配置文件相关

![image-20231011150228934](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011150228934.png)

![image-20231011150333065](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011150333065.png)

证书和私钥是yum安装的时候自动生成的，具体怎么来的，①要么安装出来的-通过 rpm -ql mod_ssl看看②要么是yum安装前或者前后的脚本跑出来的--通过rpm -q --scripts mod_ssL看看

![image-20231011150654239](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011150654239.png)

👆确实是yum 出来的两个证书和key文件。

![image-20231011150723025](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011150723025.png)

![image-20231011150749137](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011150749137.png)

但实际上都没有我的都没有，省流：因为新版本是mod_ssl模块安装后，httpd启动后就会自动生成localhost.crt和localhost.key。而不是什么安装后脚本了。

![image-20231011152735300](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011152735300.png)

视频里老师的有，是老版本的打法。

![image-20231011152628826](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011152628826.png)

👆上图确实是老版本的mod_ssl确实存在安装后脚本的，ssl证书和key确实是脚本产生的，但是新版不同。

再找一台机器yum mod_ssl可得出结论，新版本确实没有mod_ssl 安装后脚本，通过下面两个图可见证书和key文件都是启动httpd服务后自动生成的。哈哈新版本和老板的区别咯。

![image-20231011153200373](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011153200373.png)

![image-20231011153209077](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011153209077.png)



再找一台，就是不安装mod_ssl，启动服务，预判是没有这两个文件的，确实没有👇

![image-20231011153907252](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011153907252.png)

所以结论是：**mod_ssl模块+httpd服务启动一次**，就会自动生成localhost.crt和localhost.key文件了。

![image-20231011153945776](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011153945776.png)



### 不过倒是得到了一个证书生成的脚本：就叫做自签名证书脚本

![image-20231011152628826](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011152628826.png)

哈哈，天下脚本一大抄嘛，倒是个不错的思路，抄之





其实就是yum 一下 mod_ssl ，重启一下httpd服务，443就监听了，证书也有了，https也启用了

![image-20231011155543981](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011155543981.png)

此时80端口也开着，http协议不加密的服务也开着。所以http和https都可以访问，而且mod_ssl配置文件里面是这么配置的👇

![image-20231011155927569](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011155927569.png)

虚拟主机的配置方法，也不会和之前的80服务产生冲突



**冲突情况**

![image-20231011160204849](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011160204849.png)



![image-20231011160245309](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011160245309.png)



**不冲突的情况**

其实mod_ssl本质上也是 虚拟主机不过人家是443，不会和 之前的80冲突，冲突的无非是虚拟主机的80和非虚拟主机的80.

![image-20231011160346334](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011160346334.png)





curl要注意使用-k选项忽略证书的安全性

![image-20231011162654207](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011162654207.png)



### 将message大文件复制到/data/www下进行ab性能测试

![image-20231011163124837](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011163124837.png)

![image-20231011163245630](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011163245630.png)

![image-20231011163311006](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011163311006.png)

此时主站就变成了

![image-20231011163408741](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011163408741.png)







![image-20231011163911195](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011163911195.png)



### 导入证书

![image-20231011171215849](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011171215849.png)

证书怎么看，👆这样看不到任何信息，

```shell
openssl x509 -in localhost.crt -noout -text
```

![image-20231011171334915](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011171334915.png)



该

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011173905101.png" alt="image-20231011173905101" style="zoom:50%;" />

自签名证书👆，好像无法导入"受信任的根证书颁发机构"，颁发者和颁发给的都是 主机名node2，也不知道有没有问题，反正一般都是颁发给某某网站，比如👇

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231011174025361.png" alt="image-20231011174025361" style="zoom:50%;" />



上面是利用mod_ssl+httpd服务启动后自动生成的证书去实现的，下面使用手动内部签名的方式



### 利用私有CA，实现https



先回顾一下mysql里的ssl加密的所有cli

```
mkdir /etc/my.cnf.d/ssl  # 专门放证书信息，利用现成的my.cnf.d文件夹，ssl是创建的
cd /etc/my.cnf.d/ssl

①生成CA的私钥：
openssl genrsa 2048 > cakey.pem    # 以前是专门一个目录，现在简单放一起就行
可能最好加个密，或者改个cakey.pem的权限，安全些。

②利用私钥生成自签名证书
openssl req -new -x509 -key cakey.pem -out cacert.pem -days 3650

③生成master的私钥和证书申请文件
openssl req -newkey rsa:1024 -days 365 -nodes -keyout master.key > master.csr  # 利用一条命令生成私钥文件master.key，并利用该key生成证书申请文件。
注意！1024得改成2048，否则mysql起不来。可能是之前用得2048的CA私钥吧。

④有了证书申请文件，就可以签名了--也就是颁发证书
openssl x509 -req -in master.csr -CA cacert.pem -CAkey cakey.pem -set_serial 01 > master.crt

```

然后SSH里也涉及CA，下面是SSH里的操作，也是本次httpd的证书操作，一样的。

1、建立CA：genrsa生成ca私钥--cakey；利用cakey自签名证书--自己给自己颁发证书

```
(umask 077;openssl genrsa -out private/cakey.pem 4096)
openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650 <<EOF
CN
beijing
beijing
ming
devops
ca.ming.com
admin@ming.com
EOF
a.
touch /etc/pki/CA/index.txt    #  存放已经颁发的证书信息
echo 0F > /etc/pki/CA/serial   #  存放下一个颁发证书的序列号，0F改成01从第一个号开始分

```

2、搞一个证书申请文件

```
mkdir /etc/httpd/conf.d/ssl
cd /etc/httpd/conf.d/ssl
(umask 066;openssl genrsa -out httpd.key 1024)  #1024可能有问题msyql那会的经验告诉我要4096保持一致
openssl req -new -key httpd.key -out httpd.csr <<EOF
CN
beijing
beijing
ming
devops
ca.ming.com
admin@ming.com


EOF

scp /etc/httpd/conf.d/ssl/httpd.csr CAServer:/etc/pki/CA       # 把csr申请文件传到CA上，在CA上根据csr文件来颁发证书，也就是对其加密。

```

3、针对申请文件进行颁发证书-也就是签名-也就是用CA的私钥进行加密

```
openssl ca -in /etc/pki/CA/httpd.csr -out /etc/pki/CA/certs/httpd.crt -days 100

scp /etc/pki/CA/certs/httpd.crt root@httpdServer:/etc/httpd/conf.d/ssl    # 把证书复制到server上
scp /etc/pki/CA/cacert.pem root@httpdServer:/etc/httpd/conf.d/ssl      # 把ca自己的证书也复制倒server上，此举相当于windows预加载了受信任的根证书文件。
```



**实验开始👇**



在CA上：

![image-20231012102217046](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012102217046.png)

![image-20231012102459305](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012102459305.png)

实测👆上图的umask并不会影响到openssl genrsa生成文件的权限，可惜了，(umask 077;mikdir/touch)倒是ok

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012180047793.png" alt="image-20231012180047793" style="zoom:33%;" />

![image-20231012134011815](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012134011815.png)

openssl x509 -in cacert.pem -noout -text    # 查看证书内容cli

![image-20231012134046402](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012134046402.png)

👆对的，上述信息都在的。

![image-20231012134818690](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012134818690.png)



在服务器上：

![image-20231012170542789](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012170542789.png)

![image-20231012172355317](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012172355317.png) 



把httpd.csr证书申请文件传到CA上

![image-20231012173629509](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012173629509.png)

在CA上进行签名

![image-20231012175013261](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012175013261.png)

将证书复制到server上

![image-20231012175213899](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012175213899.png)

将CA证书复制到server上

![image-20231012181803972](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012181803972.png)

然后去到server上

![image-20231012181834907](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012181834907.png)

👆httpd.key就是私钥，httpd.crt就是证书-也就Sca[Pserver]-也就是经CA私钥加密后的server公钥文件。



最后就是在server上的ssl配置文件里使用上面得到的cacert.pem根证书，httpd.crt证书，httpd.key私钥

![image-20231012182316129](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012182316129.png)

![image-20231012182207779](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012182207779.png)

![image-20231012182548907](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012182548907.png)

<span id =10 >CA的证书先不配置，看下是否就会报不安全的证书信息</span>

![image-20231012182745519](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012182745519.png)

![image-20231012182804488](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012182804488.png)

名字改一下好了，结果还是报错，是不是1024和4096不匹配啊

![image-20231012182946806](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012182946806.png)



通过err_log看看，重启服务的动作会生成如下错误日志

![image-20231012183406176](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012183406176.png)

回过头去修改httpd.key的1024长度为2048试试



![image-20231012184106802](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012184106802.png)



![image-20231012184536760](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012184536760.png)

就是index.txt里有记录了，就是CN、beijing、admin@ming.com这些申请信息已经有了，重复了冲突了

![image-20231012184641483](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012184641483.png)

![image-20231012184734510](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012184734510.png)

![image-20231012184816362](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012184816362.png)

结果，就牛逼的OK了

![image-20231012184845081](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012184845081.png)

屡试不爽啊，myslq里和httpd里，然后httpd其实就是ssh的ssl一样的用法，所以CA ssl证书里的2048这种长度一定要一致啊，具体来讲就是CA的key2048，那么server的key也要2048，等等，一致肯定没问题，但是上述实验CA的key是4096哦，server的key最后2048也行的，就是不能1024，可能是1024的长度被弃用了。



测试-1-目前没有加载CA的证书呢，只是server有了自己的证书

![image-20231012185328444](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012185328444.png)

观察证书的信息，就是我们上面申请的信息

注意实验的时候不要点击 继续前往，否则就没有对比效果了

![image-20231012185412637](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012185456550.png)



注意细节，没有导入CA证书的时候，证书 详细信息里 的层次结构是看不到的

![image-20231012190425812](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012190425812.png)

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012190610290.png" alt="image-20231012190610290" style="zoom:50%;" />



待会导入证书后，就会看到多了颁发给：www.a.com 的信息 ，而不再是上图👆的颁发者和颁发给都是ca.ming.com了





![image-20231012190941204](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012190941204.png)

上图👆这里层次结构不全，[是因为我们没有加载CA证书](#10)，在httpd的/etc/httpd/conf.d/ssl.conf里



加载一下

![image-20231012191305715](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012191305715.png)

重启服务后，httpd网站的ssl证书里的层次结构就全了-就是谁下面的谁的证书。

![image-20231012191458039](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012191458039.png)





![image-20231012191527069](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012191527069.png)

这页没啥变化👆，主要是第二页 详细信息

![image-20231012191631855](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012191631855.png)

但是为什么层级里都是ca.ming.com啊，难道不是www.a.com，稍等我知道了

![image-20231012191819569](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012191819569.png)

![image-20231012191846942](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012191846942.png)



再改一下，将httpd.csr重置，



![image-20231012192001541](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012192001541.png)

![image-20231012192203297](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012192203297.png)

注意上图index.txt里由于本次颁发是www.a.com和之前的ca.ming.com不冲突了，所以不会报错

看看是不是www.a.com了👇

![image-20231012192649972](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012192649972.png)

issuer，"颁发者"

subject，这里指  "颁发给"

重启服务

![image-20231012192928512](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012192928512.png)



这下就顺眼鸟

![image-20231012193020985](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193020985.png)



![image-20231012193105481](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193105481.png)





导出CA证书，放到client PC上去，导入倒PC的 受信任的根证书机构 里去

![image-20231012193356566](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193356566.png)

如何导入呢，修改cacert.pem为caert.pem.crt，就是在linux里pem后缀就认了，但是windows里还得再加个crt后缀

![image-20231012193547231](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193547231.png)

![image-20231012193650436](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193650436.png)

注意上图左1的  "证书状态"是 不受信的。

![image-20231012193722577](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193722577.png)



![image-20231012193805662](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193805662.png)





![image-20231012193818477](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193818477.png)

![image-20231012193908665](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193908665.png)

![image-20231012193935222](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012193935222.png)



此时，再看证书状态就没问题了👇

![image-20231012194227069](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012194227069.png)



但是浏览器不管的，还是说 不安全，我怀疑浏览器就是不认私自颁发的证书了。

https://support.huaweicloud.com/ccm_faq/ccm_01_0098.html



![image-20231012195502699](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012195502699.png)



![image-20231012195434346](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012195434346.png)

下了个火狐，人家明确说了自签名，不认，啊哈哈，牛逼。



那么问题还在吖，自签名证书就没办法让浏览器认了吗？

https://blog.csdn.net/a735131232/article/details/80526859

试试这招👆不具体，再看看别的教程

https://blog.51cto.com/u_296714/5754713

这篇👆貌似可以解决，但是我就想用openssl生成的ca自签名证书呢



![image-20231012201628187](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012201628187.png)

导出后加个.crt后缀，在windows里打开看看

![image-20231012201839492](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012201839492.png)

发现没有 "使用者可选名称"啊

![image-20231012201833924](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012201833924.png)

搜索 openssl如何生成这个选项，准备重新颁发

https://blog.51cto.com/u_11508007/5674376

备份 cp -a  /etc/pki/tls/openssl.cnf  /etc/pki/tls/openssl.cnf.bak1

修改vim /etc/pki/tls/openssl.cnf

取消req下被注释的第2行

删除req_distinguished_name下的0.xxx 的标签，把0.xxx的0. 去掉

![image-20231012203337317](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012203337317.png)

在[ v3_req ]下新增最后一行内容 subjectAltName = @alt_names

新增 alt_names,注意括号前后的空格，DNS.x 的数量可以自己加

![image-20231012203634500](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012203634500.png)

上图写错了，DNS.1改为*.a.com和DNS.2=www.a.com

![image-20231012205112184](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012205112184.png)



重新走一遍所有证书的操作



```
CA上👇
cd /etc/pki/CA
mkdir certs
mkdir crl
mkdir newcerts
mkdir private
mkdir ssl

(umask 077;openssl genrsa -out private/cakey.pem 4096)
openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650



touch /etc/pki/CA/index.txt    #  存放已经颁发的证书信息
echo 0F > /etc/pki/CA/serial   #  存放下一个颁发证书的序列号，0F改成01从第一个号开始分


server上👇
mkdir /etc/httpd/conf.d/ssl
cd /etc/httpd/conf.d/ssl
(umask 066;openssl genrsa -out httpd.key 2048)  #1024可能有问题msyql那会的经验告诉我要4096保持一致

openssl req -new -key httpd.key -out httpd.csr 
scp /etc/httpd/conf.d/ssl/httpd.csr CAServer:/etc/pki/CA       # 把csr申请文件传到CA上，在CA上根据csr文件来颁发证书，也就是对其加密。


CA上👇
openssl ca -in /etc/pki/CA/httpd.csr -out /etc/pki/CA/certs/httpd.crt -days 100

scp /etc/pki/CA/certs/httpd.crt root@httpdServer:/etc/httpd/conf.d/ssl    # 把证书复制到server上
scp /etc/pki/CA/cacert.pem root@httpdServer:/etc/httpd/conf.d/ssl      # 把ca自己的证书也复制倒server上，此举相当于windows预加载了受信任的根证书文件。


```

![image-20231012205219202](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012205219202.png)

这里调了默认的[]里的值也是在配置文件里

![image-20231012205300589](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012205300589.png)



后略了，但是没有 使用者可选名称 ！！ 















浅排一下打开www.a.com慢的问题

![image-20231012200245911](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012200245911.png)

真TMD高，回头研究下为什么是每个掉用户，load这么高，

直接降低就是修改配置文件，把几个初始化多线程关掉

![image-20231012200335362](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012200335362.png)

重启服务后，逐渐就降下来了，估计是参数设置不合理，没有匹配 VM虚拟机的资源。

![image-20231012200355554](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231012200355554.png)

然后PC 浏览器打开就秒开了，呵呵呵，之前都是很慢



