# 第4节 Docker-compose单机编排工具安装和实战案例



docker-compose也是docker官方制作的



关键字编排，就是根据业务来管理，就是docker ps 看到的就是所有容器的陈列，不清楚所属的项目，但是docker compose就是业务层次清晰。也可以之启动某个业务的容器。



# 安装docker-compose



方法1：

![image-20240613174930701](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240613174930701.png)



方法2：直接yum

ubuntu查看版本

![image-20240613175049643](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240613175049643.png)



yum安装的就是版本低

![image-20240613175358965](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240613175358965.png)



方法3：直接github下载

其实就是官网上走一波install步骤就行了啊

```shell
wget https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-linux-x86_64

mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

👇可见确实是可执行文件

![image-20240614095840866](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614095840866.png)

也没有任何依赖

![image-20240614100023828](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614100023828.png)

因为完全就是go写的，静态编译的。

![image-20240614100051627](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614100051627.png)



![image-20240614095455673](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614095455673.png)





通过help可见docker-compose也和docker有些像的

![image-20240614100224554](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614100224554.png)



![image-20240614101421404](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614101421404.png)

但是这种👆副本的效果没啥意义，因为docker-compose是单机游戏，你再多的副本都是在一个机器上的，HA并不给力。

SERVERS也是要写在yml文件里的，这里是CLI格式咯，就好像ansbile的cli和ansible的playbook，就好像iptables的cli和iptalbes的配置文件。



写在yml里，然后docker-compose up就完了。



yml是按业务分文件夹存放的，根Dockerfile一个逻辑。

![image-20240614102115824](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614102115824.png)

在这个yaml文件下去执行docker-compose up就会自动读取该yaml文件，就和Dockerfile的操作一样的。

也可叫docker-compose.yml，后缀yaml，yml都行。





# docker-compose语法

和版本还有关系，需要注意；如果发现up不起来，比如之前的，比如别人的yml，你up不起来，可能就是版本问题，格式变了。



其实有工具可以将docker run的cli自动转成docker-compose的yaml的。

https://www.composerize.com/

该网站也有cli的方式

https://linux.cn/article-14970-1.html



**composerize的cli**

![image-20240614154844764](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614154844764.png)

👆cli的方式，两个docker run 好像不会给你自动合并出来，可能要手动两次 自己合并，不如web上方便。



**composerize的web ui**

将之前的wordpress的两个run 改写成 一个 compose

![image-20240614153729391](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614153729391.png)



不过这个网站只能转换docker run的cli。网络、创建存储，就不支持了。所以上图的wordpress_net这个虚拟交换机是没有的。



试试

mariadb写在前面就先执行的意思，虽然这里无所谓

```
name: <your project name>
services:
    mariadb:
        ports:
            - 3306:3306
        environment:
            - MYSQL_ROOT_PASSWORD=123456
            - MYSQL_DATABASE=wordpress
            - MYSQL_USER=wordpress
            - MYSQL_PASSWORD=123456
        container_name: mysql
        restart: always
        volumes:
            - /data/mysql:/var/lib/mysql
        image: mariadb:11.3.2
    wordpress:
        ports:
            - 8080:80
        container_name: wordpress
        volumes:
            - /data/wordpress:/var/www/html
        restart: always
        image: wordpress:php8.2-apache
```



![image-20240614161121680](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614161121680.png)





![image-20240614161137237](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614161137237.png)



然后正常设置wordpress就行了



不过可惜上面的compose是前台的

![image-20240614161221376](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614161221376.png)

退出容器也就是推出了



不过start一下也就ok了

![image-20240614161306425](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614161306425.png)



想后台，-d就行

![image-20240614161631079](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614161631079.png)



不过没有定义网络，就会自动生成一个网络

![image-20240614162105906](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614162105906.png)

这个自动生成的虚拟网络命名就是用 yaml文件里的name_default来起名字的。

![image-20240614162136325](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614162136325.png)



![image-20240614162258288](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614162258288.png)

正好👆就是一个业务一个网络，妥妥的~~但是不同业务就是不同项目之间就默认隔离了。



docker ps ，docker-compose ps就需要在yaml下才能看到

![image-20240614163508491](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614163508491.png)



images在compose里看到的就是compose这条线索的images

![image-20240614164020453](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614164020453.png)



总之docker images看的是全的，docker-compose images看的就是它自己的

![image-20240614164303684](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614164303684.png)

而且还得在yaml文件的所在目录下，不同的yaml的路径看到的结果自然是不同的。



看本项目容器里的各自的进程

![image-20240614170153152](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614170153152.png)



docker-compose up  # 创建+启动

docker-compose down # 停止+删除



扩容和缩容，0就是删除

![image-20240614182014985](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614182014985.png)

不过扩容有前提的，

①yaml文件里的container_name不要写

![image-20240614183242994](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614183242994.png)



![image-20240614183223114](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614183223114.png)



②端口冲突也就起不来了

![image-20240614183145237](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240614183145237.png)



### 复杂一点的yml

```yaml
version: '3'
services:
  db:
    image: mariadb:11.3.2
    container_name: db
    restart: unless-stopped
    environment:
      - MYSQL_DATABASE=wordpress
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=123456
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - wordpress-network
  
  wordpress:
    depends_on:
      - db
    image: wordpress:php8.2-apache
    container_name: wordpress
    restart: unless-stopped
    ports:
      - "80:80"
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=123456
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    networks:
      - wordpress-network

volumes:
  wordpress:
  dbdata:

networks:
  wordpress-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.30.0.0/16
```



![image-20240617111154223](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240617111154223.png)



由于yaml里wordpress里已经配置了DB的，打开网页，设置站点标题和管理员就进去了。

![image-20240617115817879](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240617115817879.png)





### 再来一个yml

```yaml
name: spug-001
services:
  db:
    image: mariadb:11.3.2
    container_name: spug-db
    restart: always
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    volumes:
      - /data/spug/mysql:/var/lib/mysql
    environment:
      - MYSQL_DATABASE=spug
      - MYSQL_USER=spug
      - MYSQL_PASSWORD=spug.cc
      - MYSQL_ROOT_PASSWORD=spug.cc
  
  spug:
    image: openspug/spug-service
    #image: registry.aliyuncs.com/openspug/spug # 这是国内镜像
    container_name: spug
    privileged: true
    restart: always
    volumes:
      - /data/spug/service:/data/spug
      - /data/spug/repos:/data/repos
    ports:
      - "80:80"
    environment:
      - SPUG_DOCKER_VERSION=v3.2.1
      - MYSQL_DATABASE=spug
      - MYSQL_USER=spug
      - MYSQL_PASSWORD=spug.cc
      - MYSQL_HOST=db
      - MYSQL_PORT=3306
    depends_on:
      - db
      
```

docker-compose config -q   # 语法检查

然后就docker-compose up -d，结果报错了

![image-20240617134940054](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240617134940054.png)

这里面有CF的CDN啊

![image-20240617140124758](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240617140124758.png)

emmm，改路由吧，从海外线路出去，或者image换成国内的：registry.aliyuncs.com/openspug/spug

![image-20240617141341340](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240617141341340.png)

好了👆



初始化登入密码

![image-20240617172214421](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240617172214421.png)















# 工作案例

简单加载小脚本

```shell
#!/bin/sh

char=('\' '|' '/' '—')

while true;do
for i in "${char[@]}";do
    echo -ne "\r$i"
    sleep 0.2
done;done

# 根据需要将while改成loading的实际进度执行时间就行了，不过要记得最后清除掉符号。
```





{% raw %}
<video id="my-video" class="video-js" controls preload="auto" width="50%" data-setup='{"aspectRatio":"16:9"}'>
  <source src="4-Docker-compose单机编排工具安装和实战案例.assets/shellForLoading.mp4" type='video/mp4' >
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
</video>


{% endraw %}











优化：使用斜杠转圈圈来表示加载的效果

```shell
#!/bin/sh

char=('\' '|' '/' '—')
end=$((SECONDS+10))
echo "转$end秒"
while [ $SECONDS -lt $end ]
do
    for i in "${char[@]}"
    do
        echo -ne "\r$i"
        sleep 0.2
    done
done

echo -ne "\r"  # 清除最后的加载字符
```



<img src="4-Docker-compose单机编排工具安装和实战案例.assets/loading_斜杠.gif" alt="loading_斜杠.gif" style="zoom:50%;" />





优化2：使用贪吃蛇小点点来做加载的显示。

```shell
[root@realserver2 loading2]# cat loading_gpt.sh
#!/bin/sh

#char=('\' '|' '/' '—')
char=('⠋' '⠙' '⠹' '⠸' '⠼' '⠴' '⠦' '⠧' '⠇' '⠏')
end=$((SECONDS+10))
#echo "转$end秒"
while [ $SECONDS -lt $end ]
do
    for i in "${char[@]}"
    do
        echo -ne "\r$i"
        sleep 0.2
    done
done

echo -ne "\r"  # 清除最后的加载字符

printf "\033[32m✔\033[0m\n"

[root@realserver2 loading2]#
[root@realserver2 loading2]#
[root@realserver2 loading2]# bash loading_gpt.sh
✔
```



<img src="4-Docker-compose单机编排工具安装和实战案例.assets/loading_贪吃蛇.gif" alt="loading_贪吃蛇" style="zoom:50%;" />







优化3：单任务转圈圈，汇总任务也准圈圈



