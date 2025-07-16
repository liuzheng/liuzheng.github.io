---
title: "Ansible Inventory"
description: "Ansible Inventory"
categorys: 
tags: [SA]
---


参考：<https://docs.ansible.com/intro_inventory.html>

由于实习要求我学习Ansible，故而开始看，今天遇到的问题是hosts的分组

我一直都是比较奇葩的，所以就把服务器分组就折腾的各种“优雅”

遇到的问题是未对官方文档详细查看，仅仅看了中文文档。

这里是一份 /etc/ansible/hosts 的样本

    mail.example.com
    
    [webservers]
    foo.example.com
    bar.example.com
    
    [dbservers]
    one.example.com
    two.example.com
    three.example.com
    
对于我来说，想做到的就是像盗梦空间里那样的各种嵌套，由于不熟练，以为分组仅仅是多加一个中括号而已，故而出现了连接问题

我是这样干的，/etc/ansible/hosts

    [AAA]
    IP1
    
    [BBB]
    IP2
    
    [CCC]
    AAA
    BBB

    
    $ ansible CCC -m ping
    CCC | FAILED => SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue

这里就出问题了，原因就是分组他有特殊的规则，必须后面加上`:children`，想想也对，符合逻辑，不然会出不知道的bug


    [AAA]
    IP1
    
    [BBB]
    IP2
    
    [CCC：children]
    AAA
    BBB

奇葩的我当然还没结束


    [AAA]
    IP1
    
    [BBB]
    IP2
    
    [CCC：children]
    AAA
    BBB
    
    [DDD]
    IP3

    [EEE:children]
    CCC
    DDD

当然测试EEE的时候会成功！这就初步达到了我想要的效果了

后面当然还有`:var`参数，后面再学习