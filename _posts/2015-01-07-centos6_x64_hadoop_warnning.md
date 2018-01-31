---
layout: post
title: "CentOS6 x64 Hadoop编译"
description: ""
category:
tags: [hadoop]
---


由于各种“偷懒”，架设hadoop集群的时候按照正常步骤安装，最近发现日志里
总有一堆的java报错，通过各方帖子了解是hadoop官方默认的是32位，而我这里
使用的是64位的centos6。好吧，编译了要


# 编译环境安装

    yum install cmake lzo-devel zlib-devel gcc autoconf automake libtool ncurses-devel  openssl-devel gcc-c++


# maven 配置

    wget  http://mirrors.cnnic.cn/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz

解压，并cd到/usr/local/目录

对解压的文件在/usr/local/目录下做软连接

在/etc/profile后追加

    export MAVEN_HOME=/usr/local/maven
    export PATH=$MAVEN_HOME/bin:$PATH

source一下

    # mvn -v
    Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-15T01:29:23+08:00)
    Maven home: /usr/local/maven
    Java version: 1.7.0_71, vendor: Oracle Corporation
    Java home: /opt/jdk1.7.0_71/jre
    Default locale: en_US, platform encoding: UTF-8
    OS name: "linux", version: "2.6.32-504.3.3.el6.x86_64", arch: "amd64", family: "unix"

# protoc 安装
需要gcc-c++
在网站 http://code.google.com/p/protobuf/downloads/list上可以下载 Protobuf 的源代码,目前最新版本是2.4.1.解压缩,编译,步骤如下:

    tar -xzf protobuf-2.4.1.tar.gz
    cd protobuf-2.4.1
    ./configure --prefix=/usr/local/protobuf
    make
    make check
    make install(该步骤,我是使用得root用户,若其他用户,请不要安装在/usr/下,请安装在账户目录下即可)

在/etc/profile后追加

    export PROTO_HOME=/usr/local/protobuf
    export PATH=$PROTO_HOME/bin:$PATH


# 安装Ant

    wget http://mirrors.sonic.net/apache/ant/binaries/apache-ant-1.9.4-bin.tar.gz
解压并在/usr/local/下做软连

    export ANT_HOME=/usr/local/ant
    export PATH=$PATH:$ANT_HOME/bin

# 准备编译

下载一个源文件

    wget http://apache.arvixe.com/hadoop/common/stable2/hadoop-2.6.0-src.tar.gz

常规解压

cd到目录有运行

    mvn package -DskipTests -Pdist,native -Dtar



# 参考
<http://www.eziep.net/details/136.html>
