# 第4节 Docker容器管理常见用法



# docker查看日志



**概述**

docker logs 本质是查看容器在终端打印出来的信息，只要你的容器有STDOUT就会被docker logs在外面抓到，不过一般来讲都是容器里的日志才会在屏幕打印，所以docker logs也就起了个名字叫logs否则正宗点应该叫docker stdout，哈哈。



run一个nginx看看上面说的，通过docker top 可见nginx是daemon off也就是容器里是前台运行的，这样才能保证容器run起来不会停止，然后再docker run -d 将 docker run xxx 放入后台执行--否则容器up后会占用宿主的前台。

![image-20240424095400290](4-Docker容器管理常见用法.assets/image-20240424095400290.png)



![image-20240424094708414](4-Docker容器管理常见用法.assets/image-20240424094708414.png)

<img src="4-Docker容器管理常见用法.assets/image-20240424100502765.png" alt="image-20240424100502765" style="zoom:40%;" />



简单讲一句好话：容器的run起来就是要：里面是前台，外面是后台

解释：里面是前台才能run起来不stop这是容器的固有要求；外面是后台，容器up后才不会占用宿主的前台。





**测试:docker logs其实是看的容器里的屏幕打印(stdout和stderr)**

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









