---
layout: post
title: Rust交叉编译到Aarch64
date: 2022-04-13 21:20:00
tags: [Rust,交叉编译]
---
# 前言
由于路由器是AArch64架构的，写的一些小工具也难免需要交叉编译过去运行，记录一下

# 步骤
## Step 1:安装依赖
```shell
sudo apt install gcc-aarch64-linux-gnu
sudo apt install g++-aarch64-linux-gnu
```

## Step 2:安装工具链
```shell
rustup target add aarch64-unknown-linux-gnu
```

## Step 3:指定链接器
```shell
gedit ~/.cargo/config
```
设置为如下内容
```
[target.aarch64-unknown-linux-gnu]

linker="/usr/bin/aarch64-linux-gnu-gcc"
```

## Step 4:编译
```shell
cargo build --target=aarch64-unknown-linux-gnu
```