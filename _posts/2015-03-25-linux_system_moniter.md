---
title: "[转]你值得拥有 —— 25 个 Linux 性能监控工具"
description: ""
categorys: 
tags: [Linux, 性能监控工具]
---


原文：https://linoxide.com/monitoring-2/linux-performance-monitoring-tools/作者： Adrian Dinu

译文：LCTT  https://linux.cn/article-4481-1.html 译者： andyxue

一段时间以来，我们在网上向读者介绍了如何为Linux以及类Linux操作系统配置多种不同的性能监控工具。在这篇文章中我们将罗列一系列使用最频繁的性能监控工具，并对介绍到的每一个工具提供了相应的简介链接，大致将其划分为两类，基于命令行的和提供图形化接口的。

# 基于命令行的性能监控工具
## 1. [dstat](https://linux.cn/article-3215-1.html) - 多类型资源统计工具
该命令整合了`vmstat`，`iostat`和`ifstat`三种命令。同时增加了新的特性和功能可以让你能及时看到各种的资源使用情况，从而能够使你对比和整合不同的资源使用情况。通过不同颜色和区块布局的界面帮助你能够更加清晰容易的获取信息。它也支持将信息数据导出到cvs格式文件中，从而用其他应用程序打开，或者导入到数据库中。你可以用该命令来监控cpu，内存和网络状态随着时间的变化。

![](/assets/imgs/000431dj4t4bckobrkt7uc.png)

##　2. [atop](https://linoxide.com/monitoring-2/guide-using-linux-atop/) - 相比top更好的ASCII码体验
这个使用ASCII码显示方式的命令行工具是一个显示所有进程活动的性能监控工具。它可以展示每日的系统日志以进行长期的进程活动分析，并高亮显示过载的系统使用资源。它包含了CPU，内存，交换空间，磁盘和网络层的度量指标。所有这些功能只需在终端运行atop即可。

    # atop

当然你也可以使用交互界面来显示数据并进行排序。
![](/assets/imgs/000433occszd5tslddtd17.jpg)

## 3. [Nmon](https://linoxide.com/monitoring-2/install-nmon-monitor-linux-performance/) - 类Unix系统的性能监控
Nmon是Nigel's Monitor缩写，它最早开发用来作为AIX的系统监控工具。如果使用在线模式，可以使用光标键在屏幕上操作实时显示在终端上的监控信息。使用捕捉模式能够将数据保存为CSV格式，方便进一步的处理和图形化展示。
![](/assets/imgs/000435apz51c1p00njpkjp.png)

## 4. [slabtop](https://linux.cn/article-3702-1.html) - 显示内核slab缓存信息
这个应用能够显示缓存分配器是如何管理Linux内核中缓存的不同类型的对象。这个命令类似于top命令，区别是它的重点是实时显示内核slab缓存信息。它能够显示按照不同排序条件来排序显示缓存列表。它同时也能够显示一个slab层信息的统计信息的题头。举例如下：

    # slabtop --sort=a
    # slabtop -s b
    # slabtop -s c
    # slabtop -s l
    # slabtop -s v
    # slabtop -s n
    # slabtop -s o

## 5. sar - 性能监控和瓶颈检查
sar 命令可以将操作系统上所选的累积活动计数器内容信息输出到标准输出上。其基于计数值和时间间隔参数的审计系统，会按照指定的时间间隔输出指定次数的监控信息。如果时间间隔参数为设置为0，那么[sar命令将会显示系统从开机到当时时刻的平均统计信息](https://linoxide.com/linux-command/linux-system-performance-monitoring-using-sar-command/)。有用的命令如下：

    # sar -u 2 3
    # sar -u -f /var/log/sa/sa05
    # sar -P ALL 1 1
    # sar -r 1 3
    # sar -W 1 3

## 6. Saidar - 简单的统计监控工具
Saidar是一个简单且轻量的系统信息监控工具。虽然它无法提供大多性能报表，但是它能够通过一个简单明了的方式显示最有用的系统运行状况数据。你可以很容易地看到[运行时间、平均负载、CPU、内存、进程、磁盘和网络接口](https://linoxide.com/monitoring-2/monitor-linux-saidar-tool/)统计信息。

    Usage: saidar [-d delay] [-c] [-v] [-h]
     
    -d 设置更新时间（秒）
    -c 彩色显示
    -v 显示版本号
    -h 显示本帮助

![](/assets/imgs/000448zcv6s2ewass6y2sr.png)

## 7. top - 经典的Linux任务管理工具
作为一个广为人知的Linux工具，top是大多数的类Unix操作系统任务管理器。它可以显示当前正在运行的进程的列表，用户可以按照不同的条件对该列表进行排序。它主要显示了系统进程对CPU和内存的使用状况。top可以快速检查是哪个或哪几个进程挂起了你的系统。你可以在[这里](https://linux.cn/article-2352-1.html)看到top使用的例子。 你可以在终端输入top来运行它并进入到交互模式：

    交互模式的一些快捷操作:
     
        全局命令: <回车/空格> ?, =, A, B, d, G, h, I, k, q, r, s, W, Z
        统计区的命令: l, m, t, 1
        任务区的命令：
             外观: b, x, y, z 内容: c, f, H, o, S, u 大小: #, i, n 排序: <, >, F, O, R
        色彩方案: <Ret>, a, B, b, H, M, q, S, T, w, z, 0 - 7
        窗口命令:  -, _, =, +, A, a, G, g, w

![](/assets/imgs/000450pqrrixr0u6uqdyx5.png)

## 8. Sysdig - 系统进程的高级视图
Sysdig是一个能够让系统管理员和开发人员以前所未有方式洞察其系统行为的监控工具。其开发团队希望改善系统级的监控方式，通过提供关于存储，进程，网络和内存子系统的统一有序以及粒度可见的方式来进行错误排查，并可以创建系统活动记录文件以便你可以在任何时间轻松分析。

简单例子:

    # sysdig proc.name=vim
    # sysdig -p"%proc.name %fd.name" "evt.type=accept and proc.name!=httpd"
    # sysdig evt.type=chdir and user.name=root
    # sysdig -l
    # sysdig -L
    # sysdig -c topprocs_net
    # sysdig -c fdcount_by fd.sport "evt.type=accept"
    # sysdig -p"%proc.name %fd.name" "evt.type=accept and proc.name!=httpd"
    # sysdig -c topprocs_file
    # sysdig -c fdcount_by proc.name "fd.type=file"
    # sysdig -p "%12user.name %6proc.pid %12proc.name %3fd.num %fd.typechar %fd.name" evt.type=open
    # sysdig -c topprocs_cpu
    # sysdig -c topprocs_cpu evt.cpu=0
    # sysdig -p"%evt.arg.path" "evt.type=chdir and user.name=root"
    # sysdig evt.type=open and fd.name contains /etc

![](/assets/imgs/000452i0j23a421ej2d4ra.jpg)
更多的信息参考：[如何利用sysdig改善系统层次的监控和错误排查](https://linux.cn/article-4341-1.html)。

## 9. netstat - 显示开放的端口和连接
它是Linux管理员使用来显示各种网络信息的工具，如查看什么端口开放和什么网络连接已经建立以及何种进程运行在该连接之上。同时它也显示了不同程序间打开的Unix套接字的信息。作为大多数Linux发行版本的一部分，netstat的许多命令在[netstat和它的不同输出](https://linux.cn/article-2434-1.html)中有详细的描述。最为常用的如下：

    $ netstat | head -20
    $ netstat -r
    $ netstat -rC
    $ netstat -i
    $ netstat -ie
    $ netstat -s
    $ netstat -g
    $ netstat -tapn

## 10. tcpdump - 洞察网络封包
tcpdump可以用来查看网络连接的封包内容。它显示了传输过程中封包内容的各种信息。为了使得输出信息更为有用，它允许使用者通过不同的过滤器获取自己想要的信息。可以参照的例子如下：

    # tcpdump -i eth0 not port 22
    # tcpdump -c 10 -i eth0
    # tcpdump -ni eth0 -c 10 not port 22
    # tcpdump -w aloft.cap -s 0
    # tcpdump -r aloft.cap
    # tcpdump -i eth0 dst port 80

更多的信息可以在[使用topdump捕捉包](https://linoxide.com/linux-how-to/network-traffic-capture-tcp-dump-command/)中找到详细描述。

## 11. [vmstat](https://linux.cn/article-2472-1.html) - 虚拟内存统计信息
vmstat是虚拟内存(virtual memory statistics)的缩写，作为一个内存监控工具，它收集和显示关于内存，进程，终端和分页和I/O阻塞的概括信息。作为一个开源程序，它可以在大部分Linux发行版本中找到，包括Solaris和FreeBSD。它用来诊断大部分的内存性能问题和其他相关问题。

![](/assets/imgs/000453x3hrktig73zt9fsf.png)

## 12. free - 内存统计信息
free是另一个能够在终端中显示内存和交换空间使用的命令行工具。由于它的简易，它经常用于快速查看内存使用或者是应用于不同的脚本和应用程序中。在这里你可以看到[这个小程序的许多应用](https://linux.cn/article-2443-1.html)。几乎所有的系统管理员日常都会用这个工具。:-)

![](/assets/imgs/000455wojyy7crisw3eepe.png)

## 13. Htop - 更加友好的top
Htop基本上是一个top改善版本，它能够以更加多彩的方式显示更多的统计信息，同时允许你采用不同的方式进行排序，它提供了一个用户友好的接口。

![](/assets/imgs/000456g3niaizsslnbx4eb.png)
更多的信息参考我们的文章：“[关于htop和top的比较](https://linux.cn/article-3141-1.html)”。

## 14. ss - 网络管理的现代替代品
ss是iproute2包的一部分。iproute2是用来替代一整套标准的Unix网络工具组件，它曾经用来完成[网络接口配置，路由表和管理ARP表](https://linux.cn/article-4372-1.html)任务。ss工具用来记录套接字统计信息，它可以显示类似netstat一样的信息，同时也能显示更多TCP和状态信息。一些例子如下：

    # ss -tnap
    # ss -tnap6
    # ss -tnap
    # ss -s
    # ss -tn -o state established -p

## 15. lsof - 列表显示打开的文件
lsof命令，意为“list open files”, 用于在许多类Unix系统中显示所有打开的文件及打开它们的进程。在大部分Linux发行版和其他类Linux操作系统中系统管理员用它来检查不同的进程打开了哪些文件。

    # lsof +p process_id
    # lsof | less
    # lsof –u username
    # lsof /etc/passwd
    # lsof –i TCP:ftp
    # lsof –i TCP:80

更多的信息参考我们的文章：[lsof 的使用](https://linux.cn/article-4099-1.html)。

## 16. iftop - 类似top的了网络连接工具
iftop是另一个基于网络信息的类似top的程序。它能够显示当前时刻按照带宽使用量或者上传或者下载量排序的网络连接状况。它同时提供了下载文件的预估完成时间。

![](/assets/imgs/000500naesaqcaussysm2y.png)

更多的信息参考[Linux流量监控工具：iftop](https://linux.cn/article-1843-1.html)。

## 17. iperf - 网络性能工具
iperf是一个网络测试工具，能够创建TCP和UDP数据连接并在网络上测量它们的传输性能。它支持调节关于时间，协议和缓冲等不同的参数。对于每一个测试，它会报告带宽，丢包和其他的一些参数。
![](/assets/imgs/000511acfi354w6d6i5ccw.png)
如果你想用使用这个工具，可以参考这篇文章： [如何安装和使用iperf](https://linoxide.com/monitoring-2/install-iperf-test-network-speed-bandwidth/)。

## 18. [Smem](https://linux.cn/article-4492-1.html) - 高级内存报表工具
Smem是最先进的Linux命令行工具之一，它提供关于系统中已经使用的和共享的实际内存大小，试图提供一个更为可靠的当前内存使用数据。

    $ smem -m
    $ smem -m -p | grep firefox
    $ smem -u -p
    $ smem -w -p

# 图形化或基于Web的性能工具
## 19. Icinga - Nagios的社区分支版本
Icinga是一个开源免费的网络监控程序，作为Nagios的分支，它继承了前者现有的大部分功能，同时基于这些功能又增加了社区用户要求已久的功能和补丁。

![](/assets/imgs/000521idxfbnf76l5nsex5.png)
更多信息请参考[安装和配置lcinga文章](https://linoxide.com/monitoring-2/install-configure-icinga-linux/)。

## 20. Nagios - 最为流行的监控工具
作为在Linux上使用最为广泛和最为流行的监控方案，它有一个守护程序用来收集不同进程和远程主机的信息，这些收集到的信息都通过功能强大的web界面进行呈现。
![](/assets/imgs/000528oelnr1bqraabaa2s.png)
你可以在文章“[如何安装nagios](https://linux.cn/article-2436-1.html)”里面找到更多的信息。

## 21. Linux process explorer - Linux下的procexp
Linux process explorer是一个Linux下的图形化进程浏览工具。它能够显示不同的进程信息，如进程数，TCP/IP连接和每一个进程的性能指标。作为Windows下procexp在Linux的替代品，是由Sysinternals开发的，其目标是比top和ps提供更好用户体验。
![](/assets/imgs/000543w6tja0txxv36boil.png)
查看 [linux process explorer](https://sourceforge.net/projects/procexp/) 的文章获取更多信息。

## 22. Collectl - 性能监控工具
你可以既可以通过交互的方式使用这个性能监控工具，也可以用它把报表写到磁盘上，并通过web服务器来访问。它以一种易读易管理的格式，显示了CPU，磁盘，内存，网络，网络文件系统，进程，slabs等统计信息。
![](/assets/imgs/000544g196111fm1o16n11.png)
更多信息请参看[Collectl的文章](https://linux.cn/article-3154-1.html)。

## 23. MRTG - 经典网络流量监控图形工具
这是一个采用rrdtool的生成图形的流量监控工具。作为最早的提供图形化界面的流量监控工具，它被广泛应用在类Unix的操作系统中。查看我们关于[如何使用MRTG](https://linoxide.com/tools/multi-router-traffic-grapher/)的文章获取更多关于安装和配置的信息。

![](/assets/imgs/000545zmsa6a44pr6axka6.png)

## 24. Monit - 简单易用的监控工具
Monit是一个用来监控进程，系统加载，文件系统和目录文件等的开源的Linux工具。你能够让它自动化维护和修复，也能够在运行错误的情景下执行特定动作或者发邮件报告提醒系统管理员。

![](/assets/imgs/000548kedrqzlo1x8e6qs1.png)
如果你想要用这个工具，你可以查看[如何使用Monit的文章](https://linoxide.com/monitoring-2/monit-linux/)。

## 25. Munin - 为服务器提供监控和提醒服务
作为一个网络资源监控工具，Munin能够帮助分析资源趋势和查看薄弱环节以及导致产生性能问题的原因。开发此软件的团队希望它能够易用和用户体验友好。该软件是用Perl开发的，并采用rrdtool来绘制图形，使用了web界面进行呈现。开发人员推广此应用时声称当前已有500多个监控插件可以“即插即用”。

![](/assets/imgs/000602qsw1fewgu22fo59c.png)
更多信息可以在[关于Munin的文章](https://linoxide.com/ubuntu-how-to/install-munin/)中找到。