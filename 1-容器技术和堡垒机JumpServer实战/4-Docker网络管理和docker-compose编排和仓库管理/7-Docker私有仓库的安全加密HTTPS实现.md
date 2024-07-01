# 第7节 Docker私有仓库的安全加密HTTPS实现



书接上文，再开一个135的IP机器作为nginx反代

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











