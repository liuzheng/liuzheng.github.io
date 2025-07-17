---
layout: post
title: "Valid Number"
description: ""
category: 
tags: [leetcode,python]
---


今天做了这题<https://oj.leetcode.com/problems/valid-number/>

感觉是作弊么。。。。python 还是屌

    try:
        float(s)
        return True
    except:
        return False

OK 试试不使用try玩

唉，试了字符判断，各种不够优雅

除去前后空白，将字符处理成简单符号，要判断的情况有以下几种：

    - n
    - sn
    - dn
    - nd
    - nen
    - ndn
    - snd
    - nen
    - sdn
    - nden
    - sndn
    - nesn
    - snen
    - dnen
    - sdnen
    - ndnen
    - ndesn
    - snden
    - dnesn
    - snesn
    - sdnesn
    - sndnen
    - ndnesn

    s=s.strip()
    S=' '
    for i in s:
        if i=='-' or i=='+':
            S=S+'s'
        elif i=='.':
            S=S+'d'
        elif i=='e' or i=='E':
            S=S+'e'
        elif i>='0' and i<='9':
            if not S[-1]=='n':
                S=S+'n'
        else:
            return False
    S=S.strip()
    冗余代码略
    return False
