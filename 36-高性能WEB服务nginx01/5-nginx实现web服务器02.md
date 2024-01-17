# 第5节. nginx实现web服务器02



![image-20231103105617856](5-nginx实现web服务器02.assets/image-20231103105617856.png)



![image-20231103105630727](5-nginx实现web服务器02.assets/image-20231103105630727.png)

比如统一把静态资源放到/static/下，或者将后缀写全了进行转发。



### alias和root，替换和拼接

![image-20231103113348785](5-nginx实现web服务器02.assets/image-20231103113348785.png)



![image-20231103133956115](5-nginx实现web服务器02.assets/image-20231103133956115.png)

root是拼接，而alias是看①如果访问的url里的有alias的path比如上图的about，那么就把整个url替换成alias的path。于是www.site1.com/about/index.html就访问了/opt/testdir/index.html了。



加不加/就是为了xx/xxx/zz/和xx/xxx/zz效果一样因为浏览器会自动重定向类似curl -L的效果

![image-20231103135807184](5-nginx实现web服务器02.assets/image-20231103135807184.png)

上图注意location里是/about/，用户请求的www.site1.com/about里是不包含/about/的所以不会走location的转发，而是走的非location的也就是/data/site1/的转发路径，具体见error.log

![image-20231103141055705](5-nginx实现web服务器02.assets/image-20231103141055705.png)





![image-20231103135930156](5-nginx实现web服务器02.assets/image-20231103135930156.png)

而官方的解释很清楚了已经

![image-20231103135957099](5-nginx实现web服务器02.assets/image-20231103135957099.png)

就是/user/和/user你不想合成一个URL，是可以指到不同的站点URL上去的。而一般情况都是使用/user不带/这样，用户访问/user和/user/其实都是一个站点，因为/user用户访问呢会自动301到/user/上去的。

上面讨论了location里路径的结尾带不带/的区别，下面讨论alias/root的转发路径的结尾带不带/的区别

![image-20231103142639778](5-nginx实现web服务器02.assets/image-20231103142639778.png)

### 总结下

![image-20231103143136974](5-nginx实现web服务器02.assets/image-20231103143136974.png)

### 修正理解

上图总结的处理问题不大，但是需要补充



**用户行为：**

1、curl www.site1.com/about这是访问about文件

2、curl www.site1.com/about/这是访问about/index.html文件

通常都会location里写成/about而不是/about/来应对，这样用户不管最后带不带/实际效果都是一样的，因为不带/也会匹配location然后301到/about/。



**location匹配**

1、写成/about和/about/是有区别的，首先用户访问的url里必须完整包含，比如location写的是/about/但是用户行为里是/about，就不会命中了；

2、结合用户行为，如果是/about/那么就是访问的/about/index.html此时 "alias"  就会将/about/的url整个替换成alias的路径，并在最后补上index.html；此时如果alias那里没有写成文件夹而是写成/xxx，那么整个url就有问题了，就变成了xxxindex.html中间会少了/。    不过root不会出现这个问题，因为root是拼接的，会拼接成xxx/about/index.html

![image-20231103151334087](5-nginx实现web服务器02.assets/image-20231103151334087.png)



**alias/root转发处理**

1、alias是整个替换只要用户行为里包含了location里的比如/about那么整个url都会替换掉alias的东西，当然如果用户行为最后带/会自动补上index.html的。

2、root是拼接，问题不大，总之实测一下，留意就好，实际测试oK就行，不要纠结了。









## index



![image-20231103160402671](5-nginx实现web服务器02.assets/image-20231103160402671.png)



![image-20231103154956605](5-nginx实现web服务器02.assets/image-20231103154956605.png)

上图👆有一处地方要改，就是location /about/改成lcoation /about，这样curl -L www.site1.com/about 就能利用301重定向了。



### error_page code



![image-20231103160343976](5-nginx实现web服务器02.assets/image-20231103160343976.png)



![image-20231103160426147](5-nginx实现web服务器02.assets/image-20231103160426147.png)

找不到404 Not Found哪来的，因为配置文件里404是注释的，

<img src="5-nginx实现web服务器02.assets/image-20231103163803870.png" alt="image-20231103163803870" style="zoom:50%;" />



我好像找到了，👇应该就是ngx_http_special_response.c文件

![image-20231103164208989](5-nginx实现web服务器02.assets/image-20231103164208989.png)

具体调用和最终应用到页码不知道什么弄得，不过这里由明确得html的语法，知道是源码里的404已经做好了就行了。





![image-20231103170612543](5-nginx实现web服务器02.assets/image-20231103170612543.png)

好像只是简单做一个error页面不用写location啊

![image-20231103170810342](5-nginx实现web服务器02.assets/image-20231103170810342.png)



![image-20231103171439567](5-nginx实现web服务器02.assets/image-20231103171439567.png)

![image-20231103171521884](5-nginx实现web服务器02.assets/image-20231103171521884.png)





#### location在处理error_page时的注意点



1、root没毛病

![image-20231103180142827](5-nginx实现web服务器02.assets/image-20231103180142827.png)

root的最后/写不写都行，因为时location 和 root 拼接的，location里带了/了

![image-20231103180337811](5-nginx实现web服务器02.assets/image-20231103180337811.png)



2、alias有点细节要注意

![image-20231103181604670](5-nginx实现web服务器02.assets/image-20231103181604670.png)



注意由于重定向的存在也就是location转发的存在，这些重定向的动作是不会日志中看到的，日志中只是最终的一个结果。

这里盲猜location = /404.html 被alias转成了/opt/testdir/，然后没有给你自动补一个index.html

![image-20231103182203259](5-nginx实现web服务器02.assets/image-20231103182203259.png)

总之也实现了alias和root的404location转发。



场景：浏览器劫持

输入www.site1.com/xfsdfaf后跳转的地址被修改了

![image-20240116141814641](5-nginx实现web服务器02.assets/image-20240116141814641.png)



为了防止这种浏览器的不良行为，应该如何应对，现在来讲，可能也就是你自己的配置问题，没有对应的文件存在👇比如这种：就会让别人有机可乘。当然chrome不会，可以将404报错改成200这种正确响应码。

![image-20240116151548811](5-nginx实现web服务器02.assets/image-20240116151548811.png)

当然chrome不会

![image-20240116151653677](5-nginx实现web服务器02.assets/image-20240116151653677.png)









### try_files file ...uri; try_files file ... =code;

这就是类似什么呢，类似这个

![image-20240116165451768](5-nginx实现web服务器02.assets/image-20240116165451768.png)

网工看了就懂咯

#### 配置下try_files

![image-20240116170631614](5-nginx实现web服务器02.assets/image-20240116170631614.png)

![image-20240116181611207](5-nginx实现web服务器02.assets/image-20240116181611207.png)

测试效果就是访问

http://www.site1.com/images/ajpg

http://www.site1.com/images/b.jpg

http://www.site1.com/images/default.jpg

都OK

但是访问

http://www.site1.com/images/xxx.yy

整个uri不存在，就对应到try_files 的$uri整个变量的文件不存在，于是就转到default.jpg上去。



思考👇

![image-20240116184306232](5-nginx实现web服务器02.assets/image-20240116184306232.png)

总结就是：内部重定向的/images/default.jpg，也会再次命中location的alias，于是

http://www.site1.com/images/default.jpg被alias成http://www.site1.com/data/images/default.jpg了？？？只能这么理解了啊。



![image-20240116182414458](5-nginx实现web服务器02.assets/image-20240116182414458.png)

如果/images/



### keepalive_timeout

![image-20240117105122175](5-nginx实现web服务器02.assets/image-20240117105122175.png)



![image-20240117105648767](5-nginx实现web服务器02.assets/image-20240117105648767.png)

这个65怎么体现在测试中：

通过telnet 80 并发送一次get可以感受到时间的长短，操作主要事项①get一次得到页面内容后②不要再做任何操作。此时可以观测到配置的时间。



注意这两个参数是不同的

![image-20240117134637548](5-nginx实现web服务器02.assets/image-20240117134637548.png)

keepalive_time是一个链接可以存活多久

keepalive_timeout是一个client请求的链接，空闲时间可以存活多久。



![image-20240117134950166](5-nginx实现web服务器02.assets/image-20240117134950166.png)

可见启用了keepalive_timeout但是没有说多少秒。

![image-20240117135118474](5-nginx实现web服务器02.assets/image-20240117135118474.png)

写道head里的是10秒，实际是65秒。



### 持久链接断开的情况

1、keepalive_timeout时间到了。

2、请求的资源个数超了，比如一次长连接允许请求文件为3个。测试👇

![image-20240117135632993](5-nginx实现web服务器02.assets/image-20240117135632993.png)



![image-20240117135714790](5-nginx实现web服务器02.assets/image-20240117135714790.png)



### 禁止那种浏览器使用长连接

keepalive_disable none | browser ...;

手机上各种浏览器



### 向客户端发送超时时长

send_timeout time;

向客户端发送响应报文的超时时长，此处是指两次写操作之间的间隔时长，而非整个响应时间过程的传输时长。

服务器响应发送时间间隔，也就是两次写操作的间隔，这里写，我的理解是 server要构建response的，自然是写内容的要。



### 上传文件的限制？

client_max_body_size size; 请求报文中实体的最大值,默认1M，超过了就报错413

client_body_buffer_size size;上传的文件是有buffer的，超过buffer大小默认16K，就会暂存到磁盘上的下面client_body_temp_path指令所定义的位置。

client_body_temp_path path [level1 [level2 [level3]]];

设定存储客户端请求报文的body部分的临时存储路径及子目录结构和数量

目录名为16进制的数字；用hash之后的值从后往前截取第1、2、3级作为文件名

比如： client_body_temp_path /var/tmp/client_body 1 2 2       # 三级目录1 2 2，分别用1个字符，2个字符，2个字符。

1 1级目录占1位16进制，即2^4=16个目录 0-f

2 2级目录占2位16进制，即2^8=256个目录 00-ff

2 3级目录占2位16进制，即2^8=256个目录 00-ff

👇计算下这样三级命名文件夹的好处--就是用上传的文件的哈希值作为分层目录，这样能够支持1048576个子文件，每个子文件里如果放1个文件也就是100w个文件了。好处就是如果整个100w放在一个文件夹里ls都能卡死，分层放置，查找自然就快了。L1目录顶多16个，L2目录256个，L3目录256个也是。

![image-20240117144137085](5-nginx实现web服务器02.assets/image-20240117144137085.png)

![image-20240117144200552](5-nginx实现web服务器02.assets/image-20240117144200552.png)



```shell
上传服务器配置生产案例：  
location	/upload {
	client_max_body_size 100m；  
	client_body_buffer_size 2048k;
	client_body_temp_path /apps/nginx/temp 1 2 2;
	…
}
```

