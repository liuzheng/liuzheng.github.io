---
layout: post
title: "goagent+docker"
description: ""
category: 
tags: [gfw]
---


参考：<http://kb.kerio.com/product/kerio-connect/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html>

蛋疼了一下午，本来要写man ssh都delay 了。。。

好吧，我比较偏执，docker pull一定要走官方，别人提供的服务我不相信（其实也没什么，主要探究一下你懂的技术）

git clone 最新的goagent 代码，挂上梯子，上传，这里要确保你的goagent能work！

当然咯，iplist么有时候大家懂的，需要用checkgoogleip跑一下，感谢[@moonshawdo](https://github.com/moonshawdo)的工作。

我这里是CentOS7的机器，上海教育网

用参考链接的CentOS6的方案木有问题

Install the ca-certificates package:
    
    yum install ca-certificates
    
Enable the dynamic CA configuration feature:
    
    update-ca-trust enable
    
Add it as a new file to /etc/pki/ca-trust/source/anchors/:
    
    cp /GoAgentPath/locale/CA.crt /etc/pki/ca-trust/source/anchors/
    
Use command:
    
    update-ca-trust extract

修改/etc/systemd/system/docker.service

在Service里面添加一行

    Environment='HTTP_PROXY=http://127.0.0.1:8087/'

然后reload

    systemctl daemon-reload

最后重启docker

    service doceker restart

至此，docker就能从goagent上番茄拉取数据咯！！！

这种做服务环境的还是从官方弄的好，后门什么大家还是要考虑一下，从国内拉一遍后再从官方弄一下，确保hash值对就行
