---
title: "KVM + VNC (ubuntu)"
description: ""
# category: 
# tags: [KVM]
---


之前有进行过ubuntu server的KVM安装，不过似乎没有记录下来...

# 安装gnome-core

个人一直喜欢GNOME做为桌面系统，在此用vnc的时候也希望默认桌面系统是自己熟悉的，如喜欢其他桌面系统相应安装相关桌面系统的包。

    sudo apt-get install gnome-core

# 安装vnc

vnc安装也是一条命令的事情。。。

    sudo apt-get install vnc4server

至此你就可以使用命令`vncserver`来创建你的密码，一切结束后就可以使用vnc相关连接工具进行使用了，我个人使用的是chrome的插件，https://chrome.google.com/webstore/detail/iabmpiboiopbgfabjmgeedhcmjenhbla

当然这样还是不够的，没有对vnc默认桌面进行配置

    vim .vnc/xstartup

在此我注解掉所有原始配置，添加下文

    unset SESSION_MANAGER
    metacity &
    x-terminal-emulator -geometry 800x600+10+10 -ls -title "$VNCDESKTOP Desktop" &
    gnome-settings-daemon &
    gnome-panel &

# 安装KVM及virt管理软件

    sudo apt-get install kvm qemu
    sudo apt-get install virtinst python-libvirt virt-viewer virt-manager

# 配置桥接网卡

    sudo apt-get install bridge-utils

通过`ifconfig`命令我们可以发现网卡多了一个virbr0，这个是装完KVM后自己生成的虚拟网卡

增加一个虚拟网卡br0，让这个网卡和em1进行桥接

    sudo vim /etc/networks/interfaces

在其后追加如下内容

    auto tap0
    iface tap0 inet manual
    up ifconfig $IFACE 0.0.0.0 up
    down ifconfig $IFACE down
    tunctl_user liuzheng # 这里填你自己的用户名

    auto br0
    iface br0 inet dhcp
    bridge_ports em1 tap0
    #bridge_stp off # 这里注释的原因在参考文献6
    bridge_fd 0
    bridge_maxwait 0

关于网桥的配置我转了，请参考2015-09-21的文章

保存，重起网卡设置

    sudo /sbin/ifup tap0
    sudo /sbin/ifup br0
    sudo /etc/init.d/networking restart

# 新建虚机

    qemu-img create -f qcow2 ubuntu.img 10G

创建虚机

    sudo qemu-system-x86_64 \
         -m 1024 \ # set memery with 1024M
         -smp 2 \ # use 2 CPU Processes , -smp cores=2,threads=1,sockets=1
         -hda ./ubuntu.img \ # choose img file
         -localtime \ # use localtime , if dont it will bring problem
         -clock rtc \ # if you do not use this, winXP will run slow
         -net nic,vlan=0,macaddr=aa-aa-aa-aa-aa-01 \
         -net tap,vlan=0,ifname=tap0,script=no \
         -boot c \ # boot from disk ,if -boot d will boot from cdrom
         # -cdrom /path/xxxx.iso \ # iso file

当然，这边的注释需要去掉，我没找到好办法将参数和注释放一起。。。

# 性能

检测脚本如下，<https://www.chenjie.info/download/bench.sh>，对网络的部分进行了注释

   
    #!/bin/bash
    #written chenjie.com
    
    cname=$(cat /proc/cpuinfo|grep name|head -1|awk '{ $1=$2=$3=""; print }')
    cores=$(cat /proc/cpuinfo|grep MHz|wc -l)
    freq=$(cat /proc/cpuinfo|grep MHz|head -1|awk '{ print $4 }')
    tram=$(free -m | awk 'NR==2'|awk '{ print $2 }')
    swap=$(free -m | awk 'NR==4'| awk '{ print $2 }')
    up=$(uptime|awk '{ $1=$2=$(NF-6)=$(NF-5)=$(NF-4)=$(NF-3)=$(NF-2)=$(NF-1)=$NF=""; print }')
    #cache=$((wget -O /dev/null https://cachefly.cachefly.net/100mb.test) 2>&1 | tail -2 | head -1 | awk '{print $3 $4 }')
    io=$( (dd if=/dev/zero of=test_$$ bs=64k count=16k conv=fdatasync &&rm -f test_$$) 2>&1 | tail -1| awk '{ print $(NF-1) $NF }')
    echo "CPU model : $cname"
    echo "Number of cores : $cores"
    echo "CPU frequency : $freq MHz"
    echo "Total amount of ram : $tram MB"
    echo "Total amount of swap : $swap MB"
    echo "System uptime : $up"
    #echo "Download speed : $cache "
    echo "I/O speed : $io"

结果如下图所示，彩色的是宿主机,IO和CPU frequency 有点纠结，IO居然下降这么多也是醉了，CPU应该是系统二。。。

![](/assets/imgs/2015-09-20-01.png)
![](/assets/imgs/2015-09-20-02.png)

# 参考：

<https://www.havetheknowhow.com/Configure-the-server/Install-VNC.html>

<https://blog.fens.me/vps-kvm/>

<https://wiki.ubuntu.org.cn/Kvm_%E7%BD%91%E7%BB%9C%E6%A1%A5%E6%8E%A5%E6%96%B9%E6%A1%88>

<https://www.linux-kvm.org/page/Networking>

<https://www.linuxdiyf.com/linux/13338.html>

<https://blog.csdn.net/cybertan/article/details/8160102>
