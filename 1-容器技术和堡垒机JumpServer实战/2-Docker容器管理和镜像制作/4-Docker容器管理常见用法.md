# 第4节 Docker容器管理常见用法



# docker查看日志



## **概述**

docker logs 本质是查看容器在终端打印出来的信息，只要你的容器有STDOUT就会被docker logs在外面抓到，不过一般来讲都是容器里的日志才会在屏幕打印，所以docker logs也就起了个名字叫logs否则正宗点应该叫docker stdout，哈哈。



run一个nginx看看上面说的，通过docker top 可见nginx是daemon off也就是容器里是前台运行的，这样才能保证容器run起来不会停止，然后再docker run -d 将 docker run xxx 放入后台执行--否则容器up后会占用宿主的前台。

![image-20240424095400290](4-Docker容器管理常见用法.assets/image-20240424095400290.png)



![image-20240424094708414](4-Docker容器管理常见用法.assets/image-20240424094708414.png)

<img src="4-Docker容器管理常见用法.assets/image-20240424100502765.png" alt="image-20240424100502765" style="zoom:40%;" />



简单讲一句话：容器的run起来就是要：里面是前台，外面是后台

解释：里面是前台才能run起来不stop这是容器的固有要求；外面是后台，容器up后才不会占用宿主的前台。





## **docker logs其实是看的容器里的屏幕打印(stdout和stderr)**

测试如下👇

此时就可以通过docker logs看到容器里的STDOUT

![image-20240424102128388](4-Docker容器管理常见用法.assets/image-20240424102128388.png)

证明就是STDOUT而不仅仅是LOGS👇

![image-20240424102759227](4-Docker容器管理常见用法.assets/image-20240424102759227.png)

我本以为正因为nginx将两个日志软连接到了stdout和stderr也就是屏幕打印，才会被外面宿主的docker logs抓到，结果当我直接产生屏幕输出的时候docker logs 并没有抓到。

而且cat也就卡住了--在容器里👇，<font color=red>我知道了，因为你exec -it进去不是之前run的时候的那个tty了，所以exec 进去的stdout在docker logs web01的那个tty是看不到的，echo不进去和cat不出来则是因为①看似echo >> access.log和cat access.log实则echo >> /dev/stdout和cat /dev/stdout 自然是：前置不是一个tty外面docker logs自然看不到，后缀cat /dev/stdout卡住其实就是宿主上单独cat一个道理，什么道理，哟还会抬杠啊，就是cat=cat /dev/stdout是等待STDIN将STDIN的内容输入到STDTOU上</font>

![image-20240424103528994](4-Docker容器管理常见用法.assets/image-20240424103528994.png)

![image-20240424110125297](4-Docker容器管理常见用法.assets/image-20240424110125297.png)

![image-20240424110158053](4-Docker容器管理常见用法.assets/image-20240424110158053.png)



![image-20240424110250537](4-Docker容器管理常见用法.assets/image-20240424110250537.png)



改为run -it方式进去，然后用attach也就是一个tty去测试，理当实现 ： 容器里的屏幕的东西都能在docker logs 看到了。

![image-20240424105250344](4-Docker容器管理常见用法.assets/image-20240424105250344.png)

此时logs就能拿到stdout了，这就上文红色字体所说的一个tty才行。

![image-20240424105314754](4-Docker容器管理常见用法.assets/image-20240424105314754.png)





换一种证明方式👇

![image-20240424104524790](4-Docker容器管理常见用法.assets/image-20240424104524790.png)

正因为tailf -f 是屏幕打印的内容，所以docker logs 就把这个屏幕打印的内容抓出来了，

/etc/hosts----tail--->STDOUT----docer logs---->宿主看到了

而不是说/etc/hosts变成了log，哦，不要瞎理解。



​		**前一个例子，STDOUT由于是不同的tty**，所以没法同步信息--一边屏幕的STDOUT---一遍docker logs看到，因为不是一个tty。

​		**现在一个例子tail -f /etc/hosts**，也就是上图你exec -it进去往/etc/hosts里echo点东西，这样即便不是同一个tty：一边exec 进去往这个文件里写东西，一边另一个running的容器里由于前台tail -f /etc/hosts了，只要别的tty往该文件写东西，那么容器的tail -f是会看到的，于是容器本身的屏幕也会有同步的打印。  # 落文件了所以不同tty都是同步的该文件而已。

​		而前一个例子不同tty中间可能有文件作为中转桥梁，他们是各自的tty在各自的STDOUT或者STDERR。



```shell
docker run  --name test01 busybox /bin/sh -c 'i=1;while true;do echo $i;let i++;sleep 1;done'
```

![image-20240424150411056](4-Docker容器管理常见用法.assets/image-20240424150411056.png)

logs一样可见，

![image-20240424150610068](4-Docker容器管理常见用法.assets/image-20240424150610068.png)

还是那句话logs看的容器的tty正式当前你run的那个容器的tty

你若此时exec -it 进去该容器，就是另一个tty了，此时STDOUT由于是不同的tty不会在docker logs里看到，因为docker logs看的是就是那个容器里的tty终端。



然后sh -c 要用单引号👇以下是区别，其实就是你要将里面所有的都原封不动的传递到docker run 里的sh -c里去，所以自然不能解析的，所以必须是单引号，统统是字符串扔进去。

![image-20240424151633576](4-Docker容器管理常见用法.assets/image-20240424151633576.png)

补充：为什么一直是100？因为i=100，双引号展开后为"i=1;while true;do echo 100;let i++;sleep1;done"  也就是说容器里实际执行的是echo 100，哈哈哈，你说为啥一直100呢。



**docker ps docker看到的容器的日志，在宿主机上也有文件落地的**

查看容器日志落在宿主上的文件位置：

```
/var/lib/docker/containers/0axxxx...xxxb471ca/0axxx...xxx471ca-json.log
```

![image-20240424164325068](4-Docker容器管理常见用法.assets/image-20240424164325068.png)

然后logs越来越多，会导致磁盘空间占用越来越多，常用方法

![image-20240424165444650](4-Docker容器管理常见用法.assets/image-20240424165444650.png)





# **关于替换掉容器里的原本cli**

​		比如原来容器的cli是一个脚本来启动容器的，现在该脚本被你替换为tail -f xxx了，自然也就不会启动nginx服务了，所以这种情况就要小心了。

![image-20240424170721370](4-Docker容器管理常见用法.assets/image-20240424170721370.png)



![image-20240424171550061](4-Docker容器管理常见用法.assets/image-20240424171550061.png)

虽然外面看

![image-20240424171813170](4-Docker容器管理常见用法.assets/image-20240424171813170.png)

但是里面你把脚本替了，里面端口都没开👇

![image-20240424180541705](4-Docker容器管理常见用法.assets/image-20240424180541705.png)



替代cli的使用场景，一般是用来测试查看的。





# 容器里的DNS以及hosts文件



类比宿主的hosts本地解析情况

![image-20240424181546719](4-Docker容器管理常见用法.assets/image-20240424181546719.png)

容器里多了一个容器IP，且解析到容器ID。当然127.0.0.1还是老样子

![image-20240424181529142](4-Docker容器管理常见用法.assets/image-20240424181529142.png)

还有个仅仅是验证hosts文件，可以docker run --rm 来做，演示完退出容器后直接删除容器

![image-20240424182341015](4-Docker容器管理常见用法.assets/image-20240424182341015.png)



![image-20240424182420774](4-Docker容器管理常见用法.assets/image-20240424182420774.png)



**docker run的时候就新增host解析**

host文件添加进去就好了，不过外面也可以

![image-20240425094243819](4-Docker容器管理常见用法.assets/image-20240425094243819.png)



![image-20240425095019651](4-Docker容器管理常见用法.assets/image-20240425095019651.png)

再次理解下-it -d 和cli👇

![image-20240425095406770](4-Docker容器管理常见用法.assets/image-20240425095406770.png)



### docker容器里的dns

这点我前文就总结过了，结合工作中的故障案例进行和归纳总结

![image-20240425100652839](4-Docker容器管理常见用法.assets/image-20240425100652839.png)



这里补充其他的点：比如ubuntu的情况、比如除了daemon.json 还有docker run --dns=来指定。

**ubuntu的容器的dns**

和rock-linux一样也是从docker引擎的daemon.json优先然后再结合/etc/resolv.conf里来的，但是ubuntu的宿主机的/etc/resolv.conf文件里看到的其实不是真实的宿主使用的dns，而是要通过

![image-20240425100350967](4-Docker容器管理常见用法.assets/image-20240425100350967.png)

上图的/etc/resovle.conf是容器里的dns的使用文件，该文件自然也是从宿主机的docker引擎的daemon.json 结合 宿主的dns 来生产的。

​		而宿主机的dns👇下图显示了ubuntu的特殊的查看方法（ubuntu的/etc/resolv.conf里看不到真实的dns配置的）

![image-20240425101048549](4-Docker容器管理常见用法.assets/image-20240425101048549.png)



![image-20240425101111322](4-Docker容器管理常见用法.assets/image-20240425101111322.png)

同时还有其他的cli查看👇

![image-20240425101439443](4-Docker容器管理常见用法.assets/image-20240425101439443.png)



**dns和options的优先级，从上到下：**

①docker run指定的参数优先

②daemon.json指定的参数其次

③宿主机的/etc/resolv.conf指定的参数最后

④指定的参数优先的意思就是，就两个参数其实，一个dns 一个options，比如：run 和 json里都没有指定options，而宿主机的resolv.cnf里制定了，那么就是宿主的配置优先了。

![image-20240425103309407](4-Docker容器管理常见用法.assets/image-20240425103309407.png)

注意上图的一个点：就是10.1.1.1是不会生效的，因为/etc/resolv.conf文件里的namserver只认前三个。测试过了的在前面的章节里--就是本段文字的标题下面的第一张图那边做了测试截图的。

```shell
docker run --rm --name web01 --dns=1.1.1.1 --dns=2.2.2.2 --dns=3.3.3.3 --dns-option=timeout:23 --dns-option=attempt:3 --dns-option=rotate --dns-option="single-request-reopen" busybox cat /etc/resolv.conf
```

![image-20240425105000486](4-Docker容器管理常见用法.assets/image-20240425105000486.png)

这样差不多了吧，单个容器生效，也不影响其他容器，也不用dameon.json，也不用宿主的/etc/resolv.conf，就挺专业，哈哈。



**search要删掉**

还有一个search xx.xxx一般不怎么用，错了，不是不怎么用，而是一定要删掉，防止乱给你自动添加捣乱。

![image-20240425105831171](4-Docker容器管理常见用法.assets/image-20240425105831171.png)

没啥用，谁指望search来补齐啊；而且可能带来意料之外的故障。

![image-20240425110037203](4-Docker容器管理常见用法.assets/image-20240425110037203.png)

search怎么删，不是去/etc/resolv.conf里删，这只是个动态生成的文件，源头在网卡配置文件、或者nmcli去配置修改。



ubuntu里

<img src="4-Docker容器管理常见用法.assets/image-20240425110639921.png" alt="image-20240425110639921" style="zoom:33%;" />



centos一样类似的网卡，或者用的是NetworkManager的直接用nmcli去修改就行了



![image-20240425111541638](4-Docker容器管理常见用法.assets/image-20240425111541638.png)

图中ipv4.dns和IP4.DNS的区别，一个是前面配置的，一个时候后面最终生成的，你可以理解成配置文件和最终状态，唉，这些太细了，不管了，总之实际操作的时候带点脑子就行了。不理会这句话也行。





# 容器和宿主机文件怎么交换



## **docker cp**

-a 好比cp -a，保留属性

![image-20240425113332684](4-Docker容器管理常见用法.assets/image-20240425113332684.png)



操作下，跑一个容器up着

![image-20240425113012356](4-Docker容器管理常见用法.assets/image-20240425113012356.png)



然后将宿主机的一个文件，复制进容器里

```shell
docke cp -a /etc/issue test01:/tmp/
```

拷进去：

![image-20240425114653716](4-Docker容器管理常见用法.assets/image-20240425114653716.png)

拷出来

![image-20240425115147560](4-Docker容器管理常见用法.assets/image-20240425115147560.png)





# 容器的环境变量

这东西懂的都知道很关键的：

1、cmd里的set，看的是环境变量，可能看的都不全对吧，毕竟仅仅是当前用户的吧；存在软件退出了，网络设置里的代理里也清空了，但是set里发现PROXY变量的存在，这种情况也可能导致网络问题，故障定位就很难，出现过一次，具体细节我忘了，结论就是set里也要观察是否配置了代理，容易忽略这个点。

2、env也就是linux看环境变量的cli。

3、环境变量里的PATH变量，crontable里的写清楚PATH变量.

![image-20240425135622842](4-Docker容器管理常见用法.assets/image-20240425135622842.png)

crontable正儿八经还要写

![image-20240425135652603](4-Docker容器管理常见用法.assets/image-20240425135652603.png)

这些都是环境变量



每个容器自己都有环境变量，环境变量的用途和镜像密切相关，有些是与众不同的的环境变量。

![image-20240425140531905](4-Docker容器管理常见用法.assets/image-20240425140531905.png)

再看看mysql的变量

![image-20240425141132594](4-Docker容器管理常见用法.assets/image-20240425141132594.png)

mariadb的变量

![image-20240425142502347](4-Docker容器管理常见用法.assets/image-20240425142502347.png)



还可以传递变量用-e

![image-20240425150011901](4-Docker容器管理常见用法.assets/image-20240425150011901.png)

注意上图mysql如果PASSWORD变量没给进去，是起不来的，

如果用了--restart=always，就会看到容器状态是一直重启中

![image-20240425155452186](4-Docker容器管理常见用法.assets/image-20240425155452186.png)



![image-20240425150649895](4-Docker容器管理常见用法.assets/image-20240425150649895.png)

还可以写到一个文件里统一安置

![image-20240425151235519](4-Docker容器管理常见用法.assets/image-20240425151235519.png)



不过这些例子的环境变量没有具体意义，mysql的环境变量一般这么设置

```shell
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=123456 --name mysql -d --restart=always mariadb:11.3.2
```

![image-20240425161002137](4-Docker容器管理常见用法.assets/image-20240425161002137.png)

去掉env观察

![image-20240425161017715](4-Docker容器管理常见用法.assets/image-20240425161017715.png)

可得👆是password没给。其实是you need to specify one of the following as an environment variable，是三个变量给一个就行了。

然后evn取代了docker-entrypoint.sh的脚本，也报错，可见password变量是在cmd之前就需要的。

​		然后设置为随机密码或者空密码：具体-e的传递参数的格式，这是当初创建镜像的作者指定的，要去hub.docker.com去查看mysql镜像的传递参数格式，而不是去mysql官方网站去查看。

![image-20240425162212782](4-Docker容器管理常见用法.assets/image-20240425162212782.png)



![image-20240425162337327](4-Docker容器管理常见用法.assets/image-20240425162337327.png)

然后问题来了随机密码MYSQL_RANDOM_ROOT_PASSWORD到底是多少呢，看logs就行啦~👇

![image-20240425163428625](4-Docker容器管理常见用法.assets/image-20240425163428625.png)

尝试登入mysql，由于没有做端口暴露，所以我用容器的ip去连👇

![image-20240425165204462](4-Docker容器管理常见用法.assets/image-20240425165204462.png)

好，那么问题来了：如何安全的设置mysql的密码，

1、你用随机密码，logs里可见，不安全；所以需要手动进去修改，则算是一种解决方案；

2、**√** 你用手动指定-e MYSQL_ROOT_PASSWORD=123456 来指定，结果history里可见，不安全；所以我们要vim mysql.env进去定义变量，chown 600 mysql.evn，然后source mysql.env，然后再-e MYSQL_ROOT_PASSWORD=$VAR就行了。

![image-20240425170239180](4-Docker容器管理常见用法.assets/image-20240425170239180.png)

这样就安全了，

![image-20240425170606377](4-Docker容器管理常见用法.assets/image-20240425170606377.png)



然后再试试用文件传参的方式--env-file👇

![image-20240425171756062](4-Docker容器管理常见用法.assets/image-20240425171756062.png)

上图提示No space left on device，磁盘又够了，删吧

![image-20240425172522113](4-Docker容器管理常见用法.assets/image-20240425172522113.png)

后面会讲怎么制作镜像，让用户使用镜像的人可以传递变量，也就是制作镜像的时候要考虑的事情了。

还有别忘了

![image-20240425173205188](4-Docker容器管理常见用法.assets/image-20240425173205188.png)





# 清理容器



清理前先看看：可以i用 docker system



**docker system df**

![image-20240425173906580](4-Docker容器管理常见用法.assets/image-20240425173906580.png)

停掉几个up的容器后，现在RECLAIMABLE里看到可以回收的空间了

![image-20240425174200836](4-Docker容器管理常见用法.assets/image-20240425174200836.png)

解释如下<img src="4-Docker容器管理常见用法.assets/image-20240425174355078.png" alt="image-20240425174355078" style="zoom:47%;" />



**docker system events**

![image-20240425175012671](4-Docker容器管理常见用法.assets/image-20240425175012671.png)



**docker system info = docker info**

![image-20240425175538514](4-Docker容器管理常见用法.assets/image-20240425175538514.png)



**docker system prune**

![image-20240425175918253](4-Docker容器管理常见用法.assets/image-20240425175918253.png)

四个-all分别是

所有停止的容器

所有没有使用的网络，网络后面讲

所有没有标签的镜像

所有dangling的build cache



dangling就是没有标签的镜像，产生的原因如下👇，就是容器正在用着一个image，你删image的时候还得是用的TAG，则删除了TAG，也就是没有TAG，也叫dangling。注意哦，容器用在是删不掉image的哦，这一点通过ID去删就明显看到了，通过TAG去删也只是去掉了TAG而已。

![image-20240425180425126](4-Docker容器管理常见用法.assets/image-20240425180425126.png)



容器没有UP的，Exited的，image是可以删掉的

![image-20240425180724998](4-Docker容器管理常见用法.assets/image-20240425180724998.png)

然后容器run起来是复制了image一份，否则容器停止后将image删除，为什么容器还能继续UP呢，所以正是因为容器run这个动作就是复制一份image的。

![image-20240425181140332](4-Docker容器管理常见用法.assets/image-20240425181140332.png)



