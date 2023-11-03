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
