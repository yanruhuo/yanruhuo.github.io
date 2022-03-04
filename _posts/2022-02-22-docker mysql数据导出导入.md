---
title: docker mysql数据导出导入
tags:
  - docker
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

docker 备份数据库
```p
docker exec -ti c8071b30375f mysqldump -uroot -p123456 --all-databases > 130.40-`date +%Y-%m-%d-%H-%M-%S`.sql
```

将备份传输到需要导入数据的docker容器
```p
docker cp 130.40-2022-02-25-12-53-20.sql bd2c14c566e4:/
```

导入数据
```p
docker exec -ti bd2c14c566e4 sh -c "mysql -uroot -p123456 < 130.40-2022-02-25-12-53-20.sql"
```








