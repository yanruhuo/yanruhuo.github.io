---
title: docker安装redis集群
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

#Redis集群搭建
分别使用以下命令启动3个Redis
```c
docker run --name redis-6379 -p 6379:6379 -d hub.c.163.com/library/redis
docker run --name redis-6380 -p 6380:6379 -d hub.c.163.com/library/redis
docker run --name redis-6381 -p 6381:6379 -d hub.c.163.com/library/redis

##添加密码验证 --requirepass 123456
```

使用docker ps命令，查看是否启动成功
```c
[root@localhost home]# docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS                    NAMES
e081c867f7ff        hub.c.163.com/library/redis   "docker-entrypoint..."   4 minutes ago       Up 4 minutes        0.0.0.0:6381->6379/tcp   redis-6381
cd739bd1302b        hub.c.163.com/library/redis   "docker-entrypoint..."   4 minutes ago       Up 4 minutes        0.0.0.0:6380->6379/tcp   redis-6380
4b5b623ebe8a        hub.c.163.com/library/redis   "docker-entrypoint..."   4 minutes ago       Up 4 minutes        0.0.0.0:6379->6379/tcp   redis-6379
```

分别使用docker inspect 容器ID命令，查看3个Redis内网IP地址（"IPAddress": "172.17.0.3"这个是IP）
```c
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "bb0390cc412381e39ce125383b45907ba852430f7ef9b3e983ccc504f294883d",
                    "EndpointID": "e466fb2c3142b6686253455491c8172cab849ae77aafc23856bdf67182b9bd9c",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03"
```

在Networks栏，可以看见该容器的Docker内网IP地址：
```c
redis-6379：172.17.0.2:6379
redis-6380：172.17.0.3:6379
redis-6381：172.17.0.4:6379
```

进入Docker容器内部
```c
使用redis-6379为主机，其余两台为从机
使用 docker exec -ti 容器ID /bin/bash 分别进入三个Redis容器
进入容器后，使用 redis-cli 命令，连接redis服务端
连接服务后，使用 info replication 查看当前机器的角色
未配置前，三台redis均为 master主机
```
使用上面的方法，分别进入 redis-6379、redis-6380、redis-6381容器内部，并连接redis服务端
```c
分别在redis-6380和redis-6381使用 SLAVEOF 172.17.0.2 6379 命令
在redis-6379 使用 info replication 命令，验证主从关系是否配置成功

127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.17.0.3,port=6379,state=online,offset=126,lag=0
slave1:ip=172.17.0.4,port=6379,state=online,offset=126,lag=0
master_replid:d47f0a8fc314858be3e9d9434948952d13c51284
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:126
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:126
```

这样，redis的集群环境就搭建好了

#Redis哨兵模式
Redis哨兵配置，有两种方案
```c
方案一：基于现有的3台Redis容器服务，互相启动一个Redis哨兵
方案二：重新再启动3台Redis容器服务，分别启动一个Redis哨兵
```

方案二会额外的新增3个Redis容器服务，所以这里演示方案一
分别进入3台Redis容器内部，执行以下操作
首先，进入Docker容器内部
```c
使用 docker exec -ti 容器ID /bin/bash 分别进入三个Redis容器
```

然后，编写Redis哨兵配置文件
```c
使用 cd / 命令，进入根目录
```

使用touch sentinel.conf命令，创建哨兵配置文件
在进行编辑时，需要先安装vim，命令为apt-get update ,apt-get install vim
使用 vim 命令编辑 sentinel.conf 文件，
添加以下内容
```c
sentinel monitor host6379 172.17.0.2 6379 1
```

最后，启动Redis哨兵

使用redis-sentinel /sentinel.conf启动Redis哨兵监控
使用ps –ef |grep redis命令，可以看到redis-server和redis-sentinel正在运行

至此，哨兵模式配置完毕。






























