---
layout: post
title: "AngularJS做网站的一些想法"
description: "AngularJS"
category: 
tags: [AngularJS]
---


## 页面速度

由于撰写此类网站需要用到大量js代码，而且一般都是外链，故网站相应有时候相当慢。所以我觉得应该在首页尽可能少的使用js生成的页面，将css放入head，使得能迅速加载出用户可视的页面，让用户先看一会儿，让js飞一会儿，时间应该够。努力让用户初次就看到页面看一会儿，给js时间
