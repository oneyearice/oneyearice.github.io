# 第2节 Docker的自定义网络和网络间通信



docker默认的网络环境，不管是什么模式都是自带的

![image-20240604155905829](2-Docker的自定义网络和网络间通信.assets/image-20240604155905829.png)

哦，这个已经被我改掉了，还原一下



![image-20240604160227505](2-Docker的自定义网络和网络间通信.assets/image-20240604160227505.png)

bridge就是对应docker0网卡所在的虚拟网桥

host就是对应eth0网卡所在的宿主机的真实网络

none就是不与外界通信的容器单机游戏了



这个docker network ls看到的三个网络(bridge、host、none)是默认自带一套的。如果不想用这套网络，可以自建。



# 自定义容器网络

不用原来的三个bridge、host、none，但是类型也就是DRIVER还是bridge、host、null

<img src="2-Docker的自定义网络和网络间通信.assets/image-20240604160824974.png" alt="image-20240604160824974" style="zoom:50%;" />

```shell
[root@realserver2 ~]# docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.


```

```shell
[root@realserver2 ~]# docker network create --help

Usage:  docker network create [OPTIONS] NETWORK

Create a network

Options:
      --attachable           Enable manual container attachment
      --aux-address map      Auxiliary IPv4 or IPv6 addresses used by Network driver (default map[])
      --config-from string   The network from which to copy the configuration
      --config-only          Create a configuration only network
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway strings      IPv4 or IPv6 Gateway for the master subnet
      --ingress              Create swarm routing-mesh network
      --internal             Restrict external access to the network
      --ip-range strings     Allocate container ip from a sub-range
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt map         Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label list           Set metadata on a network
  -o, --opt map              Set driver specific options (default map[])
      --scope string         Control the network's scope
      --subnet strings       Subnet in CIDR format that represents a network segment
[root@realserver2 ~]#

```



``` shell
docker network create -d <mode> --subnet <CIDR> --gateway <网关>

# 注意mode不支持host和none，默认是bridge模式
-d 写了也白写，主要是有个其他swan不怎么用的技术模式在厘面。整体来说-d可以忽略。

```



```shell
docker network create -d bridge --subnet 172.27.0.0/16 --gateway 172.27.0.1 test-net
```

![image-20240611103336882](2-Docker的自定义网络和网络间通信.assets/image-20240611103336882.png)

![image-20240611103421110](2-Docker的自定义网络和网络间通信.assets/image-20240611103421110.png)

视频里老师用的是ubuntu的，可以table补齐看得到其他的network类型的

![image-20240611103812822](2-Docker的自定义网络和网络间通信.assets/image-20240611103812822.png)

除了上面自定义的test-net类型，还有overlay、macvlan这些没见过的。



创建容器的时候使用新建的network，此时ip段就是172.27.0.0/16的了。

![image-20240611104051636](2-Docker的自定义网络和网络间通信.assets/image-20240611104051636.png)

然后容器的网卡也是和物理网卡有一个eth-pair的就是一半在外面的，

![image-20240611104414851](2-Docker的自定义网络和网络间通信.assets/image-20240611104414851.png)

里面是6：7，外面是7：6

![image-20240611104726013](2-Docker的自定义网络和网络间通信.assets/image-20240611104726013.png)

同样这个eth-pair就不是桥接到docker0这个默认网桥了，而是自定义那个test-net



使用场景，10来个容器在独立的网段里，就可以用上面讲的自定义的虚拟网桥了。

### 然后容器里的IP确实是可以固定

![image-20240611113623139](2-Docker的自定义网络和网络间通信.assets/image-20240611113623139.png)





### 自定义虚拟交换机直接ping容器名

还有一个：这种自定义网络可以直接ping容器名，存在自动解析的效果

![image-20240611151758060](2-Docker的自定义网络和网络间通信.assets/image-20240611151758060.png)



![image-20240611151724599](2-Docker的自定义网络和网络间通信.assets/image-20240611151724599.png)



**使用默认的docker0就不行**：什么不行，就是直接ping c1不会给你自动解析

![image-20240611152031373](2-Docker的自定义网络和网络间通信.assets/image-20240611152031373.png)



![image-20240611152041682](2-Docker的自定义网络和网络间通信.assets/image-20240611152041682.png)



只能使用--link来完善一下

![image-20240611152848512](2-Docker的自定义网络和网络间通信.assets/image-20240611152848512.png)



### 所以在代码层面，解决容器IP不固定的问题

方法如下：3个方法都无需担心容器里的IP发生变化，都会自动更新，除非你人为进容器里修改IP。

1、使用自定义的虚拟交换机，ip可固定

2、是用自定义的虚拟交换机，无需IP固定，直接使用对方的容器名；

3、使用默认的docker0虚拟交换机，采用--link 来实现hosts解析



测试方法2的IP是否自动更新---要知道方法3--link c1的IP如果变了只要是dhcp的，机会自动更新的。

![image-20240611153536642](2-Docker的自定义网络和网络间通信.assets/image-20240611153536642.png)

OK，没问题，就是自动更新的👇

![image-20240611153947030](2-Docker的自定义网络和网络间通信.assets/image-20240611153947030.png)



还是用wordpress来实验，这次就使用自定义虚拟网桥来做吧

```shell
docker network create --subnet 172.27.0.0/16 --gateway 172.27.0.1 wordpress_net

docker run -d --name db001 --net wordpress_net -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=123456 -v /data/mysql:/var/lib/mysql --restart=always mariadb:11.4.2

docker run -d --name wordpress -p 80:80 --net wordpress_net -v /data/wordpress:/var/www/html --restart=always wordpress:php8.3-apache
```

看到这两个cli就要知道这个版本的适配要去wordpress的相关官方信息去找

https://github.com/WordPress/WordPress

![image-20240611170946930](2-Docker的自定义网络和网络间通信.assets/image-20240611170946930.png)



**实验记录**

1、创建虚拟交换机

![image-20240612091719392](2-Docker的自定义网络和网络间通信.assets/image-20240612091719392.png)



![image-20240612092628057](2-Docker的自定义网络和网络间通信.assets/image-20240612092628057.png)



2、全部删掉，重新来一遍

![image-20240612103539537](2-Docker的自定义网络和网络间通信.assets/image-20240612103539537.png)





![image-20240612103613568](2-Docker的自定义网络和网络间通信.assets/image-20240612103613568.png)

由于是新建的虚拟交换机下的wordpress和db，所以数据库主机那项直接填写容器名db001就可以了，会自动解析的。

![image-20240612103709223](2-Docker的自定义网络和网络间通信.assets/image-20240612103709223.png)



**举例**

如果容器里没法查看路由、网关信息，可以在外面通过Inspect容器去查看的

![image-20240612110538028](2-Docker的自定义网络和网络间通信.assets/image-20240612110538028.png)





### 虚拟网桥之间的互通

1、默认就是不通的，因为有DOCKER-ISOLATION-STAGE-2这个隔离链

![image-20240612115458869](2-Docker的自定义网络和网络间通信.assets/image-20240612115458869.png)



iptables的规则查看-S比较清晰

![image-20240612134023430](2-Docker的自定义网络和网络间通信.assets/image-20240612134023430.png)

导出来再修改

```shell
iptables -S   #只能看看
iptables-save > iptables.rule
iptables-restore < iptables.rule
```

添加ACCEPT

![image-20240612135656886](2-Docker的自定义网络和网络间通信.assets/image-20240612135656886.png)

但这样就怕原来一些规则被抢了，这些规则如果仅仅是ACCEPT还好，或者是DROP你这里实验大不了就不DROP了，就怕是一些修改的动作被直接ACCEPT了。好像这里不涉及。



此时不同虚拟网桥下的容器就通了

![image-20240612142317323](2-Docker的自定义网络和网络间通信.assets/image-20240612142317323.png)

但是这仅仅实验，不会这么修改docker自动生成的iptables



恢复iptables直接重启docker就行了

此时不通虚拟交换机下的容器就恢复到不通了，我们用另外一种方式来实现

![image-20240612144728357](2-Docker的自定义网络和网络间通信.assets/image-20240612144728357.png)

```shell
docker network connect 虚拟交换机  容器
```

要让c1能够给和wordpress和db001互通，就把c1 连接到对方的虚拟交换机上。

![image-20240612145956606](2-Docker的自定义网络和网络间通信.assets/image-20240612145956606.png)



本质上docker network connect 虚拟网络B 容器A 

就是将容器A挂到虚拟网络B下，拿到了B网络的一个IP（当然会加一个接口）。所以此时容器A就可以ping通网络B下的容器了，因为在一个网段了。

![image-20240612150212282](2-Docker的自定义网络和网络间通信.assets/image-20240612150212282.png)

当然hostname -i有时候也看不全

![image-20240612150516933](2-Docker的自定义网络和网络间通信.assets/image-20240612150516933.png)

其实是hosts解析丢了

![image-20240612150956082](2-Docker的自定义网络和网络间通信.assets/image-20240612150956082.png)



删掉重来看看

![image-20240612151204276](2-Docker的自定义网络和网络间通信.assets/image-20240612151204276.png)

然后梳理一下bridge和docker0的对应，以及wordpress_net和ip a 上面的对应关系怎么查看找的

首先 ip a上关注docker0和br-xxxx这种格式就是代表虚拟网桥了

![image-20240612151524526](2-Docker的自定义网络和网络间通信.assets/image-20240612151524526.png)

然后

![image-20240612151620272](2-Docker的自定义网络和网络间通信.assets/image-20240612151620272.png)

这就找到了

同时inspect 网桥里还可以看到哪个容器挂载网桥里



然后看看正常情况hostsname -i 可以看到ip的，因为/etc/hosts里有解析就能看

![image-20240612151844095](2-Docker的自定义网络和网络间通信.assets/image-20240612151844095.png)

然后故障就复现了👇

![image-20240612152116697](2-Docker的自定义网络和网络间通信.assets/image-20240612152116697.png)

原来network connect和disconnect会破坏容器里的/etc/hosts文件里的解析



wordrpess的docker里太精简了，给他安装一下工具

https://mirrors.tuna.tsinghua.edu.cn/help/debian/

sed也行，这里直接cat > xxx << EOF灌进去就行了

![image-20240612153158594](2-Docker的自定义网络和网络间通信.assets/image-20240612153158594.png)



可见：![image-20240612153437114](2-Docker的自定义网络和网络间通信.assets/image-20240612153437114.png)

hostname -i只是看看解析的地址，并不一定是真实的ip地址👆

hostname -I是准的

![image-20240612153528380](2-Docker的自定义网络和网络间通信.assets/image-20240612153528380.png)

总之，你使用docker network connect / disconnect 要小心/etc/hosts的内容变化的，同时要小心不要踩到hostname -i的坑。







![image-20240612162538959](2-Docker的自定义网络和网络间通信.assets/image-20240612162538959.png)



慢慢来吧👆









# 工作案例-openvpn容器化



## 1、准备材料

**checkpsw.sh**

```shell
#!/bin/sh

PASSFILE="/etc/ppp/chap-secrets"
LOG_FILE="/etc/openvpn/logs/openvpn-password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

if [ ! -r "${PASSFILE}" ]; then
echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
exit 1
fi
CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $3;exit}' ${PASSFILE}`
if [ "${CORRECT_PASSWORD}" = "" ]; then
echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
fi
if [ "${password}" = "${CORRECT_PASSWORD}" ]; then
echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
exit 0
fi
echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1


```

chmod +x checkpsw.sh



**connect**

```
#!/bin/sh
day=`date +%F`
if [ -f /data/logs/openvpn/$day ]
then
        echo "`date '+%F %H:%M:%S'` User $common_name IP $trusted_ip is logged $1" >>/data/logs/openvpn/$day
else
    mkdir -p /data/logs/openvpn/
        touch /data/logs/openvpn/$day
        echo "`date '+%F %H:%M:%S'` User $common_name IP $trusted_ip is logged $1" >>/data/logs/openvpn/$day
fi
```

chmod +x connect



**entrypoint.sh**

```shell
#!/bin/sh


P_PATH=/etc/openvpn/easy-rsa
W_PATH=/etc/openvpn

sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories && apk update && apk add openvpn easy-rsa

mkdir -p $P_PATH
cp -a /usr/share/easy-rsa/* $P_PATH
cd /etc/openvpn/easy-rsa/

echo yes |./easyrsa init-pki


cat > /etc/openvpn/easy-rsa/pki/vars << EOF
export KEY_COUNTRY="CN"
export KEY_PROVINCE="SH"
export KEY_CITY="SHANGHAI"
export KEY_ORG="xxx"
export KEY_EMAIL="xxx@xxx.com"
export KEY_CN=vpn.xx.com
export KEY_NAME=vpnserver
export KEY_OU=OPS
export EASYRSA_CA_EXPIRE=36500
export EASYRSA_CERT_EXPIRE=36500
EOF
. ./pki/vars

echo yes |./easyrsa build-ca nopass

echo yes |./easyrsa build-server-full server nopass

echo "" |./easyrsa gen-dh
openvpn --genkey secret ta.key
mkdir -p /etc/openvpn/server/certs
cd /etc/openvpn/server/certs
cp /etc/openvpn/easy-rsa/pki/dh.pem ./
cp /etc/openvpn/easy-rsa/pki/ca.crt ./
cp /etc/openvpn/easy-rsa/pki/issued/server.crt ./
cp /etc/openvpn/easy-rsa/pki/private/server.key ./
cp /etc/openvpn/easy-rsa/ta.key ./


mkdir -p /var/log/openvpn
chmod 777 /var/log/openvpn/
cd /etc/openvpn/
cat >> server.conf << EOF
port 1194
proto udp
dev tun
tun-mtu 1500
topology subnet
server 10.106.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 114.114.114.114"
keepalive 10 300
cipher AES-256-CBC
#comp-lzo
max-clients 1000
persist-key
persist-tun
verb 4
mute 20
reneg-sec 0
explicit-exit-notify 1
key-direction 0
tls-crypt /etc/openvpn/server/certs/ta.key 0
ifconfig-pool-persist /etc/openvpn/ipp.txt
username-as-common-name
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
script-security 3
client-config-dir /etc/openvpn/user-static-ip
client-connect "/etc/openvpn/connect connected"
client-disconnect "/etc/openvpn/connect closed"
ca /etc/openvpn/server/certs/ca.crt
cert /etc/openvpn/server/certs/server.crt
key /etc/openvpn/server/certs/server.key
dh /etc/openvpn/server/certs/dh.pem
log /var/log/openvpn/server.log
log-append /var/log/openvpn/server.log
status /var/log/openvpn/status.log
EOF
mkdir user-static-ip
echo "ifconfig-push 10.106.0.2 255.255.255.0" > user-static-ip/user1@test.com

chmod +x connect

mkdir -p /etc/ppp
cat >> /etc/ppp/chap-secrets << EOF
user1@test.com * cisco 10.106.0.2
EOF

mkdir -p /etc/openvpn/logs
touch /etc/openvpn/logs/openvpn-password.log


chmod +x ${W_PATH}/checkpsw.sh

cd /etc/openvpn/easy-rsa
echo yes |./easyrsa build-client-full client nopass
cd ../
HOST_IP=$(wget -qO - ifconfig.me )
echo $HOST_IP
cat >> ${HOST_IP}.ovpn << EOF
client
dev tun
tun-mtu 1500
proto udp
sndbuf 0
rcvbuf 0
remote ${HOST_IP} 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
#comp-lzo
verb 3
cipher AES-256-CBC
auth-user-pass
auth-nocache
script-security 3
key-direction 1
reneg-sec 0
mute-replay-warnings
explicit-exit-notify 1
<ca>
</ca>
<cert>
</cert>
<key>
</key>
<tls-crypt>
</tls-crypt>
EOF

sed -ir "/<ca>/r ${P_PATH}/pki/ca.crt"  ${HOST_IP}.ovpn
sed -ir "/<cert>/r ${P_PATH}/pki/issued/client.crt" ${HOST_IP}.ovpn
sed -ir "/<key>/r ${P_PATH}/pki/private/client.key" ${HOST_IP}.ovpn
sed -ir "/<tls-crypt>/r ${P_PATH}/ta.key" ${HOST_IP}.ovpn


mkdir -p /dev/net
mknod /dev/net/tun c 10 200
chmod 666 /dev/net/tun

exec "$@"
```

chmod +x entrypoint.sh





**Dockerfile**

```shell
FROM alpine:3.20.0
LABEL maintainer="oneyearice <oneyearice@gmail.com>"
ADD connect checkpsw.sh /etc/openvpn/
ADD /entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["/usr/sbin/openvpn","--config","/etc/openvpn/server.conf"]
```



## 2、制作镜像

```shell
docker build -t openvpn:240606 .
```

![image-20240606141045927](2-Docker的自定义网络和网络间通信.assets/image-20240606141045927.png)





### 3、启动容器

**docker run的时候加上--cap-add NET_ADMIN**

```shell
docker run -d --name openvpnser --cap-add NET_ADMIN -p 1194:1194/udp openvpn:240606
```



然后可以通过logs查看日志

![image-20240606152725386](2-Docker的自定义网络和网络间通信.assets/image-20240606152725386.png)

2048长度的时间较长

./easyrsa gen-dh  耗时较长，会卡在Generating DH parameters, 2048 bit long safe prime 提示处，因为底层是：openssl dhparam -out /etc/openvpn/easy-rsa/pki/b03e75eb/temp.1.1 2048





能拨上去了，但是网络不通



这里有一个简单点的案例参考

https://blog.51cto.com/fengwan/1896431





