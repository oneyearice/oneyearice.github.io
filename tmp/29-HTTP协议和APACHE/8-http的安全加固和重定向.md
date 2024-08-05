# 第8节. http的安全加固和重定向



书接上文的一些其他注意点



## 测试基于https访问相应的主机



### 浏览器验证https，呵呵

![image-20231013134017446](8-http的安全加固和重定向.assets/image-20231013134017446.png)



### openssl 验证https主机

openssl s_client [-connect host:port] [-cert filename] [-CApath directory] [-CAfilefilename]

其中-cert filename应该是client的证书，这个不用写，一般server也不会验证client证书。

![image-20231013135124415](8-http的安全加固和重定向.assets/image-20231013135124415.png)![image-20231013134840428](8-http的安全加固和重定向.assets/image-20231013134840428.png)![image-20231013135207956](8-http的安全加固和重定向.assets/image-20231013135207956.png)![image-20231013135301423](8-http的安全加固和重定向.assets/image-20231013135301423.png)

上图是拼接出来的哦，无缝衔接，一字不拉，牛逼不~

如果不带ca根证书  -CApath . -CAfile cacert.pem   就会error的哦 没有上图的绿色ok了，顺带一提绿色是MobaXterm自带的着色。

![image-20231013135938820](8-http的安全加固和重定向.assets/image-20231013135938820.png)

![image-20231013135609525](8-http的安全加固和重定向.assets/image-20231013135609525.png)

![image-20231013135622839](8-http的安全加固和重定向.assets/image-20231013135622839.png)

![image-20231013135633718](8-http的安全加固和重定向.assets/image-20231013135633718.png)

以上就是原来绿色ok处的对应截图，现在都是error和 Verify return code 19报错了，原来是Verify return code code 0 ok。





### curl验证https

![image-20231013133934784](8-http的安全加固和重定向.assets/image-20231013133934784.png)

#### curl 带上根证书进行访问https网页

![image-20231013134451655](8-http的安全加固和重定向.assets/image-20231013134451655.png)





# 理解下目前实验页面显示状况

为什么https://www.a.com和http://www.a.com内容一样

![image-20231013141817589](8-http的安全加固和重定向.assets/image-20231013141817589.png)



![image-20231013141742125](8-http的安全加固和重定向.assets/image-20231013141742125.png)



①首先按https://www.a.com走的是/etc/httpd/conf.d/ssl.conf里的虚拟主机

![image-20231013140833841](8-http的安全加固和重定向.assets/image-20231013140833841.png)

而此处没有定影DocumentRoot的路径，,所以去看之前我们定义的/etc/httpd/conf.d/test.conf，要定义DocumentRoot，自然/etc/httpd/conf/httpd.conf下的DocumentRoot要注释掉的

![image-20231013141106103](8-http的安全加固和重定向.assets/image-20231013141106103.png)

其实我感觉规范的做法就是虚拟主机也配置一遍自己的，不管是否利用默认的，都配一边方便维护。

![image-20231013141144772](8-http的安全加固和重定向.assets/image-20231013141144772.png)

![image-20231013141223198](8-http的安全加固和重定向.assets/image-20231013141223198.png)

然后②看下http://www.a.com走的是/etc/httpd/conf.d/test.conf里的虚拟主机

![image-20231013141351417](8-http的安全加固和重定向.assets/image-20231013141351417.png)

浏览器里输入的是http://www.a.com，所以先看80，再看head里的host是www.a.com所以走的是/data/www/asite下的xx.txt，如果没有xx.txt就是index.html了

![image-20231013141505537](8-http的安全加固和重定向.assets/image-20231013141505537.png)

所以结合①和②，https和http都是一个页面啦



有人说：https不能说配置多个"基于主机头的-虚拟主机"。我按他的思路理解是因为https加密范围是包含了head字段的，虚拟主机处理的时候再ssl卸载之前了，这也是我猜的，尴尬的点在于server具备卸载ssl证书的能力，也就是可以看到主机头，为什么不能针对多个https站点用基于head里的host来实现路由呢。除非

<img src="8-http的安全加固和重定向.assets/image-20231013150527770.png" alt="image-20231013150527770" style="zoom:37%;" />

废话不多说，上实验

![image-20231013143806986](8-http的安全加固和重定向.assets/image-20231013143806986.png)

为了便于对比，将原来的https://www.a.com的页面改成index.html不再使用m.txt

![image-20231013144012207](8-http的安全加固和重定向.assets/image-20231013144012207.png)

![image-20231013144321180](8-http的安全加固和重定向.assets/image-20231013144321180.png)



![image-20231013144351822](8-http的安全加固和重定向.assets/image-20231013144351822.png)

这不是好好的嘛

![image-20231013144543681](8-http的安全加固和重定向.assets/image-20231013144543681.png)



![image-20231013145509556](8-http的安全加固和重定向.assets/image-20231013145509556.png)



www.b.com复制的www.a.com虚拟主机的配置，证书要改，否则 subject alternative name里的没有www.b.com网站地址，不能认为安全的👇

![image-20231013145459011](8-http的安全加固和重定向.assets/image-20231013145459011.png)





只不过www.b.com没有颁发证书而已。搞一个

①生成csr文件

![image-20231013144817447](8-http的安全加固和重定向.assets/image-20231013144817447.png)



②ca颁发一下

![image-20231013144917189](8-http的安全加固和重定向.assets/image-20231013144917189.png)

发现没有Subject Alternative Name，所以要去/etc/pki/tls/openssl.conf下去配置一下

![image-20231013145023975](8-http的安全加固和重定向.assets/image-20231013145023975.png)

然后就有了

![image-20231013145119315](8-http的安全加固和重定向.assets/image-20231013145119315.png)

回到server上配置好www.b.com的证书文件的加载，同样和www.a.com的一样，在虚拟主机里配置，在/etc/httpd/conf.d/ssl.conf里配置



![image-20231013145641249](8-http的安全加固和重定向.assets/image-20231013145641249.png)

只需要改一行👆





PC上补一个hosts记录

<img src="8-http的安全加固和重定向.assets/image-20231013145312768.png" alt="image-20231013145312768" style="zoom:33%;" />

![image-20231013145356152](8-http的安全加固和重定向.assets/image-20231013145356152.png)



然后就好了啊

![image-20231013145746384](8-http的安全加固和重定向.assets/image-20231013145746384.png)

![image-20231013145803638](8-http的安全加固和重定向.assets/image-20231013145803638.png)



所以通过实验不是发现了https的多站点，可以基于head主机头host字段进行配置多个虚拟主机啊。

![image-20231013150141363](8-http的安全加固和重定向.assets/image-20231013150141363.png)

我就觉的server自己有能力卸载ssl证书-解密，而且卸载的证书和servername都在一起配置的👆，为啥不能基于主机头进行https的多虚拟主机配置呢，对吧。实验证明就是可以的。





# 下面实现http跳转https



### 首先知道一个chrome的默认行为，

chrome浏览器会自动给你补上https的，比如

使用联想浏览器输入a.com

![image-20231013153048316](8-http的安全加固和重定向.assets/image-20231013153048316.png)



使用chrome输入a.com，**只有chrome会干这事，其他浏览器不会说 输入xx.xx.com自动用https://**

![image-20231013153149256](8-http的安全加固和重定向.assets/image-20231013153149256.png)



使用火狐，结果发现火狐的证书好像识别可能不是OS的

![image-20231013153223645](8-http的安全加固和重定向.assets/image-20231013153223645.png)

![image-20231013153519526](8-http的安全加固和重定向.assets/image-20231013153519526.png)



![image-20231013153546678](8-http的安全加固和重定向.assets/image-20231013153546678.png)



![image-20231013153608605](8-http的安全加固和重定向.assets/image-20231013153608605.png)



![image-20231013153715450](8-http的安全加固和重定向.assets/image-20231013153715450.png)



![image-20231013153744323](8-http的安全加固和重定向.assets/image-20231013153744323.png)



这样就加载进去了，火狐果然不是用的OS的受信任的证书颁发机构。

然后火狐浏览器就不再报不安全了

![image-20231013153811813](8-http的安全加固和重定向.assets/image-20231013153811813.png)

然后火狐的行为

![image-20231013154119324](8-http的安全加固和重定向.assets/image-20231013154119324.png)



![image-20231013154035221](8-http的安全加固和重定向.assets/image-20231013154035221.png)



输入https://www.a.com或https://www.a.com/都一样。

![image-20231013154235922](8-http的安全加固和重定向.assets/image-20231013154235922.png)



## 然后正儿八经开始做server的http重定向到https

区别上面说的Client的chrome的默认https行为，防止误判

<img src="8-http的安全加固和重定向.assets/image-20231013154806030.png" alt="image-20231013154806030" style="zoom:33%;" />

两种重定向：301和302👆，通过语句里的[status]区别配置

![image-20231013155242899](8-http的安全加固和重定向.assets/image-20231013155242899.png)

Location：http://www.jd.com就是301重定向过去的网址。



![image-20231013155437720](8-http的安全加固和重定向.assets/image-20231013155437720.png)



![image-20231013161503514](8-http的安全加固和重定向.assets/image-20231013161503514.png)

![image-20231013161622695](8-http的安全加固和重定向.assets/image-20231013161622695.png)

活久见http跳http，浏览器错误显示了吧





![image-20231013162032302](8-http的安全加固和重定向.assets/image-20231013162032302.png)



![image-20231013162205195](8-http的安全加固和重定向.assets/image-20231013162205195.png)

![image-20231013162221864](8-http的安全加固和重定向.assets/image-20231013162221864.png)



jd就是两次跳转，一次301 从http://www.360buy.com/跳到http://www.jd.com，一次302从http://www.jd.com跳到https://www.jd.com;  也就是第一次是 不同网站的http跳转-301；第二次是相同站点的http跳https。



![image-20231013160105742](8-http的安全加固和重定向.assets/image-20231013160105742.png)



<img src="8-http的安全加固和重定向.assets/image-20231013160350314.png" alt="image-20231013160350314" style="zoom:50%;" />





<img src="8-http的安全加固和重定向.assets/image-20231013160645510.png" alt="image-20231013160645510" style="zoom:33%;" />

好像后台自动给你转，就是一次请求就行了，好像也是有此类技术的。



## 实验

访问www.a.com跳转到www.b.com



![image-20231013161339019](8-http的安全加固和重定向.assets/image-20231013161339019.png)

将上面的访问都跳转到https://www.b.com



![image-20231013163606039](8-http的安全加固和重定向.assets/image-20231013163606039.png)



再把https://www.a.com也重定向了

![image-20231013163906812](8-http的安全加固和重定向.assets/image-20231013163906812.png)



![image-20231013164004810](8-http的安全加固和重定向.assets/image-20231013164004810.png)



这样就实现了http://www.a.com跳http://www.b.com;https://www.a.com跳https://www/b.com

![image-20231013164228767](8-http的安全加固和重定向.assets/image-20231013164228767.png)



这样就实现了http://www.a.com跳https://www.b.com

![image-20231013164642621](8-http的安全加固和重定向.assets/image-20231013164642621.png)

![image-20231013164707496](8-http的安全加固和重定向.assets/image-20231013164707496.png)



curl -L 的说明，curl默认行为就是只发一次请求，-L就会follow redirects，就是会跟随重定向们，多次也没问题

![image-20231013164939957](8-http的安全加固和重定向.assets/image-20231013164939957.png)



![image-20231013170307585](8-http的安全加固和重定向.assets/image-20231013170307585.png)



![image-20231013171330534](8-http的安全加固和重定向.assets/image-20231013171330534.png)

![image-20231013171427080](8-http的安全加固和重定向.assets/image-20231013171427080.png)

如果你再在上图上地址栏里敲一下回车，由于是chrome就会导致你访问的其实不再是http://www.b.com而TMD的是https://www.b.com了然后就会走别的虚拟主机了，重定向的配置就不生效了，或者别的地方的重新向了，我的配置如下

![image-20231013171747153](8-http的安全加固和重定向.assets/image-20231013171747153.png)

![image-20231013171818629](8-http的安全加固和重定向.assets/image-20231013171818629.png)

好烦呐，翻来覆去的，(￣▽￣)"，反正这里记住，处理思路就是类似网络转发：PHB，per-hop-behavior，这里也是一样一跳一跳的路由转发查询就行了。





![image-20231016101757370](8-http的安全加固和重定向.assets/image-20231016101757370.png)

淘宝用的301永久重定向，其他大多数用的是302临时重定向。

然后就是搜索引擎比如百度，就不会抓取301重定向前的地址页面了，302的跳转前页面还会抓的。因为301永久重定向被被认为是废弃的地址了不用了，也就不会去这个页面抓取内容了。



# 不做虚拟主机进行ssl跳转

恢复实验环境①移除/etc/httpd/conf.d/test.conf②恢复/etc/httpd/conf.d/ssl.conf里的配置，证书相关保留③恢复/etc/httpd/conf/httpd.conf里的配置

![image-20231016105055797](8-http的安全加固和重定向.assets/image-20231016105055797.png)

![image-20231016105109637](8-http的安全加固和重定向.assets/image-20231016105109637.png)

修改如下👇直接在主配置文件里补上重定向的配置就行redirect temp / https://www.a.com

![image-20231016105646725](8-http的安全加固和重定向.assets/image-20231016105646725.png)



![image-20231016110029244](8-http的安全加固和重定向.assets/image-20231016110029244.png)

最多跳转50次

![image-20231016110444897](8-http的安全加固和重定向.assets/image-20231016110444897.png)

产生这样的跳转的原因不太清楚，解决方法是换一种配置方式

```shell
RewriteEngine on
RewriteRule ^(/.*)$ https://%{HTTP_HOSTS}$1 [redirect=302]
```

![image-20231016111619372](8-http的安全加固和重定向.assets/image-20231016111619372.png)

重启服务后生效

![image-20231016111653439](8-http的安全加固和重定向.assets/image-20231016111653439.png)



# HSTS：

## ①HSTS:HTTP Strict Transport Security



解决如下问题：

![image-20231016113158169](8-http的安全加固和重定向.assets/image-20231016113158169.png)

除非第一次就给你劫持了，否则还是比较安全的，靠的是本地缓存，如果缓存被清空或者到期或缓存时间设置较短，还是存在安全风险的具体逻辑如下：



![image-20231016114024622](8-http的安全加固和重定向.assets/image-20231016114024622.png)



HSTS -- HTTP Strict Transport Security 使用该技术的案例👇。其他网址看了下，用的不多。

![image-20231016114737887](8-http的安全加固和重定向.assets/image-20231016114737887.png)

老化时间是1小时，也就是重定向缓存1小时



## ②HTST preload list



是Chrome浏览器中的HSTS预载入列表，在该列表中的网站，使用Chrome浏 览器访问时，会自动转换成HTTPS。Firefox、Safari、Edge浏览器也会采用这个列表



这里和chrome浏览器自动给你补Https的行为还不是一回事。

**①chrome你输入www.baidu.com会自动给你补https://的**

![image-20231016120045397](8-http的安全加固和重定向.assets/image-20231016120045397.png)



![image-20231016120206079](8-http的安全加固和重定向.assets/image-20231016120206079.png)



注意这里不存在重定向的，要输入http://www.baidu.com才会有302出现

当然curl 也不会触发302，有点奇怪，

![image-20231016120329760](8-http的安全加固和重定向.assets/image-20231016120329760.png)



**②chrome里的hsts列表，这是preload预加载的，**你输入http://www.bing.com内部就给你转成https://www.bing.com

![image-20231016120415272](8-http的安全加固和重定向.assets/image-20231016120415272.png)

使用无痕单开模式输入http://bing.com，不要加www哦，因为上图的found清单里并没有www.bing.com只有bing.com

然后通过F12查看是不是直接就是https出去了

![image-20231016133553512](8-http的安全加固和重定向.assets/image-20231016133553512.png)



**③就是各种重定向了，301、302、307之类的**

但是加上www，由于不在chrome浏览器的hsts清单里所以还是走的服务的重定向不过这里是307

当我在无痕里输入http://www.bing.com的时候，通过F12看到的是307，内部重定向。

![image-20231016120600435](8-http的安全加固和重定向.assets/image-20231016120600435.png)





![image-20231016134221349](8-http的安全加固和重定向.assets/image-20231016134221349.png)

![image-20231016134443449](8-http的安全加固和重定向.assets/image-20231016134443449.png)

把字符集改一下

![image-20231016134752275](8-http的安全加固和重定向.assets/image-20231016134752275.png)

没改过来！

好像是开启重定向就会这样，下图👇是不用虚拟主机方式的写法，会造成循环重定向，这里仅仅测试重定向对字符集的影响。

![image-20231016135222140](8-http的安全加固和重定向.assets/image-20231016135222140.png)



重定向都关了就好了

![image-20231016135252366](8-http的安全加固和重定向.assets/image-20231016135252366.png)

这些cil进一步研究可以查看官方手册https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html#rewriterule



# 正向代理和反向代理



![image-20231016140751549](8-http的安全加固和重定向.assets/image-20231016140751549.png)

图片不多说了，补充要给一般正向代理往往会加一个缓存功能，就是出口代理缓存服务器，比如城域网出口想必是有缓存的。比如内部的Nexus私网原站(pip源、yum源、npm源、docker源)，然后client都将源指向nexus代理服务器，所有的基于各种源的软件安装都会从nexus代理走，然后凡是从nexus下载过的软件都会在nexus本地缓存起来，这样，下一次其他人通过该代理下载就会不用再去互联网请请求了。



同样理论上也是可以在代理服务器上做acl，针对某个用户拒绝代理服务也是可以的。



反向代理服务器，通常就是一个服务器 服务不好 N多个用户了，才需要多个服务器分担。所谓服务不好主要指用户量大了，或者用户地理位置分散-服务器需要有就近服务需求。   类似myslq读写分离的调度器



正向代理，加速、缓存

反向代理，均衡、调度

正反都是欺骗用户，欺骗不太好，用户也知道有这些东西，或者叫都是代理了server服务来着。



**正向代理软件，squid老牌正向代理软件，web cache咯**http://www.squid-cache.org/有需要再研究吧

反向代理如那件，nginx较多，apache有但是用的很少，还有LVS、还有F5、HAproxy。





## 实验-apache的反向代理





一般会将背后真正的提供服务的服务器叫做real server👇

![image-20231016143736459](8-http的安全加固和重定向.assets/image-20231016143736459.png)



再设置一下反向代理服务器

### apache的反向代理-常规

```shell
ProxyPass "/" "http://192.168.126.130/"      #  client访问 我"/" 转成 身后 realServer 130
ProxyPassReverse "/" "http://192.168.126.130/"   # 身后realServer回包我，将130转成我自己"/"
```

👆就是中间代理，两头欺骗，不好意思我又用了欺骗一词。因为我找不到更简单词用来替代了。承接？也不明显。用的好就是透明代理，恶意的就是欺骗了吧。

![image-20231016150916149](8-http的安全加固和重定向.assets/image-20231016150916149.png)

记得重启131的httpd服务



然后就可以测试了，测试之前打开realserver和proxy的log

client上不知道真正的服务器s👇：访问的是proxySer

![image-20231016151151661](8-http的安全加固和重定向.assets/image-20231016151151661.png)

proxyser上知道真正的c和s👇：看到的是真正的client IP

![image-20231016151205264](8-http的安全加固和重定向.assets/image-20231016151205264.png)

realSer上不知道真正的c👇：看到的是proxy访问过来的

![image-20231016151223918](8-http的安全加固和重定向.assets/image-20231016151223918.png)



### apache的反向代理-特定URL

```
ProxyPass "/images"	"http://www.example.com/"
ProxyPassReverse "/images"	http://www.example.com/
```

找个image测试

<img src="8-http的安全加固和重定向.assets/image-20231016152134433.png" alt="image-20231016152134433" style="zoom:50%;" />



![image-20231016152240126](8-http的安全加固和重定向.assets/image-20231016152240126.png)





修改proxySer的配置，只针对特定url进行转发过去

![image-20231016152737052](8-http的安全加固和重定向.assets/image-20231016152737052.png)



测试，一般不转发ok

![image-20231016153732442](8-http的安全加固和重定向.assets/image-20231016153732442.png)



测试，特殊url转发ok

![image-20231016153750701](8-http的安全加固和重定向.assets/image-20231016153750701.png)

测试，特殊url转发ok

![image-20231016153803544](8-http的安全加固和重定向.assets/image-20231016153803544.png)



### apache的反向代理-虚拟主机处配置

这个就有点像nginx了哦，呵呵，基于主机头的负载均衡，后端用端口区分，也可以用nodes的ip区分，貌似可行。

```
<VirtualHost *>
ServerName www.magedu.com  
ProxyPass / http://localhost:8080/  
ProxyPassReverse / http://localhost:8080/
</VirtualHost>

```

浅测一下

![image-20231016154557362](8-http的安全加固和重定向.assets/image-20231016154557362.png)



配置proxySer

![image-20231016155050333](8-http的安全加固和重定向.assets/image-20231016155050333.png)





修改windows这个client上的hosts 解析到proxySer上

![image-20231016154835182](8-http的安全加固和重定向.assets/image-20231016154835182.png)



测试负载均衡

![image-20231016155112669](8-http的安全加固和重定向.assets/image-20231016155112669.png)

还不错啊，貌似用apache一样做反向代理，不过nginx用的比较多，apache用的少，所以就是那家饭店吃饭人多去哪家就是这个朴实无华的道理。





# Sendfile机制-属于零复制技术里的一种

先复习

![image-20231016163224856](8-http的安全加固和重定向.assets/image-20231016163224856.png)



一直看到

<img src="8-http的安全加固和重定向.assets/image-20231016172825931.png" alt="image-20231016172825931" style="zoom:50%;" />

然后上图👆结合下图理解一遍👇

<img src="8-http的安全加固和重定向.assets/image-20231016172807156.png" alt="image-20231016172807156" style="zoom:50%;" />



4次用户态和内核态的交互如图👇

<img src="8-http的安全加固和重定向.assets/image-20231016174113530.png" alt="image-20231016174113530" style="zoom:33%;" />

<img src="8-http的安全加固和重定向.assets/image-20231016174302592.png" alt="image-20231016174302592" style="zoom:33%;" />



图中的④未标注，图中的socket buffer到httpd，是内核态到用户态的切换，但是应该不是③的write()返回。

简单理解就是确实存在4次左右的用户态和内核态的交互。这样就发现数据的传递存在不必要的处理过程--数据本来就在内核态里非要从用户态绕一圈没必要，于是sendfile就来了。

![image-20231016174831476](8-http的安全加固和重定向.assets/image-20231016174831476.png)

简单来讲就是👇下图的绿色箭头，不过socketbuffer好理解，协议栈难不成算到接口的队列缓存里的？

<img src="8-http的安全加固和重定向.assets/image-20231016175054302.png" alt="image-20231016175054302" style="zoom:33%;" />



apache默认就启用了该👆sendfiler机制。

该技术不仅仅属于apache，nginx里也有。



sendfiler技术又名**零复制**，其意思就是没有将数据从内核空间复制到用户空间。

具体零复制，还涉及一些别的技术。

<img src="8-http的安全加固和重定向.assets/image-20231016175732158.png" alt="image-20231016175732158" style="zoom:50%;" />



