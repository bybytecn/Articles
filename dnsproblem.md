---
layout: post
title: 搭建DNS服务时一个非常诡异的问题
date: 2022-04-08 00:15:00
tags: [系统工具]
---
# 前言
笔者需要搭建一个DNS服务端，用于某个用途（这里不方便透露了，目前代码非常简单，如下
```go
package main

import (
	"github.com/miekg/dns"
	"time"
)

type name struct {
}

func (receiver name) ServeDNS(w dns.ResponseWriter, r *dns.Msg) {
	c := new(dns.Client)
    //192.168.1.1:53是笔者内网环境的网关地址，作为我这个DNS服务端的上游
	rr, _, err := c.Exchange(r, "192.168.1.1:53")
	if err != nil {
		fmt.Println(err)
	}
	c.UDPSize = 65535
	w.WriteMsg(rr)
}
func main() {
	s := dns.Server{
		Addr:         ":53",
		Net:          "udp",
		ReadTimeout:  time.Minute * 20,
		WriteTimeout: time.Minute * 20,
		IdleTimeout: func() time.Duration {
			return time.Hour * 24
		},
	}
	s.Handler = &name{}
	s.ListenAndServe()
}

```
但是遇到了一个很奇怪的问题：在浏览网页时，偶然有那么一些请求失败，具体原因就是域名解析失败, 如下图

![avatar](./3333.png)

# 问题出现的条件
经过对比分析，总结了这个问题出现的条件:
- 出现这个问题的软件不限于浏览器，微信也有
- 无论主机系统，使用ping命令，解析没任何问题
- 使用公共的DNS服务(包括我本地的网关)，没任何问题
- DNS服务端跑在Linux上，主机Windows，出现问题
- DNS服务端跑在Linux上，主机Linux，没有出现问题
- DNS服务端跑在Windows上，主机Linux，没有出现问题
- DNS服务端跑在Windows上，主机Windows，出现问题

显然，问题出现在我的主机Windows上和我的DNS服务端上，但具体原因实在没法得知，因为问题是出现在我用chrome浏览网页的时候出现的，里面具体的函数调用没源码无法得知

# 用PhantomJS复现问题
Chrome是开源，但是源码巨大，而PhantomJS是个无UI的浏览器，于是看能不能用PhantomJS尝试复现这个问题，万幸成功复现了,加载一个网站会有几个域名无法解析
![avatar](./4444.png)

## 调试定位问题
问题出现了，代码也有了，接下来是Debug追踪了，PhantomJS底层是依赖QT、QtWebKit，经过漫长的下载源码、编译、调试.......... 最终复现出如下最简代码
```c++
int main()
{
	WSADATA wsaData;
	struct addrinfo hints;
	addrinfo *res = 0;
	WSAStartup(MAKEWORD(2, 2), &wsaData);
	memset(&wsaData, 0, sizeof(WSADATA));
	memset(&hints, 0, sizeof(hints));
	hints.ai_family = PF_UNSPEC;
	int result = -1;
	for (int j = 0; j < 1000; j++) {
		for (int i = 0; i < 1; i++) {
			result = getaddrinfo("hm.baidu.com", nullptr, &hints, &res);
			std::cout << result << std::endl;
		}
		for (int i = 0; i < 1; i++) {
			result = getaddrinfo("www.zhihu.com", nullptr, &hints, &res);
			std::cout << result << std::endl;
		}
		//pica.zhimg.com
		for (int i = 0; i < 1; i++) {
			result = getaddrinfo("pica.zhimg.com", nullptr, &hints, &res);
			std::cout << result << std::endl;
		}
		//static.zhihu.com
		for (int i = 0; i < 1; i++) {
			result = getaddrinfo("static.zhihu.com", nullptr, &hints, &res);
			std::cout << result << std::endl;
		}
		Sleep(1000);
	}
	return 0;
}
```
> getaddrinfo是window的一个系统调用，用于解析域名的

运行上述代码，大部分都是正常的，但是偶然就是出现几个解析失败的。部分输出如下:
```shell
0
0
0
11001
0
0
0
0
0
0
0
0
0
0
```
> 输出11101就是解析出错的那一个

百思不得其解，怎么会偶尔才出现呢，难道是这个系统调用的问题？

# 抓包
既然应用层没法找到问题，只能看底层了，用**Microsoft Network Monitor 3.4**抓包分析，发现出错的时候服务端发了一个ICMP Port Unreaching的报文
![avatar](./555.png)

我琢磨怎么会端口不可达呢，前面的那些请求都是正常的啊.....百思不得其解，突然想到之前我把系统的IPV6关了，是不是这个问题，打开后，**一切正常**......

![avatar](./666.png)

# 问题猜测
虽然我之前把系统的ipv6协议关了，但是那个**getaddrinfo**系统调用也会调用ipv6协议进行dns解析，而我把ipv6关了的，自然也没ipv6网关和ipv6 dns服务器了，所以会无法解析，当我重新打开ipv6协议后，系统重新设置了ipv6网关和ipv6的dns服务端，所以所有协议的dns请求都正常，一切都正常了...

## 疑问
虽然现在都是IPV4+IPV6双栈请求了，但是为什么我把IPV6关了，系统还是会请求ipv6协议的dns....这是微软的BUG么....无法理解