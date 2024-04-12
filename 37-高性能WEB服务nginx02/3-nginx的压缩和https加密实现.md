# 第3节. nginx的压缩和https加密实现



# 压缩

apache也支持压缩

![image-20240125144151279](3-nginx的压缩和https加密实现.assets/image-20240125144151279.png)

------------------

![image-20240125143945781](3-nginx的压缩和https加密实现.assets/image-20240125143945781.png)

这是👆httpd的压缩篇章



nginx -t查看编译的配置选项

![image-20240125145453478](3-nginx的压缩和https加密实现.assets/image-20240125145453478.png)

配置参数如下：

gzip on|off  开关

gzip_comp_level level     压缩比修改 1到9，9 最高压缩比，9也是最耗CPU的

虽然是1-9，但是压缩的效果也不会差太多

gzip_disable regex ...     匹配客户端浏览器类型就不执行压缩，比如gzip_disable "MSIE[1-6]\\.";

gzip_min_length length   压缩的起点，什么样的大小才开始压缩，比如＞100k的才开始压缩，100Bytes就别压了，压了也没啥效果。



gzip_http_version 1.0|1.1  是支持的http版本

<img src="3-nginx的压缩和https加密实现.assets/image-20240125151438624.png" alt="image-20240125151438624" style="zoom:50%;" />



gzip_buffers number szie   压缩的时候启用多大的缓冲区

gzip_types mime-type ... 针对什么类型的文件进行压缩，默认包含text/html的格式支持，不要写，写了反而报错。



gzip_vary on|off   如果启用压缩了，是否在响应报文首部插入"Vary：Accept-Encoding"

解释：👇就是说server要插入这个标记，代理那边就会存储两个版本的web页，一个压缩一个不压缩，以此来响应支持和不支持压缩的client浏览器。当然c-ser直连的话就看client那边浏览器是否支持压缩了支持-server就发送经过gzip压缩的页面版本。

<img src="3-nginx的压缩和https加密实现.assets/image-20240125160601264.png" alt="image-20240125160601264" style="zoom:50%;" />



gzip_proxied off|expied|no-cache|no-store|private|no_last_modified|no_etag|auth|any ...;

nginx做反代的时候，后端服务器将响应报文发给nginx，nginx再回给用户的时候，是否启用压缩，就靠这个参数来做的。

off  就是nginx返给client的时候不压缩

expied|no-cache|no-store|private|no_last_modified|no_etag|auth|any ...   后端服务器响应报文中带有Cache-Control字段，然后这个字段里的值 写的是这些的时候，就启用压缩。

<img src="3-nginx的压缩和https加密实现.assets/image-20240125163350082.png" alt="image-20240125163350082" style="zoom:50%;" />



配置下

同样，可以了解下gzip的适配模块

![image-20240125163747665](3-nginx的压缩和https加密实现.assets/image-20240125163747665.png)

gzip 的总开关 默认就是off的。其他的就是on后的一些选项



![image-20240125164139869](3-nginx的压缩和https加密实现.assets/image-20240125164139869.png)



观察这个大文件，看下压缩的效果

![image-20240125164525089](3-nginx的压缩和https加密实现.assets/image-20240125164525089.png)

curl可以看到Content-Lenth大小就是文件的大小👇👆

![image-20240125164704005](3-nginx的压缩和https加密实现.assets/image-20240125164704005.png)

既是curl带上压缩功能，也没有，因为server那边没有开启

![image-20240125165131369](3-nginx的压缩和https加密实现.assets/image-20240125165131369.png)



然后server开启压缩功能

![image-20240125164039972](3-nginx的压缩和https加密实现.assets/image-20240125164039972.png)

然后curl看压缩要启用压缩功能的，不启用是没效果的

![image-20240125164917874](3-nginx的压缩和https加密实现.assets/image-20240125164917874.png)

curl带上压缩选项，应该是成功了，只是不再显示Content-Length了。

![image-20240125165242206](3-nginx的压缩和https加密实现.assets/image-20240125165242206.png)

![image-20240125170959291](3-nginx的压缩和https加密实现.assets/image-20240125170959291.png)



所以说压缩比差不多就行了👇

![image-20240125171611395](3-nginx的压缩和https加密实现.assets/image-20240125171611395.png)



# SSL证书，加密

之前httpd也弄过，自签名证书还挺有意思的，实现了client的安全访问的关键点。



然后购买证书这块的情况

阿里云上有全套的购买申请流程，可以学习下，但是买不一定，需要看哪里性价比高。

apache做ssl，server上是配置3个文件：ca证书、服务器证书、服务器私钥

nginx做ssl，服务器证书、服务器私钥



实现ssl是用的ngx_http_ssl_module模块

**①ssl on|off，   ssl开关，不推荐，建议使用listen指令，和httpd一样 listen 443 ssl**

![image-20240126180128632](3-nginx的压缩和https加密实现.assets/image-20240126180128632.png)

httpd就是listen 443 https ，不是一个意思嘛。



**②ssl_certificate_file** 和 **ssl_certificate_key file**

服务器证书文件和私钥文件；从供应商购买后会得到两个文件，一个服务器证书文件，一个私钥key



**③ssl_protocols**

支持的ssl加密协议，默认就行了

**④ssl_session_cache **

加速服务器响应速度的吧应该是，

![image-20240126181350636](3-nginx的压缩和https加密实现.assets/image-20240126181350636.png)



**⑤ssl_session_timeout **

客户端链接可以复用ssl session cache中缓存的有效时长，默认5m

![image-20240126181858915](3-nginx的压缩和https加密实现.assets/image-20240126181858915.png)





## 下面开始实验

还是使用自签名证书咯

之前http的时候用的openssl还是比较多步骤的，现在利用一个Makefile文件来弄，其实就是Makefile封装好了函数，脚本简化成了cli了。

首先你需要一个Makefile文件👇

https://exampleconfig.com/view/openssl-centos7-etc-pki-tls-certs-makefile

①下载Makefile

![image-20240131094126491](3-nginx的压缩和https加密实现.assets/image-20240131094126491.png)

②生成自签名证书

![image-20240131095441316](3-nginx的压缩和https加密实现.assets/image-20240131095441316.png)

上图漏了Common Name，就是要写网站域名的👇

![image-20240131100029436](3-nginx的压缩和https加密实现.assets/image-20240131100029436.png)

但是现在有个问题，每次重启服务都有问题，就是ming.key这个私钥文件是加密的，

![image-20240131100225835](3-nginx的压缩和https加密实现.assets/image-20240131100225835.png)

所以还需要给它解密一下👇

[root@server tls]# openssl rsa -in ming.key -out ming.net.key

![image-20240201092828066](3-nginx的压缩和https加密实现.assets/image-20240201092828066.png)

这样就得到不加密的私钥了

![image-20240201094822255](3-nginx的压缩和https加密实现.assets/image-20240201094822255.png)

将两个文件ming.crt和ming.key放到nginx的配置路径下，再看下权限是否为600

![image-20240201095107653](3-nginx的压缩和https加密实现.assets/image-20240201095107653.png)

证书文件有了，就可以改nginx的配置文件了

![image-20240201103042527](3-nginx的压缩和https加密实现.assets/image-20240201103042527.png)

然后打开浏览器

![image-20240201103332794](3-nginx的压缩和https加密实现.assets/image-20240201103332794.png)

这不就来了嘛

但是不安全没法搞，因为需要下载ming.crt证书到PC，然后导入到受信任的根证书颁发机构

既是导入了可以肯定也是不行的，因为通过

openssl x509 -in ming.crt -text -noout发现这种Makefile的方式并没有X509v3 Subject Alternative Name这个选项，所以导入也没有用的，Makefile需要优化下。

![image-20240201110359655](3-nginx的压缩和https加密实现.assets/image-20240201110359655.png)



#### 尝试优化Makefile文件

搞定

![image-20240201132012102](3-nginx的压缩和https加密实现.assets/image-20240201132012102.png)

哈哈哈，修改Makefile和处理方式如下

1、makefile的修改

![image-20240201132058522](3-nginx的压缩和https加密实现.assets/image-20240201132058522.png)

这里补充一个取消xxx.key生成的时候输入密码的要求，也就是不加密了

![image-20240205150009632](3-nginx的压缩和https加密实现.assets/image-20240205150009632.png)

去掉-aes128就行了

![image-20240205150102097](3-nginx的压缩和https加密实现.assets/image-20240205150102097.png)





2、openssl的配置文件修改

![image-20240201132150628](3-nginx的压缩和https加密实现.assets/image-20240201132150628.png)

3、重新make 一下

```
make ming.crt  # 一键生成ming.key 和 ming.crt

openssl rsa -in ming.key -out ming.net.key  # 去掉ming.key的密码

rm -rf ming.key  # 删除带密码的key

mv ming.net.key ming.key  # 后面使用不带命名的key，重命名一下

mv ming.key ming.crt /etc/nginx/ssl/  # 移动到规范目录下，供nginx调用

[root@server tls]# cat /etc/nginx/conf.d/site.conf

server {
    listen 443 ssl;
    server_name *.site1.com;
    root /data/ssl/;
    ssl_certificate /etc/nginx/ssl/ming.crt;
    ssl_certificate_key /etc/nginx/ssl/ming.key;
    ssl_session_cache shared:sslcache:20m;
    ssl_session_timeout 10m;
    access_log /data/ssl/log/site1.access.log access_json;
}

systemctl restart nginx
```



4、导出ming.crt

5、导入PC

打开浏览器就ok拉



但是通过这个可发发现，现在ssl浏览器验证果然不再看站点了，只看备用名称。

![image-20240201132646727](3-nginx的压缩和https加密实现.assets/image-20240201132646727.png)

你看哦

![image-20240201133435030](3-nginx的压缩和https加密实现.assets/image-20240201133435030.png)

这里👆写错了，其实没关系

肯定正常要一样的

![image-20240201133518104](3-nginx的压缩和https加密实现.assets/image-20240201133518104.png)

这里故意写不一样，就是验证只看Alternative Name

![image-20240201133637170](3-nginx的压缩和https加密实现.assets/image-20240201133637170.png)

![image-20240201133655841](3-nginx的压缩和https加密实现.assets/image-20240201133655841.png)

然后导入ming.crt进入PC

![image-20240201134106371](3-nginx的压缩和https加密实现.assets/image-20240201134106371.png)





## ssl跳转，ngx_http_rewrite_module

![image-20240201143722831](3-nginx的压缩和https加密实现.assets/image-20240201143722831.png)



### 先说if

![image-20240201144906698](3-nginx的压缩和https加密实现.assets/image-20240201144906698.png)



return是返回301这个code，并重定向到性的url

![image-20240201150210312](3-nginx的压缩和https加密实现.assets/image-20240201150210312.png)

上图漏了80监听，否则会实现ssl跳转

![image-20240201150704138](3-nginx的压缩和https加密实现.assets/image-20240201150704138.png)



ok了就，不过更多的不用return，而是用rewrite来做重定向。

![image-20240201150804788](3-nginx的压缩和https加密实现.assets/image-20240201150804788.png)





![image-20240201155221923](3-nginx的压缩和https加密实现.assets/image-20240201155221923.png)





curl验证

![image-20240201150947211](3-nginx的压缩和https加密实现.assets/image-20240201150947211.png)

![image-20240201155111676](3-nginx的压缩和https加密实现.assets/image-20240201155111676.png)

👆看的很清楚跳转了





然后echo的测试要用之前的编译安装的nginx

![image-20240201163825403](3-nginx的压缩和https加密实现.assets/image-20240201163825403.png)

将include移到后面就行了，主配置文件

![image-20240201164012014](3-nginx的压缩和https加密实现.assets/image-20240201164012014.png)

子配置文件👇

![image-20240201164115084](3-nginx的压缩和https加密实现.assets/image-20240201164115084.png)



OK了这样，第一个if重定向到https，然后浏览器继续发起https的请求，然后就命中第二个if，然后就会调用echo得到👇页面了：

![image-20240201164309562](3-nginx的压缩和https加密实现.assets/image-20240201164309562.png)



官网的if例子

![image-20240201165854931](3-nginx的压缩和https加密实现.assets/image-20240201165854931.png)

可以知道if可以控制，比如我的站点不让你curl

比如163肯定就做了if 不让你curl，

![image-20240201165948126](3-nginx的压缩和https加密实现.assets/image-20240201165948126.png)

但其实curl依然可以用，因为163限制curl是通过if来匹配必须要有浏览器，而curl自然可以模拟浏览器

![image-20240201170503176](3-nginx的压缩和https加密实现.assets/image-20240201170503176.png)

### 👆curl通过-AIE来模拟IE浏览器，加上-L支持重定向就行了。一般就可以绕过网站对curl的限制了



![image-20240201173327741](3-nginx的压缩和https加密实现.assets/image-20240201173327741.png)



### 测试if   user_agent

![image-20240201174133014](3-nginx的压缩和https加密实现.assets/image-20240201174133014.png)

改一下不要方括号

![image-20240201174757679](3-nginx的压缩和https加密实现.assets/image-20240201174757679.png)

![image-20240201174808765](3-nginx的压缩和https加密实现.assets/image-20240201174808765.png)

这样就if区分了curl👇

![image-20240201175042477](3-nginx的压缩和https加密实现.assets/image-20240201175042477.png)









![image-20240201175723535](3-nginx的压缩和https加密实现.assets/image-20240201175723535.png)



看下agent怎么自定义出来的👇

![image-20240201175851195](3-nginx的压缩和https加密实现.assets/image-20240201175851195.png)

![image-20240201180109985](3-nginx的压缩和https加密实现.assets/image-20240201180109985.png)



### 看下301 302 307

![image-20240201180529307](3-nginx的压缩和https加密实现.assets/image-20240201180529307.png)

区别：

1、301是永久重定向，是可以缓存的，所以会出现from disk cache。

2、



![image-20240201181641146](3-nginx的压缩和https加密实现.assets/image-20240201181641146.png)

结果第二次变成307 from disk cache了。。。

![image-20240201182145712](3-nginx的压缩和https加密实现.assets/image-20240201182145712.png)



302就是临时性的，一般现在趋势是全站加密，可能301反而更多一些。



307是浏览器内部跳转，不访问页面了，这个怎么关掉呢，就是现在我测试都是

第一次 301 

第二次 然后就307 from disk cache

我想做出出来

第一次 301 

第二次 301 from disck cache



HSTS是啥

![image-20240201183338145](3-nginx的压缩和https加密实现.assets/image-20240201183338145.png)

HSTS是





![image-20240201183554109](3-nginx的压缩和https加密实现.assets/image-20240201183554109.png)



![image-20240201184323204](3-nginx的压缩和https加密实现.assets/image-20240201184323204.png)、、





![image-20240201184336413](3-nginx的压缩和https加密实现.assets/image-20240201184336413.png)



![image-20240201184359321](3-nginx的压缩和https加密实现.assets/image-20240201184359321.png)

![image-20240201184613214](3-nginx的压缩和https加密实现.assets/image-20240201184613214.png)







# 307，导致的内部网站打不开



## 1、背景

www.sw.com 和 sw.com是我内部用的一个域名 都解析到了192.168.0.9，是内部的soft站点，套了nginx，只监听了80



但是有一天突然 我的笔记本 访问www.sw.com或sw.com的时候打不开，**无痕可以👇**，无痕不用本地的HSTS列表所以可以。

![image-20240412164334777](3-nginx的压缩和https加密实现.assets/image-20240412164334777.png)

题外话：就是这里有一个307 把444重定向为80的动作，和本段研究内容无关。



进一步发现：即使手动输入http://www.sw.com也会307到https://www.sw.com，**页面直接打不开👇**，是一个报错页面

![image-20240412162514278](3-nginx的压缩和https加密实现.assets/image-20240412162514278.png)

红色的www.sw.com其实就是被307到了https://www.sw.om也就是 浏览器地址栏 里的 https👆



然后手动输入http://sw.com，**网页打开不全👇**，原因就是http://sw.com里的部分图片，框架，走的还是http://www.sw.com；而http://www.sw.com就会被307到https://www.sw.com，多了个s。

![image-20240412164147207](3-nginx的压缩和https加密实现.assets/image-20240412164147207.png)

![image-20240412164155547](3-nginx的压缩和https加密实现.assets/image-20240412164155547.png)









## 2、原因

就是HSTS导致的

1、浏览器里的

chrome://net-internals/#hsts

可以查到www.sw.com存在的，于是就是会被浏览器直接给你在浏览器内部做了307了，很骚啊



删掉就可以了



2、为什么其他电脑的

chrome://net-internals/#hsts

里没有www.sw.com



分析：可能是www.sw.com这个，不是分析了，我实测了一下，就是你用代理访问了www.sw.com就会在

chrome://net-internals/#hsts这个里面生成www.sw.com的HSTS

![image-20240412163019479](3-nginx的压缩和https加密实现.assets/image-20240412163019479.png)

![image-20240412163037576](3-nginx的压缩和https加密实现.assets/image-20240412163037576.png)

找到原因了，并且是可以复现的故障现象，very nice



**进一步分析**，就是Strict-Transport-Security，就是这个互联网的www.sw.com的服务器端给了这个回应选项导致的。

![image-20240412163232057](3-nginx的压缩和https加密实现.assets/image-20240412163232057.png)



这一点前面的章节里也有提到，不过没有这次故障处理后得到的逻辑清晰

![image-20240412163453541](3-nginx的压缩和https加密实现.assets/image-20240412163453541.png)







所以要复现故障

1、打开代理

2、访问www.sw.com    # 拿到HSTS

3、关闭代理

4、访问www.sw.com		# 得到网页直接打不开的 故障现象1

5、访问http://sw.com    	# 得到网页打不开不全的 故障现象2







### 再研究下上面说的题外话，为什么我内部的www.sw.com会443跳80：

![image-20240412171120039](3-nginx的压缩和https加密实现.assets/image-20240412171120039.png)







![image-20240412171027692](3-nginx的压缩和https加密实现.assets/image-20240412171027692.png)







![image-20240412171322474](3-nginx的压缩和https加密实现.assets/image-20240412171322474.png)







**分析判断：** 

1、收到307 临时 重定向，就会转到location的URL上，那么local 的url哪里定义的？还以为什么会收到307 临时 重定向 ？

<img src="3-nginx的压缩和https加密实现.assets/image-20240412171413111.png" alt="image-20240412171413111" style="zoom:40%;" />

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/307



location 首部 从浏览器可知就是http://www.sw.com👇

![image-20240412171631630](3-nginx的压缩和https加密实现.assets/image-20240412171631630.png)



我觉的location 首部 这里找找，也许默认就是server name 和 listen的组合也就是http://www.sw.com👇

![image-20240412171544913](3-nginx的压缩和https加密实现.assets/image-20240412171544913.png)



为什么会产生307 临时重定向

![image-20240412172526104](3-nginx的压缩和https加密实现.assets/image-20240412172526104.png)







![image-20240412172724958](3-nginx的压缩和https加密实现.assets/image-20240412172724958.png)







无痕测试

1、输入sw     按ctrl enter，也就是访问https://www.sw.com

2、但是server那边不管是accesslog和errorlog都没有看到443的访问，也正常，因为443的流量就进不来，没监听啊

3、所以这个是浏览器自身的行为，一定是，瞬间就调到80了👇 

![image-20240412173111004](3-nginx的压缩和https加密实现.assets/image-20240412173111004.png)



有可能就是它的回退既然要到处https，万一不行，总要回退吧，我认为就是这样的。

![image-20240412173500428](3-nginx的压缩和https加密实现.assets/image-20240412173500428.png)





就是浏览的自身行为👇

![image-20240412174444478](3-nginx的压缩和https加密实现.assets/image-20240412174444478.png)

10.9压根没有HTTP的报文回过去，在80 connect 联通之前。所以"307 Temporary Redirect" 我判断是浏览器的自身行为---当

①你输入的是www.sw.com的时候由于未指定http还是https，所以它(可能就是https everywhere) 优先让你用https，

②但是多次被拒绝后，回退到http了；

③如果你直接输入https://www.sw.com，由于明确制定了，所以就打不开啦；

④如果你直接输入http://www.sw.com，由于明确指定了http，所以就没有类似https everywhere的事啦就不会有**"307 Temporary Redirect"**啦，直接就打开啦。





### 另外，无痕 里 两种 输入方式，普通模式里一样

1、www.sw.com   不是每次都能看到优先https，很多时候是没有说给你先https去443连接的，就直接http了。            # 这是因为这种行为，浏览器会参考历史数据的，删掉涉及sw的历史记录，www.sw.com 就会和 👇下面的方式一样了，也能看到优先HTTPS了就。

2、输入sw，然后ctrl + enter，每次都是优先https    #  这种行为不会，就直接补上HTTPS了优先了。





<img src="3-nginx的压缩和https加密实现.assets/image-20240412181535173.png" alt="image-20240412181535173" style="zoom:50%;" />

**历史记录 不等于 缓存 ！**



![image-20240412180429765](3-nginx的压缩和https加密实现.assets/image-20240412180429765.png)





### 无痕有历史url的证据👇

![image-20240412180653541](3-nginx的压缩和https加密实现.assets/image-20240412180653541.png)

一旦这些历史url在，也是www.sw.com能够自动补出来，www.sw.com就会直接使用http://www.sw.com去访问，因为历史数据里成功的就是http吗。而输入 sw 再ctrl + enter 就是快速提交内容，而且通过测试多次测试多次频繁测试可以说这种就是不会被历史数据影响的手段。就好像ctrl+r 一样是不用缓存刷新一样。





### 无痕开两个，第二个无痕访问第一个无痕的url没有意义了就，等于不是无痕了，所以无痕测试的时候要关闭所有浏览器窗口包括无痕

这里不演示了就，多次体验过了已经。





# 最后总结

307 Temprorary Redirect
307 Internal Redirect

浏览器的输入手法

无痕测试注意



最终都是浏览器内部的行为，但Internal才是和server交互出来的内部配置，而Temprorary才是真正内部就有的只需要server拒绝浏览器多次请求就行，是一种回退类似。



307 Internal Redirect，是网站server回应里的Strict-Transport-Security键值将浏览器的HSTS列表里[chrome://net-internals/#hsts]加上了，所以你怎么输入都会让你直接走HTTPS



307 Temprorary Redirect，是浏览器自身行为，是一种回退行为，特别是443被拒后的80回退



浏览器由于内置机制，当你输入www.sw.com，未指定http还是https的时候，会优先https的，但是也要和输入手法结合在一起的👇

​		浏览器里的历史记录会影响url输入的行为，比如有www.sw.com的记录，是http://www.sw.com,那么你输入www.sw.com就会走http:了

​		但是地址栏里输入sw，后ctrl + enter就不会 被历史记录的影响。此时就会优先Https，然后多次被拒后才会307 Temprorary Redirect回退到http



无痕测试只能一个无痕窗口，否则还是会利用其他无痕里的缓存。
