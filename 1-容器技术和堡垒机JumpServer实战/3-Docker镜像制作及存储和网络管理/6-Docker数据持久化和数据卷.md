# 第6节 Docker数据持久化和数据卷





之前VOLUME也可以实现数据持久化，就是容器删掉，数据还在宿主上放着，就是有点缺点，太乱了，存放没法自定义，或者要-v 命令行选项的方式来指定。



再来看下镜像和容器的分层数据存放

**image的**

![image-20240521111902324](6-Docker数据持久化和数据卷.assets/image-20240521111902324.png)

![image-20240521112036391](6-Docker数据持久化和数据卷.assets/image-20240521112036391.png)



**容器的**

首先容器run起来就是复制一份image

![image-20240521112142037](6-Docker数据持久化和数据卷.assets/image-20240521112142037.png)



![image-20240521112300490](6-Docker数据持久化和数据卷.assets/image-20240521112300490.png)

因为这个容器停掉了，所以数据释放掉了，running起来，才有merge才有复制image的那一份。

挂一个前台让他running

![image-20240521112729148](6-Docker数据持久化和数据卷.assets/image-20240521112729148.png)

再看分层数据

![image-20240521112808821](6-Docker数据持久化和数据卷.assets/image-20240521112808821.png)

这次就看到merged了👆



然后我将image和run起来的容器的分层目录 放一起看看

![image-20240521113630281](6-Docker数据持久化和数据卷.assets/image-20240521113630281.png)

上图是容器里的meger复制了一份image里的diff，其实inspect容器可见LowerDir分层里直接写了image的diff分层，

![image-20240521141920449](6-Docker数据持久化和数据卷.assets/image-20240521141920449.png)







容器生成的数据是放在diff下，比如日志、以及其他容器中生成的新的数据，这个diff就是可读可写层。

![image-20240521141632632](6-Docker数据持久化和数据卷.assets/image-20240521141632632.png)





看下日志，应该就在UpperDir也就是diff里。不过我上面是tail -f /dev/null，没有日志输出，所以重新run一个有std的

![image-20240521144031109](6-Docker数据持久化和数据卷.assets/image-20240521144031109.png)

然后去分层diff里看

但是显然是没有的，因为，没有落文件

![image-20240521144259910](6-Docker数据持久化和数据卷.assets/image-20240521144259910.png)

落一个文件

![image-20240521150807933](6-Docker数据持久化和数据卷.assets/image-20240521150807933.png)



现在diff里是空的

![image-20240521151133637](6-Docker数据持久化和数据卷.assets/image-20240521151133637.png)



然后生成一个新的文件

![image-20240521151152765](6-Docker数据持久化和数据卷.assets/image-20240521151152765.png)

然后UpperDir里就有了，于此同时merge作为合并层也就看得到了

![image-20240521151334814](6-Docker数据持久化和数据卷.assets/image-20240521151334814.png)



这是容器起来后生成一个新的文件，如果是容器build的时候生成文件会出现在哪个目录里呢?

这里做了日志文件和生成了test文件，不过是build的时候就已经生成test了，看看

![image-20240521151743546](6-Docker数据持久化和数据卷.assets/image-20240521151743546.png)

可见build阶段的文件是不会出现在UpperDir里的。而是出现在LowerDir里的diff里的

![image-20240521151857652](6-Docker数据持久化和数据卷.assets/image-20240521151857652.png)

### 分层目录理解：结论来了

1、diff就是容器和镜像相比 diff出来的，就是多出来的

2、而UpperDir的diff就是容器的可写层

3、而LowerDIr的diff就是基于某个FROM基础镜像创建新的镜像的时候的底层。~~是可写的，因为不管是test还是access.log都是可以写的。~~  其实是不可写的，就是build的时候打进去的，后面可写都是UpperDir里的diff才是，LowerDirr的diff只是 区别于 FROM基础镜像的diff，同样也是只读层。

![image-20240521152934039](6-Docker数据持久化和数据卷.assets/image-20240521152934039.png)

3、LowerDir里的每个路径之间用冒号分隔，代表一层层build的时候的封装。

4、merged就是联合文件整体的呈现。



# 下面开始学习持久化保存



![image-20240521160637520](6-Docker数据持久化和数据卷.assets/image-20240521160637520.png)



1、session这些就是有状态的东西，没必要持久化。

2、mysql数据需要持久化，需要持久化。



**持久化的的方法就是：**

1、容器删除，/var/lib/doker/overlay2/xxxxxxxxx..xxx这个对应的文件夹就自动没了

2、对于重要的数据，单独存放到宿主的其他路径就行了。



**具体的技术：**

1、数据卷

2、数据集容器就是数据卷的变种，不常用



### 使用方法



docker run 命令的以下格式可以实现数据卷

```shell
-v, --volume=[host-src:]container-dest[:<options>]

<options>
ro 从容器内对此数据卷是只读，不写此项默认为可读可写
rw 从容器内对此数据卷可读可写，此为默认值
host-src 宿主机目录如果不存在，会自动创建
```

不管什么方法，肯定都不在/var/lib/docker/overlay2/下了，不会随容器删除而删除



<span id=1>方法1：手动指定宿主目录</span>

```shell
#指定宿主机目录或文件格式：
—v <宿主机绝对路径的目录或文件>:<容器目录或文件>[:ro]  # 将宿主机目录挂载容器目录，两个目录都可以自动创建
```



<span id=2>方法2：卷ID</span>

```shell
# 匿名卷，只指定容器内路径，没有指定宿主机路径信息，宿主机自动生成/var/lib/docker/volumes/<卷ID>/_data目录，并挂载至容器指定路径
-v <容器内路径>

# 示例：
docker run --name nginx -v /etc/nginx nginx
```



<span id=3>方法3：卷名比卷ID识别度好，最通用的方式。</span>

```shell
#命名卷将固定的存放在/var/lib/docker/volumes/<卷名>/_data
-v <卷名>:<容器目录路径>
# 可以通过以下命令事先创建，如果没有事先创建卷名，docker run时也会自动创建。
# 里面外面都能自动创建文件夹，但是文件不行！

#示例：
docker volume create vol1  # 也可以事先不创建，就是这句可以不写的。
docker run -d -p 80:80 --name -v vol1:/usr/share/nginx/html nginx
```

docker rm 的-v选项可以删除容器的时候，同时删除相关联的匿名卷

```shell
-v,--volumes  Remove the volumes associated with the container
```





这是之前Dockerfile里的VOLUME也有类似的效果，好像就是方法2的效果一样。

![image-20240521165754387](6-Docker数据持久化和数据卷.assets/image-20240521165754387.png)





![image-20240521170455848](6-Docker数据持久化和数据卷.assets/image-20240521170455848.png)

![image-20240521170517938](6-Docker数据持久化和数据卷.assets/image-20240521170517938.png)

这种就是随机字符串的目录名，没有可读性好的名字，所以一般就称之为匿名卷。没有名字



### 实验

![image-20240521171752876](6-Docker数据持久化和数据卷.assets/image-20240521171752876.png)

把之前nginx的容器的Dockerfile来做挂载VOLUME实验

![image-20240521172201546](6-Docker数据持久化和数据卷.assets/image-20240521172201546.png)

Dockerfile.son本次实验不要太关注，就是测试ONBUILD 指令的。



[**方法2**](#2)

**先跑一个不用方法3的，因为Dockerfile理由指定VOLUME所以也就是方法2的效果**

先清一下none的镜像docker images -f "Dangling=true"

![image-20240521172858615](6-Docker数据持久化和数据卷.assets/image-20240521172858615.png)

![image-20240521172922048](6-Docker数据持久化和数据卷.assets/image-20240521172922048.png)

不run一下是没有VOLUME卷的是吧...应该是的

![image-20240521173340806](6-Docker数据持久化和数据卷.assets/image-20240521173340806.png)



run起来看看

![image-20240521181240376](6-Docker数据持久化和数据卷.assets/image-20240521181240376.png)

![image-20240521181623398](6-Docker数据持久化和数据卷.assets/image-20240521181623398.png)

**然后重点关注VOLUME**

![image-20240521182715677](6-Docker数据持久化和数据卷.assets/image-20240521182715677.png)

因为HOST变量没有值，所以文件也是空的👇

![image-20240522092534945](6-Docker数据持久化和数据卷.assets/image-20240522092534945.png)

然后在宿主机上往里面写点东西玩玩哈哈

![image-20240522092814312](6-Docker数据持久化和数据卷.assets/image-20240522092814312.png)

这不就有了吗，哈哈，其实应该去entrypoint.sh脚本里，给HOST赋值才对哦~~



顺便也瞟一眼健康检查的log

![image-20240521182916372](6-Docker数据持久化和数据卷.assets/image-20240521182916372.png)







[**方法1**](#1)

​		先看下方法2的匿名卷(也就是VOLUME名称是随机一长串字符串)的效果

我想通过runlike看到VOLUME的值，看来build阶段的VOLUME，run阶段是看不到的，runlike就是看docker run的选项的。

​		只能通过inspect去看咯

![image-20240522094828062](6-Docker数据持久化和数据卷.assets/image-20240522094828062.png)

可见这个VOLUME名称是不友好的，改之，

```shell
docker run -d --name web2-run -P -v /dirtest/html:/dirtest/html -e DO       C_ROOT=/dirtest/html/ -e HOST=ttttttt web1
```

👇图中-e DOC_ROOT 和-e HOST是传参，跳转页面转发路径，以及HOST这里赋值修复脚本里没给HOST赋值的情况。

![image-20240522095340613](6-Docker数据持久化和数据卷.assets/image-20240522095340613.png)

上图👆curl的时候HOST要注意，已经被你改成tttttt了。

![image-20240522095905010](6-Docker数据持久化和数据卷.assets/image-20240522095905010.png)

容器里面也是自动创建了这个目录

![image-20240522095434354](6-Docker数据持久化和数据卷.assets/image-20240522095434354.png)



好了然后这个VOLUME也是自定义的就比较好维护了

![image-20240522100405083](6-Docker数据持久化和数据卷.assets/image-20240522100405083.png)



[**方法3**](#3)才是最棒的

```shell
docker run -d --name web3-run -P -v vol1:/dirtest/html -e DOC_ROOT=/dirtest/html/ -e HOST=www.test.tk web1
```

不要自己定义宿主的挂载目录，就用统一的/var/lib/docker/volumes/起个名字/_data/就行。

![image-20240522101412278](6-Docker数据持久化和数据卷.assets/image-20240522101412278.png)



然后要注意就是Dockerfile里的VOLUME你也没删，docker run 的时候又-v vol1:/dirtest/html

![image-20240522102500419](6-Docker数据持久化和数据卷.assets/image-20240522102500419.png)

所以就是创建了两个挂载卷了👆。



#### 插播docker起不来排错思路

docker build ok，run的时候没有达到预期的排错

1、正向检查，Dockerfile是否ok

2、docker ps -a看看里面的信息，特别是CMD，不过看不全，没事用

```shell
docker inspect web3-run -f '{{.Args}}' 
```

看的全

3、docker run去掉-d，不挂后台，run着看看

4、docker logs看看现象

5、docker exec 进去手动其服务或则CMD的命令，起不来无非是CMD那条问题，起来还有别的问题才是进一步的配置设置之类的，用ENTRYPOINT和CMD的组合来解释，通常是初始化脚本没初始化好等等。或者RUN里的的apk add漏安装包啦之类的。

![image-20240522104202770](6-Docker数据持久化和数据卷.assets/image-20240522104202770.png)

下图说明如果挂载的宿主卷里有文件，然后Dockerfile里有COPY了同名这个文件进去，哪一个优先，宿主的优先会覆盖掉的，如果vol1里是空的，就不会覆盖同名的文件了👇

![image-20240522112001436](6-Docker数据持久化和数据卷.assets/image-20240522112001436.png)









# 工作案例分享



linux如何拨v2ray

## 1、搭建桥接client，server利旧

1、安装docker

https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/

2、pull镜像

https://hub.docker.com/r/v2fly/v2fly-core/tags

3、run容器

https://github.com/v2fly/docker



![image-20240522164614007](6-Docker数据持久化和数据卷.assets/image-20240522164614007.png)

修改配置文件

![image-20240522164836662](6-Docker数据持久化和数据卷.assets/image-20240522164836662.png)

不行，进去看，   #  **其实就是修改配置文件后没有重启容器**

![image-20240522164914462](6-Docker数据持久化和数据卷.assets/image-20240522164914462.png)

换下国内源

https://mirrors.tuna.tsinghua.edu.cn/help/alpine/

安装curl测试，在容器里

![image-20240522165008327](6-Docker数据持久化和数据卷.assets/image-20240522165008327.png)



然后正式实施的时候加上--restart always和安装runlike

```shell
docker run -d --name v2ray  --restart always -v vol1:/etc/v2ray -p 10086:1080 v2fly/v2fly-core run -c /etc/v2ray/config.json
 
yum -y install python3-pip
pip install runlike
```

![image-20240522171423795](6-Docker数据持久化和数据卷.assets/image-20240522171423795.png)



## 2、搭建nginx反代

将sockt反指到本地监听端口

别搭了了，需求搞错了，目前正确的需求是将所有流入进来的流量，转发到本地的socks5:10086上去。

而不是再配置一个nginx去暴露一个端口



## 2、构建本地流量转发

1、使用ssh试试

不行，ssh也是要指定端口的

2、iptables 可行

3、redsocks 可行

明天继续

https://tttang.com/archive/1878/



https://www.cnblogs.com/zhenyuyaodidiao/p/5494569.html

