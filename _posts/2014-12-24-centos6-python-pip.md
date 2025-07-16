---
title: "CentOS6 安装Python2.7 及 pip记录"
description: ""
# category:
# tags: []
---


实验室机器是 Centos6 yum完全不能升级成Python2.7，so 这几日
看了一些Python升级的东西。

# CentOS切记不能自己编译，内部依赖实在太多！！！

## epel !!!

epel是标配大家都懂的，这里就顺手加一下，以后查起来便利。

    sudo rpm -ivh https://download.fedora.redhat.com/pub/epel/6/x86_64/epel-release-6-5.noarch.rpm

## 安装SCL
这个可以提供如下几个版本的更新

    yum install centos-release-SCL

官方原话

`Currently, the following collections are available for CentOS 6.5 and later (package name in parenthesis):`


    Ruby 1.9.3 (ruby193)
    Python 2.7 (python27)
    Python 3.3 (python33)
    PHP 5.4 (php54)
    Perl 5.16.3 (perl516)
    Node.js 0.10 (nodejs010)
    MariaDB 5.5 (mariadb55)
    MySQL 5.5 (mysql55)
    PostgreSQL 9.2 (postgresql92)

## 安装python2.7
我这里就安装python27所有的东西了

    sudo yum install python27-python*

如下是即将安装的包

    ========================== N/S Matched: python27-python ==========================
    python27-python.x86_64 : An interpreted, interactive, object-oriented programming
    : language
    python27-python-babel.noarch : Library for internationalizing Python applications
    python27-python-bson.x86_64 : Python bson library
    python27-python-coverage.x86_64 : Code coverage testing module for Python
    python27-python-debug.x86_64 : Debug version of the Python runtime
    python27-python-devel.x86_64 : The libraries and header files needed for Python
    : development
    python27-python-docutils.noarch : System for processing plaintext documentation
    python27-python-jinja2.noarch : General purpose template engine
    python27-python-libs.x86_64 : Runtime libraries for Python
    python27-python-markupsafe.x86_64 : Implements a XML/HTML/XHTML Markup safe string
    : for Python
    python27-python-nose.noarch : Discovery-based unittest extension for Python
    python27-python-nose-docs.noarch : Nose Documentation
    python27-python-psycopg2.x86_64 : A PostgreSQL database adapter for Python
    python27-python-psycopg2-doc.x86_64 : Documentation for psycopg python PostgreSQL
    : database adapter
    python27-python-pygments.noarch : Syntax highlighting engine written in Python
    python27-python-pymongo.x86_64 : Python driver for MongoDB
    python27-python-pymongo-gridfs.x86_64 : Python GridFS driver for MongoDB
    python27-python-setuptools.noarch : Easily build and distribute Python packages
    python27-python-simplejson.x86_64 : Simple, fast, extensible JSON encoder/decoder
    : for Python
    python27-python-six.noarch : Python 2 and 3 compatibility utilities
    python27-python-sphinx.noarch : Python documentation generator
    python27-python-sphinx-doc.noarch : Documentation for python-sphinx
    python27-python-sqlalchemy.x86_64 : Modular and flexible ORM library for python
    python27-python-test.x86_64 : The test modules from the main python package
    python27-python-tools.x86_64 : A collection of development tools included with
    : Python
    python27-python-virtualenv.noarch : Tool to create isolated Python environments
    python27-python-werkzeug.noarch : The Swiss Army knife of Python web development
    python27-python-werkzeug-doc.noarch : Documentation for python-werkzeug

安装完后还没结束，需要启用python27

    scl enable python27 bash

## pip
首先查看一下当前的pip版本

    $ pip -V
    pip 1.5 from /usr/lib/python2.6/site-packages/pip-1.5-py2.6.egg (python 2.6)

我这里选择最新的pip安装包

    wget https://pypi.python.org/packages/source/p/pip/pip-1.5.tar.gz --no-check-certificate
    tar xfz pip-1.5.tar.gz
    cd pip-1.5
    sudo python setup.py install

安装玩就搞定pip的升级了，同时也支持了python2.7了

    pip -V
    pip 1.5 from /opt/rh/python27/root/usr/lib/python2.7/site-packages/pip-1.5-py2.7.egg (python 2.7)

唉，手贱还是pip upgrade吧

    pip install --upgrade pip
    pip -V
    pip 6.0.3 from /opt/rh/python27/root/usr/lib/python2.7/site-packages (python 2.7)

至此搞定！

# 参考

https://wiki.centos.org/AdditionalResources/Repositories/SCL
