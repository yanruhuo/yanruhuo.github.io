---
title: 内网动态域名配置
tags:
  - 其他配置
jekyll-theme-WuK:
  background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="//music.163.com/outchain/player?type=2&id=27876158&auto=0&height=66"></iframe>'
---

# OpenWrt配置阿里云动态域名服务DDNS
DDNS（Dynamic Domain Name Server，动态域名服务） 是将用户的动态IP地址映射到一个固定的域名解析服务上，用户每次连接网络的时候客户端程序就会通过信息传递把该主机的动态IP地址传送给位于服务商主机上的服务器程序，服务器程序负责提供DNS服务并实现动态域名解析。

# 创建Access Key
1.登录阿里云，找到AccessKey管理；

2.你可以使用管理账号的AccessKey，但为安全起见，本案例使用子用户AccessKey。

3.创建子用户，填入登录名称和显示名称，这里设置不作限制。

# 添加权限
1.选择AliyunDNSFullAccess，并添加。

2.创建AccessKey。注意：子用户的AccessKey只显示一次，请将AccessKey和Secret记下来，一会用到。

# 域名创建A记录
1.找到域名解释设置，为你的域名设置一个A记录。

#记住二级域名需在路由器上进行配置

# 设置OpenWrt DDNS
1.登录OpenWrt管理后台——服务——动态DNS，如下图点击修改。

2.填入你的查询的域名、选择aliyun.com作为DDNS服务提供商、域名、Access Key和Secret。

3.选择接口和WAN口作为更新源，用于获取和更新你本地的公网地址。如果你无法通过接口获得公网地址，也可以使用URL方式，二选一。

4.确认无误后保存并应用。

5.再进入日志查看器，读取日志文件。若提供Update successful说有配置正确。

# 验证
1.在域名解析中ip输入和实际不一样的ip

2.设置OpenWrt DDNS刷新和重新配置下

3.查看域名解析有没有变化
