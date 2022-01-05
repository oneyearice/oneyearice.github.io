# 第3节. linux入门操作和基础命令

17、**lsb_release -a**

18、关闭GUI，free可见 700M+的内存使用量，关闭前后的内存使用对比，init 3 关闭GUI，进入纯字符界面。再来看free 还剩200M+，一下500M的量省了。

19、runlevel 可见 5 3，说明之前是5模式切换到3的。

20、who  who -r可见这用户用的哪个命令

 

a

alias

b

basename

bc

c

cal 9 1752

chkconfig iptables off

cd

command

clock(hwclock)

cat /etc/rehat-release /proc/maminfo  /proc/partitions

d

dir类似ls

dirname

df

du

date

e

enable enable -n

exit

echo

f

free -h

g

getenforce disabled

h

hexdump -C

halt

history

hostname

help

hash

i

iptables -vnL

init 3字符模式  5图形   0是关机  6是重启

info

ifconfig 

id

j

k

l

lsblk

logout

ls

lsb_release -a

lshw

m

mount /dev/sr0 /mnt挂光盘到/mnt下

man

mv

mandb

n

ntpdate

o

p

ps

pwd

poweroff

ping

pstree

q

r

rpm -ivh 

rm

rz

runlevel查看当前运行模式的

s

systemctl disable firewalld

stat

shutdown

screen

sleep

source(.)

sz

sudo -i

t

touch

tty 看在哪个终端里

type

u

uname -r

unalias

v

vdir类似ll

w

which

whereis

whoami who -r

whatis

x

xxd 等价于hexdump -C

y

z

 

 

rz后再按esc可以产生如下图效果，并排两个提示符😶

![img](file:///C:/Users/MingYi/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

