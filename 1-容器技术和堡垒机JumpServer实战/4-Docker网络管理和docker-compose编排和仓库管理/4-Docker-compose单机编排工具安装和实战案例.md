# 第4节 Docker-compose单机编排工具安装和实战案例









# 工作案例

简单加载小脚本

```shell
#!/bin/sh

char=('\' '|' '/' '—')

while true;do
for i in "${char[@]}";do
    echo -ne "\r$i"
    sleep 0.2
done;done

```

<video src="4-Docker-compose单机编排工具安装和实战案例.assets/shellForLoading.mp4"></video>





