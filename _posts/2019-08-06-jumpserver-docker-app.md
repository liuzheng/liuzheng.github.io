---
title: "Jumpserver Docker App"
tagline: ""
description: ""
# category: 
# tags: [Jumpserver]
---

## 方案分析
近期遇到一些客户对于云应用的需求比较强烈，Jumpserver 团队已决定采用 Windows 域应用来实现 远端应用访问的功能，本人并不喜欢，并且也在一年前提出较为革新的技术方案，终未被技术采纳。

对于 Jumpserver 现今所决定使用的技术架构我简单总结一下优劣：

优势：

    1. 基于 Windows 生态，几乎所有工具软件均有支持
    2. 其产品架构由 Windows 背书，稳定且不用折腾

劣势：

    1. 有域账户数限制及 Windows License 限制
    2. 局限于 Windows 上的扩展性
    3. 资源消耗较重

该方案确实是 N 多年以前就成熟的方案，技术团队选择该方案无可厚非，但本人并不喜欢，故而提出容器化应用的方案。该方案的优劣如下：

优势：

    1. 无限制，容器随起随用
    2. 底层基于 Linux Docker 方案，技术稳定性可以保证
    3. 可以自定义各种内容细节
    4. 可结合容器管理平台进行应用的启停

劣势：

    1. 需要自行封装 Docker image
    2. 不一定所有软件都有 Linux 版本

## 方案细节
可以参考如下几个 repo 进行容器封装

* [DBeaver](https://github.com/liuzheng/dbeaver-in-docker)
* [mysql workbench](https://github.com/liuzheng/mysqlworkbench-in-docker)
* [wps](https://github.com/liuzheng/wps-in-docker)
* [term](https://github.com/liuzheng/term-in-docker)
* [firefox](https://github.com/liuzheng/firefox-in-docker)
* [chrome](https://github.com/liuzheng/chrome-in-docker)

这里仅列出撰文时我所封装的容器，如需要看到更多可以访问该链接<https://github.com/liuzheng?utf8=%E2%9C%93&tab=repositories&q=in-docker&type=&language=>

`term` 主要是为了我做容器做测试用的，提供了基本的vnc server和图形窗口、xterm。

其余软件我会按需求进行修改 Dockerfile, 譬如 DBeaver 我将其封装在了 awesome window manager，并设置了关闭重启进程的功能。




最后还是要感谢 Linus，Docker，xvfb, x11vnc, awesome 以及各软件的 Linux 发行版的作者或公司

