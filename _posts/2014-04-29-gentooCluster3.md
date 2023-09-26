---
layout: post
title: "Gentoo集群架设（3）"
tagline: "配置buildhost"
description: "Gentoo集群架设（3） 配置buildhost"
category: study
tags: [gentoo, 集群]
---


前文已架设完毕gentoo的分布式编译集群，系统维护不可能每台机器都编译一次或者手动拷贝软件

故配置gentoo buildhost来分发二进制包

首先安装vsftpd

	emerge -av vsftpd

添加vsftpd自启动

	rc-update add vsftpd default

新建/home/ftp/gentoo

修改/home/ftp/gentoo文件夹777权限

	mount --bind /usr/portage/packages  /home/ftp/gentoo/
	vim /etc/local.d/mount.start中添加“mount --bind /usr/portage/packages  /home/ftp/gentoo/”

修改 /etc/vsftpd/vsftpd.conf，安自己需要配置，不赘述