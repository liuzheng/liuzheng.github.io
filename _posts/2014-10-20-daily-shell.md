---
layout: post
title: "挂载 smb 盘"
description: "Daily Shell"
category: dailyShell
tags: [dailyShell]
---


挂载 smb 盘时 mount -t 是 cifs ，不是 smbfs，被坑了一晚上。。。

    mount -t cifs //IP/share /tmp/mountPoint

今天开始准备把数据导出到hdfs里

    gunzip -c 20131003.log.gz | sed 's/"" //'|sed 's/" "/"\t"/g' | awk -F"\t" 'BEGIN{}{if($13!="\"400\"" && $13!="\"403\"" && $13!="\"404\""){print $0;}}END{}' > /opt/yun/20131003.log.lz
