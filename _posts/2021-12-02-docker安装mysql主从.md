---
title: docker安装mysql主从
tags:
  - 软件安装配置
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

#Master(主)：

首先拉取docker镜像,我们这里使用5.7版本的mysql：
```p
docker pull mysql:5.7
```

创建容器：
```p
docker run -p 3339:3306 --name mymysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

#-p   数据库端口
#MYSQL_ROOT_PASSWORD=123456    可根据自己需求修改密码
```

进入容器：
```p
docker ps   #查看正在运行的容器
docker exec -it 容器ID /bin/bash
```

安装一些配置
```p
apt-get update   #更新系统

apt-get install vim     #安装编辑工具
```

编辑数据库配置文件
```p
vim /etc/mysql/my.cnf
添加如下配置：
[mysqld]
## 同一局域网内注意要唯一
server-id=100  
## 开启二进制日志功能，可以随便取（关键）
log-bin=mysql-bin
```

重启mysql服务使配置生效
```p
service mysql restart

#重启mysql服务时会使得docker容器停止，我们还需要  'docker start 容器ID' 启动容器。
```

数据库授权
```p
mysql -uroot -p

CREATE USER '账号'@'%' IDENTIFIED BY '密码';

GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO '账号'@'%';
```

主库查询mysql状态
```p
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 | db           |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
#File、Position  用于下面配置同步需要
```

#Slave(从)

首先拉取docker镜像,我们这里使用5.7版本的mysql：
```p
docker pull mysql:5.7
```

创建容器：
```p
docker run -p 3339:3306 --name mymysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

#-p   数据库端口
#MYSQL_ROOT_PASSWORD=123456    可根据自己需求修改密码
```

进入容器：
```p
docker ps   #查看正在运行的容器
docker exec -it 容器ID /bin/bash
```

安装一些配置
```p
apt-get update   #更新系统

apt-get install vim     #安装编辑工具
```

编辑数据库配置文件
```p
vim /etc/mysql/my.cnf
添加如下配置：
[mysqld]
## 设置server_id,注意要唯一
server-id=101  
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin   
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin  
```

同步主库
```p
change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos= 154, master_connect_retry=30;

master_port：Master的端口号，指的是容器的端口号
master_user：用于数据同步的用户
master_password：用于同步的用户的密码
master_log_file：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
master_log_pos：从哪个 Position 开始读，即上文中提到的 Position 字段的值
master_connect_retry：如果连接失败，重试的时间间隔，单位是秒，默认是60秒
```

```p
start slave  				#保存配置

show slave status \G;       #查询主从同步状态

Slave_IO_Running: yes
Slave_SQL_Running: yes
#Slave_IORunning及Slave_SQL_Running进程必须正常运行，即yes状态，否则说明同步失败。
```









