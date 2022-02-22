---
title: docker安装zabbix，配置agent
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 镜像准备
根据 Zabbix 架构图，需要拉取如下几个镜像：
```p
docker pull zabbix/zabbix-web-nginx-mysql:5.0-centos-latest
docker pull zabbix/zabbix-server-mysql:5.0-centos-latest
docker pull zabbix/zabbix-proxy-mysql:5.0-centos-latest
docker pull zabbix/zabbix-snmptraps:5.0-centos-latest
docker pull zabbix/zabbix-agent:5.0-centos-latest
docker pull mysql:5.7
```

## 运行 Demo
运行本地数据库:
```p
docker run --name zbx5-mysql -d \
    --network=host \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="zabbix" \
    -e MYSQL_ROOT_PASSWORD="root" \
    --restart unless-stopped mysql:5.7 \
    --character-set-server=utf8 --collation-server=utf8_bin \
    --default-authentication-plugin=mysql_native_password
``` 

启动 server 后端:
```p
docker run --name zbx5-server-mysql -d \
    -e DB_SERVER_HOST="127.0.0.1" \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="zabbix" \
    -e MYSQL_ROOT_PASSWORD="root" \
    --network=host \
    --restart unless-stopped \
	zabbix/zabbix-server-mysql:5.0-centos-latest
``` 

启动 proxy:
由于默认 Server 和 Proxy 端口都为 10051，避免冲突，这里使用 11111
```p
docker run --name zbx5-proxy-mysql -d \
    -e DB_SERVER_HOST="127.0.0.1" \
    -e ZBX_LISTENPORT=10052 \
    -e ZBX_HOSTNAME="zabbix-proxy-test1" \
    -e ZBX_SERVER_HOST="127.0.0.1" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="zabbix" \
	-e MYSQL_ROOT_PASSWORD="root" \
    --network=host \
    --restart unless-stopped \
	zabbix/zabbix-proxy-mysql:5.0-centos-latest
``` 

启动前端页面，默认端口是 8080:
```p
docker run --name zbx5-web-nginx-mysql -d \
    -e ZBX_SERVER_HOST="127.0.0.1" \
    -e DB_SERVER_HOST="127.0.0.1" \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="zabbix" \
    -e MYSQL_ROOT_PASSWORD="root" \
    --network=host \
    --restart unless-stopped \
	zabbix/zabbix-web-nginx-mysql:5.0-centos-latest
```

启动 Agent:
```p
docker run --name zbx5-agent -d \
    -e ZBX_HOSTNAME="local-agent" \
    -e ZBX_SERVER_HOST="127.0.0.1"  \
    --network=host \
    --restart unless-stopped \
    zabbix/zabbix-agent:5.0-centos-latest
```

访问 http://ip:8080 即可访问！
默认用户名/密码是：Admin/zabbix

其他服务器安装agent：
```p
docker run --name zbx5-agent -d \
    -e ZBX_HOSTNAME="名称" \
    -e ZBX_SERVER_HOST="zabiix地址"  \
    --network=host \
    --restart unless-stopped \
    zabbix/zabbix-agent:5.0-centos-latest
```











