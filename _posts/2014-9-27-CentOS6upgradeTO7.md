---
layout: post
title: "CentOS6 升级到 7"
category: study
tags: [study, 运维]
---


CentOS 7 已经发布了，很多情况不允许重装系统，我这里就写一下 CentOS6 升级到 7 的过程以及注意点吧。

# 安装 CentOS 6

由于没有 CentOS 6 的实体机玩，我就装个 VirtualBox 

VirtualBox 配置的机器如下

![CentOS 6 安装](https://ilz.me/assets/imgs/CentOS6TO7_1.png)

安装都是基本默认配置，未做特殊修改，仅仅设置磁盘大小为18GB

CentOS 版本为 CentOS-6.4-x86_64


# 准备升级

首先修改upgrade.repo

    # vim /etc/yum.repos.d/upgrade.repo

加入，注：这里baseurl我在163的源里木有看到有upg目录，so，使用原版的

    [upgrade]
    name=upgrade
    baseurl=https://dev.centos.org/centos/6/upg/x86_64/
    enabled=1
    gpgcheck=0
    
安装升级软件

    # yum -y install preupgrade-assistant-contents redhat-upgrade-tool preupgrade-assistant
    
运行检查命令，以保证升级前所有软件都ok

    # preupg
    
如出现各种包依赖什么关系，请自行 Google ，无法在此对所有情况一一做叙述

好了，现在使用 repo 文件升级,发出以下命令来导入GPG密钥。注：我这里使用的是 163 的源

    # rpm --import https://mirrors.163.com/centos/7.0.1406/os/x86_64/RPM-GPG-KEY-CentOS-7

根据手册页,使用以下命令升级CentOS 6;这将从互联网下载的包。

    # redhat-upgrade-tool --network 7.0 --instrepo https://mirrors.163.com/centos/7.0.1406/os/x86_64/

出来这个结果，看看英文吧，就是说不推荐，存在风险啥啥的
    
    setting up repos...
    No upgrade available for the following repos: base extras updates
    .treeinfo                                                | 1.1 kB     00:00
    preupgrade-assistant risk check found EXTREME risks for this upgrade.
    Run preupg --riskcheck --verbose to view these risks.
    Continuing with this upgrade is not recommended.
   
重新运行redhat-upgrade-tool 加上 --force 选项(不推荐,但这是唯一的解决办法是现在)

    # redhat-upgrade-tool --network 7.0 --force --instrepo https://mirrors.163.com/centos/7.0.1406/os/x86_64/

等他下完一定要看英文！！！有 Finished 让你 reboot 再 reboot ！

真的弄完了？那就reboot吧

    # reboot

至此就升级完毕了
    
# 参考链接

<https://www.itzgeek.com/how-tos/linux/centos-how-tos/upgrade-from-centos-6-to-centos-7.html#axzz3EV4Ux4hv>