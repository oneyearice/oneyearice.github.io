# 第5节. nginx反向代理实现



# 防盗链

盗链就是一个网站调用另一个网站的资源



**构建盗链**

site1站点盗链site2站点里的一张图片

![image-20240206111138310](5-nginx反向代理实现.assets/image-20240206111138310.png)

这是两个之前做的站点👆就利用这个现成的做实验

![image-20240206111350203](5-nginx反向代理实现.assets/image-20240206111350203.png)

这就实现了盗链，

![image-20240206112043970](5-nginx反向代理实现.assets/image-20240206112043970.png)

很明显存在两个问题①乱码②图片不能缩放了，

![image-20240206112800647](5-nginx反向代理实现.assets/image-20240206112800647.png)

![image-20240206113334211](5-nginx反向代理实现.assets/image-20240206113334211.png)

这是site1和site2 log 合在一起咯，盗链行为其实是，站在被盗方的角度去防止，所以log拆开来，去开被盗站点site2的log

![image-20240206114652752](5-nginx反向代理实现.assets/image-20240206114652752.png)

可以看到的，所以就可以防止盗链行了的



**防止盗链的解放方法：ngx_http_referer_module模块来弄**

大体意思就是先定义出哪些是合法的referer，比如百度过来的referer说明你广告费没白出。

两个参数

**valid_referers none|blocked|server_names|string ...;**      # 用来定义合法referer的值的

**none**：就是referer为空 判断为合法。一般直接访问某个网站referer都是空的。自然要写none的。

![image-20240206120416349](5-nginx反向代理实现.assets/image-20240206120416349.png)

none在浏览器里F12是看不到的吧



![image-20240206120747913](5-nginx反向代理实现.assets/image-20240206120747913.png)



![image-20240206120557421](5-nginx反向代理实现.assets/image-20240206120557421.png)





**blocked**：就是referer里写的东西不是常规的uri，这个没见过还



**server_names**：本网站内部跳转的，就会在referer里带上自己的主机名，都是自家人啊。



**arbitrary_string**：任意字符串，用通配符的方式。

![image-20240206135745293](5-nginx反向代理实现.assets/image-20240206135745293.png)



**regular_expression** ：也是任意字符串，用regex来做了，格式就是~开头代表regex。

​																		例如：        ~.*\.site1\.com





配置防盗链肯定在被盗站点上配置了👇

```
    valid_referers none block server_names
    *.site2.com site2.* ~\.site2\.
    ~\.google\. ~\.baidu\.;

    if  ($invalid_referer)  {
         return 403 "Forbidden Access";
    }

```

![image-20240206142530884](5-nginx反向代理实现.assets/image-20240206142530884.png)







![image-20240206142854980](5-nginx反向代理实现.assets/image-20240206142854980.png)



![image-20240206143005932](5-nginx反向代理实现.assets/image-20240206143005932.png)



然后如果是从baidu跳过来的就不禁止访问，因为十有八九就是广告的钱啦👇

测试：修改site1为baidu

![image-20240206154826262](5-nginx反向代理实现.assets/image-20240206154826262.png)

修改本地hosts文件

C:\Windows\System32\drivers\etc      下👇

![image-20240206154608745](5-nginx反向代理实现.assets/image-20240206154608745.png)



上图配置是不行的，因为http://www.baidu.com/daolian.html是80跳443的也就是跳到https://www.site1.com的，

![image-20240206155119867](5-nginx反向代理实现.assets/image-20240206155119867.png)

然后就会去访问https://www.site1.com/daolian.html

然后图片的访问里就有

![image-20240206155206461](5-nginx反向代理实现.assets/image-20240206155206461.png)

这样就不是从www.baidu.com跳过去的了，

测试的话倒不用改配置文件，直接访问https://www.baidu.com/daolian.html

这样就可以了，

![image-20240206155336952](5-nginx反向代理实现.assets/image-20240206155336952.png)



当然这里ssl不安全的原因也很简单啊，就是域名和ssl里的 备用名称一致。

![image-20240206155712189](5-nginx反向代理实现.assets/image-20240206155712189.png)





![image-20240206155541825](5-nginx反向代理实现.assets/image-20240206155541825.png)



如果就要实现http://www.baidu.com/daolian.html能够打开呢

可以，单独写baidu就行了

![image-20240206163545898](5-nginx反向代理实现.assets/image-20240206163545898.png)

![image-20240206163735200](5-nginx反向代理实现.assets/image-20240206163735200.png)









# 开始学习nginx的反向代理咯

正向代理后面也要学习，应用在比如手机的ssl报文抓包要抓到内容的，做ssl解密的。后话这里记一下~



据说nginx的反代还可以做健康性检查，这点比LVS强，LVS好像没有健康性检查，据说哦。

而且nginx可以做应用层和TCP层的反代，不过速度没有LVS快据说。LVS好像工作在内核层的？

![image-20240206181246911](5-nginx反向代理实现.assets/image-20240206181246911.png)





![image-20240206181536254](5-nginx反向代理实现.assets/image-20240206181536254.png)





![image-20240206181729908](5-nginx反向代理实现.assets/image-20240206181729908.png)







**模块支持**

![image-20240206181940090](5-nginx反向代理实现.assets/image-20240206181940090.png)

1、proxy、fastcgi、uwsgi这些都是http类的，所以名字都是ngx_http开头的。

​		1.1 、 proxy就是nginx对接http原本协议、fastcgi就是nginx对接php协议、uwsgi是nginx对接python协议

​				1.1.1、编写动态页面，可以用PHP、python、java、

​							对于java，如果后端是一个tomcat类似的，这个就用ngx_http_proxy_module来支持了。



所以：

​		前后端是java-tomcat之类，就用ngx_http_proxy_module了

​		前后端是python，比如jango，就用ngx_http_uwsgi_moduel了

​		前后端是php，就用ngx_http_fastcgi_module了











