# 第3节 Dockerfile常见指令ENTRYPOINT用法



# 概述



![image-20240509174316361](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509174316361.png)





## CMD是会直接全部覆盖掉

![image-20240509180227402](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509180227402.png)

同样之前实验的CMD是一串CLI，也被直接覆盖了

![image-20240509180942268](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509180942268.png)

但是ENTRYPOINT就不会了

![image-20240509181517255](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509181517255.png)

![image-20240509182406403](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509182406403.png)

没变化，再看其实是有变化的，只是docker ps 看不到太长了

![image-20240509183003429](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509183003429.png)

上图可见docker run 的cli也就是cat 123 全部都当作ENTRYPOINT的参数了。

这一点inspect看还是不明朗的👇，只能自己知道有这回事，哦cmd的所有参数都是合并进去的。

![image-20240509183228285](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509183228285.png)

### 然后看合并后的cli有专业的命令的

![image-20240509183832168](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509183832168.png)

emmm，工具真多啊

![image-20240509184316338](3-Dockerfile常见指令ENTRYPOINT用法.assets/image-20240509184316338.png)







