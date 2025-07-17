---
layout: post
title: "Gentoo集群架设（1）"
tagline: "安装系统"
description: "Gentoo集群架设（1） 安装系统"
category: study
tags: [gentoo, 集群]
---



## gentoo install

编译内核省略

## 以下是最初使用的方法 

	passwd
	/etc/init.d/sshd start
	fdisk /dev/sda
	300M
	4G
	mkfs.ext4 /dev/sda1
	mkfs.ext4 /dev/sda3
	mkswap /dev/sda2
	swapon /dev/sda2


	scp portage-*.tar.bz2 root@192.168.1.133:/mnt/gentoo/
	scp stage3-*.tar.bz2 root@192.168.1.133:/mnt/gentoo/
	tar jxf stage3-*.tar.bz2
	tar jxf portage-*.tar.bz2 -C  /mnt/gentoo/usr
	mount /dev/sda3 /mnt/gentoo 
	mkdir /mnt/gentoo/boot 
	mount /dev/sda3 /mnt/gentoo 
	mount /dev/sda1 /mnt/gentoo/boot

	mount -t proc none /mnt/gentoo/proc 
	mount -o bind /dev /mnt/gentoo/dev 

	cp -L /etc/resolv.conf /mnt/gentoo/etc/

	chroot /mnt/gentoo /bin/bash
	emerge --sync --quiet
	cd usr/src/linux

	make menuconfig

	rsync -azIHP --exclude=src/* --exclude=portage/distfiles/* -e ssh /usr/ root@192.168.1.133:/mnt/gentoo/usr/
	rsync -azIHP -e ssh /bin/ root@192.168.1.133:/mnt/gentoo/bin/
	rsync -azIHP -e ssh /sbin/ root@192.168.1.133:/mnt/gentoo/sbin/
	rsync -azIHP -e ssh /boot/ root@192.168.1.133:/mnt/gentoo/boot/
	rsync -azIHP -e ssh /lib/ root@192.168.1.133:/mnt/gentoo/lib/
	rsync -azIHP -e ssh /etc/ root@192.168.1.133:/mnt/gentoo/etc/


### ONCE

	rsync -azIHP -e ssh /home/ root@192.168.1.133:/mnt/gentoo/home/
	rsync -azIHP -e ssh /root/ root@192.168.1.133:/mnt/gentoo/root/
	rsync -azIHP -e ssh /var/ root@192.168.1.133:/mnt/gentoo/var/

	grub2-install --no-floppy /dev/sda
	#grub2-install --grub-setup=/bin/true /dev/sda
	grub2-mkconfig -o /boot/grub/grub.cfg
	#grub> root (hd0,0)    （指定您的/boot目录所在分区）
	#grub> setup (hd0)     （将GRUB安装到硬盘主引导记录）
	#grub> quit            （退出GRUB shell）

## 最新方案

制作自己的gentoo包文件，直接用脚本解压安装，前提是已组建完基本集群及buildhost和ftp的配置

脚本如下chroot脚本

	#!/bin/bash
	# coding: utf-8
	# Copyright (c) 2014
	# Gmail:liuzheng712
	#
	fdisk /dev/sda
	mkfs.ext4 /dev/sda1
	mkfs.ext4 /dev/sda3
	mkswap /dev/sda2
	mkdir /mnt/gentoo
	mount /dev/sda3 /mnt/gentoo
	mkdir /mnt/gentoo/boot
	mount /dev/sda1 /mnt/gentoo/boot/
	cd /mnt/
	ftp=$1
	while [ -z $ftp ]
	do
	  read -p "Please input the ftp server:" ftp
	done
	wget 'ftp://'$ftp'/gentoo.tb2'
	tar xjfvm gentoo.tb2
	mount -t proc none /mnt/gentoo/proc
	mount -o bind /dev /mnt/gentoo/dev
	cp -L /etc/resolv.conf /mnt/gentoo/etc/
	echo 'PORTAGE_BINHOST="ftp://'$ftp'/gentoo"' >> /mnt/gentoo/etc/make.conf
	chroot /mnt/gentoo /bin/bash

解压完后安装脚本，在/目录下

	#!/bin/bash
	# coding: utf-8
	# Copyright (c) 2014
	# Gmail:liuzheng712
	#
	emerge --sync
	emerge -auvDN world
	emerge ntp grub
	ntpdate time.qq.com
	grub2-install --no-floppy /dev/sda
	grub2-mkconfig -o /boot/grub/grub.cfg
	default_name='gentoo'
	read -p "Please input this PC-name(default $default_name):" hostname
	hostname=${hostname:-$default_name}
	echo 'hostname="'$hostname'"' >> ./etc/conf.d/hostname

如有错误敬请指正！


