---
title: "Gentoo集群架设（2）"
tagline: "分布式编译emerge"
description: "Gentoo集群架设（2） 分布式编译emerge"
categorys: [study]
tags: [gentoo, 集群]
---


主要使用到的是distcc这个分布式编译软件

	# emerge distcc
	
注意必须在每一台机器上都安装该软件，并采用相同配置

	# vim /etc/portage/make.conf
	MAKEOPTS="-jN -lM"
	FEATURES="distcc"

这里N参数大家都应该清楚，是cpu个数，这里强调一下是所有机器的cpu数，官方是这样叙述的

A common strategy is to set N as twice the number of total (local + remote) CPUs + 1 and M as number of local CPUs.

两倍的cpu个数然后还要加一，我觉得还是安小的来（两倍的cpu数还要小一点），毕竟都分布式编译了，保证系统稳定，我是这么认为的

M参数是本地的cpu参数，是为了防止分布失败时不至于线程数过多拖垮系统，注意M前面是L，不要看着像一来填一了

你可以使用

	# /usr/bin/distcc-config --set-hosts "Host1IP Host2IP Host3IP Host4IP"

来设定机器数

也可以在make.conf里添加这条

	DISTCC_HOSTS="Host1IP,Host2IP,Host3IP,Host4IP"
	
据说越前面的越会被用到

启动并添加开机启动

	# rc-update add distccd default
	# /etc/init.d/distccd start
	
好了，可以配置ssh了

通过ssh-keygen等一系列操作将集群可以通过无密码进行访问，这点从略

对于蛋疼的SA来说，端口啥啥的总要改改，官方也给了教程

	# ssh-keygen -b 2048 -t rsa -f /var/tmp/portage/.ssh/id_rsa
	# nano -w /var/tmp/portage/.ssh/config
	Host test1
		HostName 123.456.789.1
		Port 1234
		User UserName
	
	Host test2
		HostName 123.456.789.2
		Port 1234
		User UserName
		
看着改吧

如此设定host时还可以这样

	# /usr/bin/distcc-config --set-hosts "@test1 @test2"

当然，还可以添加一个ccache

ccache是一个快速的编译器缓存。当您编译一个程序的时候，它会缓存中间的结果。这样，不论什么时候您重新编译同一个程序，编译所需要得时间将被大大缩短。对于普通的编译来说，这可以提高编译速度5到10倍。

	# emerge ccache

在make.conf中添加如下

CCACHE_SIZE="2G"

具体可以参考第二个参考链接

参考：

<https://wiki.gentoo.org/wiki/Distcc>

<https://www.gentoo.org/doc/zh_cn/handbook/handbook-x86.xml?part=2&chap=3>
