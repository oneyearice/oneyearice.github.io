# 第4节. nginx实现web服务器01



# nginx的http的语句块配置

## 主要是[ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html)这个模块

![image-20231101133651770](4-nginx实现web服务器01.assets/image-20231101133651770.png)



MIME参考文档：

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_Types



![image-20231101134517371](4-nginx实现web服务器01.assets/image-20231101134517371.png)

不写include，就写type同样放到http语句块下



default_type是有默认值的

![image-20231101135939525](4-nginx实现web服务器01.assets/image-20231101135939525.png)



但是通常yum和编译都会给你改成

![image-20231101140109966](4-nginx实现web服务器01.assets/image-20231101140109966.png)

这个，这个就是除了明确的类型以为，浏览器不能识别了就，此时统统提示下载。

![image-20231101141351088](4-nginx实现web服务器01.assets/image-20231101141351088.png)

### 测试下default_type application/octet-stream的效果

![image-20231101142417793](4-nginx实现web服务器01.assets/image-20231101142417793.png)



![image-20231101142328223](4-nginx实现web服务器01.assets/image-20231101142328223.png)

如果是无痕模式的浏览器，会直接弹 另存为 的对话框，也是下载的动作。



通过curl 看看头部

![image-20231101142643657](4-nginx实现web服务器01.assets/image-20231101142643657.png)

curl 不加-I 貌似还是直接以文本打开了就

![image-20231101142729917](4-nginx实现web服务器01.assets/image-20231101142729917.png)

注释掉使其使用默认的text/plain格式

![image-20231101142956470](4-nginx实现web服务器01.assets/image-20231101142956470.png)

![image-20231101143006157](4-nginx实现web服务器01.assets/image-20231101143006157.png)



**看下php文件的识别**

![image-20231101143337930](4-nginx实现web服务器01.assets/image-20231101143337930.png)

默认就是不支持的

写一个php文件来实验

![image-20231101143448439](4-nginx实现web服务器01.assets/image-20231101143448439.png)

由于不支持.php所以打开该URL默认就是下载该文件的，对比之前apache的效果，我有两个apache①个yum安装②一个编译安装，其实不管是yum还是编译都不支持php文件的浏览器直接打开，只不过yum安装的apache，后来又yum安装了php，于是

![image-20231101145039676](4-nginx实现web服务器01.assets/image-20231101145039676.png)

这就找到了cli的出处啦，正式因为yum install php才有了httpd配置文件里的哪个php.conf且放在了conf.d下，才有了httpd浏览器打开了php文件。所以，以此类推现在nginx也要这么搞一下。

![image-20231101145204626](4-nginx实现web服务器01.assets/image-20231101145204626.png)



![image-20231101154824667](4-nginx实现web服务器01.assets/image-20231101154824667.png)

但是我现在的nginx是编译安装的，所以我需要尝试蒋这个php.conf复制到编译的路径下

![image-20231101155629430](4-nginx实现web服务器01.assets/image-20231101155629430.png)

没用php里的ngnix关键字的文件里都没有text/html .php之类的字眼，简单网速搜了下nginx对php的支持不想apache一样简单。具体操作参见以下教程👇

https://blog.51cto.com/928004321/1744675

https://www.php.cn/faq/476519.html



### tcp_nodelay on; 可优化报文确认频次，提高传输效率

![image-20231101164742749](4-nginx实现web服务器01.assets/image-20231101164742749.png)



据说是当 数据报文比较大的时候拆包传输了就，此时就一下发好几个包，然后再确认一次就比较合理了。

其实数据报文小的时候不涉及拆包，同样也能提高效率，只是数据量小，资源消耗小，系统也好硬件也罢能对付得了，数据量大的时候资源消耗大了，此时就会从各个方面优化，才需要tcp_nodely on多包确认一次。



### tcp_nopush on;  可优化报文合并提高，提高传输效率



### ![image-20231101165302878](4-nginx实现web服务器01.assets/image-20231101165302878.png)





![image-20231101165729757](4-nginx实现web服务器01.assets/image-20231101165729757.png)

默认是off不显示的

![image-20231101170035018](4-nginx实现web服务器01.assets/image-20231101170035018.png)

也可以利用"include conf.d/*.conf"去单开一个文件，我理解就是include是在这个语句块也就是这个缩进层级下的，所以新建的xxx.conf里的内容也只反映到这个语句块层级下的。

![image-20231101171212860](4-nginx实现web服务器01.assets/image-20231101171212860.png)

只是个显示而已。



### server_tokens on|off|build|string;

![image-20231101171341677](4-nginx实现web服务器01.assets/image-20231101171341677.png)

string是商业版的nginx才支持的。

![image-20231101172519258](4-nginx实现web服务器01.assets/image-20231101172519258.png)

build参数就不是简单修改了的，

![image-20231101172714900](4-nginx实现web服务器01.assets/image-20231101172714900.png)

![image-20231101172748320](4-nginx实现web服务器01.assets/image-20231101172748320.png)

![image-20231101172802960](4-nginx实现web服务器01.assets/image-20231101172802960.png)

所以build是编译的时候，通过--build=xx.xxx.v1001，然后配置文件里写server_tokens build;才会有效果的。 

或者修改源码文件里的关键字段；server_tokens on或者off都能改，改的地方不同👇

![image-20231101175953946](4-nginx实现web服务器01.assets/image-20231101175953946.png)

server_tokens on;修改👇

![image-20231101175749099](4-nginx实现web服务器01.assets/image-20231101175749099.png)

改成：

![image-20231101181415647](4-nginx实现web服务器01.assets/image-20231101181415647.png)

server_tokens off;修改👇

![image-20231101180159940](4-nginx实现web服务器01.assets/image-20231101180159940.png)

改成：

![image-20231101181506260](4-nginx实现web服务器01.assets/image-20231101181506260.png)



当然改完还需要重新编译了

那么就重新变一下看看

编译之前看下二进制主程序的哈希值

![image-20231101180854787](4-nginx实现web服务器01.assets/image-20231101180854787.png)

担心软连接的哈希，确认下，不是哦，就是源文件的哈希

![image-20231101180931154](4-nginx实现web服务器01.assets/image-20231101180931154.png)

开始编译，ctrl+r调出历史命令回车就行👇

![image-20231101181009188](4-nginx实现web服务器01.assets/image-20231101181009188.png)

![image-20231101181651972](4-nginx实现web服务器01.assets/image-20231101181651972.png)



![image-20231101181745430](4-nginx实现web服务器01.assets/image-20231101181745430.png)



那个此时之前的nginx服务还在，不会停，因为你用的之前的nginx二进制启动的服务，你重新编译其实也就是对这个二进制进行变化吧，所以之前已经启动了的服务不受影响，服务程序继续对外提供服务。



此时想用nginx -s reload看到效果是不可能的，因为我们修改的/apps/nginx/sbin/nginx二进制程序里的东西，不是配置文件，所以需要重启服务后才能看到效果



![image-20231101182807915](4-nginx实现web服务器01.assets/image-20231101182807915.png)



然后修改server_token为on，reload配置文件就可以看到效果了👇

![image-20231101183245161](4-nginx实现web服务器01.assets/image-20231101183245161.png)





## server在很多模块里都有

![image-20231102092239228](4-nginx实现web服务器01.assets/image-20231102092239228.png)

下面梳理core模块里的server

server就是虚拟主机

![image-20231102092826522](4-nginx实现web服务器01.assets/image-20231102092826522.png)

![image-20231102093458093](4-nginx实现web服务器01.assets/image-20231102093458093.png)



listen PORT|address[:port]|unix:/PATH/TO/SOCKET_FILE

listen可以写port、ip+port、unix套接字的路径。但是写unix socket就只能本机访问了。

default_server：默认虚拟主机，就是没有命中的统统用这个虚拟主机来响应，这里在apache里是不用设置，拍最前面的virtualHost就是默认的。
