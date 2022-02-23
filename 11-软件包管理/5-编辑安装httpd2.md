# 第5节. 编辑安装httpd2



## 理解源码安装

![img](5-编辑安装httpd2.assets/clip_image002.jpg)

很多较新的版本，光盘里是没有的，这就需要自行到官网下载源码进行安装（官网提供的软件通常就是源码的）。

 ![image-20220215152328491](5-编辑安装httpd2.assets/image-20220215152328491.png)



源码理论上是通过gcc从源代码-->预处理-->编译-->汇编-->链接--最终生成二进制然后执行起来。

但中间过程源码多、不仅仅源代码 还有各种文档 配置文件 说明等，这些文件你放在哪 手动拷贝工作量巨大，所以针对发布的源码 都会有一个官方的 源码编译的项目管理工具 + 配合脚本。



C一般是make这个项目管理器，把软件作为项目统筹管理。利用make里的项目管理功能便捷的部署源码。大概步骤有3：

①创建makefile文件

<img src="5-编辑安装httpd2.assets/image-20220215152942503.png" alt="image-20220215152942503" style="zoom:67%;" />

​		这个makefile就定义一些必要信息：各种文件的安装路径（二进制文件放在哪里、配置文件放在哪里、man帮助等分门别类 也可以不分同一个一个总目录放进去 各有各的子目录，这点和yum截然不同了，yum安装时自动给你安排好了，而源码得自己指定安装路径）

​		为什么要源码编译呢，其中一个原因就是，功能定制，比如源码内置100个功能，如果你是yum或rpm安装，yum的底层也是rpm，官方给你编译完了，官方觉得50个功能是常用的，就给你把这个50个功能编译在了rpm包里，剩下50功能就用不了了。或者生产环境场景单一无需已启用的50个功能这么多，所以就需要定制化。  ----所以就需要把哪些特性功能 放到Makefile里。

​		**所以makefile尤其2点：安装路径、启用特性。**所以configure 后面启用功能的参数就可能很长，cli就会很长了。

​		Makefile手动写不现实，也是使用工具configure脚本  借助Makefile.in(Makefile模板文件)来生成。

②利用C语言的项目管理工具make 自动读取Makefile的内容，进行后续工作（编译）

③利用make install 将编译后的文件复制到指定的路径，就是安装动作。

![img](5-编辑安装httpd2.assets/clip_image004.jpg)

 然后configure脚本也是通过autocnf生成的，Makefile.in这个模板也是通过automake生成的。这里不用管，源码下载下来configure脚本和makefile.in模板都是自带的，开发人员做好的。

./configure要进到当前目录运行的，不然可能人家的代码里写的都是相对路径，会存在问题的。

还有一点./configure的时候会检查依赖，所以会报各种错误，然后工程师就更加错误补上对应的包，然后再回过来继续./configure。



上述的./configure , make , make install  三大步是通用的，但是每个软件各有特点，所以还需要参考install 和readme里的说明。通常源码解压缩后也会有这两个文件。有时候把install里的东西拿出来复制粘贴就直接执行就行了。



所以源码安装就是上面那回事，easy~，源码安装说的是安装，又不是让你写源码，哈哈，难不成让你跨行理解还是执行吗，no way~。不要有畏难情绪，不要有什么都敢尝试的莽撞。

 

![img](5-编辑安装httpd2.assets/clip_image006.jpg)

 

 

## 源码安装tree

![img](5-编辑安装httpd2.assets/clip_image008.jpg)

上图👆可见tree这个工具-ql就是安装的所有文件了，除了/usr/bin/tree其他都是文档了，不重要。

很多软件都是这个cli格式xx -V   yyy --version这种查看自身版本的方式。

![img](5-编辑安装httpd2.assets/clip_image010.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image012.jpg)

右键得到下载链接后使用wget下载

 

![img](5-编辑安装httpd2.assets/clip_image014.jpg)

![img](5-编辑安装httpd2.assets/clip_image016.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image018.jpg)

 REAME里没啥东西，看看INSTALLL

![image-20220215165322580](5-编辑安装httpd2.assets/image-20220215165322580.png) 

上图👆说了它这个源码的Makefile是适合各种OS的，但是需要你改一改设置。

![img](5-编辑安装httpd2.assets/clip_image020.jpg)

CC=gcc，编译器是gcc

tree编译后的二进制文件放在/usr/bin下

MAN帮助放在了/usr/man/man1下

![img](5-编辑安装httpd2.assets/clip_image022.jpg)

默认是linux的。需要什么OS，就取消哪个模块的注释。

 

![img](5-编辑安装httpd2.assets/clip_image024.jpg)

![img](5-编辑安装httpd2.assets/clip_image026.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image028.jpg)

 源码安装的不要了，只能手工删除

看看install说明，就是常用的make

![img](5-编辑安装httpd2.assets/clip_image030.jpg)

![img](5-编辑安装httpd2.assets/clip_image032.jpg)

这和CPU核数有关。多线程

大量的文件才需要-j并行编译。

![img](5-编辑安装httpd2.assets/clip_image034.jpg)

其实这个时候已经就能用了

![img](5-编辑安装httpd2.assets/clip_image036.jpg)

![img](5-编辑安装httpd2.assets/clip_image038.jpg)

当前的tree 1.6和安装后的tree 1.8共存了，

![img](5-编辑安装httpd2.assets/clip_image040.jpg)

![img](5-编辑安装httpd2.assets/clip_image042.jpg)

修改PATH变量

你希望tree 1.8执行而不是1.6，所以就要把1.8的path路径放到/usr/bin的前面

![img](5-编辑安装httpd2.assets/clip_image044.jpg)

这是自己在这些特殊文件下创建的env.sh

![img](5-编辑安装httpd2.assets/clip_image046.jpg)

上图👆有个sb的操作，就是PATH='/apps/tree/bin:$PATH'，不信你试试，回头vi都用不了，只能/bin/vi编辑。

退出一下shell使之生效

![img](5-编辑安装httpd2.assets/clip_image048.jpg)

![img](5-编辑安装httpd2.assets/clip_image050.jpg)

![img](5-编辑安装httpd2.assets/clip_image052.jpg)

大软件和小软件

大软件是长期站着内存在系统中运行，小软件就是简单的各个选项用的时候就加载一下内存run一下就好了。

而大软件长期运行，就需要把一些设置存到一个配置文件中，下次再用的时候自动就加载配置文件里的一堆配置。

 

## 编译安装一个大点的软件httpd

![img](5-编辑安装httpd2.assets/clip_image054.jpg)

这是光盘自带的2.4.6👆

 

<img src="5-编辑安装httpd2.assets/clip_image056.jpg" alt="img" style="zoom:67%;" />

这是官网下的2.4.25（用这个简单点来感受下编译安装）。2.4.39难度较大。

 

1、tar 解包

![img](5-编辑安装httpd2.assets/clip_image058.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image060.jpg)

这次发现和tree不一样了，有绿色的configure脚本了

有这个configure脚本，就没有Makefile文件了，因为这玩意就是configure生成的。自然也就有Makefile.in文件了，in就是模板。

参考Makefile.in模板利用configure脚本来生成Makefile文件。

 

看看readme，也没啥用

![img](5-编辑安装httpd2.assets/clip_image062.jpg)

 

看看install

![img](5-编辑安装httpd2.assets/clip_image064.jpg)

![img](5-编辑安装httpd2.assets/clip_image066.jpg)

./configure --prefix=PREFIX就是跟了个路径，安装到什么地方。

![img](5-编辑安装httpd2.assets/clip_image068.jpg)

然后./config需要些指定一些配置：①安装路径指定好，②哪些特性启用哪些不启用。

![img](5-编辑安装httpd2.assets/clip_image070.jpg)

如上图不写就是默认安装在/usr/local/apache2

![img](5-编辑安装httpd2.assets/clip_image072.jpg)

![img](5-编辑安装httpd2.assets/clip_image074.jpg)

然后再看下特性的启用：

![img](5-编辑安装httpd2.assets/clip_image076.jpg)

enable 或 disable 特性

![img](5-编辑安装httpd2.assets/clip_image078.jpg)

 

###  然后开始执行./configure生成Makefile文件

![img](5-编辑安装httpd2.assets/clip_image080.jpg)

这就制定了安装的总文件夹，和配置文件的存放地点。而且这些文件夹是 不需要事先有的。

![img](5-编辑安装httpd2.assets/clip_image082.jpg)

需要enable说明默认是未启用的，需要disabled说明默认是启用的。这话对不对哦？⚪疑问

后缀很长建议这么放

![img](5-编辑安装httpd2.assets/clip_image084.jpg)

![img](5-编辑安装httpd2.assets/clip_image086.jpg)

我们编译工具之前已经安装了gcc，但是这些特性启用可能还需要依赖

 

好，下面开始一步步检查和拍错

![img](5-编辑安装httpd2.assets/clip_image088.jpg)

### 记住咯，一般提示XXX not found，就是需要XXX.devel这个包，所以需要的就是apr.devel。缺什么就加devel，通常对的。

![img](5-编辑安装httpd2.assets/clip_image090.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image091.png)

 

继续./configure

![img](5-编辑安装httpd2.assets/clip_image092.png)

![img](5-编辑安装httpd2.assets/clip_image093.png)

go on 。。。

![img](5-编辑安装httpd2.assets/clip_image095.jpg)

![img](5-编辑安装httpd2.assets/clip_image097.jpg)

继续./configure --prefx --xxx

![img](5-编辑安装httpd2.assets/clip_image099.jpg)

### 这次依据惯例，使用mode_ssl-devel是没有的

![img](5-编辑安装httpd2.assets/clip_image100.png)

 

 

![img](5-编辑安装httpd2.assets/clip_image102.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image104.jpg)

仔细观察上图，提示OpenSSL version is too old

![img](5-编辑安装httpd2.assets/clip_image106.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image107.png)

 

继续

![img](5-编辑安装httpd2.assets/clip_image109.jpg)

![img](5-编辑安装httpd2.assets/clip_image111.jpg)

MD，上图我还以为是打了个叉叉，几个月前写的了，现在梳理突然第一眼没看明白，哈哈。是成功两字哦呵呵~。

所以httpd的依赖包列表如下

![img](5-编辑安装httpd2.assets/clip_image113.jpg)

 上面就可以写脚本了~，部署就很快乐~

字词configure编译完了，就会生成Makefile文件

![img](5-编辑安装httpd2.assets/clip_image114.png)

 

![img](5-编辑安装httpd2.assets/clip_image116.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image118.jpg)

这个图的步骤在cat install里一样看得到

![img](5-编辑安装httpd2.assets/clip_image119.png)

### 然后make -j 4 可能是和CPU核数对应的。

![img](5-编辑安装httpd2.assets/clip_image121.jpg)

 

我们来复习编译的安装路径

![img](5-编辑安装httpd2.assets/clip_image122.jpg)

![img](5-编辑安装httpd2.assets/clip_image124.jpg)

### 最后一步makeinstall复制文件

![img](5-编辑安装httpd2.assets/clip_image125.png)

![image-20220215174354057](5-编辑安装httpd2.assets/image-20220215174354057.png) 主目录/apps/httpd24，配置文件/detc/httpd

![img](5-编辑安装httpd2.assets/clip_image127.jpg)

![img](5-编辑安装httpd2.assets/clip_image129.jpg)

### 这就安装好了，那么就要使用了

![img](5-编辑安装httpd2.assets/clip_image131.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image133.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image135.jpg)

 

每次写路径比较麻烦，所以还是写到PATH变量里去

![img](5-编辑安装httpd2.assets/clip_image137.jpg)

![img](5-编辑安装httpd2.assets/clip_image139.jpg)

![img](5-编辑安装httpd2.assets/clip_image141.jpg)

![img](5-编辑安装httpd2.assets/clip_image143.jpg)

 启动：

![img](5-编辑安装httpd2.assets/clip_image145.jpg)

![img](5-编辑安装httpd2.assets/clip_image147.jpg)

### 这种开机自启动的方法：它又不是sytemctl enable xxx

![img](5-编辑安装httpd2.assets/clip_image149.jpg)

![img](5-编辑安装httpd2.assets/clip_image150.png)

![img](5-编辑安装httpd2.assets/clip_image152.jpg)

这种大的软件且是持续运行的就是叫做服务。而不是一次运行的程序。

![img](5-编辑安装httpd2.assets/clip_image154.jpg)

![img](5-编辑安装httpd2.assets/clip_image156.jpg)

reboot后测试一下：

![img](5-编辑安装httpd2.assets/clip_image158.jpg)

 

### 以上过程可以写成编译安装的脚本。



## 下面再装个小软件-黑客帝国的既视感哈哈~

![img](5-编辑安装httpd2.assets/clip_image160.jpg)

 ll



![img](5-编辑安装httpd2.assets/clip_image162.jpg)

 看看 README，看看INSTALL

![img](5-编辑安装httpd2.assets/clip_image164.jpg)

![img](5-编辑安装httpd2.assets/clip_image166.jpg)

![img](5-编辑安装httpd2.assets/clip_image168.jpg)

可以通过./configure --help查看安装路径选项 和 特性启用选项

![img](5-编辑安装httpd2.assets/clip_image170.jpg)

![img](5-编辑安装httpd2.assets/clip_image171.png)

curses.h，是缺少curses的头

这个头由什么文件提供的呢？

![img](5-编辑安装httpd2.assets/clip_image173.jpg)

改为模糊搜索

![img](5-编辑安装httpd2.assets/clip_image175.jpg)

这个搜到的东西太多了，不仅仅是提供该关键字的包，还有文件，不是一目了然。

或者搜包名这种好一点：

![img](5-编辑安装httpd2.assets/clip_image177.jpg)

可见是叫ncourses

![img](5-编辑安装httpd2.assets/clip_image179.jpg)

![img](5-编辑安装httpd2.assets/clip_image181.jpg)

错误是courses.h安装晚了，应该是configure之前装好的。此时需要make clean一下。也可以把这个目录删了，重新tar xvf解压

![img](5-编辑安装httpd2.assets/clip_image183.jpg)

重走一遍configure

![img](5-编辑安装httpd2.assets/clip_image185.jpg)

再make👇：

![img](5-编辑安装httpd2.assets/clip_image187.jpg)

所以还是删除整个文件夹

![img](5-编辑安装httpd2.assets/clip_image188.png)

![img](5-编辑安装httpd2.assets/clip_image190.jpg)

![img](5-编辑安装httpd2.assets/clip_image192.jpg)

![img](5-编辑安装httpd2.assets/clip_image194.jpg)

![img](5-编辑安装httpd2.assets/clip_image195.png)

![img](5-编辑安装httpd2.assets/clip_image196.png)

 

![img](5-编辑安装httpd2.assets/clip_image198.jpg)

![img](5-编辑安装httpd2.assets/clip_image200.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image202.jpg)

![img](5-编辑安装httpd2.assets/clip_image204.jpg)

![img](5-编辑安装httpd2.assets/clip_image206.jpg)

 

 

![img](5-编辑安装httpd2.assets/clip_image208.jpg)

### 脚本写好后，放到网站上，这也是一个常见操作 

![img](5-编辑安装httpd2.assets/clip_image210.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image212.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image214.jpg)

要想执行往bash里一传就行了

![img](5-编辑安装httpd2.assets/clip_image216.jpg)

 

所以将来，你把一键安装脚本放到http网站里面，然后，随便找个机器这么一执行就本地安装成功了。

★这是个法宝啊

 

复习下

![img](5-编辑安装httpd2.assets/clip_image218.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image220.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image222.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image224.jpg)

 

 

##  Ubuntu软件管理

<img src="5-编辑安装httpd2.assets/clip_image226.jpg" alt="img" style="zoom:67%;" />

 

.deb类似.rpm包

dpkg 类似rpm安装

APT类似yum

 

![img](5-编辑安装httpd2.assets/clip_image228.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image229.png)

 du -sh *查看所有文件和文件夹大小

![img](5-编辑安装httpd2.assets/clip_image230.png)

 

![img](5-编辑安装httpd2.assets/clip_image232.jpg)

这就可以看到.deb后缀的包了

 dpkg -i xxxxx 安装即可。

![img](5-编辑安装httpd2.assets/clip_image234.jpg)

 

-l列出了所有安装的包类似rpm -qa

![img](5-编辑安装httpd2.assets/clip_image236.jpg)

 上图的这个大箭头挺抽象，几个月后也半天(5s)才看明白，

类似于rpm -ql

![image-20220215194447851](5-编辑安装httpd2.assets/image-20220215194447851.png) 



###  centos如果是minial最小安装，后来又想要GUI，于是可以安装一个包组GNOME。

![img](5-编辑安装httpd2.assets/clip_image237.png)

带空格和不带空格的区别：是grouplist[空格]不是group[空格]list

这个如果你最小安装tab不出来的哦。

![img](5-编辑安装httpd2.assets/clip_image238.png)

![img](5-编辑安装httpd2.assets/clip_image239.png)



### 16.04以后apt基本整合取代了之前的3个。

 

![img](5-编辑安装httpd2.assets/clip_image240.png)

 

![img](5-编辑安装httpd2.assets/clip_image242.jpg)

![img](5-编辑安装httpd2.assets/clip_image244.jpg)

![img](5-编辑安装httpd2.assets/clip_image245.png)

 

但是这个虽然是CN的网站，但是还是不快，所以很多人都替换成阿里的。

### 类似centos的GNOME的ubuntu里叫ubuntu-destop

![img](5-编辑安装httpd2.assets/clip_image247.jpg)

![img](5-编辑安装httpd2.assets/clip_image249.jpg)

 

![img](5-编辑安装httpd2.assets/clip_image251.jpg)

 

 

![img](5-编辑安装httpd2.assets/clip_image253.jpg)

![img](5-编辑安装httpd2.assets/clip_image255.jpg)

![image-20220215200653774](5-编辑安装httpd2.assets/image-20220215200653774.png)

yum list installed也能看，但是不知道是哪个

![image-20220215200754747](5-编辑安装httpd2.assets/image-20220215200754747.png)



![image-20220215200348085](5-编辑安装httpd2.assets/image-20220215200348085.png)





友情提示，yum 安装的过程种或者脚本执行的过程中，等等如下图👇

![image-20220215200936617](5-编辑安装httpd2.assets/image-20220215200936617.png)

不要敲任何键盘，否则后果比较危险，证明👇

![image-20220215201023088](5-编辑安装httpd2.assets/image-20220215201023088.png)

等10s后就发现

![image-20220215201043889](5-编辑安装httpd2.assets/image-20220215201043889.png)



复习

![image-20220215201228881](5-编辑安装httpd2.assets/image-20220215201228881.png)

<img src="5-编辑安装httpd2.assets/image-20220215201338134.png" alt="image-20220215201338134" style="zoom:150%;" />





补充：

tree 编译安装的其他问题处理记录

```
root@vpn tree-2.0.2]#make
gcc -O3 -pedantic -Wall -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64 -c -o tree.o tree.c
In file included from tree.c:20:0:
tree.h:63:1: warning: C++ style comments are not allowed in ISO C90 [enabled by default]
 // Start using PATH_MAX instead of the magic number 4096 everywhere.
 ^
tree.h:63:1: warning: (this will be reported only once per input file) [enabled by default]
tree.c:47:1: warning: C++ style comments are not allowed in ISO C90 [enabled by default]
 //off_t (*listdir)(char *, int *, int *, u_long, dev_t) = unix_listdir;
 ^
tree.c:47:1: warning: (this will be reported only once per input file) [enabled by default]
tree.c: In function ‘main’:
tree.c:100:3: warning: ISO C90 forbids mixed declarations and code [-Wpedantic]
   bool needfulltree;
   ^
tree.c:124:29: warning: ISO C90 forbids compound literals [-Wpedantic]
   lc = (struct listingcalls){
                             ^
tree.c:138:3: warning: ISO C90 forbids mixed declarations and code [-Wpedantic]
   char *stddata_fd = getenv(ENV_STDDATA_FD);
   ^
tree.c:145:33: warning: ISO C90 forbids compound literals [-Wpedantic]
       lc = (struct listingcalls){
                                 ^
tree.c:263:30: warning: ISO C90 forbids compound literals [-Wpedantic]
    lc = (struct listingcalls){
                              ^
tree.c:271:30: warning: ISO C90 forbids compound literals [-Wpedantic]
    lc = (struct listingcalls){
                              ^
tree.c:279:30: warning: ISO C90 forbids compound literals [-Wpedantic]
    lc = (struct listingcalls){
                              ^
tree.c: In function ‘usage’:
tree.c:659:2: warning: string length ‘3348’ is greater than the length ‘509’ ISO C90 compilers are required to support [-Woverlength-strings]
  "  --            Options processing terminator.\n");
  ^
tree.c: In function ‘patignore’:
tree.c:668:3: error: ‘for’ loop initial declarations are only allowed in C99 mode
   for(int i=0; i < ipattern; i++)
   ^
tree.c:668:3: note: use option -std=c99 or -std=gnu99 to compile your code
tree.c: In function ‘patinclude’:
tree.c:679:3: error: ‘for’ loop initial declarations are only allowed in C99 mode
   for(int i=0; i < pattern; i++)
   ^
tree.c: In function ‘getinfo’:
tree.c:712:3: warning: ISO C90 forbids mixed declarations and code [-Wpedantic]
   int isdir = (st.st_mode & S_IFMT) == S_IFDIR;

```

上面error说这个只能用c99编译，结果你的MakeFile

![image-20220221182652521](5-编辑安装httpd2.assets/image-20220221182652521.png)



注意，在我的一台centos8里是CC=gcc，没问题的，但是在另一台centos7里，报错c99，

网上搜了下，说gcc默认是使用c89，然后我rpm -ql看了一下

![image-20220221182929846](5-编辑安装httpd2.assets/image-20220221182929846.png)

果断修改Makefile里的**CC=c99**就好了，不明觉厉~

所以不管怎么说，人家要c99就给他c99好了

改成

![image-20220221182830860](5-编辑安装httpd2.assets/image-20220221182830860.png)

再次make就好了



