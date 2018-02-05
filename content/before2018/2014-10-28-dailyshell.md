---
layout: post
title: "shell变量"
description: "Daily Shell"
category: dailyShell
tags: [dailyShell]
---


shell变量

    ${varname:-word}    如varname存在且非null，则返回其值，否则返回word
    ${varname:=word}    如varname存在且非null，则返回其值，否则设置为word
    ${varname:?word}    如varname存在且非null，则返回其值，否则显示varname:word
    ${varname:+word}    如varname存在且非null，则返回word，否则返回null


    ${varname#key}  从头开始删除关键词，执行最短匹配
    ${varname##key}  从头开始删除关键词，执行最长匹配
    ${varname%key}  从尾开始删除关键词，执行最短匹配
    ${varname%%key}  从尾开始删除关键词，执行最长匹配
    ${varname/old/new}  将old替换为new，仅替换第一个出现的old
    ${varname//old/new}  将old替换为new，替换所有old
