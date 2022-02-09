---
title: docker安装gitlab
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

拉取git镜像
```p
docker pull gitlab/gitlab-ce:latest
```

创建docker容器 
```p
docker run \
    --p 443:443 --p 80:80 --p 3322:22 \                     #开放端口
    --name gitlab \											#名称
    --volume /usr/local/gitlab/config:/etc/gitlab \			# 映射github的配置目录
    --volume /usr/local/gitlab/logs:/var/log/gitlab \		#映射github的日志目录
    --volume /usr/local/gitlab/data:/var/opt/gitlab \		#映射github的 data目录
    --privileged=true \										#在运行容器的时候，给容器加入权限参数 --privileged=true，以特权方式启动容器 
    gitlab/gitlab-ce
```

配置gitlab服务器的访问地址
```p
vim /usr/local/gitlab/config/gitlab.rb

#添加地址
external_url 'http://192.168.1.21'
```

重启镜像
```p
docker start ID     				 	#Docker容器做端口映射报错 重启即可systemctl restart docker
```

查看密码
```p
cat /etc/gitlab/initial_root_password 	#这个文件将在首次执行reconfigure后24小时自动删除
```

登录
```p
等待 docker 容器启动完成后，访问 http://192.168.1.21 就进入 gitlab 访问界面
```
