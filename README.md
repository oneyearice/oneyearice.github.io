# Introduction

* [课程介绍和<font color=red>·</font><font color=green>小</font><font color=red>·</font>目标](./introduction/课程介绍和小目标.md)

 		linxu~PY，go



* 目录生成的方法：
  * 打开pycharm里的windows/summary/list/directory_generate.py，运行
  * 将运行结果去头掐尾后，复制到nodepad过滤一遍格式，然后再复制到SUMMARY.md中即可。
  * 然后再gitbook init初始化目录结构。





* github 创建项目的提示备忘：

  ### …or create a new repository on the command line

  

  ```
  echo "# oneyearice.github.io" >> README.md
  git init
  git add README.md
  git commit -m "first commit"
  git branch -M main
  git remote add origin https://github.com/oneyearice/oneyearice.github.io.git
  git push -u origin main
  ```

  ### …or push an existing repository from the command line

  

  ```
  git remote add origin https://github.com/oneyearice/oneyearice.github.io.git
  git branch -M main
  git push -u origin main
  ```

  ### …or import code from another repository

  You can initialize this repository with code from a Subversion, Mercurial, or TFS project.

## 多终端pull和push注意点：

```
1、首先pull下来，得到最新的版本，如果是第一次git clone即可
2、复制oneyearice.github.io并重命名为gitbook；如果是git clone的就复制文件夹里的内容到gitbook下，选择替换原文件，得到最新的版本。  注意gitbook是本地编辑目录，oneyearice.gitbhu.io是pull和push目录
3、进入gitbook下运行gitbook install安装插件
3、在gitbook里编辑md文件，也就是主要工作内容
4、运行脚本自动上传
```

```
1、进入D盘git clone，如果有Oneyearice.github.io文件夹，进去后git pull
2、将oneyearice.github.io文件夹复制，并改名为gitbook
3、进入gitbook，删除node_module文件夹，cmd在gitbook文件夹下运行gitbook install
4、运行脚本自动push
```

