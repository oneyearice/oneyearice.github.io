# 第7节 Docker私有仓库的安全加密HTTPS实现



书接上文，再开一个135的IP机器作为

# nginx反代

1、当前拓扑

<img src="7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624110001185.png" alt="image-20240624110001185" style="zoom:50%;" />



132和133两个harbor的双向复制上一篇已经搞定，基本简单的实现了就。

下面134就是client访问135这个nginx。



2、docker跑一个nginx

```shell
services:
  nginx:
    image: nginx:1.27.0
    container_name: nginx_server
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx/logs:/var/log/nginx
    restart: always

# 说明：
# - 使用最新的 nginx 镜像。
# - 映射容器的 80 和 443 端口到主机。
# - 映射本地的 nginx 配置文件目录到容器的配置文件目录。
# - 映射本地的 html 文件目录到容器的 html 文件目录。
# - 映射本地的日志文件目录到容器的日志文件目录。
# - 设置容器重启策略为 always。

```



up一下

![image-20240624151224914](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624151224914.png)

![image-20240624151344059](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624151344059.png)



编写配置文件

![image-20240624152623804](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624152623804.png)



同样做成服务

```shell
vim /lib/systemd/system/nginxd.service

[unit]
Description=nginxd
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service

[Service]
Type=simple
Restart=on-failure
RestartSec=5
ExecStart=/usr/local/bin/docker-compose -f /apps/nginx/docker-compose.yml up
ExecStop=/usr/local/bin/docker-compose -f /apps/nginx/docker-compose.yml down

[Install]
WantedBy=multi-user.target
```

![image-20240624153052380](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624153052380.png)

此时就实现了反代了

![image-20240624153214191](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624153214191.png)

但是此时登入有问题的，因为轮询调度，ID密码输入，再点就调度到另一台Harbor上去了。

![image-20240624154239380](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624154239380.png)

验证就用同样的密码 分别登入两台Harbor就行了，不用测了没问题的，不放心可以测的。

就是nginx调度轮询，没有session导致的。估计连三次握手都握不起来，更遑论验证密码呢！



**所以改成session调度**

1、会话绑定，也就是nginx的session调度：①基于cookie的(harbor的cookie是多少也不知道)②基于源IP的(PAT环境就不适合了)

2、session复制，tomcat里有这个功能可以做，harbor没有做不了。

3、redis，理论上可以，但是harbor不知道怎么配置redis，让harbor去redis读session。



下面基于session

1、cookie好像是sid👇

![image-20240701142626478](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701142626478.png)

但是好像不能确定

所以还是用👇

2、源IP来做会话保持吧

ip_hash说是源IP，其实是源IP的/24，不是/32的

![image-20240701150212041](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701150212041.png)

所以得用hash👇



![image-20240701151755568](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701151755568.png)

应该是做成服务了，所以出现问题了，

①docker-compose管理的就用docker-compose来管理，不要用docker；

②做成服务的，就要用服务来start，restart，stop，不要用docker来弄。



systemctl restart nginxd

就好了

![image-20240701152606436](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701152606436.png)





上传镜像要需要注意https

![image-20240701161305746](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701161305746.png)



![image-20240701161313468](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701161313468.png)



因为现在是nginx代理的，所以insecure-registries里要写nginx的地址或域名

![image-20240701162703065](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701162703065.png)



![image-20240701162900839](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701162900839.png)



![image-20240701162938728](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701162938728.png)



再次登入OK

![image-20240701163001390](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701163001390.png)



由于没做dns，直接改tag为IP👇，但是报错了

![image-20240701163149986](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701163149986.png)



![image-20240701163326799](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701163326799.png)

这是因为当初登入的是IP地址，不是域名👆，👇

![image-20240701163942386](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701163942386.png)



这样，报错就和IP  PUSH的时候一致了

![image-20240701164105412](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701164105412.png)



报错写明：Entity Too Large 

![image-20240701164454833](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701164454833.png)



![image-20240701165854891](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701165854891.png)



![image-20240701170019613](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701170019613.png)



再次push ok

![image-20240701170102133](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240701170102133.png)











# HTTPS实现

HTTPS的SSL证书可以做在nginx上

<img src="7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240624110001185.png" alt="image-20240624110001185" style="zoom:50%;" />

也可以做在单机版的Harbor上

新版Harbor本身的SSL证书比老版的要复杂些



https://goharbor.io/docs/2.11.0/install-config/configure-https/

以下是对上面链接内容的简要说明👇

**1、CA的私钥**

**2、CA用私钥自签名**

**3、CA给他用户也就是server颁发证书**

​	**3.1 server的私钥**

​	**3.2 server的申请文件**

​	**3.3 CA给server颁发证书**

```shell
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=yourdomain.com
DNS.2=yourdomain
DNS.3=hostname
EOF
```

比老版多了一个文件👆，然哥们我一看，这不就是让**下面PC浏览器显示安全**的关键操作嘛，就是subectAltname就是我<u>之前折腾的那东西</u>啊👇

​		其实不是什么  比老板 多了了，而是时代变了(屎大便了)，浏览器要人为安全就得这么干，不仅仅是Harbor的新老版本之分，而是所有的都变了。

![image-20240704100806682](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704100806682.png)

然后还有一个nginx的ssl生成的便捷脚本也是一样

![image-20240704100936765](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704100936765.png)

两处地方都是用到了一个知识点👇

![image-20240704101340839](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704101340839.png)



然后这里Harbor的ssl解决方案里自然跑不掉这个新的政策subject Alternative name的意义。





<img src="7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704111812613.png" alt="image-20240704111812613" style="zoom:45%;" />





啊~~~~  继续学习



继续说明https://goharbor.io/docs/2.11.0/install-config/configure-https/这里的内容



👇这个其实好像是无用功，理论上是转换个，其实就是内容都一样，就是改了个后缀名称

<img src="7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704114503048.png" alt="image-20240704114503048" style="zoom:43%;" />

其实官方这么些，也是对的，①改后缀②用这种方式改后缀也是大而全的cli

①因为docker deamon会认为.crt是CA的，所以要改成.cert作为client证书，client就是CA的用户，而CA的用户也就是 PC--SERVER ，的server服务器证书了。上图client就指的是server





链接里同样有不完善的地方

![image-20240704121005541](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704121005541.png)

这里点进去也不是很明朗，就是yml文件的ssl配置。







# 以下是实验

```shell
# 创建证书相关数据的目录
mkdir -p /data/harbor/certs
cd /data/harbor/certs

# 生成CA的私钥
openssl genrsa -out ca.key 4096

# 生成CA的自签名证书
openssl req -x509 -new -nodes -sha512 -days 3650 \
    -subj "/C=CN/ST=Shanghai/L=Shanghai/O=example/OU=Personal/CN=ca.ming.org" \
    -key ca.key \
    -out ca.crt

# 生成harbor主机的私钥
openssl genrsa -out harbor.ming.org.key 4096

# 生成harbor主机的证书申请
openssl req -sha512 -new \
    -subj "/C=CN/ST=Shanghai/L=Shanghai/O=example/OU=Personal/CN=harbor.ming.org" \
    -key harbor.ming.org.key \
    -out harbor.ming.org.csr
    
# 创建x509 v3扩展文件(时代变了新增内容)
cat > v3.ext <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=ming.org
DNS.2=yourdomain
DNS.3=hostname
EOF

# 给harbor主机颁发证书
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in harbor.ming.org.csr \
    -out harbor.ming.org.crt

# 最终文件列表如下
ca.crt  ca.key  ca.srl  
harbor.ming.org.crt  
harbor.ming.org.csr  
harbor.ming.org.key
v3.ext


#新版的最终文件列表如下
ca.crt
ca.key
harbor.ming.org.crt
harbor.ming.org.csr
harbor.ming.org.key
v3.ext



# 查看证书内容cli
openssl x509 -in cacert.pem -noout -text 

```



![image-20240704141838118](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704141838118.png)





然后

```shell
mkdir -p /data/harbor/certs/
cp harbor.ming.org.crt harbor.ming.org.key /data/harbor/certs/

vim /apps/harbor/harbor.yml
......
hostname: harbor.ming.org # 注意：此行必须是网站的域名，而且harbor主机的/etc/hosts可以不解析此域名，不能是IP地址，否则登入时会报如下错苏
Error response from daemon: Get "https://harbor.ming.org/v2/": Get "https://192.168.126.132/service/token?account=admin&client_id=docker&offline_token=true&service=harbor-registry": x509: cannot validate certificate for 192.168.126.132 because it doesn't contain any IP SANs
# 而且 之前Harbor HA那会也对hostname有写法要求，好像写IP就得用IP去做HA。在👇这一篇里有提到 ，不过当时没有测试域名写个解析得效果可惜了：https://oneyearice.github.io/1-%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF%E5%92%8C%E5%A0%A1%E5%9E%92%E6%9C%BAJumpServer%E5%AE%9E%E6%88%98/4-Docker%E7%BD%91%E7%BB%9C%E7%AE%A1%E7%90%86%E5%92%8Cdocker-compose%E7%BC%96%E6%8E%92%E5%92%8C%E4%BB%93%E5%BA%93%E7%AE%A1%E7%90%86/6-Docker%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93Harbor%E9%AB%98%E5%8F%AF%E7%94%A8%E5%92%8C%E5%8F%8C%E5%90%91%E5%A4%8D%E5%88%B6.html


# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /data/harbor/certs/harbor.ming.org.crt
  private_key: /data/harbor/certs/harbor.ming.org.key
......

# 使上面的配置生效
cd /apps/harbor/
./prepare
docker-compose down -v
docker-compose up -d

```



## 以下使过程截图

### 生成证书文件

1、生成ca的key，用这个key生成ca的自签名  # key就是私钥

![image-20240704152411821](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704152411821.png)

看看时间到是否10年

![image-20240704154900928](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704154900928.png)



2、生成用户的key，用这个key 生成csr证书申请文件

![image-20240704152642352](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704152642352.png)

3、生成openssl v3 扩展文件

![image-20240704153256616](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704153256616.png)

4、利用ca.crt和ca.key和用户的csr文件生成用户的证书文件，并生成serial字符串

![image-20240704155059187](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704155059187.png)

这里上图少了一个换行符，导致CLI卡住不动

![image-20240704155305248](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704155305248.png)

查看生成的用户证书，确认时限10年

![image-20240704160053107](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704160053107.png)



### 将上面的证书文件配到Harbor上

就使用，用户key和crt文件。

![image-20240704161223034](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704161223034.png)



![image-20240704161625309](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704161625309.png)



然后重新启动harbor的dockers，也就是

docker-compose down -v
docker-compose up -d

不过之前使写了service的

![image-20240704161722168](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704161722168.png)

所以直接restart就行

![image-20240704161841682](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704161841682.png)

慢慢就重新创建了👆 一定要做好数据卷的持久保存。

1分钟大概

![image-20240704161919050](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704161919050.png)



访问就看到鸟

![image-20240704161946147](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704161946147.png)







![image-20240704162009245](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704162009245.png)



然后导入证书，试试域名访问，已经IP访问的效果    

然后要测试是否可以上传下载镜像，因为开了ssl

然后还要测试HA，因为之前是用的IP还互指HA的，当时用域名还不行咧。

好一个个来



## 1、导入证书

![image-20240704162826164](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704162826164.png)

双击导入

![image-20240704162900545](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704162900545.png)





<img src="7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704162934734.png" alt="image-20240704162934734" style="zoom:50%;" />





IP访问不行，换域名试试

![image-20240704163025472](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704163025472.png)





发现

![image-20240704163210873](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704163210873.png)

少了通配符，去试试，重新颁发crt，不麻烦，因为Harbor调用路径没发生变化，文件名称不会改变，# 但是要./prepare的因为crt文件名称没变，但是内容变了，容器不是链接而是内容的复制进去到容器里的。

![image-20240704163414166](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704163414166.png)





![image-20240704163454080](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704163454080.png)

重启harbor也就是docker-compose down 然后docker-compose up -d

发现PC浏览器还是老的证书

![image-20240704164055378](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704164055378.png)





于是怀疑要./prepare一下，



![image-20240704164136643](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704164136643.png)



果然

![image-20240704164204791](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704164204791.png)

然后导出来，再导进去

不行

![image-20240704165432307](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704165432307.png)





可惜了，先试试FQDN

继续改

![image-20240704164650987](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704164650987.png)

![image-20240704164701693](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704164701693.png)



删除老的证书

![image-20240704165703243](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704165703243.png)



导出导入

![image-20240704165741745](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704165741745.png)





![image-20240704165913230](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704165913230.png)



FQDN写错了

改

![image-20240704170230614](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704170230614.png)

重启harbor试试



![image-20240704170350144](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704170350144.png)



导出导入，谁让你这么玩的，你可真牛逼，受信任的根证书颁发机构，这个是要导入CA的证书，FUCK





![image-20240704170952120](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704170952120.png)





终于可以了

![image-20240704171128514](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704171128514.png)



回到过去，通配符的设置

![image-20240704171555830](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704171555830.png)



FUCK



![image-20240704171444128](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704171444128.png)



然后修正DNS

![image-20240704171703465](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704171703465.png)







### 所以结论来了



只要 证书主题背景的备用名称 里的通配符包含哪个就行了呢？

![image-20240704171916083](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704171916083.png)



👇这个无所谓，你和域名不一样无所谓，

![image-20240704171925508](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704171925508.png)



![image-20240704172252712](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704172252712.png)



![image-20240704172338849](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704172338849.png)



### 所以结论

就是证书主体背景的备用名称  包含你访问的域名就行了，其他无所谓

然后就是要PC导入ca.crt  CA的证书到受信任的根证书颁发机构



### √ 最后修正为规范的统一的配置

```
# 创建证书相关数据的目录
mkdir -p /data/harbor/certs
cd /data/harbor/certs

# 生成CA的私钥
openssl genrsa -out ca.key 4096

# 生成CA的自签名证书
openssl req -x509 -new -nodes -sha512 -days 3650 \
    -subj "/C=CN/ST=Shanghai/L=Shanghai/O=example/OU=Personal/CN=ca.ming.org" \
    -key ca.key \
    -out ca.crt

# 生成harbor主机的私钥
openssl genrsa -out harbor.ming.org.key 4096

# 生成harbor主机的证书申请
openssl req -sha512 -new \
    -subj "/C=CN/ST=Shanghai/L=Shanghai/O=example/OU=Personal/CN=harbor.ming.org" \
    -key harbor.ming.org.key \
    -out harbor.ming.org.csr
    
# 创建x509 v3扩展文件(时代变了新增内容)
cat > v3.ext <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=*.ming.org
DNS.2=ming.org
EOF

# 给harbor主机颁发证书
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in harbor.ming.org.csr \
    -out harbor.ming.org.crt

# 最终文件列表如下
ca.crt  ca.key  ca.srl  
harbor.ming.org.crt  
harbor.ming.org.csr  
harbor.ming.org.key
v3.ext


#新版的最终文件列表如下
ca.crt
ca.key
harbor.ming.org.crt
harbor.ming.org.csr
harbor.ming.org.key
v3.ext



# 查看证书内容cli
openssl x509 -in cacert.pem -noout -text 


# 这两行无所谓，就是调用文件路径一致就行，我的没变化直接用
#mkdir -p /data/harbor/certs/
#cp harbor.ming.org.crt harbor.ming.org.key /data/harbor/certs/

vim /apps/harbor/harbor.yml
......
hostname: harbor.ming.org # 注意：此行必须是网站的域名，而且harbor主机的/etc/hosts可以不解析此域名，不能是IP地址，否则登入时会报如下错  ★然而俺实测并不会，而且HA要求必须是IP。不过我是先用的域名然后再改回去的。就是说ssl证书生成的时候都是域名来着。有什么关系，harbor的东西和ssl证书生成过程并没有关系。所以这里必须写IP，就是为了HA能够OK。然后IP这边做SSL也没有问题。可能是我的harbor版本搞吧。
Error response from daemon: Get "https://harbor.wang.org/v2/": Get "https://10.0.0.102/service/token?account=admin&client_id=docker&offline_token=true&service=harbor-registry": x509: cannot validate certificate for 10.0.0.102 because it doesn't contain any IP SANs

#之前Harbor HA那会也对hostname有写法要求，好像写IP就得用IP去做HA。在👇这一篇里有提到 ，不过当时没有测试域名写个解析得效果可惜了：https://oneyearice.github.io/1-%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF%E5%92%8C%E5%A0%A1%E5%9E%92%E6%9C%BAJumpServer%E5%AE%9E%E6%88%98/4-Docker%E7%BD%91%E7%BB%9C%E7%AE%A1%E7%90%86%E5%92%8Cdocker-compose%E7%BC%96%E6%8E%92%E5%92%8C%E4%BB%93%E5%BA%93%E7%AE%A1%E7%90%86/6-Docker%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93Harbor%E9%AB%98%E5%8F%AF%E7%94%A8%E5%92%8C%E5%8F%8C%E5%90%91%E5%A4%8D%E5%88%B6.html


# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /data/harbor/certs/harbor.ming.org.crt
  private_key: /data/harbor/certs/harbor.ming.org.key
......

# 使上面的配置生效
cd /apps/harbor/
./prepare
docker-compose down -v
docker-compose up -d

```

![image-20240704173106013](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704173106013.png)







然后要测试是否可以上传下载镜像，因为开了ssl

然后还要测试HA，因为之前是用的IP还互指HA的，当时用域名还不行咧。



## 2、检查HA

由于132这台改成域名+ssl，果然

![image-20240704175118444](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704175118444.png)



![image-20240704175218448](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704175218448.png)



![image-20240704175801294](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704175801294.png)



同样要 安装ca.crt吧

```shell
yum install -y ca-certificates  # 一般就有

# 1√ 根证书到/etc/pki/ca-trust/source/anchors目录中远程就scp啦
cp ca.crt /etc/pki/ca-trust/source/anchors/

# 2√ 执行以下命令来更新 CA 证书存储，以便系统识别新的 CA 证书。
update-ca-trust

# 如果根证书是以*.pem结尾，需要转换成crt，然后再执行上述步骤。命令如下：
openssl x509 -in ca.pem -inform PEM -out ca.crt


============

# 你可以使用 openssl 命令来验证 CA 证书是否已经成功安装。运行以下命令并检查输出是否包含你刚才添加的 CA 证书。
# 这个也可以不做，直接用curl -L 域名 测试效果就行了，排查故障倒是分布检查需要的
openssl verify -CAfile /etc/pki/tls/certs/ca-bundle.crt your_cert.pem


#确保系统信任新的 CA 证书：在安装和更新 CA 证书后，系统应该信任由这个 CA 证书签署的所有证书。可以使用以下命令测试：
# 其实没必要，直接curl -L 域名就行了，就能测试了，这个👇是没有做上面步骤的时候测试的吧，不管了。
curl --cacert /etc/pki/tls/certs/ca-bundle.crt https://your-secured-site.com

```

![image-20240704180658320](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704180658320.png)



测试

![image-20240704180906267](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704180906267.png)



![image-20240704180918115](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704180918115.png)



![image-20240704180932000](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704180932000.png)

![image-20240704180944129](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704180944129.png)



算了改回去，将132的harbor.yml里hostname改成IP地址，重新./prepare 再restart harbor

不对啊，

1、如果要做HA，那么就是nginx顶前面，就nginx自己做HTTPS就行了，HA之间就用IP。没毛病。

2、如果不做HA，那么就是单台，那么也就无需这里测试什么连接性了，对吧。

3、其实可以做到132 133 134  两台harbor自带ssl，一个nginx也带ssl的，就是hostname用IP就行了，具体如下👇测试OK：

试试看吧，改回IP

![image-20240704181301270](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704181301270.png)

![image-20240704181400986](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704181400986.png)



​	

然后去看132，harhor.yml里的hostname改为IP后的HTTPS是否OK吧。。

没问题啊

![image-20240704181425270](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704181425270.png)

谁说一定要域名的





然后要测试是否可以上传下载镜像，因为开了ssl

## 3、测试上传下载image













# 案例

**openssh升级高版本**

以下未验证👇，思路应该使对的

用telnet上去编译安装openssl

![image-20240704150630675](7-Docker私有仓库的安全加密HTTPS实现.assets/image-20240704150630675.png)



**openssl升级**

直接换rocky9





