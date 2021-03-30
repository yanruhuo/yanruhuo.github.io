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

<div align=center>
![图片1.png](https://i.loli.net/2021/03/30/Bq4K7AjEyQZWVXM.png)
![图片2.png](https://i.loli.net/2021/03/30/lDch5iM7U26ItFC.png)
</div>

4.生成容器
```c
docker run -d --name jenkins -p 8040:8080 -p 50000:50000 -v /data/jenkins_home:/var/jenkins_home jenkins/jenkins:2.222.3-centos
docker ps    （查看运行容器）
```
![图片3.png](https://i.loli.net/2021/03/30/k3c82fniZQpSd1o.png)

5.配置jenkins
- 输入 IP地址：8040 进行访问
- 前提查看防火墙是否打开，要关闭防火墙
![图片4.png](https://i.loli.net/2021/03/30/TpdR47H8PZ253cl.png)

6.查看密码
```c
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPasswd
```
![图片5.png](https://i.loli.net/2021/03/30/7IvDcxzm9J3Qf2b.png)

7.按照要求配置
> 选择跳过插件安装
![图片6.png](https://i.loli.net/2021/03/30/CNyX5GcZFWAYQ6s.png)

8.将开始备份的plugins上传 解压到目录路径下

9.在地址栏后面加上/restart，点击yes。
![图片7.png](https://i.loli.net/2021/03/30/SygaX53pHtGzsdO.png)
![图片8.png](https://i.loli.net/2021/03/30/hRDFpMy2wdTcL1B.png)

重启后可以看到插件
![图片9.png](https://i.loli.net/2021/03/30/MvWh4F9JUn3iDsY.png)
