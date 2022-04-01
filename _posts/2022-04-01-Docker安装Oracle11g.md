---
title: Docker安装Oracle11g
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

# 拉取Oracle镜像
```p
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

# 创建镜像
```p
docker run -d -p 1521:1521 --name oracle registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

# 进入Oracle容器创建用户
容器镜像系统用户root密码为 helowin
```p
#进入容器
docker exec -it oracle /bin/bash

#进入到 oracle 用户目录
cd /home/oracle/

#加载 oracle 环境变量
source .bash_profile 

#登录oracle数据库
sqlplus /nolog

#切换管理用户
conn /as sysdba

#修改system用户账号密码为system
alter user system identified by system; 

#修改sys用户账号密码为system
alter user sys identified by system; 

#创建内部管理员账号mike 密码为mikeops
create user mike identified by mikeops;

#将dba权限授权给内部管理员账号
grant connect,resource,dba to mike; 

#修改密码规则策略为密码永不过期
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED; 

#修改数据库最大连接数据
alter system set processes=2000 scope=spfile; 

#查看mike用户创建信息
select * from dba_users t where t.username = 'MIKE'; 
```p

```p
[oracle@1cab53ad073d ~]$ sqlplus /nolog

SQL*Plus: Release 11.2.0.1.0 Production on Wed Nov 18 11:10:07 2020

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

SQL> conn /as sysdba
Connected.
SQL> alter user system identified by system;

User altered.

SQL> alter user sys identified by system;

User altered.

SQL> create user mike identified by mikeops;

User created.

SQL> grant connect,resource,dba to mike;

Grant succeeded.

SQL> ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;

Profile altered.

SQL> alter system set processes=2000 scope=spfile;

System altered.

SQL> select * from dba_users t where t.username = 'MIKE';
```
