---
layout: post
title: "终端访问代理系统(2)"
tagline: "Mac iTerm2 自动启动rdp链接后端"
description: ""
category: 
tags: ["Bastion"]
---

个人日常使用均是MBP，今天研究rz/sz功能的时候想着如何自动启动Windows会话链接。

查了一下文档，配置如下：

设置Iterm2的Tirgger特性，Profiles->Default->Advanced中的Tirgger 

![iTerm2](/assets/imgs/bifrost/iterm2.png)

之后在连接Bifrost中链接rdp时会显示rdp会话的url，iTerm2会自动打开Windows Remote Desktop 去连接，前提当然得安装了该软件。

其他软件思路大体一致，后期会测试Windows下几个软件的自动打开方式。
