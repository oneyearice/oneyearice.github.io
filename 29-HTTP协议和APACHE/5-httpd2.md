# 第5节. httpd2



# 继续上一篇的配置



## 主站点页面-DocumentRoot

**定义'Main' server的文档页面路径** DocumentRoot “/path”

文档路径映射：

DocumentRoot指向的路径为URL路径的起始位置 示例：

DocumentRoot "/app/data“

http://HOST:PORT/test/index.html --> /app/data/test/index.html 注意：SELinux和iptables的状态



![image-20231007104941607](5-httpd2.assets/image-20231007104941607.png)

这里修改站点的页面存放路径，还需要保证该路径的文件的访问权限。注释掉👆上图的主站点配置。

![image-20231007105533835](5-httpd2.assets/image-20231007105533835.png)

下面对其进行修改👇，还是使用单独的配置文件，不对httpd.conf这个主文件进行修改

<img src="5-httpd2.assets/image-20231007105037065.png" alt="image-20231007105037065" style="zoom:50%;" /> 

![image-20231007105118589](5-httpd2.assets/image-20231007105118589.png)

权限👆没问题

selinux没问题👇

![image-20231007105137048](5-httpd2.assets/image-20231007105137048.png)



![image-20231007105631809](5-httpd2.assets/image-20231007105631809.png)

修改配置文件，

![image-20231007105712166](5-httpd2.assets/image-20231007105712166.png)

必须是文件夹，且其下放index.html

<img src="5-httpd2.assets/image-20231007105750876.png" alt="image-20231007105750876" style="zoom:33%;" /> 

重启OK



但是curl 发现不行

![image-20231007105841918](5-httpd2.assets/image-20231007105841918.png)

这就意味着页面一样是apache的默认页面了，因为index.html打不开

![image-20231007105921419](5-httpd2.assets/image-20231007105921419.png)

可是对应的文件夹的进入，和文件的可读权限都有的啊，奇怪了

但是就是403 Forbidden了

<img src="5-httpd2.assets/image-20231007110058968.png" alt="image-20231007110058968" style="zoom:44%;" /> 

所谓授权不是linux系统文件的授权，而是httpd服务本身配置文件里要明确授权，所以还是配置文件里的配置不全。

<img src="5-httpd2.assets/image-20231007110435402.png" alt="image-20231007110435402" style="zoom:50%;" /> 

同样写道单独的配置文件里去

<img src="5-httpd2.assets/image-20231007110625847.png" alt="image-20231007110625847" style="zoom:50%;" /> 

![image-20231007110654272](5-httpd2.assets/image-20231007110654272.png)

---------------这样就OK👆了------------------

然后取消/data文件夹的所有权限，哈哈

![image-20231007111300759](5-httpd2.assets/image-20231007111300759.png)

其上图不用重启服务，因为修改的os操作系统的权限，而不是httpd服务的配置文件，👇证明下：

<img src="5-httpd2.assets/image-20231007111415750.png" alt="image-20231007111415750" style="zoom:50%;" /> 

如果是DocumentRoot下的子目录的index就是补一个路径就好了👇。

![image-20231007115555925](5-httpd2.assets/image-20231007115555925.png)



#### 如果子目录不在DocumentRoot下呢

1、<span id = 'jump1'>用软连接行不行：👇可以的，没得问题</span>

![image-20231007115843184](5-httpd2.assets/image-20231007115843184.png)





那为什么文件默认就是index.html，是否可以修改呢？可以，在配置里面找到关键词DirectoryIndex👇

### **定义站点主页面**

DirectoryIndex		index.html

![image-20231007133654437](5-httpd2.assets/image-20231007133654437.png)

增加一个

![image-20231007133801205](5-httpd2.assets/image-20231007133801205.png)



![image-20231007134204865](5-httpd2.assets/image-20231007134204865.png)

上图👆就说明了，配置文件的写法：DirectoryIndex m.txt index.html是优先级从左到右的。



如果m.txt和index.html文件都不存在，那么这个报错的页面403 Forbidden又是哪来的呢？

![image-20231007134656311](5-httpd2.assets/image-20231007134656311.png)

<img src="5-httpd2.assets/image-20231007134714917.png" alt="image-20231007134714917" style="zoom:50%;" />



在/etc/httpd/conf.d/下有一个叫welcome.conf的配置文件，其中就有403对应的页面👇

![image-20231007134927879](5-httpd2.assets/image-20231007134927879.png)

这里的/.noindex.html就是相对于DocumentRoot主页根路径来讲的。

但是没找到这个文件，增大俺的狗眼仔细看看上图 找找看，Alias啊，/.noindex.html全文里出现了2次，还不清楚嘛~(/≧▽≦)/

删掉这个welcome.conf文件，观察页面

<img src="5-httpd2.assets/image-20231007135731812.png" alt="image-20231007135731812" style="zoom:50%;" /> 

重启服务后👇

<img src="5-httpd2.assets/image-20231007135856543.png" alt="image-20231007135856543" style="zoom:50%;" />

![image-20231007135922545](5-httpd2.assets/image-20231007135922545.png)

所以修改一下

![image-20231007141003559](5-httpd2.assets/image-20231007141003559.png)



![image-20231007141102616](5-httpd2.assets/image-20231007141102616.png)

页面一样效果👇

<img src="5-httpd2.assets/image-20231007141112133.png" alt="image-20231007141112133" style="zoom:50%;" />



### 站点访问控制常见机制

可基于**两种机制**指明对**哪些资源**进行何种访问控制

​		**两种机制**：客户端来源地址，用户账号

​		**哪些资源**：文件系统路径、URL路径



**文件系统路径：**

①针对文件夹②针对文件③写正则匹配

```
<Directory	“/path">
...
</Directory>
<File	“/path/file”>
...
</File>
<FileMatch	"PATTERN">
...
</FileMatch>

```

**URL路径：**

```
<Location	"">
...
</Location>
<LocationMatch "">
...
</LocationMatch>

```

**示例：**

```
<FilesMatch "\.(gif|jpe?g|png)$">       	# FilesMatch就是REGEX正则
<Files “?at.*”>	通配符       			  	  # 应该是Files 开头就是通配符
<Location /status>                			# Location就是URL
<LocationMatch "/(extra|special)/data">     # LocationMatch就是url里的regex正则
```



**具体的写法-带上源**



```
<Directory>中“基于源地址”实现访问控制

(1) Options：后跟1个或多个以空白字符分隔的选项列表
	在选项前的+，- 表示增加或删除指定选项

常见选项：
Indexes：指明的URL路径下不存在与定义的主页面资源相符的资源文件时，返回索引列表给用户
FollowSymLinks：允许访问符号链接文件所指向的源文件 
None：全部禁用
All：	全部允许


示例：
<Directory /web/docs>
	Options Indexes FollowSymLinks
</Directory>
<Directory /web/docs/spec>
	Options FollowSymLinks
</Directory>


```

### Indexes：指明的URL路径下不存在与定义的主页面资源相符的资 源文件时，返回索引列表给用户

**就是访问页面不存在，返回当前路径下的列表，页面显示出来的文件夹和文件都可以点击，类似yum源网站的点击浏览一样**

![image-20231007143821355](5-httpd2.assets/image-20231007143821355.png)

然后页面访问是默认的报错页面

![image-20231007144403825](5-httpd2.assets/image-20231007144403825.png)

删掉该文件也没用！

找找官网说明

![image-20231007153451751](5-httpd2.assets/image-20231007153451751.png)

这个模块也加载了啊

![image-20231007153507904](5-httpd2.assets/image-20231007153507904.png)

搞不懂，继续折腾，发现还是welcome.conf搞的鬼，这样就可以看到了，其实就是👇这里得options -Indexes。  - 减号就是禁止啊，禁用了目录索引功能。

![image-20231007162756638](5-httpd2.assets/image-20231007162756638.png)



然后dir1不现实，dir2可以的，研究下dir1为啥不显示

![image-20231007155411441](5-httpd2.assets/image-20231007155411441.png)

搞不懂dir1为啥不显示了，[之前可以的啊](#jump1)

![image-20231007155538433](5-httpd2.assets/image-20231007155538433.png)



![image-20231007161148352](5-httpd2.assets/image-20231007161148352.png)

就是dir1好好的不行了，哈哈

![image-20231007161141031](5-httpd2.assets/image-20231007161141031.png)



![image-20231007161656178](5-httpd2.assets/image-20231007161656178.png)



👇结果发现是软连接得问题，但是之前可以得啊！

![image-20231007163745035](5-httpd2.assets/image-20231007163745035.png)



![image-20231007164158405](5-httpd2.assets/image-20231007164158405.png)

权限OK👆，找到元婴了，👇**就是开启indexes就不支持软连接了，奇怪了，先记着这个点**

![image-20231007175831767](5-httpd2.assets/image-20231007175831767.png)

👆同时注意：curl 不同于浏览器http://192.168.126.130/dir2 ,  浏览器这样回车自动就是dir2/ 补一个斜杠的/；  curl 没斜杆/有问题的。



来一张总结图👇，结论就是 **options indexes会导致ln -s 的403 Forbidden。**

![image-20231007180604795](5-httpd2.assets/image-20231007180604795.png)





如果要把软连接也显示出来，需要再配置一下

### FollowSymLinks：允许访问符号链接文件所指向的源文件

当然，welcome.conf里的-Indexes要去掉的

![image-20231007181947268](5-httpd2.assets/image-20231007181947268.png)



![image-20231007182031235](5-httpd2.assets/image-20231007182031235.png)

![image-20231007182500958](5-httpd2.assets/image-20231007182500958.png)



补上对应参数后就可以显示软连接了👇

![image-20231007182638822](5-httpd2.assets/image-20231007182638822.png)

<img src="5-httpd2.assets/image-20231007182931735.png" alt="image-20231007182931735" style="zoom:33%;" />





