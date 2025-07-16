---
title: "Leap second"
description: ""
# category: 
# tags: [运维]
---


今天经历了闰秒，好像很牛逼的样子。不过我胆子小，关了ntp服务。

据说是kernel crash，kernel version 2.6.18-164.e15之后的版本解决了这个问题，所以我还是希望运维人员掌握kernel权限，但是实际情况是木有。。。

不过还是得摘录一下群内的解决方案和问题。

这是群内某人的实况截图

![](/assets/imgs/LeapSecond.jpg)

高亮处其实是kernel的信息，下面的ntpd才是正真的ntp服务的log

## CPU在过闰秒后100%

    service ntpd stop
    date -s "`date`"
    service ntpd restart

部分人是由于ntp server开了，然后似乎是开了Java进程，不过我觉得还是用了有bug的kernel，能运维掌控kernel多好。。。

据说是闰秒过后，系统本该调用 clock_was_set ，但是bug 的kernel没有调用

我觉得此文日后绝对用不到了，还是mark一下，以后查完kernel版本就淡定的let it go吧！

