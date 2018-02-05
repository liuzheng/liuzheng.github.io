---
layout: post
title: "Ubuntu add ppa Error"
description: ""
category: 
tags: [ubuntu, ppa]
---

参考<http://askubuntu.com/questions/212132/i-cant-add-ppa-repository-behind-the-proxy>

    sudo add-apt-repository ppa:gnome3-team/gnome3
    Cannot add PPA: 'ppa:gnome3-team/gnome3'.

该问题是由于是使用代理上网导致的

    export http_proxy=http://username:password@host:port/
    export https_proxy=https://username:password@host:port/

并且在 /etc/sudoers 末尾追加   `Defaults env_keep="https_proxy"` 

之后再运行

    sudo -E add-apt-repository ppa:gnome3-team/gnome3

    