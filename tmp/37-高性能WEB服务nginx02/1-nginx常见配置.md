# 第1节. nginx常见配置



## 对客户端进行限制

### 下载限速



准备下载的资源

![image-20240117150403381](1-nginx常见配置.assets/image-20240117150403381.png)

下载速度

![image-20240117150513176](1-nginx常见配置.assets/image-20240117150513176.png)

限速成功👇

![image-20240117164800136](1-nginx常见配置.assets/image-20240117164800136.png)







### 限制method

```
limit_except method ... { ... }，仅用于location
限制客户端使用除了指定的请求方法之外的其它方法
method:GET, HEAD, POST, PUT, DELETE，MKCOL, COPY, MOVE,  OPTIONS, PROPFIND, PROPPATCH, LOCK, UNLOCK, PATCH
limit_except GET {
	allow 192.168.1.0/24;
	deny all;
}
除了GET和HEAD 之外其它方法仅允许192.168.1.0/24网段主机使用

```



![image-20240117172113121](1-nginx常见配置.assets/image-20240117172113121.png)

![image-20240117172203447](1-nginx常见配置.assets/image-20240117172203447.png)

这里的Not Allowed应该不是没有放行，而是网站不支持的意思，证明如下👇

配置

![image-20240117183840413](1-nginx常见配置.assets/image-20240117183840413.png)

翻译：限制所有，除了GET和OPTIONS以及GET放了就带了HEAD，IP方面全部包含。

![image-20240117183951126](1-nginx常见配置.assets/image-20240117183951126.png)

这才是没有放行的报错👇

![image-20240117184015696](1-nginx常见配置.assets/image-20240117184015696.png)

而这个不是没有放行，而是网站不支持的现象👇

![image-20240117184102196](1-nginx常见配置.assets/image-20240117184102196.png)



### 文件操作优化的配置

**aio on | off | threads[=pool];**

​	是否启用异步io，默认off

**directio size | off;**

​	当文件大于等于给定大小时，同步(直接)写磁盘，而非写缓存，默认off

```
location /video/ {
	sendfile	on;
	aio			on;
	directio	8m;
}
```



**open_file_cache off;**

**open_file_cache max=N [inactive=time];**

​	nginx可以使用缓存来加快文件访问速度，可以缓存以下三种信息：

​	①文件元数据：文件描述符、文件大小和最近一次的修改时间

​	②打开的目录结构

​	③没有找到的或者没有权限访问的文件的相关信息

​	max=N：可缓存的缓存项上限，达到上限后会使用LRU算法来管理

​	inactive=time：缓存项的非活动时长，在此处指定的时长内未命中的或命中次数少于open_file_cahce_min_users指令所指定的次数的缓存项即为非活动项，将被删除。

这里面就体现了缓存的一个使用逻辑：①缓存的空间多少能存多少数据量数据量是条数来定义的吗？②存多久，不用就删，不用的标准是什么，用的少就删，用的少的标准是什么。这里就有一个指定时间。所以上面的逻辑就是答案了。



**open_file_cache_min_uses number;**

​	open_file_cache指令的inactive参数指定的时长内，至少被命中此处指定的次数方可被归类为活动项。默认为1，1就是等价于LRU算法了--只要命中一次就将优先级提为最高。可以改大点。



**open_file_cache_errors on | off**，是否缓存查找时发生错误的文件一类的信息，默认off

​	用户经常访问错误的页面，可以开启这个选项



**open_file_cache_valid time**，缓存项有效性的检查频率，默认值60s

​	检查到了非活动项，就会删除了。



### ngx_http_access_module 类似acl

该模块，可实现基于ip的访问控制功能

allow address | CIDR | unix: | all;

deny address | CIDR | unix: | all;

​	 上下文所属：http，server，location，limit_execpt

​	自上而下检查，一旦匹配，将生效，条件严格的置前。



示例：

```
location /about {  # 这玩意会301重定向为/about/然后走root拼接，就是
	root /data/nginx/html/pc;   # 拼接为/data/nginx/html/pc/about/index.html
	index index.html;
	deny 192.168.1.1;
	allow 192.168.1.0/24;
	allow 10.1.1.0/16;
	allow 2001:0db8::/32;
	deny all;
}
```

![image-20240118103756729](1-nginx常见配置.assets/image-20240118103756729.png)

调整一下顺序

![image-20240118103834111](1-nginx常见配置.assets/image-20240118103834111.png)



### ngx_http_auth_basic_module

该模块基于用户访问控制，使用basic机制进行用户认证

auth_basic string | off

auth_basic_user_file file;

​	location /admin/ {

​		auth_basic "Admin Area";     # 提示语句，弹出对话框里的信息

​		auth_basic_user_file /etc/nginx/.ngxpasswd;        # 你看，和httpd一样，密码都是用.xxx隐藏文件来做一般

}

用户口令文件：

1、明文文本：格式name:passswd:comment

2、加密文本：由htpasswd命令实现

​		httpd-tools所提供



好像和httpd也就是apache一样？肯定一样了，都是一个工具httpd-tools提供的

![image-20240118111154983](1-nginx常见配置.assets/image-20240118111154983.png)



![image-20240118111415509](1-nginx常见配置.assets/image-20240118111415509.png)

按这走一遍咯👆



![image-20240118111745835](1-nginx常见配置.assets/image-20240118111745835.png)

或者这样更简单

![image-20240118112244323](1-nginx常见配置.assets/image-20240118112244323.png)



注意-c 只有首次才能加哦

![image-20240118112323831](1-nginx常见配置.assets/image-20240118112323831.png)

以后就这么配置👇

![image-20240118112403537](1-nginx常见配置.assets/image-20240118112403537.png)

配置以下使用该用户密码

![image-20240118112928596](1-nginx常见配置.assets/image-20240118112928596.png)

去掉index test.html才会使认证生效👇 不对，不对，不去掉也可以的，这里实验效果生效可能慢了

![image-20240118114702350](1-nginx常见配置.assets/image-20240118114702350.png)





![image-20240118115933695](1-nginx常见配置.assets/image-20240118115933695.png)





![image-20240118133653704](1-nginx常见配置.assets/image-20240118133653704.png)



要注意这个basic认证的密码，是明文的，抓包可见

抓这个口的报文

![image-20240118134111599](1-nginx常见配置.assets/image-20240118134111599.png)

![image-20240118134234807](1-nginx常见配置.assets/image-20240118134234807.png)

这就是为什么要采用https的原因



### apache那会讲过status统计页面，nginx同样也有



![image-20240118141004245](1-nginx常见配置.assets/image-20240118141004245.png)

这有什么用呢，就是将来可以curl xx/xxx/status，就能快捷地获得各个指标，联动报警了。

```
location /nginx_status {
	stub_status;
	allow 127.0.0.1;
	allow 172.16.0.0/16;  # 自己修改成所需要地白名单
	deny all;
}
```

![image-20240118143637173](1-nginx常见配置.assets/image-20240118143637173.png)



![image-20240118143734204](1-nginx常见配置.assets/image-20240118143734204.png)



ab打一下

![image-20240118144850694](1-nginx常见配置.assets/image-20240118144850694.png)



注意：第二行参数地值也就是2、3两行 是累计地值

最后一行是当前的值。





### 第三方插件-echo

echo就是变量 显示出来比较方便可能。

![image-20240118174400274](1-nginx常见配置.assets/image-20240118174400274.png)

这是没有echo功能的，需要安装插件

https://github.com/openresty/echo-nginx-module

![image-20240118181306590](1-nginx常见配置.assets/image-20240118181306590.png)



然后下载nginx源码

https://nginx.org/en/download.html

然后编译的时候，把echo插件加进去

![image-20240118183113715](1-nginx常见配置.assets/image-20240118183113715.png)



![image-20240118182856859](1-nginx常见配置.assets/image-20240118182856859.png)



ok👇

![image-20240118183408407](1-nginx常见配置.assets/image-20240118183408407.png)

图中error只是摸粑粑的着色

然后make -j 4 && make install报错👇

![image-20240119093506284](1-nginx常见配置.assets/image-20240119093506284.png)

处理方法

![image-20240119093616854](1-nginx常见配置.assets/image-20240119093616854.png)

完整的编译选项如下👇

```
./configure \
--with-cc-opt='-fPIE -fPIC' \
--with-ld-opt='-pie' \
--prefix=/apps/nginx \
--user=nginx --group=nginx \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-http_perl_module \
--with-pcre \
--with-stream \
--with-stream_ssl_module \
--with-stream_realip_module \
--add-module=/usr/local/src/echo-nginx-module

make -j 4 && make install

```

![image-20240119094102949](1-nginx常见配置.assets/image-20240119094102949.png)



创建测试配置文件

![image-20240119100352179](1-nginx常见配置.assets/image-20240119100352179.png)



![image-20240119100404330](1-nginx常见配置.assets/image-20240119100404330.png)

看上图不行哦👆，nginx -t看到的是nginx.conf test OK，其实没有检测test.conf的，因为主配置文件里没有指定includ xxx

加一下

![image-20240119155746463](1-nginx常见配置.assets/image-20240119155746463.png)



![image-20240119174905216](1-nginx常见配置.assets/image-20240119174905216.png)

这就是效果了，然后浏览器是由于mime.types应该没有识别所以就统一下载

![image-20240119174948167](1-nginx常见配置.assets/image-20240119174948167.png)

下载下来打开就是

![image-20240119175018672](1-nginx常见配置.assets/image-20240119175018672.png)

优化下指一下default_type为text/html;👇

![image-20240119175108511](1-nginx常见配置.assets/image-20240119175108511.png)

ok，如果注释掉echo，就是index.html内容了

![image-20240119175154033](1-nginx常见配置.assets/image-20240119175154033.png)



简化配置👇

![image-20240119175347006](1-nginx常见配置.assets/image-20240119175347006.png)

如果用echo，其实不必要指root的什么文件也无需存在的。



同样echo插件可以打印变量

![image-20240122094417340](1-nginx常见配置.assets/image-20240122094417340.png)

![image-20240122101657095](1-nginx常见配置.assets/image-20240122101657095.png)

解释上图👇

![image-20240122101641024](1-nginx常见配置.assets/image-20240122101641024.png)

