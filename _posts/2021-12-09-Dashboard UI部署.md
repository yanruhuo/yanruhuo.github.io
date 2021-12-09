---
title: Dashboard UI部署
tags:
  - k8s
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

# 部署 Dashboard UI
默认情况下不会部署 Dashboard。可以通过以下命令部署：
```p
#https://share.weiyun.com/ChJ8Rwul 下载上传
bash image.sh        

#https://share.weiyun.com/xHTLq6JZ  下载上传
kubectl apply -f ./recommended.yaml

#对外暴露Dashboard
kubectl -n kubernetes-dashboard edit svc kubernetes-dashboard 
#type: ClusterIPs 改为 type: NodePort

#查看svc
kubectl -n kubernetes-dashboard get svc

#看到端口3xxxx
#https://10.95.130.84:3xxxx
```

# 配置一下证书
```p
#删除默认创建的secret
kubectl delete secret kubernetes-dashboard-certs  -n kubernetes-dashboard

#重新创建secret，主要用来指定证书的存放路径
kubectl create secret generic kubernetes-dashboard-certs --from-file=/etc/kubernetes/pki/ -n kubernetes-dashboard

#删除dashboard的pod，主要让它重新运行，加载证书
kubectl delete pod -n kubernetes-dashboard --all
```

# 登录
```p
#创建服务账户，https://share.weiyun.com/i4JG9kBo
kubectl apply -f ./dashboard-adminuser.yaml

#创建一个ClusterRoleBinding，https://share.weiyun.com/zbUyIhc8
kubectl apply -f ./dashboard-ClusterRoleBinding.yaml

#获取token
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

把token粘贴登录，开始愉快的访问吧！
```













