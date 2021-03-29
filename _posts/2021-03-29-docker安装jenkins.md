---
title: Docker安装Jenkins
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

##Docker安装Jenkins

1.安装环境
CentOS 7.6
Docker 18.09.6

2.拉取镜像
我这里安装的版本是2.222.3-centos，可以去这里获取你需要的版本: https://hub.docker.com/_/jenkins?tab=tags
```c
docker pull jenkins/jenkins:2.222.3-centos
```

3.创建本地数据卷
我这里映射本地数据卷的路径为/data/jenkins_home/，你想放别的地方可以自行修改
```c
mkdir -p /data/jenkins_home/
```

需要修改下目录权限，因为当映射本地数据卷时，/data/jenkins_home/目录的拥有者为root用户，而容器中jenkins用户的 uid 为 1000。
```c
chown -R 1000:1000 /data/jenkins_home/
```

4.创建容器
```c
docker run -d --name jenkins -p 8030:8080 -p 50000:50000 -v /data/jenkins_home:/var/jenkins_home jenkins/jenkins:2.222.3-centos
```

- -d 标识是让 docker 容器在后台运行
- --name 定义一个容器的名字，如果没有指定，那么会自动生成一个随机数字符串当做UUID
- -p 8040:8080 端口映射，我本地的8080被占用了，所以随便映射了一个8040
- -p 50000:50000 端口映射
- -v /data/jenkins_home:/var/jenkins_home 绑定一个数据卷，/data/jenkins_home是刚才创建的本地数据卷

5.打开 Jenkins
通过浏览器访问 http://IP:8030/（注意替换成你自己的IP和端口）进入初始页，如果 Jenkins 还没有启动完成，会显示如下内容

6.输入管理员密码
这里要求输入初始的管理员密码，根据提示密码在/var/jenkins_home/secrets/initialAdminPassword这个文件中，注意这个路径是 Docker 容器中的，所以我们通过如下命令获取一下
```c
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
85770376692448b7b6a8e301fb437848
```

别忘了我们映射了本地数据卷/data/jenkins_home/，所以也可以通过如下命令输出
```c
cat /data/jenkins_home/secrets/initialAdminPassword 
85770376692448b7b6a8e301fb437848
```
