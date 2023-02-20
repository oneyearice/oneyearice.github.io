# 第3节. DNS主服务器实现



dnsmasq就是比较简单的hosts复用就行

bind的它分域，针对一个一级域名进行其下域名解析记录的编写。

前2节内容说白了，就是两行的事

<img src="3-DNS主服务器实现.assets/image-20230215174217781.png" alt="image-20230215174217781" style="zoom:60%;" /> 



下面继续了解其他配置，针对xxx.com某个域进行dns服务，

### 第一步，打开主配置文件

配置文件类似C预言风格，都是;分号结尾的

其中有options语句块，用{}花括号括起来的。定义了dns的选项

<img src="3-DNS主服务器实现.assets/image-20230215174649803.png" alt="image-20230215174649803" style="zoom:50%;" /> 

下面还有logging日志的语句块。

<img src="3-DNS主服务器实现.assets/image-20230215174703727.png" alt="image-20230215174703727" style="zoom:50%;" /> 



要建立的DNS域信息，区域zone就是针对某个域建立dns解析用的。

<img src="3-DNS主服务器实现.assets/image-20230215174750392.png" alt="image-20230215174750392" style="zoom:50%;" />  

zone : 

oneyearice.asia <---> dbfile1			   #   zone就是一级域名对应的数据库文件

xkyearsice.asia<--->dbfile2  # 这是第二个区域

<img src="3-DNS主服务器实现.assets/image-20230215181130406.png" alt="image-20230215181130406" style="zoom:50%;" /> 

上图的file就是数据库的文件名 而且是根，根的tye就是hint，主dns就是写master。这里的file只写了名称，路径没有写。路径在option里定义的👇

<img src="3-DNS主服务器实现.assets/image-20230216091730061.png" alt="image-20230216091730061" style="zoom:50%;" /> 

还有dns是主还是从

图中的.就是根咯，因为这里填的是域名，而根的域名就是个.点，所以file指向的是named.ca文件，该文件里放的及时13个根域。



根据option里的directory 指定的路径，和zone里自己定义的file 名称，创建数据库文件

![image-20230216101456504](3-DNS主服务器实现.assets/image-20230216101456504.png)

![image-20230216101545405](3-DNS主服务器实现.assets/image-20230216101545405.png)



![image-20230216101750222](3-DNS主服务器实现.assets/image-20230216101750222.png)

既然以named用户运行，那么数据库就要有访问权限





![image-20230216105821734](3-DNS主服务器实现.assets/image-20230216105821734.png)



![image-20230216105832833](3-DNS主服务器实现.assets/image-20230216105832833.png)





格式

<img src="3-DNS主服务器实现.assets/image-20230216103443548.png" alt="image-20230216103443548" style="zoom: 33%;" /> 

name就是域名

TTL是域名的缓存时间，单位是秒

IN就是固定格式，internet记录，这里固定格式，其实dns还可以做别的工作，

rr_type常规就是A记录，AAAA ipv6地址记录，

value就是IP地址，当然如果是PTR的类型，前面的name就是IP地址，这里的value值就是域名了。



这些就是rrtype类型：除此之外还有PTR--ip解析成域名的记录，也就是A记录的反向动作。

![image-20230216103124850](3-DNS主服务器实现.assets/image-20230216103124850.png)

其中rr_type灰常重要的一种是SOA类型

数据库中可以有很多条记录，SOA必须是第一条记录

<img src="3-DNS主服务器实现.assets/image-20230216105036181.png" alt="image-20230216105036181" style="zoom:50%;" /> 

SOA就是对该域的初始定义，比如当前域由哪个主dns服务器对外提供服务;写清 主从 主从数据库同步的方式，管理员邮箱 故障通知，

可以使用freenom获得免费域名，然后NS到CF，在使用CF的免费CDN就挺好的。





开始编写，参考模板

![image-20230216110141406](3-DNS主服务器实现.assets/image-20230216110141406.png)



![image-20230216110358341](3-DNS主服务器实现.assets/image-20230216110358341.png)

接着修改下

<img src="3-DNS主服务器实现.assets/image-20230216111621421.png" alt="image-20230216111621421" style="zoom:50%;" />  

下面的NS、A、AAAA分别一行行的，是3条解析记录

![image-20230216112907544](3-DNS主服务器实现.assets/image-20230216112907544.png)

上图的SOA记录解释<img src="3-DNS主服务器实现.assets/image-20230216113413149.png" alt="image-20230216113413149" style="zoom:50%;" /> 

---------------

1、name就是@这个，写全就是oneyearice.asia.   注意最后一个点，不写就当作前缀会自动给你补上zone定义的一级域名oneyearice.asia   也就说写成oneyearice.asia等价于oneyearice.asia.oneyearice.aisa。结尾加上.就不会自动再补了。

然后，@就是zone里指过来的一级域名就可以用@代替。



2、TTL，可以用第一行定义的$TTL 1D，就不用再写了；也可以不用它的，采用自定义比如@ 864000 这样就是10天的缓存



3、IN照抄

4、类型rr_type就是SOA啦

5、value，soa的值不是一个IP地址，而是一个定义的集合，包括

​		①当前这个域的主DNS服务器



![image-20230216134215489](3-DNS主服务器实现.assets/image-20230216134215489.png)

理论上面的master.oneyearice.asia.可以简写为master，不是说自动补齐嘛，为啥写这么长呢。

通过这里再次明确根就是<font color=red size=5>.</font>咯，

![image-20230216134620318](3-DNS主服务器实现.assets/image-20230216134620318.png)



下面的

0 ：serial  表示当前区域数据文件的版本号，数字越大，表示当前内容越新。当我们修改了记录后，就需要手动的递增该版本号。

这个版本的作用就是：主从同步的判断依据，同步考的就是主上的版本号变大了（自己和自己比较，MMP，这同步的思路牛逼了）。只看版本号，不看内容 不看哈希，就看数字。数字还是人写的。牛逼~，看来还得辅以脚本自动判断哈希值 递增主的版本号。



1D：refresh 是从服务器 1 天 去主服务器上拉一次数据库信息。这么理解的，关于主从的同步 分为 推 和 拉

推就是主发现自己的版本号变了，而且是变大了，就会推；

拉就是从没法知道主什么时候变，于是周期性的去拉，周期就是这里的1D设置的。



1H：retry，就是从没拉成功，不用等1D后的周期，只需要1H就会再次尝试去拉。那么问题来，你确定1H不涉及主推的不成功的重试时间？



1W：如果从服务器长达1W一周没能同主服务器同步，这话不准确啊，是没拉成功还是没被主push成功呢，我初步感觉应该是看日期，不管是主push还是从get，数据库文件的mtime如果小于当前时间 超过1W，就算失联了，对吧。这个就合理了，就算它的逻辑不是我想的这样，我TM写个脚本给你改掉。



3H：minimum是针对client请求的缓存时间，比如client请一个域名，我第一次本地没有的就会去上游dns问，然后缓存3小时内，client再次请求，就会直接把缓存里的记录交给client。如果是本地有的记录呢，是不是也会缓存，应该也不会查本地数据库。错了！

​		这3H和上面的TTL是两码事，minimum和TTL都是针对client的请求的记录缓存，minimun是针对本地不存的记录需要去上游问的，不管问到还是没问到的缓存；而TTL是针对本地有记录的用户请求的缓存。

​		都不对，两个缓存的理解如下：

 		所以dnsmasq估计也有这两种缓存，也就是说 修改hosts，说不定不用重启dnasmq，只要等他缓存到期了就行了。

**更正** https://www.zytrax.com/books/dns/apd/rfc2308.txt上讲了

```
   Negative caching was an optional part of the DNS specification and
   deals with the caching of the non-existence of an RRset [RFC2181] or
   domain name.
```

https://www.zytrax.com/books/dns/ch8/soa.html中讲了

"the negative caching time - the time a NAME ERROR = NXDOMAIN result may be cached by any resolver."

然后人家也说了，older documentation也就是bind4and8不是这么玩的，那会儿表示TTL的默认值。那会还没有explicit TTL呢

| nx = nxdomain ttl | Signed 32 bit value in seconds. [RFC 2308](https://www.zytrax.com/books/dns/apd/rfc2308.txt) (implemented by BIND 9) redefined this value to be the negative caching time - the time a NAME ERROR = NXDOMAIN result may be cached by any resolver. The maximum value allowed by RFC 2308 for this parameter is 3 hours (10800 seconds). **Note:** This value was historically (in BIND 4 and 8) used to hold the default TTL value for any RR from the zone that did not specify an explicit TTL. RFC 2308 (and BIND 9) uses the $TTL directive as the zone default TTL. You may find older documentation or zone file configurations which reflect the old usage. [BIND Time format](https://www.zytrax.com/books/dns/apa/time.html). |
| ----------------- | ------------------------------------------------------------ |

 https://www.zytrax.com/books/dns/apd/rfc2308.txt上讲了

```
   Negative caching was an optional part of the DNS specification and
   deals with the caching of the non-existence of an RRset [RFC2181] or
   domain name.
```

所以关于两个缓存需要重新认识一下

吃饭，

回来了，去的路上我在想，雨果我来开发，我怎么弄，我就会将 请求到的做一个缓存，没请求到的另外做一个缓存，就行拉，



找证据，肯定要去官网了，首先进入到这里：https://www.isc.org/bind/

页面找了一圈没有看到document关键字，点这里看看

<img src="3-DNS主服务器实现.assets/image-20230216190406578.png" alt="image-20230216190406578" style="zoom:33%;" /> 



继续点

![image-20230216190426563](3-DNS主服务器实现.assets/image-20230216190426563.png)



点



![image-20230216190454119](3-DNS主服务器实现.assets/image-20230216190454119.png)

搜

![image-20230216190507489](3-DNS主服务器实现.assets/image-20230216190507489.png)





结果在1.4里，哈哈

![image-20230216190601091](3-DNS主服务器实现.assets/image-20230216190601091.png) 

所以我开发的思路也是对了，就是不存的解析一个缓存。并不是 通过dns server代理查询上游的本地没有的记录的缓存时间。没这个说法！

然后上图的  For details of what all these fileds mean,please see the authoritatvie server document.去看看明细解释。

# SOA Records

There is only one SOA that is guaranteed to exist on the internet and that is the one for the root zone (called `.`). As of 2018, it looks like this:

```
.   86400   IN   SOA   a.root-servers.net. nstld.verisign-grs.com. 2018032802 1800 900 604800 86400
```

This says: the authoritative server for the root zone is called `a.root-servers.net`. This name is however only used for diagnostics. Secondly, [nstld@verisign-grs.com](mailto:nstld@verisign-grs.com) is the email address of the zone maintainer. Note that the `@` is replaced by a dot. Specifically, if the email address had been `nstld.maintainer@verisign-grs.com`, this would have been stored as **nstld\\.maintainer.verisign-grs.com**，This name would then still be 3 labels long, but the first one has a dot in it.

The following field, `2018032802`, is a serial number. Quite often, but by all means not always, this is a date in proper order (YYYYMMDD), followed by two digits indicating updates over the day. This serial number is used for replication purposes, as are the following 3 numbers.

Zones are hosted on 'masters`. Meanwhile, 'slave' servers poll the master for updates, and pull down a new zone if they see new contents, as noted by an increase in serial number.

The numbers 1800 and 900 describe how often a zone should be checked for updates (twice an hour), and that if an update check fails it should be repeated after 900 seconds. Finally, 604800 says that if a master server was unreachable for over a week, the zone should be deleted from the slave. This is not a popular feature.

The final number, 86400, denotes that if a response says a name or RRSET does not exist, it will continue to not exist for the next day, and that this knowledge may be cached.

所以

1、邮箱的格式要注意 识别不了的时候 \\.的写法

2、时间默认单位是s秒

3、Finally, 604800 says that if a master server was unreachable for over a week, the zone should be deleted from the slave. This is not a popular feature.这句话是说①这是在从服务器上起作用的配置，从服务器判断master不可达持续一定时间后，就将zone删除，是从服务器上删除zone，也就是从自行惭愧了，知道自己差了很久，补提供该方面的服务了②这个功能不受欢迎，很简单啊，master虽然从服务器不可达了，其实大概率就是master挂了，从服务器又不提供服务，那网络里dns一个都没了，喜欢才怪。

4、最后的minimum时间早期是没有定义TTL的时候就用来做默认的TTL，有了$TTL就在BIND9版本里作为新的功能了--NAME ERROR = NXDOMAIN 就是上游dns的response 说不存在，则本地缓存 这个判断 的持续时间。在此时间段里用户请求该域名就直接用缓存回给用户。

5、TTL的作用看来就不是仅仅本地RR，理应包括response 的来的RR。我再去确认，

TTL in the DNS context defines the duration in seconds that the record may be cached by any resolver.

https://powerdns.org/hello-dns/basic.md.html这里点击

<img src="3-DNS主服务器实现.assets/image-20230216200733947.png" alt="image-20230216200733947" style="zoom:50%;" /> 

<img src="3-DNS主服务器实现.assets/image-20230216201229819.png" alt="image-20230216201229819" style="zoom:50%;" />

所以没找到  针对本地RR这种说法，所我有理由认为就是所有得有结果得解析记录，后面敲实验的时候验证下就好赖，有啥难的，不查了。



还是这篇就不错https://www.zytrax.com/books/dns/ch8/soa.html

<img src="3-DNS主服务器实现.assets/image-20230216193239373.png" alt="image-20230216193239373" style="zoom:33%;" /> 

点进去就知道缓存RR超时时间的前生今世了https://www.zytrax.com/books/dns/apa/ttl.html

其中**Note:** [RFC 1912](https://www.zytrax.com/books/dns/apd/rfc1912.txt) cautions that 0 = no caching may not be widely supported, however most modern DNS software does support the feature. ，我学这个bind本意就是用来做DNS的请求的负载均衡将奇数偶数分担到10.2和10.3，所以肯定是可以设置为0的，因为10.2 和10.3上就有缓存。不过bind有缓存好像也不影响结果。

**然后**In BIND 8 the SOA record (minimum parameter) was used to define the zone default TTL value. In BIND 9 the [SOA 'minimum' parameter](https://www.zytrax.com/books/dns/ch8/soa.html) is used as the negative (NXDOMAIN) caching time (defined in RFC 2308).这就是前生今世拉

**设置的频率也是一个学问**RFC 1912 recommends that the $TTL value be set to 1 day or longer and that certain RRs which rarely change, such as the MX records for the domain, use an explicit TTL value to set even longer values such as 2 to 3 weeks. The value of any TTL is a balance between how frequently you think the DNS records will change vs load on the DNS server.

**操作细节**In most cases this will not be a problem since IP address changes are normally planned in advance, in which case in advance of the change process the TTL could be reduced to 3h to 12h and then restored to a higher value when the change has stabilized. 

**然后DNS最长缓存时间是68年**The TTL field is defined to be an unsigned 32 bit value with a valid range from 0 to 2147483647 (clarified in RFC 2181) - which is a long time! - somewhere on the other side of 68 years.

<img src="3-DNS主服务器实现.assets/image-20230216194850441.png" alt="image-20230216194850441" style="zoom:50%;" /> 

看来首位置为0了。

**想看再看吧**https://www.zytrax.com/books/dns/info/ttl.html







![image-20230216161127660](3-DNS主服务器实现.assets/image-20230216161127660.png)



关于www，往往不是A记录，而是CNAME(别名)做CDN这是比较常见的，当然你也可以直接A记录，个人网站其实也有CF的免费CDN也是用CNAME咯。所以除非你不懂，否则还真是用别名，不用A记录。

![image-20230216161501720](3-DNS主服务器实现.assets/image-20230216161501720.png) 







![image-20230216165820453](3-DNS主服务器实现.assets/image-20230216165820453.png) 



![image-20230216165803355](3-DNS主服务器实现.assets/image-20230216165803355.png)



<img src="3-DNS主服务器实现.assets/image-20230216165658948.png" alt="image-20230216165658948" style="zoom:50%;" /> 



还是dig看着最舒服，它能够标注出来CNAME，不过host和nlsookup其实也是明确的，要注意它的排版格式就能识别谁CNAME到谁了。

![image-20230216170015511](3-DNS主服务器实现.assets/image-20230216170015511.png) 

配置CNAME多个看看：

![image-20230216170756532](3-DNS主服务器实现.assets/image-20230216170756532.png)







![image-20230216171217920](3-DNS主服务器实现.assets/image-20230216171217920.png)

上图的3H更正为：代理查询 没查到 这么一个没查到的结果 缓存的时间。



![image-20230216171856287](3-DNS主服务器实现.assets/image-20230216171856287.png) 



编辑bind配置文件

<img src="3-DNS主服务器实现.assets/image-20230216173255906.png" alt="image-20230216173255906" style="zoom:50%;" /> 



不过一般不推荐在这个主配置文件里这么直接加，可以指一个目录，然后统一放在那里，这是常规宇宙法则，其实就是上图倒数第二行的文件，就是统一放区域rr记录文件的。

![image-20230216173707778](3-DNS主服务器实现.assets/image-20230216173707778.png) 

这里才是专门放区域文件的地方，所以修改一下

<img src="3-DNS主服务器实现.assets/image-20230216174024997.png" alt="image-20230216174024997" style="zoom:50%;" /> 

至此，该有的配置就配置完了，

1、语法检查：named-checkconf   named-checkzone

检查配置文件的命令，也只能检测配置文件，它不能检查数据库文件也就是rr记录文件

<img src="3-DNS主服务器实现.assets/image-20230216174243897.png" alt="image-20230216174243897" style="zoom:50%;" /> 

<img src="3-DNS主服务器实现.assets/image-20230216174329348.png" alt="image-20230216174329348" style="zoom:50%;" /> 

检测区域数据的命令：

<font color=green size=4>OK</font>就是成功拉，loaded serial 0就是版本号0的意思。

![image-20230216174658224](3-DNS主服务器实现.assets/image-20230216174658224.png)

这是报错👇

![image-20230216175231724](3-DNS主服务器实现.assets/image-20230216175231724.png)

![image-20230216175456269](3-DNS主服务器实现.assets/image-20230216175456269.png)



![image-20230216175555243](3-DNS主服务器实现.assets/image-20230216175555243.png)

![image-20230216175726956](3-DNS主服务器实现.assets/image-20230216175726956.png)

所以不管最大多少，人家推荐的是  年月日nn   nn是当天修订的次数。<img src="3-DNS主服务器实现.assets/image-20230216175837688.png" alt="image-20230216175837688" style="zoom:60%;" />

<img src="3-DNS主服务器实现.assets/image-20230216175932326.png" alt="image-20230216175932326" style="zoom:50%;" />

验证一下，果然4294967295 是最大值，就是32位，4个Byte的空间。其实你要有这个思路，就是 凡是计算机里的最大值都是几个BYTE空间得出来的。比如IP报文里的 首部长度也是4个B。要知道2^32已经很大了。

![image-20230216180032315](3-DNS主服务器实现.assets/image-20230216180032315.png) 

![image-20230216180049041](3-DNS主服务器实现.assets/image-20230216180049041.png) 





2、重新加载配置文件：rndc reload

此时解析就OK

![image-20230216203426348](3-DNS主服务器实现.assets/image-20230216203426348.png)







![image-20230216204903434](3-DNS主服务器实现.assets/image-20230216204903434.png)

注意区别，



![image-20230216204858327](3-DNS主服务器实现.assets/image-20230216204858327.png)

![image-20230216205018879](3-DNS主服务器实现.assets/image-20230216205018879.png)





我的测试有点小问题，不递归去问上游dns了，也就是默认的13个根了







![image-20230216205617400](3-DNS主服务器实现.assets/image-20230216205617400.png)

![image-20230216205726225](3-DNS主服务器实现.assets/image-20230216205726225.png)

开启海外网络，本地dig就成功了，毛仔细看下图 ask的是192.168.10.2.

![image-20230216210037532](3-DNS主服务器实现.assets/image-20230216210037532.png)



还是不行，dns server的firewalld和selinux都关掉还是不行，明天再看吧

![image-20230216210511308](3-DNS主服务器实现.assets/image-20230216210511308.png)

奇怪了

不过firewalld确实也造成了故障



![image-20230216210559725](3-DNS主服务器实现.assets/image-20230216210559725.png)





![image-20230216210727237](3-DNS主服务器实现.assets/image-20230216210727237.png)





![image-20230216211007346](3-DNS主服务器实现.assets/image-20230216211007346.png)



就是不递归了，下班下班，回去睡觉



好了，好像还是selinux没有disabled之前是permissive，反正今天过来一开始还是老样子，防火墙，selinux，重启linux，就没有啥其他动作了。无非时各种dig，顺带把windows的dig和tcping工具给完善和一下



---

奇怪的一点时，我怎么查dig都看不到AUTHORITY权威DNS服务器了

<img src="3-DNS主服务器实现.assets/image-20230217182907243.png" alt="image-20230217182907243" style="zoom:50%;" /> 

<img src="3-DNS主服务器实现.assets/image-20230217182953291.png" alt="image-20230217182953291" style="zoom:50%;" /> 

以上就完成了主dns的简单搭建





<img src="3-DNS主服务器实现.assets/image-20230217183322152.png" alt="image-20230217183322152" style="zoom:50%;" /> 

![image-20230217183358946](3-DNS主服务器实现.assets/image-20230217183358946.png) 



![image-20230217183513525](3-DNS主服务器实现.assets/image-20230217183513525.png) 



dig的用法

![image-20230217183918396](3-DNS主服务器实现.assets/image-20230217183918396.png)





此外还有

从服务器、反向解析、智能DNS、CDN、DNS委托



下一节继续



