# 第2节 centos7和rocky_linux安装详解

1、安装前最好校验一下，防止iso损坏。

2、密码不符合安全要求，需要点击两次done。

3、文件格式化，linux centos6默认是ext4，也支持windows里的vfat，也支持centos7里的xfs。就用默认的就行

4、分区的sda1或sda2对应/还是/boot，这个序号对应关系不大，只要空间够就行。

5、ctrl + alt f2切到命令行界面，f6切回图形界面，**cat /proc/meminfo** 便于你在装机阶段查看内存大小。shift pageUp往上翻，memTotoal可见总内存大小。当然你要说 | less | more ，我也没办法。

6、在分区的时候，扩展分区是自动给你分的，你只要知道你在划分sda5的时候，会自动给你划分sda4—扩展分区就行了。

7、**cat /proc/partition** 可见当前只有一个sda

8、**ls /dev/sda\*** 可见只有一个以sda开头的文件

![partitionShow](./pics\2\partitionShow.png)

9、等你的分区，确定格式化，werite chages to disk或done的时候，再去看ls /dev/sda*去看就看到分区开始实施了， cat /proc/partitions 也有了。



10、boot loader环节后面再说，这跳过

11、工作中一般都是minimal最小化安装，节约资源。学习选择Desktop或Server with GUI。

12、desktop，图形默认是GNOME，还有一种是KDE，一般不用KDE的（选择customize now—Desktops—KDE Desktop）。

13、centos7的安装注意：

1、内存2G，1G会导致系统安装报错-内存不够。

2、分区一样的，选择I will configure partitioning -Done-进入分区界面

  单位可以输入比如1G

3、KDUMP别管，内核分析记录用的，系统崩溃查看用的，不是一般人玩的。默认是enabled，建议关掉，反正用不到。

4、关于ens33将来要改成eth1的

5、安装后，做好快照

14、ubuntu-18.04.1的安装 略，开发的，我看到的 智能小车，图像识别什么的用的就是ubuntu系统。

15、在安装过程中，可以 ctrl + alt f1 f2去切命令行界面 如果需要的话

16、centos允许你用root登陆，ubuntu不允许使用root，可以改passwd使能root的。

