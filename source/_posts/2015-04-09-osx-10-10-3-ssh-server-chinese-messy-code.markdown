---
layout: post
title: "max升级10.10.3后的坑"
date: 2015-04-09 17:52:09 +0800
comments: true
categories: mac
---

* ssh 到服务器中文乱码，巨坑，网上查到了解决方案。来自[这里](http://kqwd.blog.163.com/blog/static/41223448201211604620454/)。不让mac发送自己的编码过去。方案如下：

	这个问题跟Mac的终端程序有关。修正方法是：

	编辑Mac系统下/etc/ssh_config这个文件
	将其中SendEnv LANG LC_*这一行前面加一个井号，注释掉，保存退出

	再使用ssh连接远程Linux服务器时，中文就不会有问题了~	