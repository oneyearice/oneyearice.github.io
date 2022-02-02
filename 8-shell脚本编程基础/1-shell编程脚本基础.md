# 第1节. shell编程脚本基础





## 

 

<img src="1-shell编程脚本基础.assets/clip_image006.jpg" alt="img" style="zoom:67%;" /> 

##  编程基础 

<img src="1-shell编程脚本基础.assets/image-20220202124713699.png" alt="image-20220202124713699" style="zoom:67%;" /> 

shell和py都是边解释边执行

![image-20220202124832678](1-shell编程脚本基础.assets/image-20220202124832678.png)



![image-20220202124810083](1-shell编程脚本基础.assets/image-20220202124810083.png)

 

![img](1-shell编程脚本基础.assets/clip_image012.jpg) 

![image-20220202125928948](1-shell编程脚本基础.assets/image-20220202125928948.png) 



gcc是个编译软件，可以把高级语言转换成机器代码

![img](1-shell编程脚本基础.assets/clip_image016.jpg)

![img](1-shell编程脚本基础.assets/clip_image018.jpg)

gcc就是编译器



![img](1-shell编程脚本基础.assets/clip_image020.jpg)

![img](1-shell编程脚本基础.assets/clip_image022.jpg)

在执行的时候有python解释器，会读到内存里翻译成机器码了。但是这个机器码是在内存里的，不是个放在硬盘里的文件。它是边执行边翻译。

 

## 编程基本概念

<img src="1-shell编程脚本基础.assets/image-20220202131248703.png" alt="image-20220202131248703" style="zoom:67%;" /> 



## shell脚本基础

<img src="1-shell编程脚本基础.assets/image-20220202131333656.png" alt="image-20220202131333656" style="zoom:67%;" /> 



## 创建shell脚本

<img src="1-shell编程脚本基础.assets/image-20220202131356090.png" alt="image-20220202131356090" style="zoom:67%;" /> 



脚本规范

<img src="1-shell编程脚本基础.assets/image-20220202131417037.png" alt="image-20220202131417037" style="zoom:67%;" /> 

## 脚本的基本结构

<img src="1-shell编程脚本基础.assets/image-20220202131454261.png" alt="image-20220202131454261" style="zoom:67%;" /> 



## vim的初始化

![image-20220202133342212](1-shell编程脚本基础.assets/image-20220202133342212.png)

## 

## 脚本执行的方法1:bash xxx

![image-20220202133529479](1-shell编程脚本基础.assets/image-20220202133529479.png)

## 方法2，source xxx和. xxxx

![image-20220202140114392](1-shell编程脚本基础.assets/image-20220202140114392.png) 

## 方法3：添加执行权限

![image-20220202133748656](1-shell编程脚本基础.assets/image-20220202133748656.png) 

👆直接运行脚本，就是外部命令了，是要到PATH变量里找路径的，而当前目录是/root并不在PATH变量里，所以找不到。 

添加到PATH变量

![image-20220202134023344](1-shell编程脚本基础.assets/image-20220202134023344.png) 

![img](1-shell编程脚本基础.assets/clip_image030.jpg)

![img](1-shell编程脚本基础.assets/clip_image032.jpg)

![img](1-shell编程脚本基础.assets/clip_image034.jpg)

## 👆其实也可以用ln -s 软连接来实现path变量的

但是如果你以后很多脚本都统一放到/data/scipts下的话，还是加/data/scripts为PATH变量好一点

![image-20220202134635596](1-shell编程脚本基础.assets/image-20220202134635596.png)



## 脚本运行方法4：传递给bash命令

![img](1-shell编程脚本基础.assets/clip_image036.jpg)

![img](1-shell编程脚本基础.assets/clip_image038.jpg)

evn.sh，只要是sh后缀就行了。

 

 

##  例子，写个脚本创建用户

![image-20220202142901764](1-shell编程脚本基础.assets/image-20220202142901764.png)

![image-20220202142924425](1-shell编程脚本基础.assets/image-20220202142924425.png) 



![img](1-shell编程脚本基础.assets/clip_image040.jpg)

让其口令立即过期

![img](1-shell编程脚本基础.assets/clip_image042.jpg)

![image-20220202144956523](1-shell编程脚本基础.assets/image-20220202144956523.png)

chage -d 0 test等价于passwd -e tezt都是修改date of last password chage这个值为0，意思就是登入后强制修改密码

![image-20220202145056787](1-shell编程脚本基础.assets/image-20220202145056787.png) 

![image-20220202145456337](1-shell编程脚本基础.assets/image-20220202145456337.png) 

 

## 语法错误检查方法

![image-20220202145818690](1-shell编程脚本基础.assets/image-20220202145818690.png)

![image-20220202145941589](1-shell编程脚本基础.assets/image-20220202145941589.png) 

### 两种语法检查方法

![image-20220202150027129](1-shell编程脚本基础.assets/image-20220202150027129.png) 



![image-20220202150125551](1-shell编程脚本基础.assets/image-20220202150125551.png)

![image-20220202150442302](1-shell编程脚本基础.assets/image-20220202150442302.png)

删除if那行后 再次执行就OK了

![image-20220202150600969](1-shell编程脚本基础.assets/image-20220202150600969.png) 



 
