# 第5节 Docker配置优化





docker 工作是基于镜像，而镜像存储的驱动，通过docker info可见是overlay2。

<img src="5-Docker配置优化.assets/image-20240401113801442.png" alt="image-20240401113801442" style="zoom:50%;" />



![image-20240401114202132](5-Docker配置优化.assets/image-20240401114202132.png)



这些存储引擎不管是那种，都是联合文件系统

![image-20240401114700537](5-Docker配置优化.assets/image-20240401114700537.png)

比如ext4 xfs的文件系统，磁盘挂载到目录，是不可以重复挂载的，也就是一个目录你两个硬盘挂载的时候，前面一个就被顶掉了。

而docker的联合文件系统时可以把多个设备(所谓设备就是硬盘、分区、lvm这些)挂载到一个目录下，其效果就是这个目录下既可以看到设备1上的数据，也能看到设备2上的数据。

​		为什么可以采用联合文件系统，就是因为docker的image是多层镜像。

![image-20240401115713714](5-Docker配置优化.assets/image-20240401115713714.png)

image是一层层构建出来的

容器生成，是复制一份镜像，并在其上又添加了一层writable可读可写层，这层就用来存放容器自己的数据--比如LOG。

docker stop xxx ，就是把复制过来的image层删了，其上的容器子深深的writable层还在的。

![image-20240401120512888](5-Docker配置优化.assets/image-20240401120512888.png)



容器启动的时候其实是可以修改存储驱动的

![image-20240401133623870](5-Docker配置优化.assets/image-20240401133623870.png)

不过，想来一般是不会去改的。确认是overlay2就行了。万一不是就换docker版本~

查看支持的文件系统👇注意aufs是aufs，autofs是autofs，不要瞎搞

![image-20240401134116982](5-Docker配置优化.assets/image-20240401134116982.png)



# docker优化

## 下载源，其实应该私有库作缓存来弄比如nexus

可以使用阿里的个人镜像加速地址

![image-20240401135538590](5-Docker配置优化.assets/image-20240401135538590.png)



![image-20240401140127212](5-Docker配置优化.assets/image-20240401140127212.png)



其实最好是用私有缓存库比如nexushttps://cloud.tencent.com/developer/article/1764866



### socket通信-docker-client和server之间的通信

![image-20240401142344030](5-Docker配置优化.assets/image-20240401142344030.png)

默认走的本地socket文件，无法支持远端的调用。

配置方法👇下图绿框框；但是用的不太多。一般就是本地连接。

![image-20240401142523157](5-Docker配置优化.assets/image-20240401142523157.png)

该配置文件类似cli里的-H选项，既有socket文件也有tcp，可能就是写两个-H咯

![image-20240401143339943](5-Docker配置优化.assets/image-20240401143339943.png)

还可以改service文件

![image-20240401143508966](5-Docker配置优化.assets/image-20240401143508966.png)



**先通过修改daemon.json去试下**

![image-20240401165325655](5-Docker配置优化.assets/image-20240401165325655.png)

但是报错了👆



**换个思路，去service文件里改改**

![image-20240401170558409](5-Docker配置优化.assets/image-20240401170558409.png)

然后把daemon.json里的hosts那行删了，修改为

<img src="5-Docker配置优化.assets/image-20240402171915073.png" alt="image-20240402171915073" style="zoom:50%;" />

再reload 和 restart

![image-20240401172813797](5-Docker配置优化.assets/image-20240401172813797.png)

此时tcp和socket文件都好了

![image-20240401172847800](5-Docker配置优化.assets/image-20240401172847800.png)



![image-20240401173610011](5-Docker配置优化.assets/image-20240401173610011.png)





此时就可以远端client 调用 server 了，也就是docker cli 连接 docker server引起后 调用命令了

![image-20240402173332905](5-Docker配置优化.assets/image-20240402173332905.png)

然后远程跑一个nginx看看，此时容器都是退出的

![image-20240402173559934](5-Docker配置优化.assets/image-20240402173559934.png)

去远端windows的cmd里执行远程cli，run一个nginx

![image-20240402173718678](5-Docker配置优化.assets/image-20240402173718678.png)

然后去server看下docker ps

![image-20240402173933777](5-Docker配置优化.assets/image-20240402173933777.png)

这就实现了远程执行docker cli

同时curl 瞧瞧，因为是容器里的ip没有暴露出来呢，所以ipsect看到ip后宿主机上curl下👇

![image-20240402174041031](5-Docker配置优化.assets/image-20240402174041031.png)

要注意docker engine也就是服务端在哪，容器就运行在哪里，客户都的cli只是一个连接调用工具，很好理解，好比mysql 远程执行命令一样。



将来在K8S里的跨主机的远程docker调用可能就是用的这个。目前docker学习阶段还用不到。

<img src="5-Docker配置优化.assets/image-20240402180552713.png" alt="image-20240402180552713" style="zoom:50%;" />



<img src="5-Docker配置优化.assets/image-20240402180924506.png" alt="image-20240402180924506" style="zoom:50%;" />



看来是查不到就往下一个地址查找。



现在都是systemd了👇<img src="5-Docker配置优化.assets/image-20240402181155813.png" alt="image-20240402181155813" style="zoom:55%;" />

不用改了，以前是这么改的👇也是daemon.json里

<img src="5-Docker配置优化.assets/image-20240402181223650.png" alt="image-20240402181223650" style="zoom:80%;" />







还有这个容器和镜像存放的路径

![image-20240402181409786](5-Docker配置优化.assets/image-20240402181409786.png)

默认是这里👆，这么改👇改到单独的高性能ssd上去，前文讲过了，目录不该直接挂到新磁盘上去，或者该目录这个目录挂过去也行👇

<img src="5-Docker配置优化.assets/image-20240402181801828.png" alt="image-20240402181801828" style="zoom:80%;" />

改目录后，老数据--就是当前已经产生的容器和镜像文件都是还在老地方的。



现在立即改会出现的问题





