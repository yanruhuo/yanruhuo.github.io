---
title: Elasticsearch 配置用户名和密码
tags:
  - 软件安装配置
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

启动 Elasticsearch 程序
```p
[elastic@console bin]$ ./elasticsearch -d
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_181/jre] does not meet this requirement
```
创建密码
```p
[elastic@console bin]$ ./elasticsearch-setup-passwords interactive
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_181/jre] does not meet this requirement

Unexpected response code [500] from calling GET http://192.168.108.126:9200/_security/_authenticate?pretty
It doesn't look like the X-Pack security feature is enabled on this Elasticsearch node.
Please check if you have enabled X-Pack security in your elasticsearch.yml configuration file.

ERROR: X-Pack Security is disabled by configuration.
```

需要设置 X-Pack
```p
[elastic@console bin]$ vim ../config/elasticsearch.yml
	http.cors.enabled: true
	http.cors.allow-origin: "*"
	http.cors.allow-headers: Authorization
	xpack.security.enabled: true
	xpack.security.transport.ssl.enabled: true
```

添加密码
```p
[elastic@console bin]$ ./elasticsearch-setup-passwords interactive
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_181/jre] does not meet this requirement
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]:
Reenter password for [elastic]:
Passwords do not match.
Try again.
Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [apm_system]:
Reenter password for [apm_system]:
Enter password for [kibana]:
Reenter password for [kibana]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Enter password for [beats_system]:
Reenter password for [beats_system]:
Enter password for [remote_monitoring_user]:
Reenter password for [remote_monitoring_user]:
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

修改kibana
```p
[root@console bin]# vim ../config/kibana.yml
	elasticsearch.username: "elastic"
	elasticsearch.password: "passwd"
```

修改密码
```p
POST /_security/user/elastic/_password
{
  "password": "123456"
}
修改密码之后，需要重新设置kibana的配置文件，才可以重新使用kibana
```