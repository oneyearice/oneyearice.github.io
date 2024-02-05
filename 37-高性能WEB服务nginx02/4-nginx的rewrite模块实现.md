# 第4节. nginx的rewrite模块实现



# rewrite



![image-20240202183308606](4-nginx的rewrite模块实现.assets/image-20240202183308606.png)



先看下官网的例子，关注下last

![image-20240202141210125](4-nginx的rewrite模块实现.assets/image-20240202141210125.png)





![image-20240202162208107](4-nginx的rewrite模块实现.assets/image-20240202162208107.png)

这种是服务器内部的跳转，client是看不到的，是没有什么301 302 的，直接就是server内部重定向好了直接给你200的。而不是return你301让你重新去访问一个新的url的。

通过https://www.site1.cm/test1/可见直接是200返回码，内容是server自己rewrite成了/data/ssl/site2/inedex22.html了，上图的第二个location匹配到的

![image-20240202174522073](4-nginx的rewrite模块实现.assets/image-20240202174522073.png)

这点就和return 301 httpxxxx不同了，301是告诉用户重新访问发起的。这个rewrite只server自己内部处理后直接返给用户的。节省了c/s交互吧。



👇echo会抢了rewrite的

![image-20240202180635535](4-nginx的rewrite模块实现.assets/image-20240202180635535.png)

反正last不推荐放到location里





![image-20240202183647746](4-nginx的rewrite模块实现.assets/image-20240202183647746.png)

👆上图是官网的说法，我反正这么测试👇没看到什么10次循环500code的结果

![image-20240204142206572](4-nginx的rewrite模块实现.assets/image-20240204142206572.png)



![image-20240204142422429](4-nginx的rewrite模块实现.assets/image-20240204142422429.png)





所以我认为👇

![image-20240204142511481](4-nginx的rewrite模块实现.assets/image-20240204142511481.png)



**总之：last就是下一个location匹配；break就是不继续匹配了**



![image-20240204143405874](4-nginx的rewrite模块实现.assets/image-20240204143405874.png)

好像上图的说法也不对，哈哈，原因看下图👇

![image-20240204143600536](4-nginx的rewrite模块实现.assets/image-20240204143600536.png)

可能echo优先级高于rewrite吧，所以echo启用后，访问xxx/xxx/test就默认行为重定向301到xxx/xxx/test/然后  echo优先级高，于是直接echo出来了。

而访问xxx/xxx/test/，按理说也是命中location /test，但是此时echo又没有生效！奇怪了

两次现象的区别就是第一次有一个301，而直接访问/test/是没有301的。就这个区别了，所以我认为：

**对于last：**

**/test 301到/test/就会echo优先**

**/test/ 直接访问echo就会rewrite优先，又或者就是rewrite更希望没有重新向过的行为。**





![image-20240204144521796](4-nginx的rewrite模块实现.assets/image-20240204144521796.png)

![image-20240204144720921](4-nginx的rewrite模块实现.assets/image-20240204144720921.png)

**对于break和last一样的逻辑**

**1、如果存在echo，那么echo就会优先，和last的区别就是 /xxx和/xxx/ 一样都是echo优先了**

**2、不存在echo，就是常规理解了，就是跳出继续的location匹配了，**



我认为，这里写的就是一坨，而且echo是插件，没有弄好一些细节，这块技术落地的时候要测试好你自己的业务场景的。





### redirect，http跳转https

![image-20240204153934297](4-nginx的rewrite模块实现.assets/image-20240204153934297.png)



![image-20240204152134078](4-nginx的rewrite模块实现.assets/image-20240204152134078.png)

第一个301，默认的/xxx跳转/xxx/行为

![image-20240204152226622](4-nginx的rewrite模块实现.assets/image-20240204152226622.png)

第二个302 就是conf文件里的redirect导致

![image-20240204152259551](4-nginx的rewrite模块实现.assets/image-20240204152259551.png)



![image-20240204160837395](4-nginx的rewrite模块实现.assets/image-20240204160837395.png)

curl 里的log要看到框框里的重定向里的完整url需要这么写，要注意request_uri和uri还是有区别的！

比如/xxx/是request_uri，而uri确实/xxx/index.html。

![image-20240204160941628](4-nginx的rewrite模块实现.assets/image-20240204160941628.png)





### ssl的基于域名进行转发

1、给site2.com创建ssl证书相关的文件

①修改openssl配置文件里的dns为*.site2.com

![image-20240205144050110](4-nginx的rewrite模块实现.assets/image-20240205144050110.png)

![image-20240205144125792](4-nginx的rewrite模块实现.assets/image-20240205144125792.png)

上图改错了，必须改成*.site2.com才行，否则证书无法验证通过！



好了两个文件有了

![image-20240205144328103](4-nginx的rewrite模块实现.assets/image-20240205144328103.png)



![image-20240205144413520](4-nginx的rewrite模块实现.assets/image-20240205144413520.png)



配置好虚拟主机，基于域名的

![image-20240205144538198](4-nginx的rewrite模块实现.assets/image-20240205144538198.png)

![image-20240205144640954](4-nginx的rewrite模块实现.assets/image-20240205144640954.png)



![image-20240205144655257](4-nginx的rewrite模块实现.assets/image-20240205144655257.png)

然后就实现了👇

![image-20240205144824245](4-nginx的rewrite模块实现.assets/image-20240205144824245.png)



由于上面的site2.com没有写成*.site2.com通配，所以ssl验证失败，这里改一下，顺便用一个ssl证书搞定两个网站

![image-20240205153835616](4-nginx的rewrite模块实现.assets/image-20240205153835616.png)



去掉Makefile的xxx.key加密选项和添加v3的dns选项的👇

![image-20240205153234675](4-nginx的rewrite模块实现.assets/image-20240205153234675.png)

![image-20240205154712875](4-nginx的rewrite模块实现.assets/image-20240205154712875.png)



![image-20240205153355453](4-nginx的rewrite模块实现.assets/image-20240205153355453.png)

ssl证书两个虚拟主机就用一个样的

![image-20240205153525994](4-nginx的rewrite模块实现.assets/image-20240205153525994.png)

![image-20240205153558613](4-nginx的rewrite模块实现.assets/image-20240205153558613.png)

然后看最关键的浏览器

通过sz site.crt导入到本地电脑的桌面上，然后双击导入 受信任的根证书颁发机构 就行了

![image-20240205154734173](4-nginx的rewrite模块实现.assets/image-20240205154734173.png)





![image-20240205154841969](4-nginx的rewrite模块实现.assets/image-20240205154841969.png)

👆这就实现了一个证书搞定多个域名啦，自然也是一个ip+80对应不同域名的。

![image-20240205155421367](4-nginx的rewrite模块实现.assets/image-20240205155421367.png)





据说nginx支持这种ssl的多域名对应一个ip+port是因为TLS SNI功能

![image-20240205160212167](4-nginx的rewrite模块实现.assets/image-20240205160212167.png)

TLS SNI就是TLS的sever name indication（sni)

![image-20240205160402176](4-nginx的rewrite模块实现.assets/image-20240205160402176.png)





### 访问错误页面时的处理

需要利用request_filename变量来判断访问的页面不存在

![image-20240205164935118](4-nginx的rewrite模块实现.assets/image-20240205164935118.png)





![image-20240205165000916](4-nginx的rewrite模块实现.assets/image-20240205165000916.png)







<img src="4-nginx的rewrite模块实现.assets/image-20240205165914695.png" alt="image-20240205165914695" style="zoom:50%;" />





![image-20240205170759575](4-nginx的rewrite模块实现.assets/image-20240205170759575.png)

![image-20240205170957699](4-nginx的rewrite模块实现.assets/image-20240205170957699.png)



这就实现了访问不存在的网页跳转到首页了，不过要和80转443共存，可能需要这么修改

![image-20240205171443340](4-nginx的rewrite模块实现.assets/image-20240205171443340.png)

这是80转443的过程，里面也涉及一个不存在的页面(https://www.site1.com/)这个页面本来是应该存在的啊，奇怪了？这么写把原来的默认/就是访问/index.html的行为给搞没了。

![image-20240205172000501](4-nginx的rewrite模块实现.assets/image-20240205172000501.png)

这是不存在页面转的过程👇

![image-20240205171745224](4-nginx的rewrite模块实现.assets/image-20240205171745224.png)



奇怪的点看看日志👇

![image-20240205173230871](4-nginx的rewrite模块实现.assets/image-20240205173230871.png)

![image-20240205173246718](4-nginx的rewrite模块实现.assets/image-20240205173246718.png)

好了不管了





