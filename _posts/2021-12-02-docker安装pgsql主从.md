---
title: docker安装pgsql主从
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

# 主库安装部署
## 创建数据盘
```p
mkdir -p  /data/postgresql/data
```

## 运行主库容器
```p
docker run --name pg_test --restart=always  -v /data/postgresql/data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=密码 -p 5432:5432 -d postgres:11.6
```

## 进入启动的 docker 容器内部
```p
docker exec -it pg_test /bin/bash
```

## 进入 postgres 客户端，新建用于同步数据的用户
```p
#切换到 postgres 用户
su postgres

#进去 postgres 客户端
psql

#新建用于同步数据的用户
CREATE ROLE replica login replication encrypted password 'replica';  #可自行修改密码

#利用 \du 命令可以查看 postgres 的用户列表
\du
#如下图
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 replica   | Replication                                                | {}
```

## 返回容器，安装vim编辑器
```p
\q     #退出pgsql数据库
exit   #退出pgsql账号

apt-get update  		 #更新系统
apt-get install vim      #安装编辑工具
```

## 修改 配置文件
```p
vim /var/lib/postgresql/data/pg_hba.conf
#添加
host   replication      replica       从库IP/32          trust   #允许从库访问

vim /var/lib/postgresql/data/postgresql.conf
#修改或者在最下方增加
listen_addresses = '*'   			# 监听所有IP
archive_mode = on  					# 允许归档
archive_command = '/bin/date' 		# 用该命令来归档logfile segment,这里取消归档。
wal_level = replica 				# 开启热备
max_wal_senders = 32 				# 这个设置了可以最多有几个流复制连接，差不多有几个从，就设置几个
wal_keep_segments = 64 				# 设置流复制保留的最多的xlog数目，一份是 16M，注意机器磁盘 16M*64 = 1G
wal_sender_timeout = 60s 			# 设置流复制主机发送数据的超时时间
max_connections = 200 				# 这个设置要注意下，从库的max_connections必须要大于主库的
```

## 重启容器
```p
docker restart pg_test
```

# 从库安装配置
## 创建数据盘
```p
mkdir -p  /fhpt/postgresql/data
```

## 运行从库容器
```p
docker run --name pg_test --restart=always  -v /fhpt/postgresql/data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=Fhpt@2021! -p 5432:5432 -d postgres:11.6
```

## 进入启动的 docker 容器内部
```p
docker exec -it pg_test /bin/bash
```

## 拷贝主服务器数据进入从节点容器并同步
```p
su postgres									#切换数据库用户

rm -rf /var/lib/postgresql/data/*			#删除data下面数据

#拷贝主服务器数据
pg_basebackup -h 主库IP -U replica -D /var/lib/postgresql/data -X stream -P

执行完上述命令之后，容器自动退出
```

## 修改配置文件
```p
cd /fhpt/postgresql/data

vim /fhpt/postgresql/data/recovery.conf
#添加下列内容
standby_mode = on
primary_conninfo = 'host=主库IP port=5432 user=replica password=replica'
recovery_target_timeline = 'latest'

vim /fhpt/postgresql/data/postgresql.conf
#替换之前配置的参数
wal_level = replica
max_connections = 1000
hot_standby = on
max_standby_streaming_delay = 30s
wal_receiver_status_interval = 10s 
hot_standby_feedback = on
```

## 重启容器
```p
docker stop pg_test    	#停止容器
docker start pg_test	#启动容器
docker restart pg_test	#重启容器
```

## 验证
### 验证方法一：
```p
docker exec -it pg_test /bin/bash
su postgres
psql
select client_addr,sync_state from pg_stat_replication;
#下面出现从库IP代表成功
 client_addr  | sync_state 
--------------+------------
 10.95.130.37 | async
(1 row)
```

### 验证方法二：
```p
主库上：
docker exec -it pg_test /bin/bash
su postgres
ps -ef | grep wal
postgres     26      1  0 14:46 ?        00:00:00 postgres: walwriter  
postgres     32      1  0 14:47 ?        00:00:00 postgres: walsender replica 10.95.130.23(57476) streaming 0/9000328
postgres    159    125  0 16:17 pts/0    00:00:00 grep wal

从库上:
ps -ef | grep wal
postgres     27      1  0 14:47 ?        00:00:03 postgres: walreceiver   streaming 0/9000328
postgres     79     67  0 16:17 pts/0    00:00:00 grep wal
```

测试同步状态
```p
我们再主库上创建一张表：
create table t1(name varchar(10));
insert into t1 values('liu');
select * from t1;
 name
----
 liu
(1 row)

仓库上查看
insert into t1 values('liu');
select * from t1;
 name
----
 liu
(1 row)
```

## 主备切换测试
```p
在主服务器上执行:
docker stop pg_test         		#关闭主库

#备库上执行pg_ctl promote命令
docker exec -it pg_test bash  		#进入备库容器

su postgres 						#切换数据库用户

/usr/lib/postgresql/11/bin/pg_ctl promote -D $PGDATA   #命令执行后，如果原来的 recovery.conf 更名为 recovery.done, 表示切换成功

#时如果需要将老的主库切换成备库，在老的主库的$PGDATA目录下也创建recovery.conf文件（创建方式跟之前介绍的一样，内容可以和原从库pghost2的一样，只是primary_conninfo的IP换成对端pghost2的IP）

主库创建recovery.conf文件：
recovery_target_timeline = 'latest'
standby_mode = on
primary_conninfo = 'host=10.95.130.83 port=5432 user=replica password=replica'

#启动老的主库pghost1。这时观察主、备进行是否正常
```

## 检测
```p
新主库（pghost2）上执行：
postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
(1 row)

新备库（pghost1）上继续执行：
postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)

发现它目前的角色也已经切换为备库了
```




