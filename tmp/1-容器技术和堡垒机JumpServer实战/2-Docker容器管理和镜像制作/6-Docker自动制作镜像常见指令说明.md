# 第6节 Docker自动制作镜像常见指令说明





# Dockerfile的编写



## 官方文档：

https://docs.docker.com/reference/dockerfile/

一行一个指令

\# 表示是注释



## 首先指定基础镜像:FROM

```shell
cd /data/dockerfile/base/apline

touch Dockerfile

```



1、基于alpine来

明确下较新的版本

![image-20240429154039336](6-Docker自动制作镜像常见指令说明.assets/image-20240429154039336.png)

```shell
FROM alpine:3.19.1   # 从该镜像继承
```

alpine:3.19.1 又是从拿来的呢？这个就要去官网查看了

![image-20240429154719125](6-Docker自动制作镜像常见指令说明.assets/image-20240429154719125.png)



![image-20240429154736097](6-Docker自动制作镜像常见指令说明.assets/image-20240429154736097.png)



![image-20240429154752781](6-Docker自动制作镜像常见指令说明.assets/image-20240429154752781.png)



可见alpine的父镜像是scratch来着。而且ubuntu、centos、busybox都是from的她。然后jdk，tomcat这些APP又是基于OS的。这个scratch就是所有镜像的祖先，这就是一个空镜像![image-20240429161151287](6-Docker自动制作镜像常见指令说明.assets/image-20240429161151287.png)

。



https://hub.docker.com/_/scratch

![image-20240429155846707](6-Docker自动制作镜像常见指令说明.assets/image-20240429155846707.png)





再看回alpine的Dockerfile

![image-20240429162658367](6-Docker自动制作镜像常见指令说明.assets/image-20240429162658367.png)

参考人家的alpine制作，其实就是from一个scratch原始空镜像，再ADD一个迷你rootfs就行了这个rootfs其实就是OS的用户空间，而OS的内核空间就是用宿主机的。

​		你要自己手动创建一个类似alpine的镜像，就可以定制以下这个rootfs文件

![image-20240429162955179](6-Docker自动制作镜像常见指令说明.assets/image-20240429162955179.png)

也就是说确实还有进一步缩减的空间。



去看看busybox的Dockerfile，里面rootfs更小。

![image-20240429163517390](6-Docker自动制作镜像常见指令说明.assets/image-20240429163517390.png)

<img src="6-Docker自动制作镜像常见指令说明.assets/image-20240429163550816.png" alt="image-20240429163550816" style="zoom:46%;" />



其实，没必要折腾，直接官方的基础镜像拿来用就行了

### 既然没必要折腾那么就折腾下吧 # 非常规操作

目标：重新自己做一个最小OS，参考busybox咯

![image-20240429173617863](6-Docker自动制作镜像常见指令说明.assets/image-20240429173617863.png)



然后下载下来

![image-20240429173820449](6-Docker自动制作镜像常见指令说明.assets/image-20240429173820449.png)



然后看看是否要修改

![image-20240429174407133](6-Docker自动制作镜像常见指令说明.assets/image-20240429174407133.png)

还是要加压，因为本来我想直接用tar -r -f 追加工具进去，但是-r不支持压缩格式，单单一个tar包不带gz是可以的

![image-20240429174942327](6-Docker自动制作镜像常见指令说明.assets/image-20240429174942327.png)

进一步缩减cli文件，再加点比如curl命令

![image-20240429175100483](6-Docker自动制作镜像常见指令说明.assets/image-20240429175100483.png)

删除就怕这些bin文件之间有依赖关系就不好弄了，先不管，删除没见过的所有bin文件

搞不懂了👇，每删一个就会出现新的文件来占用空间

![image-20240429183620033](6-Docker自动制作镜像常见指令说明.assets/image-20240429183620033.png)

删掉差不多了，再弄个curl进去试试

![image-20240430092936364](6-Docker自动制作镜像常见指令说明.assets/image-20240430092936364.png)

![image-20240430092958639](6-Docker自动制作镜像常见指令说明.assets/image-20240430092958639.png)

打包

![image-20240430094351076](6-Docker自动制作镜像常见指令说明.assets/image-20240430094351076.png)

定制的busybox.tar.gz反而变大了，呵呵

预览以下包里内容当作检查

![image-20240430094500474](6-Docker自动制作镜像常见指令说明.assets/image-20240430094500474.png)

![image-20240430100020262](6-Docker自动制作镜像常见指令说明.assets/image-20240430100020262.png)

注释要单独一行，不能跟再CLI后面

![image-20240430100117086](6-Docker自动制作镜像常见指令说明.assets/image-20240430100117086.png)

这样就好了

![image-20240430100206879](6-Docker自动制作镜像常见指令说明.assets/image-20240430100206879.png)

![image-20240430100318621](6-Docker自动制作镜像常见指令说明.assets/image-20240430100318621.png)

这就是LABEL打进去的标签

然后run就报错啦

![image-20240430100449964](6-Docker自动制作镜像常见指令说明.assets/image-20240430100449964.png)

因为我把sh删掉了，哈哈哈



重做，只要sh等3个bin文件

![image-20240430101041335](6-Docker自动制作镜像常见指令说明.assets/image-20240430101041335.png)

打包后，build

![image-20240430101222329](6-Docker自动制作镜像常见指令说明.assets/image-20240430101222329.png)

run一下

![image-20240430101459800](6-Docker自动制作镜像常见指令说明.assets/image-20240430101459800.png)

可惜curl还是没有把依赖库弄好，搞笑的是ls也被我删掉了，搞得没法ls，然后find也没有，curl到底在哪也不知道。

好了，总之思路就是curl和依赖都要打包，然后过程如上。这就是定制os的思路。一般os还是不自己折腾的。



**再来一遍**

![image-20240430101916340](6-Docker自动制作镜像常见指令说明.assets/image-20240430101916340.png)



![image-20240430102205622](6-Docker自动制作镜像常见指令说明.assets/image-20240430102205622.png)



![image-20240430102304457](6-Docker自动制作镜像常见指令说明.assets/image-20240430102304457.png)

上面错了，cp -a是复制的软连接，

![image-20240430102446520](6-Docker自动制作镜像常见指令说明.assets/image-20240430102446520.png)



**再来一遍吧**

![image-20240430102800002](6-Docker自动制作镜像常见指令说明.assets/image-20240430102800002.png)

很好报别的依赖库了，所以你知道了OS定制还是挺麻烦的，不像基于别人的os，直接安装工具就好了，比如commit方式最简单，

不过上面的情况，我也可以用docker cp 的方式来处理，不过要处理的就多了去了👇

![image-20240430103005106](6-Docker自动制作镜像常见指令说明.assets/image-20240430103005106.png)

尝试把这些ldd依赖弄进去

![image-20240430103753353](6-Docker自动制作镜像常见指令说明.assets/image-20240430103753353.png)

但是继续测试发现，这些库弄进来也把sh的依赖搞出问题了

![image-20240430105527475](6-Docker自动制作镜像常见指令说明.assets/image-20240430105527475.png)

那么就这个文件不弄进去

对比了原来sh依赖的这个库和curl依赖的这个库发现不一样

![image-20240430105909577](6-Docker自动制作镜像常见指令说明.assets/image-20240430105909577.png)

应该是curl的依赖把这个libc.so.6覆盖了，导致sh启动不了

用官方的tar包里的源文件，将这个库还原回去，可行~下文**方法1**就是上面折腾的总结



### 

### 方法1：从tar包入手的镜像定制，ldd是关键

这种方法就是从scratch开了，呵呵，比较叼，一般不这么用，属于非正规方法。

```shell
mkdir -p /data/dockerfile/os/busybox-my-base
cd /data/dockerfile/os/busybox-my-base
vim Dockerfile
FROM scratch
ADD busybox.tar.gz /       # 这句其实就是解包，解开并解压缩到容器里的/根路径下
LABEL multi.label1="value1" \
	  multi.label2="value2" \
	  maintainer="oneyearice <oneyearice@gmail.com>" \
	  version=1.0 \
	  other="value3"
CMD ["sh"]
```

**以下是完整的一个os自定义过程**

需求，将busybox里的cli只保留sh find ls 并打入curl

<img src="6-Docker自动制作镜像常见指令说明.assets/image-20240430111415867.png" alt="image-20240430111415867" style="zoom:40%;" />

找到官方Dockerfile

![image-20240430111455112](6-Docker自动制作镜像常见指令说明.assets/image-20240430111455112.png)



就在Dockerfile文件的同级目录下找到这个tar包

<img src="6-Docker自动制作镜像常见指令说明.assets/image-20240430111554714.png" alt="image-20240430111554714" style="zoom:45%;" />

结果是个软连接，找到正主，

![image-20240430111638421](6-Docker自动制作镜像常见指令说明.assets/image-20240430111638421.png)

复制Raw进行下载

down下来

![image-20240430111724344](6-Docker自动制作镜像常见指令说明.assets/image-20240430111724344.png)

解压

![image-20240430111758101](6-Docker自动制作镜像常见指令说明.assets/image-20240430111758101.png)

并将原压缩包移入上层目录，待会会用里面的lib库做修正

进入bin删除sh ls find以外的所有文件

![image-20240430111945922](6-Docker自动制作镜像常见指令说明.assets/image-20240430111945922.png)



复制curl以及其依赖的库

![image-20240430112039319](6-Docker自动制作镜像常见指令说明.assets/image-20240430112039319.png)

![image-20240430112151552](6-Docker自动制作镜像常见指令说明.assets/image-20240430112151552.png)

![image-20240430112202798](6-Docker自动制作镜像常见指令说明.assets/image-20240430112202798.png)

此时lib里的东西有一个sh依赖的libc.so.6也被curl的依赖覆盖了，curl的依赖是从宿主复制过去的。所以要还原，否则容器里的sh用不了，连容器都启动不了

![image-20240430112433456](6-Docker自动制作镜像常见指令说明.assets/image-20240430112433456.png)

覆盖回去

![image-20240430112510064](6-Docker自动制作镜像常见指令说明.assets/image-20240430112510064.png)



打包

![image-20240430112641935](6-Docker自动制作镜像常见指令说明.assets/image-20240430112641935.png)

编写Dockerfile

build后再run就搞定了

![image-20240430112827154](6-Docker自动制作镜像常见指令说明.assets/image-20240430112827154.png)

![image-20240430113538875](6-Docker自动制作镜像常见指令说明.assets/image-20240430113538875.png)

贪狼，不计后果，勇往直前，好吧，就是好累。

以上就是完成了一个busybox的OS定制，打入curl和去掉N多cli的操作。不过一般不会这么做的，这里只是我的一个操作排错的记录而已，存思路验证过程。也算完成了



这里记录下images大小，回头用正规方式将curl封装进busybox，再看看那时的镜像大小

![image-20240430113909039](6-Docker自动制作镜像常见指令说明.assets/image-20240430113909039.png)

```shell
# 宿主机上编译安装curl

wget https://curl.se/download/curl-8.7.1.tar.xz && \
    tar -xJf curl-8.7.1.tar.xz && \
    cd curl-8.7.1 && \
    ./configure --prefix=/opt/curl-static --enable-static --disable-shared  --with-ssl && \
    make && \
    make install


./configure --prefix=/usr/local/curl --disable-shared --enable-static --without-libidn --without-ssl --without-librtmp --without-gnutls --without-nss --without-libssh2 --without-zlib --without-winidn --disable-rtsp --disable-ldap --disable-ldaps --disable-ipv6

# 以上还不是完全静态编译，复制到容器里还是缺库
```

我觉得还是应该找一个bin二进制的独立文件，或ldd出来放好，build的时候COPY进去。



### 方法2：别人的镜像增加一个软件，ldd也是关键

**上面的编译老是依赖共享库，算了，我直接ldd弄一个出来得了**

```shell

cd /data/dockerfile/os/busybox-curl/curl-static

# 复制出curl的所有依赖库
ldd `which curl` |awk -F '=>' '{print $2}' |awk -F '(' '{print $1}' |grep -E '/' |xargs -i bash -c 'cp -L {} ./'

mkdir lib
mv * lib

# 复制curl的bin文件
mkdir bin
cp -L `which curl` lib

```

得到👇

![image-20240430145854982](6-Docker自动制作镜像常见指令说明.assets/image-20240430145854982.png)

然后编写Dockerfile

```shell
# 使用 BusyBox 基础镜像
FROM busybox

# 复制编译好的 curl 目录到镜像中
COPY /curl-static /curl-static

# 将curl的bin文件和库合并(不覆盖的方式)进入镜像的bin和lib里
RUN cp -n /curl-static/bin/curl /bin/
RUn cp -n /curl-static/lib/* /lib/

# 设置默认命令
CMD ["sh"]
```

![image-20240430150946941](6-Docker自动制作镜像常见指令说明.assets/image-20240430150946941.png)

搞定

![image-20240430151014583](6-Docker自动制作镜像常见指令说明.assets/image-20240430151014583.png)



### 方法3：下载一个独立的bin文件COPY进去

这种更简单，就是要找到人家一个bin文件就是独立的curl这种，all-in-one吧

https://curl.se/download.html

找到linux系统下的binary

![image-20240430152404821](6-Docker自动制作镜像常见指令说明.assets/image-20240430152404821.png)



![image-20240430152818634](6-Docker自动制作镜像常见指令说明.assets/image-20240430152818634.png)

搞定

![image-20240430152921492](6-Docker自动制作镜像常见指令说明.assets/image-20240430152921492.png)





## LABEL指定镜像元数据

镜像的说明：标签、版本、作者等

```shell
#一行格式
LABEL multi.label1="valuel" multi.label2="value2" maintainer="oneyearice <oneyearice@gmail.com> other="value3"

#多行格式
LABEL multi.label1="value1" \
	  multi.label2="value2" \
	  maintainer="oneyearice <oneyearice@gmail.com>" \
	  other="value3"
```

作者的指令早期是

```shell
MAINTAINER <NAME>    # 现在改掉了
LABEL maintainer="oneyearice <oneyearice@gmail.com>"
```





## RUN命令

alpine里安装常用工具

![image-20240430160159876](6-Docker自动制作镜像常见指令说明.assets/image-20240430160159876.png)

这些默认没有，现在需要安装一点，这个比上面busybox里安装curl要简单多咯。

先替换为国内镜像源

![image-20240430160421794](6-Docker自动制作镜像常见指令说明.assets/image-20240430160421794.png)

点击问好得到镜像源替换信息

![image-20240430160504241](6-Docker自动制作镜像常见指令说明.assets/image-20240430160504241.png)

复制

```shell
sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
```

好在一般linux里没有vim但是有sed命令的，alpine容器镜像自然是sed命令的。



上面是清华源的使用，我下面用中科大的

https://mirrors.ustc.edu.cn/help/alpine.html

![image-20240430160750161](6-Docker自动制作镜像常见指令说明.assets/image-20240430160750161.png)



```shell
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
```

其实可以先run一个alpine进去手动安装看看，这样比较清楚具体过程是否ok

比如tcping其实说到底就是tcptraceroute写出来的脚本，你自己也可以写啊👇写好了：

![image-20240430163016058](6-Docker自动制作镜像常见指令说明.assets/image-20240430163016058.png)

```shell
FROM alpine:3.19.1
LABEL maintainer="oneyearice <oneyearice@gmail.com>"
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
# 跟新源后再安装
# 一个RUN产生一层镜像，6个就6层镜像，层越多，效率越低，减少层的方式
RUN apk update
RUN apk add curl
RUN apk add vim
RUN apk add ss
RUN apk add telnet
RUN apk add tcptraceroute

# 将上面的6个RUN合成一个RUN，就不会打6层镜像了
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories && apk update && apk add curl vim tcptraceroute iproute2

```

apk update 类似 ubuntu的apt update 但是完全不是yum update哦，别搞错啦，哈哈。

apk update是yum makecache更新镜像源

apk upgrade是yum update，更新软件

![image-20240430164009513](6-Docker自动制作镜像常见指令说明.assets/image-20240430164009513.png)

apk确实好比apt，而apt的老版本就是apt-get

<img src="6-Docker自动制作镜像常见指令说明.assets/image-20240430164049841.png" alt="image-20240430164049841" style="zoom:50%;" />



search一下telnet包在哪？

![image-20240430170313069](6-Docker自动制作镜像常见指令说明.assets/image-20240430170313069.png)

也可以用busybox里的telnet

![image-20240430170347651](6-Docker自动制作镜像常见指令说明.assets/image-20240430170347651.png)

busybox瑞士军刀啊，这个叼，很多命令都不用找，直接busybox，不过alpine就是利用的busybox

![image-20240430170849758](6-Docker自动制作镜像常见指令说明.assets/image-20240430170849758.png)

当然不是FROM busybox而是定制的tar包

<img src="6-Docker自动制作镜像常见指令说明.assets/image-20240430170915413.png" alt="image-20240430170915413" style="zoom:50%;" />





telnet 如果不用busybox就用search到的，不过要去掉后面的版本好之类的

![image-20240430171347329](6-Docker自动制作镜像常见指令说明.assets/image-20240430171347329.png)





```shell
FROM alpine:3.19.1
LABEL maintainer="oneyearice <oneyearice@gmail.com>"
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
# 跟新源后再安装
# 一个RUN产生一层镜像，6个就6层镜像，层越多，效率越低，减少层的方式
RUN apk update
RUN apk add curl
RUN apk add vim
RUN apk add ss
RUN apk add busybox
RUN apk add tcptraceroute

# 将上面的6个RUN合成一个RUN，就不会打6层镜像了
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories && apk update && apk --no-cache add curl vim busybox tcptraceroute iproute2

# 但是如果一层，将来镜像复用还能复用得到吗？如果将RUN的替换镜像源、各个工具的安装都单行来写，将来是不是其他镜像也能更好的复用呢，否则上面虽然整合成一条RUN，但是一个分层，别人如何复用呢？

# 常规使用上来考虑还是写成一个RUN比较好，不同基础镜像的工具也不太可能复用，而且复用里面到底怎么复用的也不是很清楚比如UinonFS(overlay2)。



```

![image-20240430172901141](6-Docker自动制作镜像常见指令说明.assets/image-20240430172901141.png)

所以你写在Dockerile里的RUN apk add xx xxx xxx xx 如果前面没有这个包，就直接报错了。

没办法只能先run一个进去安装成功了，再写道Dockerfile里了。



apk没有类似ubuntu里的apt clean这个命令，所以要用apk --no-cache来安装软件，这样安装后安装包就会自动删除。简称不缓存安装信息。减少镜像的空间。

![image-20240430173319162](6-Docker自动制作镜像常见指令说明.assets/image-20240430173319162.png)



可以参考别人的安装工具集

![image-20240430174826254](6-Docker自动制作镜像常见指令说明.assets/image-20240430174826254.png)



### 一个自用的alpine:Dockerfile

经过上面的不断校对，下面就是一个相对完善的Dockerfile啦

```shell
FROM alpine:3.19.1
LABEL maintainer="oneyearice <oneyearice@gmail.com>"

# 跟新源后再安装
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories && apk update && apk --no-cache add tzdata && ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && apk update && apk --no-cache add iotop gcc libgcc libc-dev libcurl libc-utils pcre-dev zlib-dev libnfs make pcre pcre2 zip unzip net-tools pstree wget libevent libevent-dev iproute2 vim curl tcptraceroute busybox-extras tcpdump

```

赶快去试试吧

![image-20240430181443839](6-Docker自动制作镜像常见指令说明.assets/image-20240430181443839.png)

build 灰常顺利

![image-20240430181515081](6-Docker自动制作镜像常见指令说明.assets/image-20240430181515081.png)

run起来看看

![image-20240430181703008](6-Docker自动制作镜像常见指令说明.assets/image-20240430181703008.png)

一些依赖python的可能由于py没装好就报错，其他还是ok的👆这个节后再折腾，无所谓了~

抓包也是ok的

![image-20240430182034237](6-Docker自动制作镜像常见指令说明.assets/image-20240430182034237.png)





