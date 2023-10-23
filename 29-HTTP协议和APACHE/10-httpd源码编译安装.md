# 第10节. httpd源码编译安装





![image-20231020112831574](10-httpd源码编译安装.assets/image-20231020112831574.png)

确实有：https://httpd.apache.org/test/flood/

但是搜不到👇

![image-20231020112804497](10-httpd源码编译安装.assets/image-20231020112804497.png)







APR是统一的调用OS的接口，便于开发。而apache也就是httpd软件就是基于APR开发的。

<img src="10-httpd源码编译安装.assets/image-20231020095437175.png" alt="image-20231020095437175" style="zoom:33%;" />

随着开发项目的越来越多，就需要APR这种接口、工具、或者引擎之类的专项小组，这样规模起来了，就需要统一接口开发小组来节省整体的开发成本。



其实软件是否基于什么软件通常都可以查的👇

<img src="10-httpd源码编译安装.assets/image-20231020100022104.png" alt="image-20231020100022104" style="zoom:40%;" />



讲这个就是要明白，编译安装的时候，要注意apache(httpd)的版本，和APR的版本时候适配。



一般yum安装的版本都相对较旧一点

![image-20231020103640017](10-httpd源码编译安装.assets/image-20231020103640017.png)

官方有新的且稳定的版本，也不是就要安装最新的，要安装稳定的。判断依据有①该版本发布时间通常晚于最新1年？②和最新的隔1-2个版本。

![image-20231020103620377](10-httpd源码编译安装.assets/image-20231020103620377.png)







![image-20231020105651342](10-httpd源码编译安装.assets/image-20231020105651342.png)

理论上要先编译安装较新的apr和apr-util软件，再编译安装httpd的较新版本。

也可以把三个软件的源代码拷在一起，一起编译就行了。





### 方法一：逐个编译，这样就要注意依赖关系

![image-20231020132054259](10-httpd源码编译安装.assets/image-20231020132054259.png)

比如👆apr-util依赖于apr，所以--with要知名apr的安装路径。

![image-20231020132251445](10-httpd源码编译安装.assets/image-20231020132251445.png)

--with-apr和--with-apr-util就是依赖包的指定，这些依赖都是已经安装好的。



### 方法二：一起编译

![image-20231020132819890](10-httpd源码编译安装.assets/image-20231020132819890.png)

--with-included-apr就表示httpd里的源码包里已经有了par的源码包了。

![image-20231020133556285](10-httpd源码编译安装.assets/image-20231020133556285.png)



下面实验快开始，用rockyliux9.2弄得，思路还是和上面一样

**1、找源码包**

https://httpd.apache.org/

![image-20231020140019610](10-httpd源码编译安装.assets/image-20231020140019610.png)



**2、找APR包**

https://apr.apache.org/

![image-20231020135548189](10-httpd源码编译安装.assets/image-20231020135548189.png)

好像没有看到各个apache版本和apr的一个对应表，但是有推荐版本，所以猜测用这些就行了。也许人家软件向下兼容的。



**3、下载**

![image-20231020140544384](10-httpd源码编译安装.assets/image-20231020140544384.png)



**4、解压**

安装bzip2

```
yum -y install bzip2
```

tar解压，通过history查看和awk过滤的比较优的方法如下： 

awk的思路就是：第二列是tar并且不包含字符*，然后去掉第一列序列号啦，再去掉行首行尾的空格。over~👇

```shell
[root@server httpd_new]# history  |awk '$2=="tar" && $0!~"*" {$1="";print $0}' |awk '$1=$1' |sort |uniq
tar xvf apr-1.7.4.tar.bz2
tar xvf apr-util-1.6.3.tar.bz2 > /dev/null
tar xvf httpd-2.4.58.tar.bz2

```

取history里的需要cli的方法①：

![image-20231020153518135](10-httpd源码编译安装.assets/image-20231020153518135.png)

![image-20231020153600507](10-httpd源码编译安装.assets/image-20231020153600507.png)

![image-20231020153802972](10-httpd源码编译安装.assets/image-20231020153802972.png)

方法②：

![image-20231020155322009](10-httpd源码编译安装.assets/image-20231020155322009.png)

合并一下cli

![image-20231020155851081](10-httpd源码编译安装.assets/image-20231020155851081.png)



![image-20231020160318585](10-httpd源码编译安装.assets/image-20231020160318585.png)



![image-20231020162217358](10-httpd源码编译安装.assets/image-20231020162217358.png)

![image-20231020162241467](10-httpd源码编译安装.assets/image-20231020162241467.png)

必要时可以去掉行尾



![image-20231020170512504](10-httpd源码编译安装.assets/image-20231020170512504.png)



优化下

![image-20231020172107417](10-httpd源码编译安装.assets/image-20231020172107417.png)

![image-20231020172333361](10-httpd源码编译安装.assets/image-20231020172333361.png)

将两个awk合并一下



得到最屌的cli

![image-20231020174856311](10-httpd源码编译安装.assets/image-20231020174856311.png)

再来一个更吊的cli

![image-20231020175156022](10-httpd源码编译安装.assets/image-20231020175156022.png)



妈的我在干嘛...... 以上就得到了之运行的几个tar命令，(￣▽￣)"

![image-20231020175643614](10-httpd源码编译安装.assets/image-20231020175643614.png)



**5、合并**

把apr和apr-util的东西复制到httpd源码指定目录下。

```
cp -a apr-1.7.4 httpd-2.4.58/srclib/apr
cp -a apr-util-1.6.3 httpd-2.4.58/srclib/apr-util
```

![image-20231020182249573](10-httpd源码编译安装.assets/image-20231020182249573.png)



**6、安装一些常规依赖包**

```shell
yum -y install gcc pcre-devel openssl-devel expat-devel; yum -y groupinstall  "Development Tools"

```



**7、编译安装**

```shell
cd httpd-2.4.58

./configure \
--prefix=/app/httpd24 \
--enable-so \
--enable-ssl \
--enable-cgi \
--enable-rewrite \
--with-zlib \
--with-pcre \
--with-included-apr \
--enable-modules=most \
--enable-mpms-shared=all \
--with-mpm=prefork

make && make install
```

![image-20231023095213226](10-httpd源码编译安装.assets/image-20231023095213226.png)

上图是找到cpu核数，然后并发编译的



编译的过程也会自动记录下来

![image-20231023134143573](10-httpd源码编译安装.assets/image-20231023134143573.png)





**8、进入编译指定安装路径下查看一下安装的情况，并添加PATH变量或做软连接**

```shell
cd /app/httpd24
echo 'PATH=/app/httpd24/bin:$PATH' > /etc/profile.d/httpd.sh
. /etc/profile.d/httpd.sh

或者用ln -s /app/httpd24/bin/apachectl /usr/local/sbin/
但是要注意如果当前已经安装了httpd其他版本，还需要去掉其bin文件调用优先。

但是整个bin文件夹 链过去是没有用的，必须是二进制文件在$PATH下不能子目录。
```

![image-20231023104123496](10-httpd源码编译安装.assets/image-20231023104123496.png)



----



![image-20231023095610899](10-httpd源码编译安装.assets/image-20231023095610899.png)

awk的继续优化之去除前几列和空格的方法

![image-20231023101519099](10-httpd源码编译安装.assets/image-20231023101519099.png)



![image-20231023103302186](10-httpd源码编译安装.assets/image-20231023103302186.png)

**①软链接是文件还好👇**

![image-20231023111159610](10-httpd源码编译安装.assets/image-20231023111159610.png)

**②软链接是文件夹就一定要小心了**

![image-20231023111743987](10-httpd源码编译安装.assets/image-20231023111743987.png)



**9、启动服务**

```
apachectl start
```

![image-20231023133122746](10-httpd源码编译安装.assets/image-20231023133122746.png)

![image-20231023132853457](10-httpd源码编译安装.assets/image-20231023132853457.png)

但是查看不行，

![image-20231023134319670](10-httpd源码编译安装.assets/image-20231023134319670.png)

通过修改systemctl的启动配置修正



👇这是yum安装的httpd的systemctl的配置，参考修改

![image-20231023134519725](10-httpd源码编译安装.assets/image-20231023134519725.png)

修改之👇

![image-20231023141800153](10-httpd源码编译安装.assets/image-20231023141800153.png)

systemctl的配置修改后需要reload加载一下👉：systemctl daemon-reload，再启动服务

![image-20231023135846393](10-httpd源码编译安装.assets/image-20231023135846393.png)

死活起不来，这里不太好排查，但是ps aux可以看到

![image-20231023140013638](10-httpd源码编译安装.assets/image-20231023140013638.png)

找到个这个，用的是daemon用户，而不是apache这个用户，不过也没关系，①有该daemon用户②httpd可以使用该用户执行



当服务启动的时候

![image-20231023141221943](10-httpd源码编译安装.assets/image-20231023141221943.png)

status显示超时

![image-20231023141416575](10-httpd源码编译安装.assets/image-20231023141416575.png)

继续排查通过错误日志可见，

![image-20231023141236029](10-httpd源码编译安装.assets/image-20231023141236029.png)

可以看到  caught SIGWINCH，shutting down gracefully的情况。



改回apachectl start发现能起来，就是ss -tlnup 等3s就能看到80端口打开了，说明使用这种方式启动服务没问题。

然后去ps auxf看看

![image-20231023144451421](10-httpd源码编译安装.assets/image-20231023144451421.png)



尝试将systemctl配置文件里的启动cli改成上图的试试

还是不行，算了换种方式吧，systemctl 有问题实在不行就换成脚本的方式👇

![image-20231023150641197](10-httpd源码编译安装.assets/image-20231023150641197.png)



![image-20231023150934872](10-httpd源码编译安装.assets/image-20231023150934872.png)

![image-20231023150944856](10-httpd源码编译安装.assets/image-20231023150944856.png)

