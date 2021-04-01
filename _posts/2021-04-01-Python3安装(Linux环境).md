---
title: Python3安装(Linux环境)
tags:
  - python
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

#maven私有仓库搭建

**Python3安装(Linux环境)**

在安装python时或者在用到python的时候，会依赖一些环境。为了防止在安装时出现问题，请确保机器上有一下包。

**安装路径**
```c
yum -y install zlib zlib-devel
yum -y install bzip2 bzip2-devel
yum -y install ncurses ncurses-devel
yum -y install readline readline-devel
yum -y install openssl openssl-devel
yum -y install openssl-static
yum -y install xz lzma xz-devel
yum -y install sqlite sqlite-devel
yum -y install gdbm gdbm-devel
yum -y install tk tk-devel
yum install gcc
```

**下载Python源码**
```c
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
```

如果想下载其他版本：https://www.python.org/downloads/

**解压源码**
```c
tar zxf Python-3.6.5.tgz
```

**指定安装路径**

在Python-3.6.5目录下 执行

```c
./configure --with-ssl --prefix=/service/python3
```

**执行make命令**
```c
make
```

**执行make install**
在Python-3.6.5目录下 执行
```c
make install
```

**修改Python软连接**

Linux默认的python是2.X版本，现在我们需要把默认的软连接改成新安装的版本。
- 备份原有软连接，注意这里需要root权限。

```c
sudo mv /usr/bin/python /usr/bin/python2.back
```

- 创建新的软连接

```c
sudo ln -s /service/python3/bin/python3 /usr/bin/python
```

- 查看python版本

```c
python --version
```
查看到对应的版本信息就成功了















