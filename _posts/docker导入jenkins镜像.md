---
title: docker导入jenkins镜像
tags:
  - 随笔
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 实验过程
1.下载、上传镜像
[jenkins镜像]（https://fhpt-rj.obs.cn-east-2.myhuaweicloud.com:443/%E5%B8%B8%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%AE%89%E8%A3%85%E5%8C%85/docker/jenkins.tar?AccessKeyId=0ZWMCUR84GQVKRHXUY6X&Expires=1616436765&x-obs-security-token=gQpjbi1ub3J0aC00i0YG1TO-r2LejrgLyGPIUUf8mVlv7sLIfG3BhwC8-PWUy4iHGNVKHFATbUATP07OtkRiL75ZlRHg25jxrwGmkO2GI9kVHsJQK3rkm6DgvmkHX-n-nlyOr0x96yfK3xtII_VKdfVCp4pA1J4QSVhJhvX3NWAAYn38VorDxSISH3aO-ibl2M694RnyI89aRBM5jirzc7LzneSNvFEji7e5_sJCuIfE0oMvHtxLX89X2NDfc76lix0UyKOYfkKYjKm2P6lXUBbV2X4zAJOtrUDOojo3U-U3MZXSHWihwtpnYCYcRpLFsl5vUADxQfMDPcIs-HcjRZojMGAs9RwlzYL70_BILamUETzzR5i8RZg1uevX4d7yjcMCb3WDMS_LVnSIJgv86Qe2ZmTzajRn4yumIax06oEhQhucsvajnvNQ_OccHu90hYxXRznupSzkdX9jhyrQu-Dx9Zy1AC41Z4QE_KBln8aqZT2bwhGfwRt6bzKrXGKGaDGxW1QLFze72ACy88QgbrghVN1aO8jPGLIgfAdCTAWuHWbwLz68CCtb61UsOoQJ2bOQLVHnKeDP9Mr4WviLMLqip4Qvhu7uGQTVQis5Iz_fMRcwTPpndkezeZo185CqKcuy0nBGMCw8CpAoNTMfFAanWyKErRizoZ7SWF_Q5i-5pwVp6OjORsOcgQXA20FONqn7KzJtJZNl85hwTJLG_LEo0WaNc9qUZRDd-uj13_vyqfZbGLgw4Hi6dcYVsEdrdNslbVToiFSFWqMBYvgEYagvy_FhM3O3H2TwKqG490XuaawD_xbFBDkBRvgd&Signature=lFA1b6YA5JrdwQ657cBK9UVD0l4%3D）

2.创建数据存放目录
```c
# mkdir -p /data/jenkins_home/
#chown -R 1000:1000 /data/jenkins_home/
```

3.docker load --input 包名称 （导入镜像）
```c
-docker images     （查看镜像
```

4.生成容器
```c
docker run -d --name jenkins -p 8040:8080 -p 50000:50000 -v /data/jenkins_home:/var/jenkins_home jenkins/jenkins:2.222.3-centos
docker ps    （查看运行容器）
```
5.配置jenkins
-输入 IP地址：8040 进行访问
-前提查看防火墙是否打开，要关闭防火墙

6.查看密码
```c
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPasswd
```

7.按照要求配置
-选择跳过插件安装

8.将开始备份的plugins上传 解压到目录路径下

9. 在地址栏后面加上/restart，点击yes。

注： 插件可以在外面有网的服务器按照同样的镜像包进行安装镜像，然后把插件目录压缩打包，根据自己的需求进行选择。


