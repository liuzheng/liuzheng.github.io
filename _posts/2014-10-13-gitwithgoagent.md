---
title: "gitWithGoAgent"
description: ""
categorys: [proxy]
tags: [proxy, goagent]
---


找到.gitconfig隐藏目录，添加两行设置

    [http] 
        proxy = https://127.0.0.1:8087 
        sslVerify = false
        sslCAinfo = $goagent/local/CA.crt
    [https] 
        proxy = https://127.0.0.1:8087 
        sslVerify = false
        sslCAinfo = $goagent/local/CA.crt
        
    ssh-keygen -t rsa -P '' -f /root/.ssh/id_rsa