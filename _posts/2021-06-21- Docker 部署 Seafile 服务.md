---
title: Docker 部署 Seafile 服务
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

安装 docker-compose
因为 Seafile v7.x.x 容器是通过 docker-compose 命令运行的，所以您应该先在服务器上安装该命令。
```p
# for CentOS
yum install docker-compose -y

# for Ubuntu
apt-get install docker-compose -y
```

下载并修改 docker-compose.yml
```p
下载 docker-compose.yml 示例文件到您的服务器上，然后根据您的实际环境修改该文件。尤其是以下几项配置：
1.MySQL root 用户的密码 (MYSQL_ROOT_PASSWORD and DB_ROOT_PASSWD)
2.持久化存储 MySQL 数据的 volumes 目录 (volumes)
3.持久化存储 Seafile 数据的 volumes 目录 (volumes)
```

启动 Seafile 服务
```p
执行以下命令启动 Seafile 服务
docker-compose up -d
需要等待几分钟，等容器首次启动时的初始化操作完成后，您就可以在浏览器上访问http://seafile.example.com 来打开 Seafile 主页。
```
注意：您应该在 docker-compose.yml 文件所在的目下执行以上命令。

自定义管理员用户名和密码
```p
默认的管理员账号是 me@example.com 并且该账号的密码是 asecret，您可以在 docker-compose.yml 中配置不同的用户名和密码，为此您需要做如下配置：
seafile:
    ...
    environment:
        ...
        - SEAFILE_ADMIN_EMAIL=me@example.com
        - SEAFILE_ADMIN_PASSWORD=a_very_secret_password
        ...
```

使用 Let's encrypt SSL 证书
如果您把 SEAFILE_SERVER_LETSENCRYPT 设置为 true，该容器将会自动为您申请一个 letsencrypt 机构颁发的 SSL 证书，并开启 https 访问，为此您需要做如下配置：
seafile:
    ...
    ports:
        - "80:80"
        - "443:443"
    ...
    environment:
        ...
        - SEAFILE_SERVER_LETSENCRYPT=true
        - SEAFILE_SERVER_HOSTNAME=docs.seafile.com
        ...
```

如果您想要使用自己的 SSL 证书，而且如果用来持久化存储 Seafile 数据的目录为 /opt/seafile-data，您可以做如下处理：

```p
1.创建 /opt/seafile-data/ssl 目录，然后拷贝您的证书文件和密钥文件到ssl目录下。
2.假设您的站点名称是 seafile.example.com，那么您的证书名称必须就是 seafile.example.com.crt，密钥文件名称就必须是 seafile.example.com.key。
```

修改 Seafile 服务的配置
```p
Seafile 的配置文件存放在 shared/seafile/conf 目录下，您可以根据Seafile 手册的指导来修改这些配置项。
一旦修改了配置文件，您需要重启服务以使其生效：
docker-compose restart
```

查找日志
```p
Seafile 容器中 Seafile 服务本身的日志文件存放在 /shared/logs/seafile 目录下，或者您可以在宿主机上 Seafile 容器的卷目录中找到这些日志，例如：/opt/seafile-data/logs/seafile
同样 Seafile 容器的系统日志存放在 /shared/logs/var-log 目录下，或者宿主机目录 /opt/seafile-data/logs/var-log。
```

增加一个新的管理员
```p
docker exec -it seafile /opt/seafile/seafile-server-latest/reset-admin.sh
根据提示输入用户名和密码，您现在有了一个新的管理帐户。
```

我们假设您的 seafile 数据卷路径是 /opt/seafile-data，并且您想将备份数据存放到 /opt/seafile-backup 目录下。
```p
您可以创建一个类似以下 /opt/seafile-backup 的目录结构：
/opt/seafile-backup
---- databases/  用来存放 MySQL 容器的备份数据
---- data/  用来存放 Seafile 容器的备份数据
```

要备份的数据文件：
```
/opt/seafile-data/seafile/conf                   # configuration files
/opt/seafile-data/seafile/seafile-data        # data of seafile
/opt/seafile-data/seafile/seahub-data       # data of seahub
```

备份数据库：
```p
# 建议每次将数据库备份到一个单独的文件中。至少在一周内不要覆盖旧的数据库备份。
cd /opt/seafile-backup/databases
docker exec -it seafile-mysql mysqldump  -uroot --opt ccnet_db > ccnet_db.sql
docker exec -it seafile-mysql mysqldump  -uroot --opt seafile_db > seafile_db.sql
docker exec -it seafile-mysql mysqldump  -uroot --opt seahub_db > seahub_db.sql
```
备份 Seafile 资料库数据：
```p
1.直接复制整个数据目录
cp -R /opt/seafile-data/seafile /opt/seafile-backup/data/
cd /opt/seafile-backup/data && rm -rf ccnet

2.使用 rsync 执行增量备份
rsync -az /opt/seafile-data/seafile /opt/seafile-backup/data/
cd /opt/seafile-backup/data && rm -rf ccnet
```

恢复数据库：
```p
docker cp /opt/seafile-backup/databases/ccnet_db.sql seafile-mysql:/tmp/ccnet_db.sql
docker cp /opt/seafile-backup/databases/seafile_db.sql seafile-mysql:/tmp/seafile_db.sql
docker cp /opt/seafile-backup/databases/seahub_db.sql seafile-mysql:/tmp/seahub_db.sql

docker exec -it seafile-mysql /bin/sh -c "mysql -uroot ccnet_db < /tmp/ccnet_db.sql"
docker exec -it seafile-mysql /bin/sh -c "mysql -uroot seafile_db < /tmp/seafile_db.sql"
docker exec -it seafile-mysql /bin/sh -c "mysql -uroot seahub_db < /tmp/seahub_db.sql"
```

恢复 seafile 数据：
```p
cp -R /opt/seafile-backup/data/* /opt/seafile-data/seafile/
```

垃圾回收
```p
在 seafile 中，当文件被删除时，组成这些文件的块数据不会立即删除，因为可能有其他文件也会引用这些块数据(用于去重功能的实现)。为了真正删除无用的块数据，还需要额外运行"GC"程序。GC 会自动检测到哪些数据块不再被任何文件所引用，并清除它们。
GC 脚本被放在docker容器的 /scripts 目录下。执行 GC 的方法很简单：docker exec seafile /scripts/gc.sh。对于社区版来说，该程序会暂停 Seafile 服务，但这是一个相对较快的程序，一旦程序运行完成，Seafile 服务也会自动重新启动。而专业版提供了在线运行 GC 的功能，不会暂停 Seafile 服务。
```
