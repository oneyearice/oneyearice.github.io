# 第7节 Docker自动制作镜像常见指令说明



# ENV 设置环境变量

注意是shell的环境变量，而不是普通变量

之前 run一个mysql容器设置变量的方法，就是基于EVN实现的👇

![image-20240506092101215](7-Docker自动制作镜像常见指令说明.assets/image-20240506092101215.png)

![image-20240506094014243](7-Docker自动制作镜像常见指令说明.assets/image-20240506094014243.png)

当时那个mysql容器里的变量就是通过-e来传递的。



```shell
#变量赋值格式1
ENV <key> <value>  # 此格式只能对一个key赋值，<key>之后的所有内容钧会被视作其<value>

#变量赋值格式2，此格式可以支持多个key赋值，定义多个变量建议使用该格式，减少镜像层
ENV <key1>=<valule1> <key2>=<value2>\
	<key3>=<value3> ...
	
#如果<value>中包含空格，可以反斜线\进行转义，也可以通过对<value>加引号进行标识；另外，反斜线也可以用于续行。  续行，可还行~ 老师词汇不错~

#只使用一次变量
RUN <key>=<value> <command>

#引用变量
RUN $key

#变量支持高级赋值格式
${key:-word}
${key:+word}

```



去看看官方mysql的ENV配置

![image-20240506100134599](7-Docker自动制作镜像常见指令说明.assets/image-20240506100134599.png)

![image-20240506100200240](7-Docker自动制作镜像常见指令说明.assets/image-20240506100200240.png)

发现就2行，也没看到密码的变量，发现有个脚本

![image-20240506100222778](7-Docker自动制作镜像常见指令说明.assets/image-20240506100222778.png)

找啊找啊，找到了👇

![image-20240506100718564](7-Docker自动制作镜像常见指令说明.assets/image-20240506100718564.png)



就是说人家还是通过脚本整体去搞定变量这件事了。

下面还是用ENV的方式来定义吧

![image-20240506102200330](7-Docker自动制作镜像常见指令说明.assets/image-20240506102200330.png)

run起来看下变量是否进去了

![image-20240506102315119](7-Docker自动制作镜像常见指令说明.assets/image-20240506102315119.png)

ok👆



![image-20240506102453928](7-Docker自动制作镜像常见指令说明.assets/image-20240506102453928.png)

注意观察CACHED

![image-20240506102620575](7-Docker自动制作镜像常见指令说明.assets/image-20240506102620575.png)

再次build可见

![image-20240506102640551](7-Docker自动制作镜像常见指令说明.assets/image-20240506102640551.png)

现在的RUN sed 那一行也是之前做过的，所以这里就是CACHED，利用的缓存了，这就是分层了。



**这是build的时候定义的环境变量，如何在后面再去修改呢，-e咯**

![image-20240506103043884](7-Docker自动制作镜像常见指令说明.assets/image-20240506103043884.png)

多个-e就是改多个

![image-20240506103157590](7-Docker自动制作镜像常见指令说明.assets/image-20240506103157590.png)



## ENV的跨阶段有效性

再看个阶段状态，从Dockerfile build成image ENV是有效的，然后ENV还在image RUN 成容器的时候也有效，也就是说ENV环境变量设置跨了两个阶段，其实肯定的，第一个阶段是定义，第二个阶段是应用嘛，但是作为类比，其他的指令就不是了比如RUN 比如CMD ARG等👇

![image-20240506103353530](7-Docker自动制作镜像常见指令说明.assets/image-20240506103353530.png)

**举例RUN指令只在build的时候才有效**

![image-20240506104159523](7-Docker自动制作镜像常见指令说明.assets/image-20240506104159523.png)



![image-20240506104219254](7-Docker自动制作镜像常见指令说明.assets/image-20240506104219254.png)

RUN只在build阶段有效，RUN起来是不会生效的，证明如下

![image-20240506105053144](7-Docker自动制作镜像常见指令说明.assets/image-20240506105053144.png)



这小段👆内容呢，其实作为我来讲，压根不会有这个困惑，①build build， build完了Dockerfile里的东西就算用过了，不会再持续生效，这才是常理 ②build的东西持续到run生效，其实ENV只是设置了环境变量，从而再run起来变量当然生效，你不能说run起来ENV这个动作还在，对吧，压根没有的困惑，被解释成了一片所谓的知识，这就是这段文字的我的个人理解。



但是这个图确实有意义，它明确指出了哪些命令在哪个阶段起作用的，除了ENV我认为不应该也出现在run阶段，因为build完了的变量是持续的，而不是ENV这个动作是二次生效的。以下有证明改图的右边的EVN是不存在的，常识性的东西。

![image-20240506103353530](7-Docker自动制作镜像常见指令说明.assets/image-20240506103353530.png)



### 变量高级用法

![image-20240506113509111](7-Docker自动制作镜像常见指令说明.assets/image-20240506113509111.png)

Dockerfile里一样这么用

![image-20240506115809884](7-Docker自动制作镜像常见指令说明.assets/image-20240506115809884.png)

讨厌的咧~

![image-20240506120150099](7-Docker自动制作镜像常见指令说明.assets/image-20240506120150099.png)



### 变量的作用范围

**然后注意两个RUN指令之间没有继承关系，我说的是变量不会继承**

![image-20240506134708031](7-Docker自动制作镜像常见指令说明.assets/image-20240506134708031.png)

👆图中红箭头并不会集成变量，所以结果test里还是ming   # 因为第二个RUN应该是独立的，name依旧为空，所以author=ming👇

![image-20240506134818900](7-Docker自动制作镜像常见指令说明.assets/image-20240506134818900.png)

改成一条RUN就行啦

![image-20240506135010515](7-Docker自动制作镜像常见指令说明.assets/image-20240506135010515.png)

由于是一个RUN指令里的name，所以能够继承，所以author高级赋值就依据name里的值为123了👇

![image-20240506135112611](7-Docker自动制作镜像常见指令说明.assets/image-20240506135112611.png)

两个RUN之间不能继承，但是RUN可以继承上面的ENV啊，ENV是环境变量了，肯定OK的，RUN好比函数里的变量属于local本地局部变量。

![image-20240506135553262](7-Docker自动制作镜像常见指令说明.assets/image-20240506135553262.png)

![image-20240506135643373](7-Docker自动制作镜像常见指令说明.assets/image-20240506135643373.png)

上图👆奇怪没有看到ENV赋值的语句在build中呈现。

总之ENV和RUN的研究就差不多了，当然-e去覆盖只是覆盖的一摸一样的那个变量，不能说你build的时候z=x+y，然后-e 赋值x，z就变了，这是不可能的，又不是python的列表字典赋值--指针的意思。所以要注意别被人带偏了。上图的-e都是无所谓的参数只是放在这里看看。



### 然后变量高级赋值的+-补充理解

![image-20240506140707629](7-Docker自动制作镜像常见指令说明.assets/image-20240506140707629.png)



这里有一个高级赋值的案例，是用在监听socket上的写法

![image-20240506143041542](7-Docker自动制作镜像常见指令说明.assets/image-20240506143041542.png)





# COPY复制指令

把脚本复制到镜像里

copy的时候可以顺带改属性

![image-20240506160952866](7-Docker自动制作镜像常见指令说明.assets/image-20240506160952866.png)

给执行权限

![image-20240506161028525](7-Docker自动制作镜像常见指令说明.assets/image-20240506161028525.png)

看下效果

![image-20240506161157244](7-Docker自动制作镜像常见指令说明.assets/image-20240506161157244.png)

这就在容器里的根目录下生成了test.log文件里面的内容如上👆

COPY的是源文件，必须是  ①相对路径 + 且 ②是Dockerfile的同级或下级目录



![image-20240506163716377](7-Docker自动制作镜像常见指令说明.assets/image-20240506163716377.png)



# ADD复制和解包指令

比COPY多一个直接解包的功能



**1、先准备一个打包文件**

![image-20240506165144296](7-Docker自动制作镜像常见指令说明.assets/image-20240506165144296.png)

![image-20240506165205368](7-Docker自动制作镜像常见指令说明.assets/image-20240506165205368.png)

![image-20240506165637376](7-Docker自动制作镜像常见指令说明.assets/image-20240506165637376.png)

所以**mv xxx ~-/**  就是将当前文件xxx移动到上一个工作目录中



**2、然后用ADD指令复制并解压到镜像里**

![image-20240506170621713](7-Docker自动制作镜像常见指令说明.assets/image-20240506170621713.png)



![image-20240506170700584](7-Docker自动制作镜像常见指令说明.assets/image-20240506170700584.png)



看看alpine官方的Dockerfile也是用的ADD

![image-20240506171203361](7-Docker自动制作镜像常见指令说明.assets/image-20240506171203361.png)





# CMD容器启动命令

CMD就是指定一个命令作为容器启动的默认命令。

一般服务性的容器就是CMD是挂前台，这样容器运行后就不会退出

这个前面梳理过了👇

![image-20240506174334716](7-Docker自动制作镜像常见指令说明.assets/image-20240506174334716.png)



![image-20240506174247140](7-Docker自动制作镜像常见指令说明.assets/image-20240506174247140.png)



启动服务的时候，将服务最终的前台服务进程写在CMD后面

### CMD和RUN的不同

CMD和RUN都是执行命令，但是阶段不同，RUN是build的时候用过就用过了，CMD是容器run起来的时候才会生效的。

<img src="7-Docker自动制作镜像常见指令说明.assets/image-20240506173713164.png" alt="image-20240506173713164" style="zoom:70%;" />



也就是说CMDbuild的时候是不生效的，而docker run的时候才起作用，这个确实要注意的。





# 制作基于alpine的自定义nginx镜像



```shell
# 准备相关文件
# 先定义文件目录结构，分门别类base、web分开来
mkdir -p /data/dockerfile/{base/{ubuntu,centos,alpine,busybox},web/{jdk,nginx,tomcat}}

mkdir /data/dockerfile/web/nginx/1.26.0-alpine/
cd /data/dockerfile/web/nginx/1.26.0-alpine/


wget https://nginx.org/download/nginx-1.26.0.tar.gz

echo Test Page based nginx-alpine > index.html
cp ../1.26.0-centos7/nginx.conf .
cat nginx.conf
user nginx
worker_processes 1;
daemon off;
...
location / {
		root /data/nginx/html;
....


# 编写Dockerfile
vim Dockerfile
cat Dockerfile
FROM alpine-my-base:3.19.1
LABEL maintainer="oneyearice <oneyearice@gmail.com>"
ENV NGINX_VERSION=1.26.0
ADD nginx-$NGINX_VERSION.tar.gz /usr/local/src/

RUN cd /usr/local/src/nginx-$NGINX_VERSION && ./configure --prefix=/apps/nginx && make && make install && ln -s /apps/nginx/sbin/nginx /usr/bin && addgroup -g 2024 -S nginx && adduser -s /sbin/nologin -S -D -u 2024 -G nginx nginx

COPY nginx.conf /apps/nginx/conf/nginx.conf
ADD index.html /data/nginx/html/index.html
RUN chown -R nginx.nginx /data/nginx/ /apps/nginx/
EXPOSE 80 443
# 这里由于修改了配置文件，配置文件里有daemon off前台运行了，所以直接nginx一敲就行了
CMD ["nginx"]
# 如果利用默认配置文件就需要手动前台运行
#CMD ["nginx","-g","daemon off;"]


# 构建镜像
vim build.sh
cat build.sh
#!/bin/bash
#-----------------------------------------------------
docker build -t nginx-alpine:1.16.1 .
```



下面是实验过程，比上面的配置文件少了一个自定义配置文件 还少了 端口暴露，要注意下

![image-20240507105727608](7-Docker自动制作镜像常见指令说明.assets/image-20240507105727608.png)



build一下

![image-20240507110018986](7-Docker自动制作镜像常见指令说明.assets/image-20240507110018986.png)

很顺利👆



run一下看看

![image-20240507110223128](7-Docker自动制作镜像常见指令说明.assets/image-20240507110223128.png)

ok跑起来了👆



![image-20240507110306409](7-Docker自动制作镜像常见指令说明.assets/image-20240507110306409.png)

测试OK

![image-20240507110249015](7-Docker自动制作镜像常见指令说明.assets/image-20240507110249015.png)

这样就制作出了一个镜像，同时也是按照  OS基础镜像-----安装常用命令得基础镜像------应用镜像得思路来弄得。

![image-20240507110656088](7-Docker自动制作镜像常见指令说明.assets/image-20240507110656088.png)





可惜的是上面的镜像没有把log做出来，人家官方的log直接就是能👇看得到下图是官方的log

![image-20240507112259346](7-Docker自动制作镜像常见指令说明.assets/image-20240507112259346.png)

看看人家官方的日志，关于日志前面有讲过的

![image-20240507112416238](7-Docker自动制作镜像常见指令说明.assets/image-20240507112416238.png)





分析下为什么我的alpine定制的nginx没有logs

![image-20240507112843599](7-Docker自动制作镜像常见指令说明.assets/image-20240507112843599.png)

所以其实是少了这个动作应该，

![image-20240507113147587](7-Docker自动制作镜像常见指令说明.assets/image-20240507113147587.png)

试试，在web01的容器里直接修改看看

![image-20240507113337116](7-Docker自动制作镜像常见指令说明.assets/image-20240507113337116.png)

好像不行没日志出来👇

![image-20240507114253792](7-Docker自动制作镜像常见指令说明.assets/image-20240507114253792.png)

去看看人家的好像这么tail也看不到

![image-20240507114516861](7-Docker自动制作镜像常见指令说明.assets/image-20240507114516861.png)

于是再看看自己的docker logs

有了！！，哈哈，原来上面的做法是对的，

![image-20240507114622490](7-Docker自动制作镜像常见指令说明.assets/image-20240507114622490.png)

只不过tail 看不到stdout罢了，所以将上面的操作改写成Dockerfile就行了



### 自定义的nginx-alpine做好日志



**1、自己折腾方法1--就不丑**

![image-20240507114923068](7-Docker自动制作镜像常见指令说明.assets/image-20240507114923068.png)

重新build

run

就ok啦

![image-20240507115056844](7-Docker自动制作镜像常见指令说明.assets/image-20240507115056844.png)

而且官方就是这么做的👇

![image-20240507134645470](7-Docker自动制作镜像常见指令说明.assets/image-20240507134645470.png)





**2、人家的做法2-也挺好**

你tail access.log 那error.log呢，这不是错误日志没有STDERR了，没有屏幕输出，怎么docker logs查看呢。  所以视频里老师就tail /apps/nginx/logs/*    牛逼哈哈哈也行~

![image-20240507115937478](7-Docker自动制作镜像常见指令说明.assets/image-20240507115937478.png)

buid后run，再logs看看👇

![image-20240507120052429](7-Docker自动制作镜像常见指令说明.assets/image-20240507120052429.png)





**这里再次复习一问题**

docker run运行一个容器，端口映射为8080，一段时间之后忘记当初运行容器的命令，请问如何修改映射的端口为80。

1、首先题目要看到，你说停掉重新run，run的选项你知道嘛？你说就那几个，对，万一人家用了非常规的选项呢。

2、所以处理方法有三，



①就是前文讲过的去修改分层文件里的配

![image-20240507143540302](7-Docker自动制作镜像常见指令说明.assets/image-20240507143540302.png)



②或者还可以，在你已经安装了runlike第三方工具的前提下，可以直接看出当初run的所有默认和非默认的选项，![image-20240507143239234](7-Docker自动制作镜像常见指令说明.assets/image-20240507143239234.png)





③或者还可以，inpsec自己去一个个看也可以找到当初run的配置选项。

![image-20240507143138563](7-Docker自动制作镜像常见指令说明.assets/image-20240507143138563.png)







# 汇总

![image-20240507151411444](7-Docker自动制作镜像常见指令说明.assets/image-20240507151411444.png)



还有一些问题得能够讲清楚，还得继续学习

![image-20240507151535456](7-Docker自动制作镜像常见指令说明.assets/image-20240507151535456.png)

