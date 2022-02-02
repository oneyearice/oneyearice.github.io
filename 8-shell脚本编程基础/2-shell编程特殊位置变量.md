# 第2节. shell编程特殊位置变量

## 举个例子

![img](2-shell编程特殊位置变量.assets/clip_image002.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image004.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image006.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image008.jpg)

 

![img](2-shell编程特殊位置变量.assets/clip_image010.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image012.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image014.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image016.jpg)

之前接触过%s/xx/yy/g，现在又看到了.,$s/XX/yy/g

<font color=red>.</font>点表示当前行号<font color=red>,</font>逗号是一直到整个文件最后一行

![img](2-shell编程特殊位置变量.assets/clip_image018.jpg)

u撤销后，改成

![img](2-shell编程特殊位置变量.assets/clip_image020.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image022.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image024.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image026.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image028.jpg)

引号替换一下

![img](2-shell编程特殊位置变量.assets/clip_image030.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image032.jpg)

## 变量

![img](2-shell编程特殊位置变量.assets/clip_image034.jpg)

变量代表着内存空间 

<img src="2-shell编程特殊位置变量.assets/clip_image036.jpg" alt="img" style="zoom:67%;" /> 

内存中的一个地址块放了magedu，而name就表示地址值。于是就是name中存放了magedu。



变量，值可变化，当然也有不可变

 

![img](2-shell编程特殊位置变量.assets/clip_image038.jpg)

python和shell都不需要事先申明变量

##  变量起名规范

![img](2-shell编程特殊位置变量.assets/clip_image040.jpg)

##  特殊变量

![img](2-shell编程特殊位置变量.assets/clip_image042.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image044.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image046.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image048.jpg)

##  👆变量的正儿八经的写法，很重要 

![img](2-shell编程特殊位置变量.assets/clip_image050.jpg)

 

![img](2-shell编程特殊位置变量.assets/clip_image052.jpg)

### 如果此时Y的值变成了30，问X的值是多少，这个在PYTHON里面叫变量赋值，如果是列表、字典是需要.copy()的

![img](2-shell编程特殊位置变量.assets/clip_image054.jpg)

<img src="2-shell编程特殊位置变量.assets/clip_image056.jpg" alt="img" style="zoom:67%;" />

<img src="2-shell编程特殊位置变量.assets/clip_image058.jpg" alt="img" style="zoom:67%;" />

##  变量取消

![img](2-shell编程特殊位置变量.assets/clip_image060.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image062.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image064.jpg)

上图替换语法存在错误

![img](2-shell编程特殊位置变量.assets/clip_image066.jpg)

### 不用加g，%s/xx/yy/g，的g如果是每行只有一个不需要加g全局

![img](2-shell编程特殊位置变量.assets/clip_image068.jpg)

一般脚本结束了变量也就没了。不过还是建议删掉。

 

![image-20220202182620984](2-shell编程特殊位置变量.assets/image-20220202182620984.png) 

## 把命令放到变量里

![img](2-shell编程特殊位置变量.assets/clip_image072.jpg)



![image-20220202182715995](2-shell编程特殊位置变量.assets/image-20220202182715995.png)



![img](2-shell编程特殊位置变量.assets/clip_image076.jpg)

![img](2-shell编程特殊位置变量.assets/clip_image078.jpg)

第一题答案就有了

 

 cp -a  的a等价于-dR

### 文件夹不存在cp会直接创建的

![img](2-shell编程特殊位置变量.assets/clip_image080.jpg)

第二题答案

 

![img](2-shell编程特殊位置变量.assets/clip_image081.png)

 

![img](2-shell编程特殊位置变量.assets/clip_image083.jpg)

 

 nl 和 cat -b一个意思，不过不能列出空行行号

 ![image-20220202184946332](2-shell编程特殊位置变量.assets/image-20220202184946332.png) 

![image-20220202185054359](2-shell编程特殊位置变量.assets/image-20220202185054359.png) 

