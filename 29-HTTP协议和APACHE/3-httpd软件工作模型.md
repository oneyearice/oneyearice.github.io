# 第3节. httpd软件工作模型



# URI

**URI: Uniform Resource Identifier 统一资源标识，分为URL和URN**

​	>	URN: Uniform Resource Naming，统一资源命名

​					示例： P2P下载使用的磁力链接是URN的一种实现 magnet:?xt=urn:btih:6605589.....890EF888666

​	>	URL: Uniform Resorce Locator，统一资源定位符，用于描述某服务器某特定资源位置。

两者区别：URN如同一个人的名称，而URL代表一个人的住址。换言之， URN定义某事物的身份，而URL提供查找该事物的方法。URN仅用于命名， 而不指定地址

<img src="3-httpd软件工作模型.assets/image-20230926170209512.png" alt="image-20230926170209512" style="zoom:50%;" />

额，看来URI真的很多啊，哈哈哈



其实URL使用是最多的，URI里面虽然包含了很多，但是URL是最最多的，所以通常人们提到屌丝的时候，大概也不是什么赞美，哎哎哎，通常工作中URI和URL其实是混为一谈的，不必较真。但是你自己说出来的就得是URL





# URL组成

![image-20230926174512659](3-httpd软件工作模型.assets/image-20230926174512659.png)

URL格式：

```
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
```

**scheme**:方案，访问服务器以获取资源时要使用哪种协议，比如https://，这就是scheme，ftp://也是schema，不同的scheme其实就是不同的协议，自然后面的port也就不同。
**user**:用户，某些方案访问资源时需要的用户名      # 基本不用
**password**:密码，用户对应的密码，中间用：分隔      # 基本不用
**Host**:主机，资源宿主服务器的主机名或IP地址
**port**:端口,资源宿主服务器正在监听的端口号，很多方案有默认端口号
**path**:路径,服务器资源的本地名，由一个/将其与前面的URL组件分隔
**params**:参数，指定输入的参数，参数为名/值对，多个参数，用;分隔      # 名/值对，就是在说 键值对的形式
**query**:查询，传递参数给程序，如数据库，用？分隔,多个查询用&分隔    #  这个比较常见，👇cat是类型的简写。

<img src="3-httpd软件工作模型.assets/image-20230926181328783.png" alt="image-20230926181328783" style="zoom:50%;" />

上图👆?cat=670,671,672实际上就是mysql的select * from goods where cat='670,671,672';



**frag**:片段,一小片或一部分资源的名字，此组件在客户端使用，用#分隔      这个是跳转，比如业内跳转到下面的内容。







---



![image-20230926181801624](3-httpd软件工作模型.assets/image-20230926181801624.png)

&就是select里的and，翻译成DQL就是select * from goods where cat='670,671,672' and ev='exbrand_华为(HUAWEI)^' and cid3='672';   这里还涉及%2C就是,逗号     然后%5E好像是^脱字符，

<img src="3-httpd软件工作模型.assets/image-20230926182329805.png" alt="image-20230926182329805" style="zoom:50%;" />





<img src="3-httpd软件工作模型.assets/image-20230926182640685.png" alt="image-20230926182640685" style="zoom:50%;" />



