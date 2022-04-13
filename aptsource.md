---
layout: post
title: Ubuntu一行命令换源
date: 2022-04-13 19:48:00
tags: [Ubuntu]
---
记录一下，方便使用
```shell
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```