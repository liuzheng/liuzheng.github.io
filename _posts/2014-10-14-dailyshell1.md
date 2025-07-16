---
title: "某日志处理"
description: "Daily Shell"
categorys: [dailyShell]
tags: [dailyShell]
---


 cat 20131001.log | sed 's/"" //'|sed 's/" "/"\t"/g' | awk -F"\t" 'BEGIN{}{if($13!="\"400\"" && $13!="\"403\"" && $13!="\"404\""){print $0;}}END{}' > 20131001.log.lz
