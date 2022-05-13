---
title: Docker方式部署redis-cluster集群
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 下载镜像
```
docker pull redis:5.0.5
```

## 创建 6 个 Redis 容器
```
docker create --name redis-node1 --net host -v /data/redis-data/node1:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-1.conf --port 6379
 
docker create --name redis-node2 --net host -v /data/redis-data/node2:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-2.conf --port 6380
 
docker create --name redis-node3 --net host -v /data/redis-data/node3:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-3.conf --port 6381
 
docker create --name redis-node4 --net host -v /data/redis-data/node4:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-4.conf --port 6382
 
docker create --name redis-node5 --net host -v /data/redis-data/node5:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-5.conf --port 6383
 
docker create --name redis-node6 --net host -v /data/redis-data/node6:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-6.conf --port 6384

分参数解释：
--cluster-enabled：是否启动集群，选值：yes 、no
--cluster-config-file 配置文件.conf ：指定节点信息，自动生成
--cluster-node-timeout 毫秒值： 配置节点连接超时时间
--appendonly：是否开启持久化，选值：yes、no
```

## 启动 Redis 容器
```
docker start redis-node1 redis-node2 redis-node3 redis-node4 redis-node5 redis-node6
```

## 组建 Redis 集群
```
# 这里以 redis-node1 实例为例
docker exec -it redis-node1 /bin/bash
 
# 组建集群192.168.4.201为当前物理机的ip地址
redis-cli --cluster create 192.168.4.201:6379 192.168.4.201:6380 192.168.4.201:6381 192.168.4.201:6382 192.168.4.201:6383 192.168.4.201:6384 --cluster-replicas 1
#  中间要输入yes
#  创建成功后 进入redis：redis-cli
#  看集群信息：cluster nodes 查看一下集群节点信息
```

## 设置密码
```
config set requirepass 'password'   // 设置密码
config set masterauth 'password'	// 设置从节点连接主节点的密码
```
