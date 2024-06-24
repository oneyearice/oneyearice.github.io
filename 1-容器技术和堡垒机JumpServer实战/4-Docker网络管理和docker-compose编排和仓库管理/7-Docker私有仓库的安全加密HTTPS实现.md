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



所以改成session调度







