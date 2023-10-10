# 第7节. 基于主机头的多虚拟主机和实现HTTPS加密



书接上回，虚拟主机的实现继续，上一篇实现了基于dstIp，基于dstPort

## 虚拟主机-基于主机头head里的host字段实现负载均衡

原理就是client发送http报文请求的时候，http头部里就携带了访问网站的FQDN



抓包是看不到https的head的，TCP以上都是加密的。

![image-20231010161941842](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010161941842.png)



<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010162139465.png" alt="image-20231010162139465" style="zoom:50%;" />

HTTP不加密可以抓到head内容

![image-20231010162417207](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010162417207.png)



**实验**

👇就是统一监听80，80本来在/etc/httpd/conf/httpd.conf这个主配置文件里就启用了，

然后documentroot及其all granted授权，documentroot和directory是一对，根和授权。

最后就是servername转发依据和LOG

![image-20231010163336943](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010163336943.png)

搞定👆



疑问：为什么cstie没写servername，结果转发到www.a.com了

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010163715609.png" alt="image-20231010163715609" style="zoom:50%;" />



估计就是 不写servername，然后用的就是第一个 从上往下找 <img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010164647324.png" alt="image-20231010164647324" style="zoom:50%;" />

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010164702128.png" alt="image-20231010164702128" style="zoom:50%;" /> 



![image-20231010165801318](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010165801318.png)

![image-20231010165841739](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010165841739.png)

通过官网查看就会自然知道现在servername还不够，还需要知道serveralais

![image-20231010171040782](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171040782.png)



![image-20231010171054597](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171054597.png)



<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171156199.png" alt="image-20231010171156199" style="zoom:33%;" />



<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171227653.png" alt="image-20231010171227653" style="zoom:33%;" />







看个现象

![image-20231010171802366](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171802366.png)![image-20231010171828433](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010171828433.png)

问，为什么192.168.126.130的访问变成了www.a.com了

答，因为www.a.com在前面

![image-20231010172237006](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010172237006.png)

问，为什么不是原来的 非虚拟主机的页面了，答：从结果判断，就是虚拟主机的转发配置抢先了

![image-20231010172331424](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010172331424.png)

![image-20231010172307449](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010172307449.png)

其实就是虚拟主机的80和非虚拟主机的80冲突了，结果就是虚拟主机胜利

![image-20231010173211780](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010173211780.png)

80变成www.c.com排在第一个，所以和

<img src="7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010173242299.png" alt="image-20231010173242299" style="zoom:50%;" /> 

冲突了，存在优先级了就。





**如果ip+port+servername一起用会如何**

![image-20231010175649095](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010175649095.png)

![image-20231010175657444](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010175657444.png)

**说明端口 port 优先> servername**

![image-20231010180239229](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010180239229.png)



![image-20231010180601048](7-基于主机头的多虚拟主机和实现HTTPS加密.assets/image-20231010180601048.png)



虚拟主机一般就是网站业务量不大用用吧







