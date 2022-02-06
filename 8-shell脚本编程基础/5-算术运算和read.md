# 第5节. 算术运算和read

![img](5-算术运算和read.assets/clip_image002.jpg) 

## && || 和 || &&先后效果不一样

![img](5-算术运算和read.assets/clip_image004.jpg)

 

![img](5-算术运算和read.assets/clip_image006.jpg)

= 是比字符串

-eq是比整数的，小数不行。

上图的两个等号，一般一个就行了，双综括号里可以用两个等号，然后双综括号里一般用=~正则表达式。

![img](5-算术运算和read.assets/clip_image008.jpg)

<img src="5-算术运算和read.assets/clip_image010.jpg" alt="img" style="zoom:67%;" />

![img](5-算术运算和read.assets/clip_image012.jpg)

 

##  read

read varXX

unset varXX

这两个后面跟的都是变量名，不需要加$xxx这样。就是变量了。

![img](5-算术运算和read.assets/clip_image013.png)

![img](5-算术运算和read.assets/clip_image015.jpg)

![img](5-算术运算和read.assets/clip_image017.jpg)

 

![img](5-算术运算和read.assets/clip_image019.jpg)

![img](5-算术运算和read.assets/clip_image021.jpg)

![img](5-算术运算和read.assets/clip_image022.png)

优化不换行

![img](5-算术运算和read.assets/clip_image024.jpg)

### echo 不换行

![img](5-算术运算和read.assets/clip_image026.jpg)

### 再次优化，read的本身就自带提示语句

![img](5-算术运算和read.assets/clip_image028.jpg)

![img](5-算术运算和read.assets/clip_image030.jpg)

![img](5-算术运算和read.assets/clip_image032.jpg)

![image-20220206152843515](5-算术运算和read.assets/image-20220206152843515.png)



写个脚本实现鸡兔同笼算法

![image-20220206153142175](5-算术运算和read.assets/image-20220206153142175.png) 

![image-20220206153207370](5-算术运算和read.assets/image-20220206153207370.png) 





## read一下赋值多个

失败案例

![image-20220206153332685](5-算术运算和read.assets/image-20220206153332685.png) 

 

man bash可见

<img src="5-算术运算和read.assets/clip_image039.jpg" alt="img" style="zoom:67%;" />

![img](5-算术运算和read.assets/clip_image041.jpg)

![img](5-算术运算和read.assets/clip_image043.jpg)

## 管道符后面是一个子进程，所以要括起来，你用小括号就是子进程后面再接一个子进程了。

花括号就是管道符-子进程后面直接一个整体。

总之作为一个整体就行了。

![img](5-算术运算和read.assets/clip_image045.jpg)

##  证明管道符确实开启了子进程

![img](5-算术运算和read.assets/clip_image047.jpg)

![img](5-算术运算和read.assets/clip_image049.jpg)

![img](5-算术运算和read.assets/clip_image051.jpg)

![img](5-算术运算和read.assets/clip_image053.jpg)

<img src="5-算术运算和read.assets/image-20220206154003520.png" alt="image-20220206154003520" style="zoom:67%;" /> 

![image-20220206154151545](5-算术运算和read.assets/image-20220206154151545.png) 

![image-20220206154255928](5-算术运算和read.assets/image-20220206154255928.png) 



 

![img](5-算术运算和read.assets/clip_image059.jpg)

先来一个还阔以但是有点不太推荐的解法，因为短路与或用的太多了

![image-20220206161258588](5-算术运算和read.assets/image-20220206161258588.png)



![img](5-算术运算和read.assets/clip_image061.jpg)

![img](5-算术运算和read.assets/clip_image062.png)

![img](5-算术运算和read.assets/clip_image064.jpg)

 

## if条件判断

<img src="5-算术运算和read.assets/image-20220206161859279.png" alt="image-20220206161859279" style="zoom:67%;" /> 



