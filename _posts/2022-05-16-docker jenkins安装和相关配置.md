---
title: docker jenkins安装和相关配置
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---


一、安装jenkins
```p
 docker run -d -p 80:8080 -p 50000:50000 -v jenkins:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkinsfrank  docker.io/jenkins/jenkins
```


二、添加插件
```p
NodeJS   			#java打包工具
Publish Over SSH		#添加服务器和路径插件
Git Parameter 			#构建选择分支
```

三、项目配置
```p
NodeJS配置路径：
	系统管理-全局工具配置-NodeJS-NodeJS安装-新增NodeJS

maven配置路径：
	系统管理-全局工具配置-Maven-Maven安装-新增Maven

#git是默认勾选自动安装，上面都选择自动安装即可。
```

四、centos配置
```p
jenkins容器内部操作
docker ps 			#查看安装jenks容器

docker exec -ti 容器ID bash	#进入容器

ssh-keygen			#生成密钥

ssh-copy-id 部署服务器IP		#公钥文件复制到部署服务器IP


部署服务器操作：
adduser jenkins			#创建用户

passwd jenkins			#设置用户密码

su jenkins			#进入jenkins账户

cd /home/jenkins		#进入用户目录

mkdir 文件路径			#创建文件路径
```

五、配置jenkins shell脚本
```p
#!/bin/bash
serve_ip="121.5.182.42"
serve_user="root"
work_dir="/home/chuangyuan/workspace/web/dev-contest.woweds.com"
serve=$serve_user@$serve_ip

echo $PATH

node -v
npm -v

npm install
ret=$?
if [[ ${ret} -ne 0 ]] ; then
	echo "[error] 打包错误"
	exit 1
fi

npm run build
ret=$?
if [[ ${ret} -ne 0 ]] ; then
	echo "[error] 打包错误"
	exit 1
fi
echo "编译完成"

time=$(date "+%Y-%m-%d-%H.%M.%S")

tar_name=dist-${time}.tar.gz

echo "开始打包完成"
tar -zcvf ${tar_name} dist/
echo "打包完成"

echo "开始上传压缩包"
scp ${tar_name} ${serve}:/${work_dir}/
echo "压缩包上传完成"

echo "删除压缩包"
rm ${tar_name}
echo "删除压缩包完成"

echo "远程执行解压"
## 上传后执行启动脚本
ssh $serve  << bash
source /etc/profile
cd ${work_dir}
pwd
echo "删除原文件夹"
rm -rf dist
echo "解压"
tar zxvf ${tar_name}
bash
```