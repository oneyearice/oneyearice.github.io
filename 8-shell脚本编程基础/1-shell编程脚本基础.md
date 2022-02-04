# 第1节. shell编程脚本基础





 

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



 

## 举个例子

![img](1-shell编程脚本基础.assets/clip_image002-16439583340341.jpg)

![img](1-shell编程脚本基础.assets/clip_image004-16439583340352.jpg)

![img](1-shell编程脚本基础.assets/clip_image006-16439583340353.jpg)

![img](1-shell编程脚本基础.assets/clip_image008.jpg)

 

![img](1-shell编程脚本基础.assets/clip_image010.jpg)

![img](1-shell编程脚本基础.assets/clip_image012-16439583340364.jpg)

![img](1-shell编程脚本基础.assets/clip_image014.jpg)

![img](1-shell编程脚本基础.assets/clip_image016-16439583340365.jpg)

之前接触过%s/xx/yy/g，现在又看到了.,$s/XX/yy/g

<font color=red>.</font>点表示当前行号<font color=red>,</font>逗号是一直到整个文件最后一行

![img](1-shell编程脚本基础.assets/clip_image018-16439583340366.jpg)

u撤销后，改成

![img](1-shell编程脚本基础.assets/clip_image020-16439583340367.jpg)

![img](1-shell编程脚本基础.assets/clip_image022-16439583340368.jpg)

![img](1-shell编程脚本基础.assets/clip_image024-16439583340369.jpg)

![img](1-shell编程脚本基础.assets/clip_image026-164395833403610.jpg)

![img](1-shell编程脚本基础.assets/clip_image028-164395833403611.jpg)

引号替换一下

![img](1-shell编程脚本基础.assets/clip_image030-164395833403612.jpg)

![img](1-shell编程脚本基础.assets/clip_image032-164395833403613.jpg)

## 变量

![img](1-shell编程脚本基础.assets/clip_image034-164395833403614.jpg)

变量代表着内存空间 

<img src="1-shell编程脚本基础.assets/clip_image036-164395833403615.jpg" alt="img" style="zoom:67%;" /> 

内存中的一个地址块放了magedu，而name就表示地址值。于是就是name中存放了magedu。



变量，值可变化，当然也有不可变

 

![img](1-shell编程脚本基础.assets/clip_image038-164395833403617.jpg)

python和shell都不需要事先申明变量

##  变量起名规范

![img](1-shell编程脚本基础.assets/clip_image040-164395833403616.jpg)

##  特殊变量

![img](1-shell编程脚本基础.assets/clip_image042-164395833403618.jpg)

![img](1-shell编程脚本基础.assets/clip_image044-164395833403719.jpg)

![img](1-shell编程脚本基础.assets/clip_image046-164395833403720.jpg)

![img](1-shell编程脚本基础.assets/clip_image048-164395833403721.jpg)

##  👆变量的正儿八经的写法，很重要 

![img](1-shell编程脚本基础.assets/clip_image050-164395833403722.jpg)

 

![img](1-shell编程脚本基础.assets/clip_image052-164395833403723.jpg)

### 如果此时Y的值变成了30，问X的值是多少，这个在PYTHON里面叫变量赋值，如果是列表、字典是需要.copy()的

![img](1-shell编程脚本基础.assets/clip_image054-164395833403724.jpg)

<img src="1-shell编程脚本基础.assets/clip_image056-164395833403825.jpg" alt="img" style="zoom:67%;" />

<img src="1-shell编程脚本基础.assets/clip_image058-164395833403826.jpg" alt="img" style="zoom:67%;" />

##  变量取消

![img](1-shell编程脚本基础.assets/clip_image060.jpg)

![img](1-shell编程脚本基础.assets/clip_image062.jpg)

![img](1-shell编程脚本基础.assets/clip_image064.jpg)

上图替换语法存在错误

![img](1-shell编程脚本基础.assets/clip_image066.jpg)

### 不用加g，%s/xx/yy/g，的g如果是每行只有一个不需要加g全局

![img](1-shell编程脚本基础.assets/clip_image068.jpg)

一般脚本结束了变量也就没了。不过还是建议删掉。

 

![image-20220202182620984](1-shell编程脚本基础.assets/image-20220202182620984.png) 

## 把命令放到变量里

![img](1-shell编程脚本基础.assets/clip_image072.jpg)



![image-20220202182715995](1-shell编程脚本基础.assets/image-20220202182715995.png)



![img](1-shell编程脚本基础.assets/clip_image076.jpg)

![img](1-shell编程脚本基础.assets/clip_image078.jpg)

第一题答案就有了

 

 cp -a  的a等价于-dR

### 文件夹不存在cp会直接创建的

![img](1-shell编程脚本基础.assets/clip_image080.jpg)

第二题答案

 

![img](1-shell编程脚本基础.assets/clip_image081.png)

 

![img](1-shell编程脚本基础.assets/clip_image083.jpg)

 

 nl 和 cat -b一个意思，不过不能列出空行行号

 ![image-20220202184946332](1-shell编程脚本基础.assets/image-20220202184946332.png) 

![image-20220202185054359](1-shell编程脚本基础.assets/image-20220202185054359.png) 

