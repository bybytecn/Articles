---
layout: post
title: 分享我的科学上网的姿势和问题
date: 2022-03-26 18:50:00
tags: [科学上网]
---
# 前言
🚀科学上网是程序员必备技能的，发展了这么多年，各种姿势也层出不穷，我在这里分享下我的解决方案和存在的痛点，以下两个方案都是我尝试过的而有在用着的。

# 方案一：浏览器单独设置使用
用到的工具如下：
1. 能科学上网的Http/Socket代理端口
2. Chrome/FireFox

## 能科学上网的Http/Socket端口
这个一般是指你用的小猫咪、酸酸乳、或者别的，在启动后，都会在本地监听一个Http/Socket协议的代理端口（一般是1080），通过这个端口你就可以科学上网

## Chrome/Firefox + 插件
在有了能科学上网的Http/Socks端口后，在Chrome里装个插件，或者在Firefox里设置（Firefox默认就有了不需要装插件），然后要用的时候单独打开该浏览器即可

![avatar](github.png)

## 优点
- 方便

直接打开浏览器使用即可，如果在国内架设一台服务器跑小猫咪、酸酸乳，本地都不用开了，甚至手机，别人的电脑都不用下载软件，直接设置系统代理就能用。

## 缺点
- 范围短

只有设置到的浏览器能用，当然一些支持HTTP_PROXY环境变量的工具也行，但是还是局限性小，没法做到所有应用层的流量都走代理

# 方案二：旁路由作透明代理
用到的工具如下：
1. Linux+iptables+chnroute+redsocks2
2. 无污染的DNS服务器
3. 能科学上网的Http/Socket代理端口

第二点不在本人范围内，第三点介绍过了，就看第一点了

## Linux+iptables+chnroute+redsocks2
使用iptables的REDIRECT把流量转发给redsocks2，redsocks2再转发给socks5服务端，同时再配合ipset，把chnroute以外的IP走代理。这样就在路由层做到分流了
```shell
root@ubuntu:~/chinadns-ng# iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
DNS        all  --  anywhere             anywhere            
PROXY      all  --  anywhere             anywhere             ! match-set chnroute dst

```

```shell
Chain PROXY (1 references)
target     prot opt source               destination         
REDIRECT   tcp  --  anywhere             anywhere             redir ports 8088

```

```shell
Chain DNS (1 references)
target     prot opt source               destination         
DNAT       udp  --  anywhere             anywhere             udp dpt:domain to:192.168.1.7:54

```

## 优点
- 范围大

因为是在路由层作用的，所以TCP协议都可以走代理

- 分流

用ipset配合iptables分流，这样不影响国内流量速度了

## 缺点
- 需要一台设备做旁路由

需要一台设备跟主机在同一内网，不过也可以在主机上用虚拟机

- 某些国内网站的服务器在香港

某讯视频、B某站、某读。等服务器都部署在香港。。导致流量跑出去了，严重影响速度。。


# 未来方案
以后我想的是在主机上一行命令就开启、关闭全局科学上网，或者一个快捷键，极速切换。


# 总结
方案一是我用得最久的，也一直再用、因为简单稳定，且方便。方案二目前还不完美，主要是没想到那么多服务商都部署在香港了。。