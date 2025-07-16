---
title: "phpmyadmin去除iframe限制"
description: ""
categorys: 
tags: [phpmyadmin]
---


无聊要将phpmyadmin页面嵌入到某页面iframe中，做成统一的控制台管理界面，在此记录一下对phpmyadmin源代码的一些操作

修改 /usr/share/phpmyadmin/libraries/Header.class.php 461 行

    //'X-Frame-Options: DENY'

将其注释

修改 /usr/share/phpmyadmin/js/cross_framing_protection.js

    /* vim: set expandtab sw=4 ts=4 sts=4: */
    /**
    * Conditionally included if framing is not allowed
    */
    //if(self == top) {
    document.documentElement.style.display = 'block' ;
    //} else {
    //    top.location = self.location ;
    //}

至此phpmyadmin可以在iframe内展示咯
