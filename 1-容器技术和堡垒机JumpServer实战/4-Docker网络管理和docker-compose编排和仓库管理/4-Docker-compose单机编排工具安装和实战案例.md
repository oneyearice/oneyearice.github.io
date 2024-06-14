# 第4节 Docker-compose单机编排工具安装和实战案例



docker-compose也是docker官方制作的



关键字编排，就是根据业务来管理，就是docker ps 看到的就是所有容器的陈列，不清楚所属的项目，但是docker compose就是业务层次清晰。也可以之启动某个业务的容器。



# 安装docker-compose



方法1：

![image-20240613174930701](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240613174930701.png)



方法2：直接yum

ubuntu查看版本

![image-20240613175049643](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240613175049643.png)



yum安装的就是版本低

![image-20240613175358965](4-Docker-compose单机编排工具安装和实战案例.assets/image-20240613175358965.png)



方法3：直接github下载

其实就是官网上走一波install步骤就行了啊











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

# 根据需要将while改成loading的实际进度执行时间就行了，不过要记得最后清除掉符号。
```





{% raw %}
<video id="my-video" class="video-js" controls preload="auto" width="100%" data-setup='{"aspectRatio":"16:9"}'>
  <source src="4-Docker-compose单机编排工具安装和实战案例.assets/shellForLoading.mp4" type='video/mp4' >
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
</video>

{% endraw %}





优化：

```shell
#!/bin/sh

char=('\' '|' '/' '—')
end=$((SECONDS+10))
echo "转$end秒"
while [ $SECONDS -lt $end ]
do
    for i in "${char[@]}"
    do
        echo -ne "\r$i"
        sleep 0.2
    done
done

echo -ne "\r"  # 清除最后的加载字符
```



