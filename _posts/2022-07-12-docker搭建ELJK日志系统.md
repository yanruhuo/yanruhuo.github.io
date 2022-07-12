---
title: docker ELK日志系统
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 一、安装docker
一键安装最新docker
```p
curl -sSL https://get.docker.com/ | sh 
```

## 二、安装elasticsearch镜像
```p
#搜素ES镜像
docker search elasticsearch
#下载ES镜像包
docker pull elasticsearch:7.7.1
```
等待下载完成，查看elasticsearch镜像是否已加载。
```p
#查看下载镜像
[root@elasticsearch ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
logstash            7.7.1               7f059e3dee67        20 months ago       788MB
kibana              7.7.1               6de54f813b39        20 months ago       1.2GB
elasticsearch       7.7.1               830a894845e3        20 months ago       804MB
```

创建挂载目录
```p
mkdir -p /data/elk/es/{config,data,logs}
```

赋予权限
```p
#docker中elasticsearch的用户UID是1000.
chown -R 1000:1000 /data/elk/es
```

创建挂载用配置
```p
cd /data/elk/es/config
touch elasticsearch.yml
vim elasticsearch.yml
-----------------------配置内容----------------------------------
cluster.name: "my-es"
network.host: 0.0.0.0
http.port: 9200
```

运行elasticsearch
```p
通过镜像，启动一个容器，并将9200和9300端口映射到本机（elasticsearch的默认端口是9200，我们把宿主环境9200端口映射到Docker容器中的9200端口）。此处建议给容器设置固定ip，我这里没设置。
docker run -it  -d -p 9200:9200 -p 9300:9300 --name es -e ES_JAVA_OPTS="-Xms1g -Xmx1g" -e "discovery.type=single-node" --restart=always -v /data/elk/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /data/elk/es/data:/usr/share/elasticsearch/data -v /data/elk/es/logs:/usr/share/elasticsearch/logs elasticsearch:7.7.1
```

验证安装是否成功
```p
[root@elasticsearch home]# curl http://localhost:9200
{
  "name" : "0adf1765ac08",
  "cluster_name" : "my-es",
  "cluster_uuid" : "MpKqrEKySnSdwux0m7AlEA",
  "version" : {
    "number" : "7.7.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ad56dce891c901a492bb1ee393f12dfff473a423",
    "build_date" : "2020-05-28T16:30:01.040088Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## 三、部署kibana
安装kibana
```p
docker pull kibana:7.7.1
```

获取elasticsearch容器ip
```p
[root@elasticsearch home]# docker inspect --format '{{ .NetworkSettings.IPAddress }}' es
172.17.0.2
```

配置文件
在服务器上新建配置文件，用于docker文件映射。所使用目录需对应新增。
vi /data/elk/kibana/kibana.yml
```p
#Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
elasticsearch.hosts: ["http://172.17.0.2:9200"]
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

运行kibana
```p
docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2 --name kibana -p 5601:5601 -v /data/elk/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.7.1
```

访问
```p
浏览器上输入：http://ip:5601，如无法访问进容器检查配置是否生效
```

## 四、部署logstash
获取logstash镜像
```p
docker pull logstash:7.7.1
```

编辑logstash.yml配置文件。所使用目录需对应新增
新建挂载目录
```p
mkdir -p /data/elk/logstash/conf.d/
```

vim /data/elk/logstash/logstash.yml
```p
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://172.17.0.2:9200" ]
xpack.monitoring.elasticsearch.username: elastic
xpack.monitoring.elasticsearch.password: changeme
#path.config: /data/elk/logstash/conf.d/*.conf
path.config: /data/docker/logstash/conf.d/*.conf
path.logs: /var/log/logstash
```

编辑logstash.conf文件，此处先配置logstash直接采集本地数据发送至es
vim /data/elk/logstash/conf.d/syslog.conf 
```p
input {
  syslog {
    type => "system-syslog"
    port => 5044
  }
}
output {
  elasticsearch {
    hosts => ["192.168.1.214:9200"]      # 定义es服务器的ip
    index => "system-syslog-%{+YYYY.MM}" # 定义索引
  }
}
```

编辑本地rsyslog配置增加：
```p
vi /etc/rsyslog.conf       #增加一行
*.* @@192.168.200.94:5044
```

配置修改后重启服务
```p
systemctl restart rsyslog
```

运行logstash
```p
docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2 -p 5044:5044 --name logstash -v /data/elk/logstash/logstash.yml:/usr/share/logstash/config/logstash.yml -v /data/elk/logstash/conf.d/:/data/docker/logstash/conf.d/ logstash:7.7.1
```

测试es接收logstash数据
```p
[root@elk logstash]# curl http://localhost:9200/_cat/indices?v
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .apm-custom-link         WBgbpphkQCS73sfjjIG0-Q   1   0          0            0       208b           208b
green  open   .kibana_task_manager_1   xmBASGi9QheR-r8hG2XLZA   1   0          5            0       28kb           28kb
green  open   .apm-agent-configuration MsvsgveHSCOhBQRCgTnsRg   1   0          0            0       208b           208b
yellow open   system-syslog-2022.02    1Vcjw7Q-TTqVscpknyK7HA   1   1          6            0     20.7kb         20.7kb
green  open   .kibana_1                vJ-B5wakRSmOrwM6ri-xgw   1   0         84            2      115kb          115kb
#获取到system-syslog-相关日志，则es已能获取来自logstash的数据，kibana中也同步显示数据。
```

## 五、部署filebeat
在需要监测的机器yum安装filebeat
```p
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.7.1-x86_64.rpm
rpm -ivh filebeat-7.7.1-x86_64.rpm
```

filebeat配置，此处先配置filebeat直接发送数据到es
vim /etc/filebeat/filebeat.yml
```p
#=========================== Filebeat inputs =============================
filebeat.inputs:
- type: log
  enabled: true.            #此处需要修改
  paths:
    #- /var/log/*.log
    - /var/log/messages     #此处需要修改，修改日志路径。
    #- c:\programdata\elasticsearch\logs\*
#-------------------------- Elasticsearch output ------------------------------
output.elasticsearch:                    #此处需要去掉#
  # Array of hosts to connect to.
  hosts: ["192.168.200.94:9200"]         #此处需要去掉#   
#----------------------------- Logstash output --------------------------------
#output.logstash:
  # The Logstash hosts
  #hosts: ["192.168.200.94:5044"]
```

启动服务
```p
systemctl restart filebeat.service
```

es接收数据查询
```p
[root@elk conf.d]# curl http://localhost:9200/_cat/indices?v
health status index                            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   filebeat-7.7.1-2022.02.04-000001 vHSJTNWARySiC7IOOSABtw   1   1      11528            0      2.9mb          2.9mb
green  open   .apm-custom-link                 WBgbpphkQCS73sfjjIG0-Q   1   0          0            0       208b           208b
green  open   .kibana_task_manager_1           xmBASGi9QheR-r8hG2XLZA   1   0          5            0     64.1kb         64.1kb
green  open   .apm-agent-configuration         MsvsgveHSCOhBQRCgTnsRg   1   0          0            0       208b           208b
green  open   .kibana_1                        vJ-B5wakRSmOrwM6ri-xgw   1   0        158            3    101.5kb        101.5kb
#可查到filebeat-7.7.1-*数据，kibana中也显示对应数据。
```

filebeat采集数据，logstash过滤，在kibana中显示
删除之前的logstash生成的测试数据
```p
curl -XDELETE http://localhost:9200/system-syslog-2022.02
```

修改filebeat.yml，后重启服务
vim /etc/filebeat/filebeat.yml
```p
#-------------------------- Elasticsearch output ------------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["192.168.200.94:9200"]
#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: [“192.168.1.214:5044"]
```

将之前的syslog.conf重命名为syslog.conf.bak，增加logstash配置 ，其中可增加过滤相关配置，此处未配置。
```p
cd /data/elk/logstash/conf.d/
mv syslog.conf syslog.conf.bak
```

vi /data/elk/logstash/conf.d/logstash.conf
```p
input {
  beats {
    port => 5044
  }
}
output {
  elasticsearch {
    hosts => ["172.17.0.2:9200"]
    index => "filebeat_g-%{+YYYY.MM.dd}"
  }
}
```

查看es是否获取数据
```p
[root@elk conf.d]# curl http://localhost:9200/_cat/indices?v
health status index                            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .apm-custom-link                 WBgbpphkQCS73sfjjIG0-Q   1   0          0            0       208b           208b
yellow open   filebeat-2022.02.04              sVdMntvdQb6YIdjpeSFNrg   1   1         43            0    130.9kb        130.9kb
yellow open   filebeat-7.7.1-2022.02.04-000001 vHSJTNWARySiC7IOOSABtw   1   1      11623            0        3mb            3mb
green  open   .kibana_task_manager_1           xmBASGi9QheR-r8hG2XLZA   1   0          5            0     64.1kb         64.1kb
green  open   .apm-agent-configuration         MsvsgveHSCOhBQRCgTnsRg   1   0          0            0       208b           208b
yellow open   filebeat_g-2022.02.04           njA2A64zS16W3YMpPMzHQA   1   1          5            0    103.4kb        103.4kb
green  open   .kibana_1                        vJ-B5wakRSmOrwM6ri-xgw   1   0        163            3     81.5kb         81.5kb
#filebeat_g-*数据已经获取，kibana中增加相关索引即可。
```

  



