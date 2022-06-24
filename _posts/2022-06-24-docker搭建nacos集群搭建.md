---
title: Docker方式部署nacos
集群
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 第一步、安装数据库
```p
docker pull mysql:5.7

docker run -p 3339:3306 --name mymysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

## 第二步、数据库导入文件
```p

https://github.com/alibaba/nacos/releases

下载nacos/nacos-server:2.0.2，本地解压 
nacos/conf/schema.sql 文件导入mysql创建好的nacos库里面
```

## 第二步、安装nacos
···p
docker run -d \
 -e PREFER_HOST_MODE=hostname \
 -e MODE=cluster \
 -e NACOS_SERVER_PORT=8848 \
 -e NACOS_SERVER_IP=172.21.134.228 \
 -e NACOS_SERVERS="172.21.134.228:8848 172.21.134.226:8848 172.21.134.234:8848" \
 -e SPRING_DATASOURCE_PLATFORM=mysql \
 -e MYSQL_SERVICE_HOST=172.21.134.227 \
 -e MYSQL_SERVICE_PORT=3306 \
 -e MYSQL_SERVICE_USER=root \
 -e MYSQL_SERVICE_PASSWORD=1PGCwrs! \
 -e MYSQL_SERVICE_DB_NAME=nacos \
 -p 8848:8848  \
 --name nacos1  nacos/nacos-server:2.0.2

docker run -d \
 -e PREFER_HOST_MODE=hostname \
 -e MODE=cluster \
 -e NACOS_SERVER_PORT=8849 \
 -e NACOS_SERVER_IP=172.21.134.226 \
 -e NACOS_SERVERS="172.21.134.228:8848 172.21.134.226:8848 172.21.134.234:8848" \
 -e SPRING_DATASOURCE_PLATFORM=mysql \
 -e MYSQL_SERVICE_HOST=172.21.134.227 \
 -e MYSQL_SERVICE_PORT=3306 \
 -e MYSQL_SERVICE_USER=root \
 -e MYSQL_SERVICE_PASSWORD=1PGCwrs! \
 -e MYSQL_SERVICE_DB_NAME=nacos \
 -p 8848:8848  \
 --name nacos2  nacos/nacos-server:2.0.2


docker run -d \
 -e PREFER_HOST_MODE=hostname \
 -e MODE=cluster \
 -e NACOS_SERVER_PORT=8850 \
 -e NACOS_SERVER_IP=172.21.134.234 \
 -e NACOS_SERVERS="172.21.134.228:8848 172.21.134.226:8848 172.21.134.234:8848" \
 -e SPRING_DATASOURCE_PLATFORM=mysql \
 -e MYSQL_SERVICE_HOST=172.21.134.227 \
 -e MYSQL_SERVICE_PORT=3306 \
 -e MYSQL_SERVICE_USER=root \
 -e MYSQL_SERVICE_PASSWORD=1PGCwrs! \
 -e MYSQL_SERVICE_DB_NAME=nacos \
 -p 8848:8848  \
 --name nacos3  nacos/nacos-server:2.0.2
```

