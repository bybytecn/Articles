---
layout: post
title: 斐讯N1做二级路由
date: 2022-04-21 23:32:00
tags: [OpenWRT,Phicomm]
---
# 前言
家里有个电信天猫，我的笔记本目前是直接连接天猫的，但是我现在手头有个斐讯N1盒子，我想把天猫做斐讯N1的上级网关，我电脑连接斐讯的WIFI，然后电脑的数据先经过斐讯路由，斐讯再交给天猫路由，这样就可以对数据透明地进行一些骚操作了。

# N1盒子
我的版本是**openwrt_s905d_n1_R21.11.11_k5.4.157-flippy-66+o**,其实如果你是armbian也可以，需要自己命令行设置，openwrt有界面操作就方便点。

# 步骤
## Step1: 设置LAN接口
一般刷完OpenWRT后会有一个br-lan接口，这个是N1的以太网接口和WIFI接口桥接起来的，我们不需要桥接。

先设置接口的IP地址、网关、DNS。

![](./1.png)

然后设置接口，我们这里选以太网接口，就是eth0

![](./2.png)

## Step2: 设置WLAN接口
这个是需要自己新建的，点**添加新的接口**即可，我的名称填的是WLAN，具体设置跟上面一样，这里给出我设置的参数。

![](./3.png)

![](./4.png)

## Step3: 添加SNAT
上面两个步骤完成后，ssh进去。
```shell
root@OpenWrt:~# ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:06:A0:66:C9
          inet addr:172.31.0.1  Bcast:172.31.0.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

eth0      Link encap:Ethernet  HWaddr FC:7C:02:6D:57:7A
          inet addr:192.168.1.11  Bcast:255.255.255.0  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:394476 errors:0 dropped:341 overruns:0 frame:0
          TX packets:212558 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:439754433 (419.3 MiB)  TX bytes:147424817 (140.5 MiB)
          Interrupt:20

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:2137 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2137 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:238443 (232.8 KiB)  TX bytes:238443 (232.8 KiB)

wlan0     Link encap:Ethernet  HWaddr FC:7C:02:6D:57:79
          inet addr:192.168.0.1  Bcast:255.255.255.0  Mask:255.255.255.0
          inet6 addr: fe80::fe7c:2ff:fe6d:5779/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:222239 errors:0 dropped:0 overruns:0 frame:0
          TX packets:400683 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:144885946 (138.1 MiB)  TX bytes:450312482 (429.4 MiB)
```

然后是路由表

```shell
root@OpenWrt:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    0      0        0 eth0
172.31.0.0      0.0.0.0         255.255.255.0   U     0      0        0 docker0
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 wlan0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```
到这里当我们用手机连上N1的WIFI后，能分配到192.168.0.0/24的IP地址，但是还是上不了网的，因为这个地址的数据直接转发给天猫的话是不处理的，因为天猫的网络号是192.168.1.1/24，所以需要先进行一个SNAT转为网络号192.168.1.1/24。

```shell
root@OpenWrt:~# iptables -t nat -N WLAN
root@OpenWrt:~# iptables -t nat -A WLAN -s 192.168.0.1/24 -j SNAT --to 192.168.1.11
root@OpenWrt:~# iptables -t nat -I POSTROUTING -j WLAN
```

成功上网

# 总结
设置好N1的两张网卡的地址后，还要注意把eth0那个接口的dhcp关闭，否则会和天猫的dhcp服务冲突了，之后用iptables做个SNAT，把从WIFI接口过来的数据的地址做一个SNAT，然后数据路由到天猫处理。