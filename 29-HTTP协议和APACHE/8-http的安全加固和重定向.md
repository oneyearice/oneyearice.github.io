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







