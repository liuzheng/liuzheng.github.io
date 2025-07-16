---
title: "Dell 服务器 SAS硬盘热插拔识别问题解决"
description: ""
categorys: [server]
tags: [Dell, Server, Disk]
---


前日BOSS将硬盘取下倒腾了一下，然后插回服务器，一切看似如此淡定。

昨日收到小伙伴的报告，说hadoop无法启动。下面就是解决的详细过程。

首先验证他说的

  $ bin/stop-all.sh
  $ bin/start-all.sh

然后在浏览器上查看信息，发现果真宕了，好吧，相信他的话。

  Safe mode is ON. The reported blocks is only 0 but the threshold is 0.9990 and the total blocks \*\*\*\*. Safe mode will be turned off automatically.
  \*\*\*\* files and directories, \*\*\*\* blocks = \*\*\*\* total. Heap Size is \*\*\*\* GB / \*\*\*\* GB (\*%) 

然后到节点服务器上查看运行情况

  $ jps

呵呵，少了`DataNode`，手动加上看看

  $ hadoop-daemon.sh start DataNode
  Error: Could not find or load main class DataNode

呵呵，无效，好吧。查看一下系统有木有认出缺失的硬盘

  $ blkid

恩，确实系统没认出出去玩蛋的硬盘，重启依然认不出。那就是在BIOS里要折腾了

重启进入系统的

