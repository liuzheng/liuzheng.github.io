---
layout: post
title: "Python, Hbase, Thrift"
description: "Python, Hbase, Thrift"
category: hbase
tags: [hbase]
---


# 使用 Python 对 Hbase 做操作初期配置

首先下载 thrift 用来生成 python hbase 包

<https://thrift.apache.org/download>

下载 Hbase 源码包

<http://www.apache.org/dyn/closer.cgi/hbase/>

## Windows 篇

<http://apache.fayea.com/apache-mirror/thrift/0.9.1/thrift-0.9.1.exe>

将其拷贝到默认 PATH 路径下，若喜欢安文件分类，手动加PATH吧，我懒。。。直接加到Git/bin下面

找到 Hbase 源码文件中 hbase.thrift 。当然，你会发现上级目录有一个thrift2，其对比我在该链接(Thrift介绍与应用（三）—hbase的thrift接口)[http://blog.csdn.net/guxch/article/details/12163047]找到了他们的不同，深入了解的话可以细看。

cd 到刚才找到的文件，运行

    thrift --gen py hbase.thrift

会生成 gen-py 文件夹，将其内部的 hbase 复制到 $PYTHON/Lib\site-packages 下即可 import 了

## Ubuntu 篇

安装必要的包

    sudo apt-get install python-dev automake libtool flex bison pkg-config g++
     
安装boost

    sudo apt-get install libboost-test-dev

cd 到 thrift 文件夹，运行

    ./configure --without-ruby --without-php --without-cpp

我就要一个 Python ， 运行结果如下

    Building C++ Library ......... : no
    Building C (GLib) Library .... : no
    Building Java Library ........ : no
    Building C# Library .......... : no
    Building Python Library ...... : yes
    Building Ruby Library ........ : no
    Building Haskell Library ..... : no
    Building Perl Library ........ : no
    Building PHP Library ......... : no
    Building Erlang Library ...... : no
    Building Go Library .......... : no
    Building D Library ........... : no

很好，可以 make 了

    make -j8

表忘了 make install

## 一些二的问题

### thrift.transport.TTransport.TTransportException: Could not connect to xxxx:9090

木有开启hbase的thrift啊啊啊啊啊啊

    hbase-daemon.sh start thrift
