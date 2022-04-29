---
title: Docker安装集群rabbitMQ
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 环境准备
### Centos 7.5虚拟机三台：
```p
192.168.0.128
192.168.0.130
192.168.0.131
```

### 三台机器分别配置如下所示的hosts文件，以供rabbitmq容器使用
```p
$ vim /home/rabbitmq/hosts 
# 文件中写入以下内容
192.168.102.128 rabbit1 rabbit1
192.168.102.130 rabbit2 rabbit2
192.168.102.131 rabbit3 rabbit3
```

## 搭建过程
### 在三台机器上，分别management版本的rabbitmq镜像
```p
docker pull rabbitmq:management
```

### 在三台机器上分别创建rabbitmq容器
在192.168.102.128上创建容器rabbit1
```p
docker run --restart=unless-stopped -h rabbit1 -d -p 5672:5672 -p 15672:15672 -p 25672:25672 -p 4369:4369 \
--name myrabbit1 \
-v /home/rabbitmq:/var/lib/rabbitmq:z \
-v /home/rabbitmq/hosts:/etc/hosts \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=123456 \
-e RABBITMQ_ERLANG_COOKIE='xxx_2019' \
rabbitmq:management
```

在192.168.102.130上创建容器rabbit2
```p
docker run --restart=unless-stopped -h rabbit2 -d -p 5672:5672 -p 15672:15672 -p 25672:25672 -p 4369:4369 \
--name myrabbit2 \
-v /home/rabbitmq:/var/lib/rabbitmq:z \
-v /home/rabbitmq/hosts:/etc/hosts \
-e RABBITMQ_ERLANG_COOKIE='xxx_2019' \
rabbitmq:management
```

在192.168.102.131上创建容器rabbit3
```p
docker run --restart=unless-stopped -h rabbit3 -d -p 5672:5672 -p 15672:15672 -p 25672:25672 -p 4369:4369 \
--name myrabbit3 \
-v /home/rabbitmq:/var/lib/rabbitmq:z \
-v /home/rabbitmq/hosts:/etc/hosts \
-e RABBITMQ_ERLANG_COOKIE='xxx_2019' \
rabbitmq:management
```

-d 表示容器后台运行
-h rabbit1 容器的主机名是rabbit1，容器内部的hostname
-v /home/rabbitmq:/var/lib/rabbitmq:z 将宿主机目录/home/rabbitmq挂载到容器的/var/lib/rabbitmq目录。z是一个标记，在selinux环境下使用
-e RABBITMQERLANGCOOKIE='rabbit_cluster' 设置rabbitmq的cookie，该值可以任意设置，只需要三个容器保持一致即可

### 绑定集群
重置myrabbit1节点
```p
docker exec -it myrabbit1 /bin/bash
rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl start_app
```

加入myrabbit2节点到集群中
```p
docker exec -it myrabbit2 /bin/bash
rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster rabbit@rabbit1 && rabbitmqctl start_app
```

加入myrabbit3节点到集群中
```p
docker exec -it myrabbit3 /bin/bash
rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster rabbit@rabbit1 && rabbitmqctl start_app
```

## 查询集群状态
```p
rabbitmqctl cluster_status
```

## 故障节点的处理
```p
docker exec -it rabbit2 /bin/bash
rabbitmqctl stop_app
```

## 在一个正常的节点上移除一个异常的节点
```p
docker exec -it rabbit1 /bin/bash
rabbitmqctl forget_cluster_node rabbit@rabbit2
```

## 访问地址
```p
192.168.0.128+15672
账号密码，在主节点启动容器时填写的
```