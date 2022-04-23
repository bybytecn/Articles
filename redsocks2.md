---
layout: post
title: 交叉编译redsocks2到斐讯N1盒子
date: 2022-04-23 19:50:00
tags: [编译]
---
# 前言
我的N1盒子上需要跑redsocks,但是总感觉redsocks有点问题，想试试redsocks2，所以得自己交叉编译一下了。

# 准备工具
[redsocks2仓库地址](https://github.com/semigodking/redsocks)

[libevent2-2.1.12](https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz)

aarch64-linux-gnu-gcc



# 交叉编译
## Step1: 编译libvent2
```shell
export CC=aarch64-linux-gnu-gcc
./configure --prefix=/root/c --host=aarch64-unknown-gnu
make
make install
```

## Step2: 编译redsocks2
```shell
export CC=aarch64-linux-gnu-gcc
```
修改Makefile,添加以下几个CFLAGS
```
CFLAGS+=-I/root/c/include
CFLAGS+=-L/root/c/lib
CFLAGS+=-I/root/open/include
CFLAGS+=-L/root/open/lib
CFLAGS+=-static
```
编译
```shell
make DISABLE_SHADOWSOCKS=true
```

# 打包
为了方便，把打包完成的文件放在这里了[redsocks2](./redsocks.jpg) **(需要把文件后缀改为zip)**