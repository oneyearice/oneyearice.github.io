# 第7节 Docker镜像导出导入



## 镜像导出和导入-比如互联网下载后导入开发内网去-导出要用name:tag

![image-20240410181848729](6-Docker镜像搜索和下载.assets/image-20240410181848729.png)

docker save 默认是标准输出也就是屏幕打印了，所以需要-o write to a file ,instead of STDOUT

![image-20240410182058372](6-Docker镜像搜索和下载.assets/image-20240410182058372.png)

👆上图是屏幕拒绝了这个输出，其实还是有STDOUT动作，也就是说可以在上图的cli后使用重定向的。

因为save出来是tar包格式，所以要加上.tar

![image-20240410182145259](6-Docker镜像搜索和下载.assets/image-20240410182145259.png)

tar包只是打包的，没有压缩的这里，所以还需要压缩一下，减少磁盘空间占用

![image-20240410182332654](6-Docker镜像搜索和下载.assets/image-20240410182332654.png)



优化命令，就是导出来就是压缩的效果

1、首先使用重定向

![image-20240410182718741](6-Docker镜像搜索和下载.assets/image-20240410182718741.png)



2、然后

![image-20240410183641772](6-Docker镜像搜索和下载.assets/image-20240410183641772.png)

怎么直接打出来的，大小还少了11KB了👆，呵呵，应该数据没丢哦，哈哈哈；回头导入的时候看一下就知道了。



3、复制到其他docker机器上load

![image-20240411103714519](7-Docker镜像导出导入.assets/image-20240411103714519.png)

load的时候👆-i或<使用TDIN都行，不过此时load会导致dangling镜像的产生；而导致danling镜像也就是\<none\>

的原因是当初save导出镜像的时候用的是ID而不是名称+TAG，所以源头就错了



即使你想通过tag去补也是不行的，因为你压根就不知道这个ID是什么镜像inspect也看不到了

![image-20240411104321475](7-Docker镜像导出导入.assets/image-20240411104321475.png)

除非你run起来进去看看是个啥容器，再看看image的哈希值，然后去官网比对哈希值，然后打上对应的版本哈哈

![image-20240411104528999](7-Docker镜像导出导入.assets/image-20240411104528999.png)



搞错了，再来，导出的时候不带tag就是所有tag达成一个包，带tag就是那个tag的导出咯👇

![image-20240411112234030](7-Docker镜像导出导入.assets/image-20240411112234030.png)

![image-20240411112444403](7-Docker镜像导出导入.assets/image-20240411112444403.png)



### 把一个机器的所有镜像导出导入到别的机器







补充，你也可以指定下载哪个，而不是通过自动判断你是amd64的，

![image-20240411110840564](7-Docker镜像导出导入.assets/image-20240411110840564.png)



![image-20240411110801379](7-Docker镜像导出导入.assets/image-20240411110801379.png)

就是没有tag了，自己打咯用上面的docker tag 来打





### 所有镜像导出



1、也算是一条命令搞定吧

![image-20240411133601695](7-Docker镜像导出导入.assets/image-20240411133601695.png)

```shell
docker save `docker images --format "{{.Repository}}" |sort |uniq |xargs` -o  all_images.tar


# 推荐这个吧👇，虽然uniq那种也不错
docker save `docker images --format "{{.Repository}}:{{.Tag}}"` |gzip > all_images.tar.gz


docker save `docker images --format "{{.Repository}}" |sort |uniq |xargs` |gzip > all_images_end.tar.gz
```

因为压缩要时间的，所以你自己选择具体cli👆

![image-20240411135308318](7-Docker镜像导出导入.assets/image-20240411135308318.png)

![image-20240411135547479](7-Docker镜像导出导入.assets/image-20240411135547479.png)

综上所述，推荐的cli是

```shell
docker save `docker images --format "{{.Repository}}"` |sort |uniq |gzip > all_images.tar.gz          # 和上图的区别就是省略掉了xargs，不用也一样的效果

👆上面的cli，有错误，反引号写错了，导致sort cpu占用率100% 无限永久卡住了，修改后👇就恢复了

docker save `docker images --format "{{.Repository}}" |sort |uniq` |gzip > all_images_no_xargs.tar.gz       # 这个就行了★
```

sort卡住不动的CPU👇

![image-20240411140012298](7-Docker镜像导出导入.assets/image-20240411140012298.png)

一个sort 100% cpu 啊？不正常。正常，稍微分析下就知道答案了：

​		卡的原因也好理解--就是docker save xxx 其实就是images的导出内容了，是内容啊，所以很大，一直在sort排序，所以sort处理大量的数据了就，什么你表示怀疑，哦，那你用docker save xxx |cat 看看就知道啦

![image-20240411141836886](7-Docker镜像导出导入.assets/image-20240411141836886.png)

你说sort忙的过来不，这还是一部分，下面没完没了的乱码 二进制 格式👇

![image-20240411142046969](7-Docker镜像导出导入.assets/image-20240411142046969.png)

纠正截图👇

![image-20240411140813895](7-Docker镜像导出导入.assets/image-20240411140813895.png)



**删除所有镜像**

```shell
docker rm -f `docker images -aq`
```

，再导入就全部回来了👇

![image-20240411133837034](7-Docker镜像导出导入.assets/image-20240411133837034.png)

这样也行：

![image-20240411142722570](7-Docker镜像导出导入.assets/image-20240411142722570.png)



### 还有虽然是tar包，但是解开后是看不到images的，而是一层一层文件，还得是load才行

![image-20240411143234917](7-Docker镜像导出导入.assets/image-20240411143234917.png)



![image-20240411143311984](7-Docker镜像导出导入.assets/image-20240411143311984.png)



### 然后看看tar包里的这些文件吧

![image-20240411143814596](7-Docker镜像导出导入.assets/image-20240411143814596.png)

一个解包多出这么多，继续

![image-20240411143856629](7-Docker镜像导出导入.assets/image-20240411143856629.png)

发现👆layer.tar解开里面就是一个精简的 linux的 根目录 / 

![image-20240411144041029](7-Docker镜像导出导入.assets/image-20240411144041029.png)

对比发现，少了boot这个么一个重要的文件，这很好理解，bootfs可不是在容器里，而在kernel里是宿主的内核，容器里只有rootfs层以及以上--这些是用户空间的东西。

<img src="7-Docker镜像导出导入.assets/image-20240411144259762.png" alt="image-20240411144259762" style="zoom:33%;" /> <img src="7-Docker镜像导出导入.assets/image-20240411144320260.png" alt="image-20240411144320260" style="zoom:33%;" />



然后继续看

![image-20240411144844245](7-Docker镜像导出导入.assets/image-20240411144844245.png)

这个精简镜像里的bin下的二进制都是busybox这个二进制的别名。

所以这些cat 、 df 、grep、mv、ls各种linux的命令都是busybox这个二进制模拟出来的，果真很busy啊~

尝试使用该牛逼的二进制，结果发现用不了👇

![image-20240411145241074](7-Docker镜像导出导入.assets/image-20240411145241074.png)

难道有库依赖，还真有👇

![image-20240411145409842](7-Docker镜像导出导入.assets/image-20240411145409842.png)



<img src="7-Docker镜像导出导入.assets/image-20240411145620231.png" alt="image-20240411145620231" style="zoom:44%;" />



然后busybox容器里的分层文件里，果然有这依赖库的👇

![image-20240411145758413](7-Docker镜像导出导入.assets/image-20240411145758413.png)

只要把这个依赖复制到宿主的/lib下应该就可以了，因为要在宿主用，宿主找的这种so文件还是会从/lib下去找吧

![image-20240411150232990](7-Docker镜像导出导入.assets/image-20240411150232990.png)

将依赖复制到宿主的/lib/或/lib64/下，记得退出一下，否则还是识别不到

![image-20240411150418943](7-Docker镜像导出导入.assets/image-20240411150418943.png)

可见busybox --list 得到具体的指令

![image-20240411150822717](7-Docker镜像导出导入.assets/image-20240411150822717.png)

我用xargs 做了横向排列，看到了ping

于是busybox用ping

![image-20240411150908557](7-Docker镜像导出导入.assets/image-20240411150908557.png)

优秀👆

![image-20240411153150965](7-Docker镜像导出导入.assets/image-20240411153150965.png)

very good

![image-20240411153922548](7-Docker镜像导出导入.assets/image-20240411153922548.png)

👆上图可见busy的大小是极简的，不仅仅cli少，而且cli里的某个命令的功能也是少很多。但是也够用了👇

![image-20240411154645001](7-Docker镜像导出导入.assets/image-20240411154645001.png)



### 补充：打tag的

![image-20240411161318249](7-Docker镜像导出导入.assets/image-20240411161318249.png)

下载的时候也要指定使用哪里的镜像，否则就是从deamon.json配置的仓库下载，什么没有json文件，那就是默认的官网了。

![image-20240415093442286](7-Docker镜像导出导入.assets/image-20240415093442286.png)



上图👆是tag打了要上传到服务器的，所以要这么打，到时候，push的时候也就是推送到对应的服务器和目录的。

如果是本地用，就只写个镜像名和tag就行了

![image-20240411162141986](7-Docker镜像导出导入.assets/image-20240411162141986.png)



总结

1、docker安装，还是要先把仓库配置好的，具体操作很简单，就是

https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/



2、离线安装，见<img src="7-Docker镜像导出导入.assets/image-20240411164931857.png" alt="image-20240411164931857" style="zoom:50%;" />

离线安装也分官方脚本和非官方的，哈哈

https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/



3、小结下

![image-20240411173615058](7-Docker镜像导出导入.assets/image-20240411173615058.png)

