---
title: Ubuntu添加用户未指定shell，ll别名等无法使用
tags:
  - Ubuntu
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

## 实验过程
1.添加用户并指定shell：

```p
useradd -r -m -s  /bin/bash 用户    #用户为新增用户
passwd 用户  #修改用户密码
```

2.没有产生用户目录
```p
useradd 用户     #添加用户
passwd  用户    #修改用户密码
mkdir /home/用户  #创建用户目录
chown 用户:用户  /home/用户   #修改用户所属者
```

用户没有添加shell， echo $SHELL查看下
```p
usermod -m /bin/bash  用户
```

3.如果没有/home/用户/.bashrc文件，可以拷贝其他用户目录下的.bashrc文件, 并修改所属者。
```p
source ~/.bashrc   #使.bashrc文件立即生效
```

4.如果每次登录命令ll不可以使用，则配置需要添加.profile文件用来配置用户环境，可以从其他用户拷贝。内容如下：
```p

# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.
 
 
# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
#umask 022
 
 
# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
 
 
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"

```