# 第1节 Docker运行容器的常见用法



# docker怎么管理容器的



## 概述





 <img src="1-Docker运行容器的常见用法.assets/image-20240415101723667.png" alt="image-20240415101723667" style="zoom:50%;" />

![image-20240415103201163](1-Docker运行容器的常见用法.assets/image-20240415103201163.png)









![image-20240415103824654](1-Docker运行容器的常见用法.assets/image-20240415103824654.png)

互联网上的镜像，不需要登入，直接下载，一般公司私有的镜像才需要login后去下载



## COMMAND

![image-20240415104528966](1-Docker运行容器的常见用法.assets/image-20240415104528966.png)



![image-20240415105021283](1-Docker运行容器的常见用法.assets/image-20240415105021283.png)

​		可以修改为自己希望执行的命令，以及加上ARG也就是跟上参数，但是加的的cli必须是人家容器里有的命令否则无法执行的。比如上图hello-world容器里只有/hello这个命令，其他的红色框框👆里的cil都是没有的， 所有都是报错的。   # 我就想到其实容器里是否可以封装一个busybox呢，哈哈基本很多常用cli就都有了。我觉得还不错~

​		镜像就是没有boot(内核)的根文件系统+app程序文件(mysql、nginx、等各自的程序包括各自依赖的库)。





下个busybox看看多大是否可以后面集成到我工作中的images里

1、首先hub.docker.com搜索latest

![image-20240415113859282](1-Docker运行容器的常见用法.assets/image-20240415113859282.png)

2、然后找哈希值一样的版本号，

有同学问，那你为什么不直接下latest，你猜我为啥

![image-20240415114000088](1-Docker运行容器的常见用法.assets/image-20240415114000088.png)



![image-20240415114057219](1-Docker运行容器的常见用法.assets/image-20240415114057219.png)

5MB不到，可以接受吧~







### alpine举例，说明commond的是前台cli，否则就是运行后即退出

![image-20240415112457390](1-Docker运行容器的常见用法.assets/image-20240415112457390.png)

​		容器运行起来不退出，要求容器运行时对应的COMMAND的是前台执行，而👆/bin/sh是一个后台执行的cli，所以运行后就提出了。类似docker run nginx执行了就是前台占用。

​		所谓前台执行就是执行后就霸占了前台，而传统systemctl start nginx这种启动方式在容器里就不能使用了，因为后台执行，容器里执行就退出了。

![image-20240415112958240](1-Docker运行容器的常见用法.assets/image-20240415112958240.png)



![image-20240415113014809](1-Docker运行容器的常见用法.assets/image-20240415113014809.png)



![image-20240415131028393](1-Docker运行容器的常见用法.assets/image-20240415131028393.png)  

怎么停不掉啊👆，关闭窗口还是up的，因为没有-it交互，你的ctrl c 发不进去👆

![image-20240415131139775](1-Docker运行容器的常见用法.assets/image-20240415131139775.png)



加个-d 放后台，但是不会退出，因为用的cli 是前台就行。

![image-20240415131210932](1-Docker运行容器的常见用法.assets/image-20240415131210932.png)





![image-20240415131722121](1-Docker运行容器的常见用法.assets/image-20240415131722121.png)



![image-20240415131829745](1-Docker运行容器的常见用法.assets/image-20240415131829745.png)



### --name选项，指定容器NAME

比如指定容器的NAME方便后面，比如容器之间的互访

![image-20240415133543953](1-Docker运行容器的常见用法.assets/image-20240415133543953.png)

随机的名字肯定可读性不好👆

![image-20240415133729937](1-Docker运行容器的常见用法.assets/image-20240415133729937.png)





![image-20240415134402626](1-Docker运行容器的常见用法.assets/image-20240415134402626.png)





### -it交互模式进入容器里，看看文件，看看状态等操作

![image-20240415135101975](1-Docker运行容器的常见用法.assets/image-20240415135101975.png)

-i是交互，还不够，还需要一个tty接口，类似交换机的vty

-i -t 还不够，还需要交互的sh否则👇

![image-20240415135433555](1-Docker运行容器的常见用法.assets/image-20240415135433555.png)

-it 配合什么shell，就是尝试尝试就行了

![image-20240415135719648](1-Docker运行容器的常见用法.assets/image-20240415135719648.png)

👆这就进来了



不仅仅有bash也有/bin/sh

![image-20240415140235224](1-Docker运行容器的常见用法.assets/image-20240415140235224.png)



### 只是用了bash交互，run的command还是原来的那个脚本

![image-20240415160059055](1-Docker运行容器的常见用法.assets/image-20240415160059055.png)

![image-20240415160130449](1-Docker运行容器的常见用法.assets/image-20240415160130449.png)

进来后，

![image-20240415160311606](1-Docker运行容器的常见用法.assets/image-20240415160311606.png)

通过👆boot里是空的就知道了，容器用的是宿主的boot(内核)，所以这里只是空的。



![image-20240415160438367](1-Docker运行容器的常见用法.assets/image-20240415160438367.png)

这个nginx是基于Debian做出来的，Debian系列的典型代表就是ubuntu了，apt就是yum了



### 安装ps

![image-20240415161859681](1-Docker运行容器的常见用法.assets/image-20240415161859681.png)



![image-20240415161921232](1-Docker运行容器的常见用法.assets/image-20240415161921232.png)

ps就有了👇

![image-20240415161938718](1-Docker运行容器的常见用法.assets/image-20240415161938718.png)



### 容器里用sed修改为清华源，安装更快，调试更方便

容器安装东西大多数都是调试用的。

```shell
sed -i 's|http://deb.debian.org/debian|https://mirrors.tuna.tsinghua.edu.cn/debian|g' /etc/apt/sources.list.d/debian.sources
sed -i 's|http://deb.debian.org/debian-security|https://mirrors.tuna.tsinghua.edu.cn/debian-security|g' /etc/apt/sources.list.d/debian.sources

```



容器exit就真的将容器退出了，即使容器本来时run起来就前台执行的那种，你用-it bash进去再exit，容器就不再时running了👇

![image-20240415163327923](1-Docker运行容器的常见用法.assets/image-20240415163327923.png)

如果不想exit退出的时候把本来应该前台执行的容器退出也可以用ctrl p q 组合键来安全退出，不过此时nginx就不再时前台了，而是后台up了，类似于docker run -d nginx了

![image-20240415163851622](1-Docker运行容器的常见用法.assets/image-20240415163851622.png)

上图👆不是nginx这种run起来就up的，一样也会被ctl p q变成后台up👇

![image-20240415164200367](1-Docker运行容器的常见用法.assets/image-20240415164200367.png)

这下倒学到了一个让hello-world后台UP的方法，赶紧试试👇，不行人家压根就没有shell，所以无法进去。而有shell的倒是可以用这种方式后台UP着。



### run -it用的少，大多数都是exec -it来针对运行中的容器进行交互排错

比如run了一个nginx

![image-20240415170121263](1-Docker运行容器的常见用法.assets/image-20240415170121263.png)

发现某个问题，需要进去排错，于是docker exec -it进去

![image-20240415170306952](1-Docker运行容器的常见用法.assets/image-20240415170306952.png)



![image-20240415170755466](1-Docker运行容器的常见用法.assets/image-20240415170755466.png)



### hostname只在容器内部生效，不具备容器互通的功效，只是在容器内部可以拿来本地通信。

通过这个hostname和本机的程序通信

![image-20240415171241453](1-Docker运行容器的常见用法.assets/image-20240415171241453.png)



### 容器运行起来后，退出自动删除，临时测试屁股擦的比较干净

![image-20240415172738881](1-Docker运行容器的常见用法.assets/image-20240415172738881.png)

只要容器run完退出的，那么就给你删除

![image-20240415173656774](1-Docker运行容器的常见用法.assets/image-20240415173656774.png)





​                                <img src="1-Docker运行容器的常见用法.assets/image-20240415175042057.png" alt="image-20240415175042057" style="zoom:66%;" /> 

所以，要做守护式容器，就得至少有一个进程是前台永久运行的👆

很多添加前台永久运行的手段：

```shell
docker run alpine:3.19.1 tail -f /dev/null
```

![image-20240415175639814](1-Docker运行容器的常见用法.assets/image-20240415175639814.png)

👆上图就是前台运行不会退出，防止被ctrl c退出，于是加一个-d，是先保证一个前台cli再放到后台👇

![image-20240415175624436](1-Docker运行容器的常见用法.assets/image-20240415175624436.png)

然后前文也说了，这里也复制过来一并看下👇

![image-20240415131028393](1-Docker运行容器的常见用法.assets/image-20240415131028393.png)  

怎么停不掉啊👆，因为没有-it交互，你的ctrl c 发不进去👆 放屁-it也不行

![image-20240415182339492](1-Docker运行容器的常见用法.assets/image-20240415182339492.png)

关闭窗口也没用

![image-20240415180943058](1-Docker运行容器的常见用法.assets/image-20240415180943058.png)

只能docker rm -f了 👇

![image-20240415180617565](1-Docker运行容器的常见用法.assets/image-20240415180617565.png)









### 那么问题来了，

1、docker run alpine tail -f /dev/null       无法ctrl c 中断

2、docker run nginx        为啥可以ctrl c 中断

难道nginx的COMMAND里的也就是xxx.sh里是用了-it的？

不行1：用sh -c 包装也不行，就是不接受ctrl c，不知道为啥

![image-20240415182831970](1-Docker运行容器的常见用法.assets/image-20240415182831970.png)















# 镜像的制作



