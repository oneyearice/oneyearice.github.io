# 第3节. nginx全局配置和性能优化



### nginx的配置文件

![image-20231030165456967](3-nginx全局配置和性能优化.assets/image-20231030165456967.png)



## nginx的配置

![image-20231030170248146](3-nginx全局配置和性能优化.assets/image-20231030170248146.png)

![image-20231030170447357](3-nginx全局配置和性能优化.assets/image-20231030170447357.png)

上面一段是全局配置，包括，nginx软件以谁的身份来运行，worker进程是几个，

![image-20231030170515271](3-nginx全局配置和性能优化.assets/image-20231030170515271.png)

下面的http打头的段，是泛指http，包括http，已经fastCGI-这个其实不是http但是是web服务也和http相关，所以也放在这里；但是后面做tcp的反代后单独块了不在http块里了。

![image-20231030171102618](3-nginx全局配置和性能优化.assets/image-20231030171102618.png)

然后语句里面还有嵌套，就是类似字典里还有字典。不同的指令只能出现在不同的语句块中。

比如上图👆的sendfile on，能够出现在什么语句块里，就需要通过官网来了解

![image-20231030171602825](3-nginx全局配置和性能优化.assets/image-20231030171602825.png)

![image-20231030172041304](3-nginx全局配置和性能优化.assets/image-20231030172041304.png)

然后配置文件，好习惯其实就是规范配置，是一个网站一个配置文件独立存放的，不是堆在一个主配置文件里的。

yum 安装的 nginx的主配置文件还是有点东西的👇

![image-20231030173014900](3-nginx全局配置和性能优化.assets/image-20231030173014900.png)

编译安装的主配置文件好像少了的意思👇

![image-20231030173041721](3-nginx全局配置和性能优化.assets/image-20231030173041721.png)



![image-20231030173557491](3-nginx全局配置和性能优化.assets/image-20231030173557491.png)





```
cd /apps/nginx/conf
mkdir conf.d
稍后配置文件都放到这个conf.d里

```

![image-20231030173955575](3-nginx全局配置和性能优化.assets/image-20231030173955575.png)





总结PPT

![image-20231030174429601](3-nginx全局配置和性能优化.assets/image-20231030174429601.png)



![image-20231030174453518](3-nginx全局配置和性能优化.assets/image-20231030174453518.png)



​	![image-20231030174503611](3-nginx全局配置和性能优化.assets/image-20231030174503611.png)



![image-20231030174528269](3-nginx全局配置和性能优化.assets/image-20231030174528269.png)





![image-20231030174806573](3-nginx全局配置和性能优化.assets/image-20231030174806573.png)

看上图👆default默认user就是叫nobody的这个用户，结果编译安装的时候ps auxf看到的是nginx用户，原因就是编译的时候指定了用户了

![image-20231030175142092](3-nginx全局配置和性能优化.assets/image-20231030175142092.png)

![image-20231030175110426](3-nginx全局配置和性能优化.assets/image-20231030175110426.png)



看看yum安装的

![image-20231030175425814](3-nginx全局配置和性能优化.assets/image-20231030175425814.png)



### 看看PID的路径

yum 安装的

![image-20231030175655247](3-nginx全局配置和性能优化.assets/image-20231030175655247.png)

我在想，编译安装的是不是都写到conf里了，应该不是，否则conf不可能那么短的

然后就是conf应该优先于nginx -V看到的配置选项咯，实验过了，是的。



### include包含的文件



![image-20231030182219330](3-nginx全局配置和性能优化.assets/image-20231030182219330.png)



![image-20231030182303040](3-nginx全局配置和性能优化.assets/image-20231030182303040.png)









### load_module 模块加载文件![image-20231030182348532](3-nginx全局配置和性能优化.assets/image-20231030182348532.png)



![image-20231030182209337](3-nginx全局配置和性能优化.assets/image-20231030182209337.png)

### ![image-20231030182137443](3-nginx全局配置和性能优化.assets/image-20231030182137443.png)





## 下面看看和性能相关的配置



