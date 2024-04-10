# 第6节 Docker镜像搜索和下载



# 镜像管理



比如从网上下载了基础镜像，然后在上面封装了一些网络检测工具，

<img src="6-Docker镜像搜索和下载.assets/image-20240408093932755.png" alt="image-20240408093932755" style="zoom:50%;" />



然后继续封装jdk，tomcat，最后跑容器就是又添加了一层可写层，而可写容器层往下都是镜像层是只读的。

<img src="6-Docker镜像搜索和下载.assets/image-20240408094218199.png" alt="image-20240408094218199" style="zoom:50%;" />



未来打镜像就是采用这种逻辑不断地打，最终看到的镜像其实是多个镜像累积起来的结果。有点像mysql的增量备份，每次增量备份一点点，最终累加起来的才是完整的数据。



分层构建，将来每一部分 都可以独立抽取出来，用来构建其他镜像，也就是说镜像层是可以复用的，正以为分层所以复用起来很丝滑。



根镜像、父镜像、子镜像、子子镜像，大概有人会这么称呼

<img src="6-Docker镜像搜索和下载.assets/image-20240408095550089.png" alt="image-20240408095550089" style="zoom:50%;" />



![image-20240408095725657](6-Docker镜像搜索和下载.assets/image-20240408095725657.png)



这个每一层的将来复用，不知道是不是联合文件系统 自己去复用啊，我感觉应该是docker自身能够复用才行啊。而不是用户手动去复用，用户没办法说去复用哪一层吧，没这个操作的空间啊。



<img src="6-Docker镜像搜索和下载.assets/image-20240408100554285.png" alt="image-20240408100554285" style="zoom:50%;" />





通过docker image history nginx查看人家是怎么构建每一层的

![image-20240408101235237](6-Docker镜像搜索和下载.assets/image-20240408101235237.png)



镜像有了，创建好了，拉好了，落在本地其实都是表现为一个个文件夹的

![image-20240408101919729](6-Docker镜像搜索和下载.assets/image-20240408101919729.png)



最大的文件夹就是overlay2，里面的东西其实就是很多层，overlay就是层的意思，是所有images的层级铺开来的，才能共用，复用。这就是所谓的联合文件系统里的overlay2这种。

![image-20240408134242272](6-Docker镜像搜索和下载.assets/image-20240408134242272.png)

删除容器后

![image-20240408135105604](6-Docker镜像搜索和下载.assets/image-20240408135105604.png)

删除容器后，发现这些分层的东西就少了很多👇

![image-20240408135127082](6-Docker镜像搜索和下载.assets/image-20240408135127082.png)

<img src="6-Docker镜像搜索和下载.assets/image-20240408141122357.png" alt="image-20240408141122357" style="zoom:50%;" />

docker rm删除容器和镜像👆

进一步再删除镜像，后发现overlay2里就空了

![image-20240408141639847](6-Docker镜像搜索和下载.assets/image-20240408141639847.png)

也就是说容器和镜像删光了，overlay2文件夹里就空了，现在pull一个nginx看看

![image-20240408142343942](6-Docker镜像搜索和下载.assets/image-20240408142343942.png)

overlay2文件夹里的就是分层的数据了👆。一个文件夹就是一层但我没仔细数哦，哈哈，大概如此吧



# 镜像去哪找

1、hub.docker.com

要用官方的镜像

![image-20240408150112585](6-Docker镜像搜索和下载.assets/image-20240408150112585.png)

![image-20240408150128977](6-Docker镜像搜索和下载.assets/image-20240408150128977.png)



2、cli，命令行搜索的方式不全，没有上面去网站搜索的方式好，就是去hub.docker.com还有其他的比如谷歌的站点。

![image-20240408150236517](6-Docker镜像搜索和下载.assets/image-20240408150236517.png)

自己做的镜像也可以上传上去，只是别人一般是不敢用的。

还有比如k8s的一些镜像，k8s就是谷歌开发的，对于谷歌来讲，docker公司肯定没有他们家的大，所以谷歌的镜像也就没有放到docker的仓库里，而是谷歌自己的仓库里的。

比如下图的这些很多就是谷歌的，但是docker上也有，就是别人上传的方便下载的

![image-20240408151447682](6-Docker镜像搜索和下载.assets/image-20240408151447682.png)



3、代理的配置方法

其实就是科学上网咯

<img src="6-Docker镜像搜索和下载.assets/image-20240408152824946.png" alt="image-20240408152824946" style="zoom:55%;" />

上图就是配置你的代理机器，走代理的意思，可惜是错误的👆实测这样配置服务都起不来，其实你完全可以做在路由器上，就不用管这里的怎么配置。

不过这里的配置方式也要知道，方法越多，应用起来就越灵活。



设置配置文件里的配置，还有一个脚本是修改系统层面的代理，同样也会被docker读进去的👇

<img src="6-Docker镜像搜索和下载.assets/image-20240408154822122.png" alt="image-20240408154822122" style="zoom:50%;" />



以上是部分咯，修改代理的方法，三选一就行了👇参考https://blog.csdn.net/peng2hui1314/article/details/124267333

总结如下：

1、网络里的**路由器层面**直接做路由，略



2、**主机层面**，的系统代理

![image-20240408161503948](6-Docker镜像搜索和下载.assets/image-20240408161503948.png)



3、**docker服务层面**的代理，就是上面的脚本，其实也就是服务文件里的三行内容，其他都是配套语句呵呵👇

![image-20240408162836631](6-Docker镜像搜索和下载.assets/image-20240408162836631.png)



**4、容器里面，**

![image-20240408161945908](6-Docker镜像搜索和下载.assets/image-20240408161945908.png)



5、可惜daemon.json这个文件的配置我没有找到上图图示的配置成功的案例，也许是老版本的配置，类似hosts了吧，不管了。   不过容器里的配置倒是和第3点-容器里的配置一样的，哈哈。





# 镜像制作

比如java程序需要一个镜像，要做java镜像，

然后java又依靠JDK，又要做JDK层对吧

然后JDK又依赖linux，又要做linux层，

一般来讲，是系统镜像就从官方拉取(系统镜像前面的文章也解释过了，其实是没有内核的用户空间的部分)，而业务镜像才是自己制作。

![image-20240408164100095](6-Docker镜像搜索和下载.assets/image-20240408164100095.png)



这就回答了下图👇的问题，用户空间里是有os的，只有os的一部分，然后再结合下层宿主的os的内核 就能构成完整的os了。以此来提供服务的。

![image-20240408164330074](6-Docker镜像搜索和下载.assets/image-20240408164330074.png)







然后docker制作镜像的时候，底层不是要linux嘛，一般也不会选择ubuntu，centos、rocky、redhat都不会，因为都太大了，压缩后还要25MB

![image-20240408164809117](6-Docker镜像搜索和下载.assets/image-20240408164809117.png)



![image-20240408164832512](6-Docker镜像搜索和下载.assets/image-20240408164832512.png)



都太大了，所以会选择一些积极精简的linxu，比如aphine或busybox👇

![image-20240408165011412](6-Docker镜像搜索和下载.assets/image-20240408165011412.png)





![image-20240408165111182](6-Docker镜像搜索和下载.assets/image-20240408165111182.png)



alpine就是基于busybox开发的



## 生产中用的比较多的alpine

1、包安装工具

2、alpine的仓库配置

3、用法也不一样



ubuntu的仓库文件

![image-20240408173623224](6-Docker镜像搜索和下载.assets/image-20240408173623224.png)



alpine是不同的

<img src="6-Docker镜像搜索和下载.assets/image-20240408174130320.png" alt="image-20240408174130320" style="zoom:67%;" />

ustc是中科大的



alpine也是有完全镜像和极简镜像的，好比centos的all-in-one和mini一样👇

![image-20240408181837435](6-Docker镜像搜索和下载.assets/image-20240408181837435.png)





![image-20240408183923431](6-Docker镜像搜索和下载.assets/image-20240408183923431.png)





指定版本下载，要去网站看看版本的格式

![image-20240409132940645](6-Docker镜像搜索和下载.assets/image-20240409132940645.png)

然后再复制，粘贴就行了

![image-20240409133022555](6-Docker镜像搜索和下载.assets/image-20240409133022555.png)

然后网站上看到的镜像是压缩后的，pull下来会变大就是解压了应该👆



### cli 用法  只显示镜像id：docker images -q   # 操作镜像靠ID是不好识别

<img src="6-Docker镜像搜索和下载.assets/image-20240409133914202.png" alt="image-20240409133914202" style="zoom:50%;" /> 



批量删除就很方便 xargs -i 就行了；或者直接docker rmi \`docker images -q\`

![image-20240409135506997](6-Docker镜像搜索和下载.assets/image-20240409135506997.png)





### cli用法，显示镜像和tag： docker image ls --format   # 操作镜像靠名称和tag，也可以识别

![image-20240409134322669](6-Docker镜像搜索和下载.assets/image-20240409134322669.png)



![image-20240409135240371](6-Docker镜像搜索和下载.assets/image-20240409135240371.png)

删除也要加上tag的，否则就是删除就是删除latest这个tag

<img src="6-Docker镜像搜索和下载.assets/image-20240409135320973.png" alt="image-20240409135320973" style="zoom:50%;" /> 





### 容器跑着的时候删除image，不会真的删掉



![image-20240409140224451](6-Docker镜像搜索和下载.assets/image-20240409140224451.png)

被占用的镜像如果使用docker rmi -f 'IMAGE ID'  这种方式 delete 肯定和写 image name tag不会delete，而且 连untag动作都不会执行。



业务也ok

![image-20240409140236335](6-Docker镜像搜索和下载.assets/image-20240409140236335.png)

此时即使停掉容器，删掉容器，那个镜像也不会自动删掉，就成为了一个无名的镜像，也叫dangling镜像

![image-20240409140418121](6-Docker镜像搜索和下载.assets/image-20240409140418121.png)

只查看这种dangling镜像的方法，docker images -f 就是filter过滤出来哪些特征的images

![image-20240409141939861](6-Docker镜像搜索和下载.assets/image-20240409141939861.png)

删除这种就这样docker rmi -f $(docker images -f dangling=true -q)     # 就行啦

或者docker images -f dangling=true -q |xargs docker rmi -f     # 也行

![image-20240409143211992](6-Docker镜像搜索和下载.assets/image-20240409143211992.png)

不过要停掉占用该镜像的容器，否则还是会删不掉👆



### 系统中有一个清理的命令docker system 

![image-20240409140817817](6-Docker镜像搜索和下载.assets/image-20240409140817817.png)

这样不仅仅是没有用的images，其他的停掉的容器和没用的网络，缓存都清了

![image-20240409140931115](6-Docker镜像搜索和下载.assets/image-20240409140931115.png)



![image-20240409141113725](6-Docker镜像搜索和下载.assets/image-20240409141113725.png)

有些容器停了，但是不是说就可以删的，你就不能用这个命令了！所以这个prune还是慎用！





### inpsect是通用型命令

![image-20240409164325795](6-Docker镜像搜索和下载.assets/image-20240409164325795.png)



![image-20240409165437840](6-Docker镜像搜索和下载.assets/image-20240409165437840.png)

上图的问题就是，images里看到的container应该就是当初这个镜像是通过容器commit出来的，所以会带上容器字眼👇

<img src="6-Docker镜像搜索和下载.assets/image-20240409172720830.png" alt="image-20240409172720830" style="zoom:45%;" />





然后详情如下

```shell
[root@nginxproxy ~]# docker inspect 05455a08881e
[
    {
        "Id": "sha256:05455a08881ea9cf0e752bc48e61bbd71a34c029bb13df01e40e3e70e0d007bd",
        // 镜像的唯一标识符。两个镜像的内容一样(也就是RepoDisgests一样)，但是构建时间不同或者名称tag不同，id也是不同的。

        "RepoTags": [
            "alpine:latest"
        ],
        // 镜像的标签，这里表示这是 alpine 镜像的最新版本。

        "RepoDigests": [
            "alpine@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b"
        ],
        // 镜像的摘要信息，用于验证镜像的完整性。

        "Parent": "",
        // 父镜像的ID，为空表示此镜像没有父镜像。

        "Comment": "",
        // 镜像的注释。

        "Created": "2024-01-27T00:30:48.743965523Z",
        // 镜像的创建时间。

        "Container": "4189cbc534955765760c227f328ec1cdd52e8550681c2bf9f8f990b27b644f9c",
        // 用于创建此镜像的容器的ID。看来很多都是从容器直接commit提交出来的镜像，而不是docker build出来的咯，可能是~

        "ContainerConfig": {
            // 创建该镜像的容器的配置信息。
            "Hostname": "4189cbc53495",
            // 容器的主机名。
            "Domainname": "",
            // 容器的域名。
            "User": "",
            // 容器内命令运行的用户。
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            // 这些设置控制是否将stdin、stdout、stderr附加到容器。
            "Tty": false,
            // 是否为容器分配一个tty设备。
            "OpenStdin": false,
            "StdinOnce": false,
            // 这些设置控制容器的stdin。
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            // 容器的环境变量。
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/sh\"]"
            ],
            // 容器的默认命令和参数。
            "Image": "sha256:9a5ce069f40cfe0f2270eafbff0a0f2fa08f1add73571af9f78209e96bb8a5e9",
            // 创建容器时使用的镜像ID。
            "Volumes": null,
            // 容器使用的卷。
            "WorkingDir": "",
            // 容器的工作目录。
            "Entrypoint": null,
            // 容器的入口点。
            "OnBuild": null,
            // Dockerfile中的ONBUILD触发器指令。
            "Labels": {}
            // 容器的标签。
        },

        "DockerVersion": "20.10.23",
        // 创建镜像时使用的Docker版本。

        "Author": "",
        // 镜像的作者。

        "Config": {
            // 镜像配置，类似于ContainerConfig，但用于运行时。
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh"
            ],
            // 容器启动时执行的命令。
            "Image": "sha256:9a5ce069f40cfe0f2270eafbff0a0f2fa08f1add73571af9f78209e96bb8a5e9",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },

        "Architecture": "amd64",
        // 镜像的架构。

        "Os": "linux",
        // 镜像的操作系统。

        "Size": 7377074,
        // 镜像的大小（字节）。

        "VirtualSize": 7377074,
        // 镜像的虚拟大小。

        "GraphDriver": {
            // 镜像的存储驱动信息。
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/bb16c3d711597eff0baab5640f474ef4c8c1c40df0351e0a21343e1362676504/merged",
                "UpperDir": "/var/lib/docker/overlay2/bb16c3d711597eff0baab5640f474ef4c8c1c40df0351e0a21343e1362676504/diff",
                "WorkDir": "/var/lib/docker/overlay2/bb16c3d711597eff0baab5640f474ef4c8c1c40df0351e0a21343e1362676504/work"
            },
            "Name": "overlay2"
        },

        "RootFS": {
            // 镜像的根文件系统信息。
            "Type": "layers",
            "Layers": [
                "sha256:d4fc045c9e3a848011de66f34b81f052d4f2c15a17bb196d637e526349601820"
            ]
        },

        "Metadata": {
            // 镜像的元数据。
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]

```

