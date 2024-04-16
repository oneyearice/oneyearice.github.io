# 第2节 Docker容器查看常见用法





# 容器里的root

docker 运行的时候，在容器里面，通常是以root运行的。

![image-20240416144445946](2-Docker容器查看常见用法.assets/image-20240416144445946.png)



![image-20240416145359646](2-Docker容器查看常见用法.assets/image-20240416145359646.png)

容器本身的文件还是分层落在宿主机上的overlay2目录里的👇，至于为什么是2个m53.txt这个后面再说，涉及分层结构的梳理。

![image-20240416145535817](2-Docker容器查看常见用法.assets/image-20240416145535817.png)

该文件的所有者也是root

![image-20240416145656551](2-Docker容器查看常见用法.assets/image-20240416145656551.png)

就是容器里的m53.txt所有者是root，容器外面宿主机上看到的m53.txt所有者也是root。

是一个root嘛？容器里的root是用的宿主机上的root嘛？并不是，容器里的root是一个假root就是一个普通用户。证明👇。

![image-20240416152328302](2-Docker容器查看常见用法.assets/image-20240416152328302.png)

而宿主机是可以的👇，所以证明了容器里的root并不是传统意义的root超级用户。

![image-20240416152425027](2-Docker容器查看常见用法.assets/image-20240416152425027.png)

df看不到挂载覆盖的效果，lsblk可以

![image-20240416155821756](2-Docker容器查看常见用法.assets/image-20240416155821756.png)

顺便知道下dm0 dm1这些

![image-20240416160008963](2-Docker容器查看常见用法.assets/image-20240416160008963.png)

所以dm-0  dm-1  这种就是lvm了，逻辑块。



## privileged--容器里使用宿主机的root，除非特地这么要求，否则别加该选项。

![image-20240416154939569](2-Docker容器查看常见用法.assets/image-20240416154939569.png)

索然/dev/mapper下面没有lvm的逻辑块，但是要知道这个目录下面其实就是软连接

真正的rl-root其实是连接到dm-0去的

这一点，在宿主机上可以看到👇

![image-20240416160438846](2-Docker容器查看常见用法.assets/image-20240416160438846.png)

所以容器里直接mount /dev/dm-0就行了

![image-20240416160522694](2-Docker容器查看常见用法.assets/image-20240416160522694.png)



![image-20240416160638530](2-Docker容器查看常见用法.assets/image-20240416160638530.png)



然后

![image-20240416160804794](2-Docker容器查看常见用法.assets/image-20240416160804794.png)

此时再到宿主上直接就看到了

![image-20240416160818508](2-Docker容器查看常见用法.assets/image-20240416160818508.png)



**这样容器里的某些操作就危险了，比如**

![image-20240416161903877](2-Docker容器查看常见用法.assets/image-20240416161903877.png)

此时宿主上的echo就没了

![image-20240416162022527](2-Docker容器查看常见用法.assets/image-20240416162022527.png)

但是发现echo还能用，

![image-20240416162240919](2-Docker容器查看常见用法.assets/image-20240416162240919.png)

忽然反应过来，type 一下看到了builtin，说明什么，说明echo是buildin在/bin/bash里的，是内部命令。

**好，mv echo看不到效果，mv /bin/bash吧哈哈**

![image-20240416162524895](2-Docker容器查看常见用法.assets/image-20240416162524895.png)

直接GG👇

![image-20240416162513703](2-Docker容器查看常见用法.assets/image-20240416162513703.png)

所以除非生产中有明确要求容器里可以操作宿主机的权限，否则别加该--privilieged选项。



### 那么如何知道run的时候加了那些选项，比如是否加了危险的provilieged

![image-20240416163844094](2-Docker容器查看常见用法.assets/image-20240416163844094.png)

看全部的run的时候加的选项

![image-20240416164100328](2-Docker容器查看常见用法.assets/image-20240416164100328.png)

不过还不是太明显，不是tmd一目了然，使用一个第三方工具runlike来一目了然。

![image-20240416170139668](2-Docker容器查看常见用法.assets/image-20240416170139668.png)



![image-20240416170554324](2-Docker容器查看常见用法.assets/image-20240416170554324.png)



![image-20240416172309411](2-Docker容器查看常见用法.assets/image-20240416172309411.png)



-p就是--pretty 更清晰点排版

![image-20240416172340168](2-Docker容器查看常见用法.assets/image-20240416172340168.png)





# 查看容器信息



前面一直在用ps images这些，

这里再补充下-f选项的过滤



https://docs.docker.com/reference/cli/docker/container/ls/

下图链接有问题，纠正为👆

然后支持--filter选项的又有很多dockers cli  ：  https://docs.docker.com/config/filter/



![image-20240416173604368](2-Docker容器查看常见用法.assets/image-20240416173604368.png)



![image-20240416181800216](2-Docker容器查看常见用法.assets/image-20240416181800216.png)

-f等价于--filter，就是短选项和长选项的意思，好比-h ＝ --help

然后--format就是不想其他的，它只有短选项。

![image-20240416182514073](2-Docker容器查看常见用法.assets/image-20240416182514073.png)



### 推荐的cli



**删除所有退出的容器**

```shell
docker ps -f 'status=exited' -q  # 退出容器的编号，删除👇起来方便
docker rm $(docker ps -f 'status=exited' -q)
```



**查看容器空间占用**

容器分层，image共用，所以括号里的是image大小，容器自身的可写层，一般是一些log，占用501B

![image-20240416183600605](2-Docker容器查看常见用法.assets/image-20240416183600605.png)

![image-20240416184059894](2-Docker容器查看常见用法.assets/image-20240416184059894.png)



