# 第1节. vim

大篇幅编辑使用软件如vscode的plugin弄就行了。

 

**会用到的备忘**：

 

:set nu

删除100dd,dGG,dgg

^$切换行尾行首w光标移动一个空格

 

```
vim +10 xxx  # 打开就进入了第10行
```

vim 颜色也不是都加的，/etc/passwd，复制到/data下再vim就会发现颜色没了。/etc/下的属于系统配置文件，所以给你加颜色了。

vim -m 只读打开

/XXX搜索，nN

 

u撤销

U 改行的修改全部撤销

 

s/要查找的内容/替换的内容/修饰符







![image-20220110205751552](1-vim.assets/image-20220110205751552.png) 

![image-20220110205806067](1-vim.assets/image-20220110205806067.png) 

![image-20220110205817934](1-vim.assets/image-20220110205817934.png) 



![image-20220110205831520](1-vim.assets/image-20220110205831520.png) 

![image-20220110205904424](1-vim.assets/image-20220110205904424.png) 





![image-20220111113314428](1-vim.assets/image-20220111113314428.png) 



这是正则，后面到了正则再说

![image-20220111113535161](1-vim.assets/image-20220111113535161.png) 





![image-20220111113548143](1-vim.assets/image-20220111113548143.png) 



![image-20220111113624830](1-vim.assets/image-20220111113624830.png) 













![image-20220111113657395](1-vim.assets/image-20220111113657395.png) 

整个文件的内容vim里替换用%s

```
:%s/XXX/YYY/g
```

/可以替换的

```
:%s#/dev#/tmp#g
```

 

 

 

![img](1-vim.assets/clip_image004.jpg)

![img](1-vim.assets/clip_image006.jpg)

0的ASCI码

![img](1-vim.assets/clip_image008.jpg)

![img](1-vim.assets/clip_image010.jpg)

![img](1-vim.assets/clip_image012.jpg)

cat也看不到

可以这么看二进制文件

![img](1-vim.assets/clip_image014.jpg)

00000000这是位置，后面00 00 00是内容

![image-20220111133808914](1-vim.assets/image-20220111133808914.png) 





![img](1-vim.assets/clip_image018.jpg)

![image-20220111133840511](1-vim.assets/image-20220111133840511.png) 

批量注释会有用



![img](1-vim.assets/clip_image022.jpg)

 

![image-20220111141200363](1-vim.assets/image-20220111141200363.png) 





复制到vim里，空行格式错位，可以试试

```
:set paste
```

![image-20220111142817455](1-vim.assets/image-20220111142817455.png) 





![image-20220112132942632](1-vim.assets/image-20220112132942632.png) 



这个比较好用，

![img](1-vim.assets/clip_image030.jpg)

![img](1-vim.assets/clip_image032.jpg)

![img](1-vim.assets/clip_image034.jpg)

 

![img](1-vim.assets/clip_image036.jpg)

跳行同学的福音



![img](1-vim.assets/clip_image038.jpg)

 ![img](1-vim.assets/vi-vim-cheat-sheet.gif)



 

 
