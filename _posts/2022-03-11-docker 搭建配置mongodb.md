---
title: docker 搭建配置mongodb
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 拉取镜像 默认最新版本
```p
docker pull mongo
```

##  查看镜像
```p
docker images
```

## 启动镜像
```p
docker run -p 27017:27017 -td mongo
```

## 查看启动的镜像
```p
docker ps 
```

## 进入容器
```p
docker exec -it 镜像id /bin/bash 
```

## 创建管理账户用户和普通用户
```p
mongo 			#进入数据库

#创建admin账户
use admin
db.createUser(
{
user: "admin",
pwd: "admin",
roles: [ { role: "root", db: "admin" } ]  
}
);

mongo --port 27017 -u admin -p admin --authenticationDatabase admin #以刚建立的用户登录数据库 创建driving用户

db.createUser(
{
user: "beautifulsoup",
pwd: "password",
roles: [
{ role: "readWrite", db: "driving" }
]
}
);

show dbs 				#显示数据库

use dbname 				#切换到数据库

show collections 		#显示表

db.find.表名 			#查看表数据
```

## Mongodb数据账户权限详细信息
```p
数据库用户角色：read、readWrite；
数据库管理角色：dbAdmin、dbOwner、userAdmin;
集群管理角色：clusterAdmin、clusterManager、4. clusterMonitor、hostManage；
备份恢复角色：backup、restore；
所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
超级用户角色：root
内部角色：__system

Read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向system.users集合写入，可以在指定数据库里创建、删除和管理用户
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，超级权限
```
注：mongodb创建库的时候需要导入数据才能生效