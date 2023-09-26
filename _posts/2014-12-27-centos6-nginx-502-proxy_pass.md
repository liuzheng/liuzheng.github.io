---
layout: post
title: "CentOS6 Nginx 502 proxy_pass"
description: ""
category:
tags: [SA]
---


困扰一天了。。。

SELinxu在这台服务器上居然开了

## 查看SELinux状态：

1、/usr/sbin/sestatus -v      ##如果SELinux status参数为enabled即为开启状态

SELinux status:                 enabled

2、getenforce                 ##也可以用这个命令检查

Nginx做反代啊什么各种502

# 临时

    setsebool -P httpd_can_network_connect 1

# 永久
修改/etc/selinux/config 文件

将SELINUX=enforcing改为SELINUX=disabled


# 有时候iptables也会来凑热闹

    chkconfig --level 12345 iptables off
    chkconfig --level 12345 ip6tables off
    service iptables stop
    service ip6tables stop

参考

<http://stackoverflow.com/questions/25995060/nginx-cannot-connect-to-jenkins-on-centos-7>

<http://www.cnblogs.com/xiangxiaodong/>
