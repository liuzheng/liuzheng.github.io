---
title: "DOM 双向绑定"
description: ""
categorys: 
tags: [JavaScript]
---


因为觉得AngularJS太重了，一般都会想尝试重新造轮子，先做一些基础调查吧。

<https://www.html-js.com/article/A-day-to-learn-JavaScript-using-the-native-JavaScript-data-binding>

ps: 上面链接的讨论圆点功能不错考虑后期加上^_^!

<https://www.jianshu.com/p/ee014d86cd3f>

<https://github.com/xufei/Make-Your-Own-AngularJS/blob/master/01.md>

AngularJS是脏值检测，想着用原生办法解决，试试吧，如果不用jQuery的话。

<https://www.zhuowenli.com/frontend/easy-two-way-data-binding-in-javascript.html>

这篇文章讲的比较细致

但是对于操作上需要使用set函数，太麻烦了。

其实想想jquery 80，90 kb，angularjs1.x 123kb，加起来就是200kb。
按照手机10kb/s就是20秒，但是大多都是牛逼的手机吧，应该两秒内可以下完，但是，哎哎哎，为什么要考虑手机呢。。。我也就是自己想重新造个轮子。。。不过200kb确实似乎有点耗时间啊>_<


再贴一篇：<https://www.ituring.com.cn/article/48463>

囫囵吞枣的了解了大概，估计过两天就忘了。。。

大致原理就是通过html的自定义属性绑定DOM标签，加上$watch （突然想到了硬件上的watchdog，一直觉得当时老师说喂狗，让狗叫很萌啊。。。）

找到了`前端乱炖`的那个圆点了，叫tips.js，不知道LICENSE是如何，觉得可能需要自己重写一个，那个真的好好玩啊

