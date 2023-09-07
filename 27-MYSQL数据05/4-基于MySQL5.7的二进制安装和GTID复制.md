# 第4节. 基于MySQL5.7的二进制安装和GTID复制



# slave配置的是省去指定masterbinlog位置的方式

之前主复制都是指定MASTER_LOG_FILE='mariadb-bin.000023', 和 Master_log_pos=xxx

```
CHANGE MASTER TO  MASTER_HOST='192.168.126.129',  MASTER_USER='repluser',  
MASTER_PASSWORD='cisco',
MASTER_LOG_FILE='mariadb-bin.000023',  
MASTER_LOG_POS=691,
MASTER_SSL=1;
```



