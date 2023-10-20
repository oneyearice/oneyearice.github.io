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

