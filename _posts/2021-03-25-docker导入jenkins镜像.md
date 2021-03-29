---
title: docker导入jenkins镜像
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 实验过程
1.下载、上传镜像
> [jenkins镜像](https://share.weiyun.com/v3Vb0cGD)

2.创建数据存放目录
```c
# mkdir -p /data/jenkins_home/
#chown -R 1000:1000 /data/jenkins_home/
```

3.docker load --input 包名称 （导入镜像）
```c
-docker images     （查看镜像)
```
（https://github.com/yanruhuo/yanruhuo.github.io/blob/main/image/docker%E5%AF%BC%E5%85%A5jenkins/%E5%9B%BE%E7%89%871.png?raw=true）

4.生成容器
```c
docker run -d --name jenkins -p 8040:8080 -p 50000:50000 -v /data/jenkins_home:/var/jenkins_home jenkins/jenkins:2.222.3-centos
docker ps    （查看运行容器）
```

5.配置jenkins
- 输入 IP地址：8040 进行访问
- 前提查看防火墙是否打开，要关闭防火墙

6.查看密码
```c
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPasswd
```

7.按照要求配置
> 选择跳过插件安装

8.将开始备份
