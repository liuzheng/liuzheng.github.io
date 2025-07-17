---
layout: post
title: "Mac 刷路由器固件"
tagline: ""
description: ""
category: 
tags: []
---

今天写一下刷路由器等固件时所需的环境，mac版，因为网上也大多是Windows的，可能mac用户本身买mac就是解决生产力，懒的折腾吧。

## Mac 上自带TFTP Server
### 配置 
我是未修改配置文件，直接用命令启动，暂时不考究是什么意思

    sudo launchctl start com.apple.tftpd

## 配置网络
这块根据路由器的设置去设定ip和网关即可，不同路由器不一样

为啥我感觉啥也没说，哎，纯粹记录一下吧。

## 编译系统
这块还是得靠日常技术经验和对openwrt等系统的需求，编译我比较懒，是在docker里编译的，这样环境依赖可控且复用性比较好。


参考
<https://www.jianshu.com/p/f230ae5b865c>
