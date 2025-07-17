---
layout: post
title: "Node for windows Install"
tagline: ""
description: ""
category: Node
tags: [Node, guide]
---


# 安装 Node 环境
## 下载NodeJs
前往 https://nodejs.org/download/ 下载node 最新版本，这里选择exe版本，个人喜欢自主控制

## 创建目录
创建D:\dev\nodejs目录，并将node.exe保存在这个目录中。并将"D:\dev\nodejs"加入系统环境变量PATH中，便于在任意位置执行node应用。

## 下载npm源代码：
    cd  /D/dev/nodejs
	git clone https://github.com/npm/npm --depth 1
	node cli.js install -gf
   这里需要注意一下，在写这篇文章时npm最新版本为1.0.106，但是这个最新版本及1.0.105在Windows平台下都有问题。所以我选择了安装1.0.104版本：
   https://github.com/isaacs/npm/zipball/v1.0.104