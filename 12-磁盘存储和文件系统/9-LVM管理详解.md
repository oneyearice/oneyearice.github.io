# 第9节. LVM管理详解

## 1、创建管理

分区一旦划分好，就没法扩了。而LVM就能扩和缩

LVM的底层可能就是raid为主，当然也可以是多块硬盘和分区。

10块注册raid10，OS看到的就是sda一块盘，以前就直接sda上分区，一旦分区就固定了，分区不够用也不能扩，所以就再sda上做LVM来管理，如何管，假设一组raid10得到sda，还有一组sdb、再来一个sdc1分区，这些raid也好、硬盘也好、分区也好，都是linux Block Devices。这些块设备大小不限，无所谓。

<img src="9-LVM管理详解.assets/image-20220310175954098.png" alt="image-20220310175954098" style="zoom:80%;" />

把这些块设备第一步通过pvcreate变成Physical Volumes物理卷，所以物理卷就是打上标签--意思就是这些块设备不作为独立硬盘用，将来要做逻辑卷的。

有几个块设备(硬盘或分区)就生成几个物理卷，就是打标签吗，原来设备叫啥名，生成后还叫啥名。



然后通过vgcreate将物理卷们 组成一个大的集合 也就是卷组。此时卷组就是一个大的硬盘咯。



然后通过lvcreate将卷组 分成一个个 逻辑卷出来。



逻辑卷不关心上面的数据存放在哪块物理设备上，

逻辑卷的地位就是分区了，该创建文件系统就创建文件系统，该挂载就挂载。



如果LVM1 500G ，LVM2 200G，下面的卷组是1T，将来LVM1满了不够用了，还可以**在线**扩展到800G。所谓在线就是用户无感--使用不受影响，无需取消挂载，直接在线挂载的状态下，一条命令就挂上去了。

如果lvm1 800G也满了，怎么办，可以再底层块设备上再加入新的硬盘。这一套还是在线扩展吗？⚪



所以linux默认安装的时候默认就是逻辑卷。



逻辑卷的概念了解后，还有一个PE，叫物理盘区**physical Extent**。在创建卷组的时候要指定PE，假设PE是16M，卷组就会显示有多少个PE，创建LVM逻辑卷的时候可以从这么多个PE中取出多少个PE来作为逻辑卷。扩展的时候也可以增加多少个PE。PE就是分配的最小单元了。



到现在已经是LVM2代了，LVM2。



在IBM AIX系统unix小型机，就只用LVM，不用分区。



LVM要建立在raid之上，硬盘坏了有冗余。①担心LVM多了一层逻辑层，硬盘坏了导致整个卷组出问题②多一层逻辑层，不是直接面对硬盘，性能是否不好。据说2点担心都是多余的。反正很多企业在用，也有企业不用。



学完再来这里补充发表自己的观点，和网上搜一下。



下面开操作

![image-20220310201034729](9-LVM管理详解.assets/image-20220310201034729.png)

![image-20220310201114620](9-LVM管理详解.assets/image-20220310201114620.png)

要注意修改ID为8e

![image-20220310201318855](9-LVM管理详解.assets/image-20220310201318855.png)

![image-20220310201420872](9-LVM管理详解.assets/image-20220310201420872.png)

存盘退出

![image-20220310201446432](9-LVM管理详解.assets/image-20220310201446432.png)

同步下

![image-20220310201516109](9-LVM管理详解.assets/image-20220310201516109.png)

![image-20220310201549170](9-LVM管理详解.assets/image-20220310201549170.png)

![image-20220310201647537](9-LVM管理详解.assets/image-20220310201647537.png)





### 第一步，物理卷的生成-pvcreate x y z\pvs\pvdisplay

pvs查看下，当前pv是空的

![image-20220310201825556](9-LVM管理详解.assets/image-20220310201825556.png)

把上面的一个分区、一个硬盘变成物理卷

![image-20220310202010422](9-LVM管理详解.assets/image-20220310202010422.png)

👆因为sdb1之前做个swap，所以有标记位，所以先dd清空前10M就能覆盖到了。也可以直接敲y就wipe一样也擦掉了。

然后再pvcreate /dev/sdb1 /dev/sdd也可以合起来些。

这样pvs和pvdisplay就能看到了

![image-20220310202323610](9-LVM管理详解.assets/image-20220310202323610.png)

pvs就是summary概况，其中lvm2就是逻辑卷2代；VG列是空 表示卷组没组建呢还。

pvdisplay同样可见VG是空；PE Size也是0 这个只有创建VG的时候指定才会出现。



### 第二步 vgcreate创建卷组

vg开头的也是一大堆命令

![image-20220310202633432](9-LVM管理详解.assets/image-20220310202633432.png)

同样有vgs和vgdisplay，然后pvdisplay肯定也能看VG和PE也就是vgcreate后的信息。

![image-20220310202653185](9-LVM管理详解.assets/image-20220310202653185.png) 

使用帮助看下

![image-20220310202921686](9-LVM管理详解.assets/image-20220310202921686.png) 

-s 指定PE的大小 pe--physical extent size

![image-20220310203126338](9-LVM管理详解.assets/image-20220310203126338.png) 

上图忘记加单位了，不过有默认值的pvdisplay可见。

vg整合好后

①pvs看

![image-20220310203214128](9-LVM管理详解.assets/image-20220310203214128.png) 

![image-20220310203233312](9-LVM管理详解.assets/image-20220310203233312.png)PE的默认值也看到了是4MB，上图PE size是4MB，total PE 1023就是1023个的意思。总容量也就是4G咯。同样/dev/sdd也就是4MB*2559=10G

②然后通过vgs 和vgdisplay看看

![image-20220310203621836](9-LVM管理详解.assets/image-20220310203621836.png)

👆vgs可见，vg0这个卷组里有2个PV--物理卷，LV还未划分，大小是两个PV的总和14g的样子。

<img src="9-LVM管理详解.assets/image-20220310203730947.png" alt="image-20220310203730947" style="zoom: 50%;" />

全部PE个数3582个，Alloc PE / Size 0 / 0就是没有分配出去呢--因为逻辑卷lvm还没有分配呢。

由于逻辑卷lvm还没有创建，所以还看不到vg0这个设备

![image-20220310203932380](9-LVM管理详解.assets/image-20220310203932380.png)



### 第三步 lvcreate创建lvm

![image-20220310204019058](9-LVM管理详解.assets/image-20220310204019058.png)

![image-20220310204030419](9-LVM管理详解.assets/image-20220310204030419.png) 



lvcreate --help可见用法：从哪个VG--卷组里 通过-L取 多大size  ；通过-l 取多少个PE，这事个数

<img src="9-LVM管理详解.assets/image-20220310204201872.png" alt="image-20220310204201872" style="zoom:80%;" /> 

上图有striped和raid1|mirror，这就是raid了，完全可以用lvm再做一层raid，实际上底层物理设备做raid后，上层lvm不会再做raid了。

![image-20220310204550779](9-LVM管理详解.assets/image-20220310204550779.png) 



![image-20220310204636582](9-LVM管理详解.assets/image-20220310204636582.png)

👆起个名字mysql，-L 直接写大小，-l数个数还要算--不用，从vg0里面划分lvm。

lvs 和 lvdisplay

![image-20220310204806905](9-LVM管理详解.assets/image-20220310204806905.png)

上图说明：

<img src="9-LVM管理详解.assets/image-20220310204926640.png" alt="image-20220310204926640" style="zoom:60%;" /> 这个就是完整的设备名，这其实是软连接👇

![image-20220310204852464](9-LVM管理详解.assets/image-20220310204852464.png)

实际上叫dm-0，还有一个和它一样指向的软连接

![image-20220310205112608](9-LVM管理详解.assets/image-20220310205112608.png)

所以待会可能看到的是上图这个名字/dev/mapper/vg0-mysql

看下现在的dm设备有几个：如果再创建一个lvm，这里就会多一个dm-1了

<img src="9-LVM管理详解.assets/image-20220310205245763.png" alt="image-20220310205245763" style="zoom:80%;" /> 

<img src="9-LVM管理详解.assets/image-20220310205349023.png" alt="image-20220310205349023" style="zoom:50%;" />



![image-20220310205704500](9-LVM管理详解.assets/image-20220310205704500.png)

上图的8G到底用的哪个物理卷，通过pvdisplay可以看到的

![image-20220310205817274](9-LVM管理详解.assets/image-20220310205817274.png)

可见/dev/sdb1全部空闲就是没有用，所以这个8G都用的/dev/sdd的空间4MB*2048也就是8G。做raid也是不关心具体到底存在那块盘上的，都是当作整体在用的。

![image-20220310210034104](9-LVM管理详解.assets/image-20220310210034104.png) 

2048个PE，在LVM叫LE，就是PE在LVM逻辑卷里的名称。

![image-20220310210211533](9-LVM管理详解.assets/image-20220310210211533.png)

一旦LVM划分了，blkid就能看到了

![image-20220310210312714](9-LVM管理详解.assets/image-20220310210312714.png)

现在由于还没有在LVM上创建文件系统呢，sdb1和sdd的UUID肯定是各归各的。所以还看不到一个sdb1和sdd合成一个的情况，其实合成一个情况是blkid是看不到清楚的，只能再下面的格式化看到多出来一个mysql的逻辑卷。而sdb1和sdd是逻辑卷的成员而已，具体是哪个lvm的成员还需要具体去看的。

### 第四步 mkfs.xfs格式化

![image-20220310210605821](9-LVM管理详解.assets/image-20220310210605821.png)

这个时候就看到，增加了一个新的记录，mysql这个逻辑卷就有文件系统了。

挂载的时候，要写设备名，可以写这里的/dev/mapper/vg0-mysql可以写当初创建的lvm起的名称/dev/vg0/mysql，这两个都是可以的，因为都是软连接。

<img src="9-LVM管理详解.assets/image-20220310211042285.png" alt="image-2022031021104228 5" style="zoom:67%;" /> 

![image-20220310211122143](9-LVM管理详解.assets/image-20220310211122143.png)

上面是临时挂载，永久挂要写fstab

![image-20220310211221800](9-LVM管理详解.assets/image-20220310211221800.png)

![image-20220310211241343](9-LVM管理详解.assets/image-20220310211241343.png)

![image-20220310211314788](9-LVM管理详解.assets/image-20220310211314788.png)

刚才挂过了，所以就行了，reboot也不怕了，正常就是mount -a就行了。



### 看看性能

![image-20220310211646809](9-LVM管理详解.assets/image-20220310211646809.png)

上图是不是有点夸张了，逻辑卷能提速这么快的？逻辑卷反正不会比物理分区慢就行了。

![image-20220310211829155](9-LVM管理详解.assets/image-20220310211829155.png)

上图就是conv=fdatasync就是结果显示的是已经同步到硬盘里的了。上上图的是写到内存里了就显示结果了。











## 2、管理详解





## 3、快照管理
