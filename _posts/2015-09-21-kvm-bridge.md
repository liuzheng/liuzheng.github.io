---
layout: post
title: "[转]桥网络配置"
description: ""
category: 
tags: [KVM]
---

转：<https://blog.csdn.net/cybertan/article/details/8160102>


在QEMU/KVM的网络使用中，网桥（bridge）模式可以让客户机和宿主机共享一个物理网络设备连接网络，客户机有自己的独立IP地址，可以直接连接与宿主机一模一样的网络，客户机可以访问外部网络，外部网络也可以直接访问客户机（就像访问普通物理主机一样）。即使宿主机只有一个网卡设备，使用bridge的方式也可知让多个客户机与宿主机共享网络设备，其使用非常方便，其应用也非常广泛。

在qemu-kvm的命令行中，关于bridge模式的网络参数如下：

    -net tap[,vlan=n][,name=str][,fd=h][,ifname=name][,script=file][,downscript=dfile][,helper=helper][,sndbuf=nbytes][,vnet_hdr=on|off][,vhost=on|off][,vhostfd=h][,vhostforce=on|off]

该配置表示连接宿主机的TAP网络接口到n号VLAN中，并使用file和dfile两个脚本在启动客户机时配置网络和在关闭客户机时取消网络配置。

tap参数，表明使用TAP设备。TAP是虚拟网络设备，它仿真了一个数据链路层设备（ISO七层网络结构的第二层），它像以太网的数据帧一样处理第二层数据报。而TUN   与TAP类似，也是一种虚拟网络设备，它是对网络层设备的仿真。TAP被用于创建一个网络桥，而TUN与路由相关。

vlan=n  设置该设备VLAN编号，默认值为0。

name=name  设置名称，在QEMU monior中可能用到，一般由系统自动分配即可。

fd=h  连接到现在已经打开着的TAP接口的文件描述符，一般来说不要设置该选项，而是让QEMU会自动创建一个TAP接口。当使用了fd=h的选项后，ifname、script、downscript、helper、vnet_hdr等选项都不可使用了（不能与fd选项同时出现在命令行中）。

ifname=name  设置在宿主机中添加的TAP虚拟设备的名称（如tap1、tap5等等），不设置这个参数时，QEMU会根据系统中目前的情况，产生一个TAP接口的名称。

script=file  设置宿主机在启动客户机时自动执行的网络配置脚本。如果不指定，其默认值为"/etc/qemu-ifu"”这个脚本，可指定自己的脚本路径以取代默认值；如果不需要执行脚本，则设置为"script=no"。

downscript=dfile  设置宿主机在客户机关闭时自动执行的网络配置脚本。如果不设置，其默认值为"/etc/qemu-ifdown"；若客户机关闭时宿主机不需要执行脚本，则设置为"downscript=no"。

helper=helper  设置启动客户机时在宿主机中运行的辅助程序，包括去建立一个TAP虚拟设备，它的默认值为/usr/local/libexec/qemu-bridge-helper，一般不用自定义，采用默认值即可。

sndbuf=nbytes  限制TAP设备的发送缓冲区大小为n字节，当需要流量进行流量控制时可以设置该选项。其默认值为"sndbuf=0"，即不限制发送缓冲区的大小。

其余几个选项都是与virtio相关的，这里暂不做过多的介绍。



上面介绍了使用TAP设备的一些选项，接下来通过在宿主机中执行如下几个步骤来实现网桥方式的网络配置。

（1）要是用bridge模式的网络配置，首先需要安装两个RPM包，即：bridge-utils和tunctl，它们提供所需的brctl、tunctl命令行工具。可以用yum工具安装它们，如下：

    [root@jay-linux ~]# yum install bridge-utils tunctl

（2）查看tun模块是否加载，如下：

    [root@jay-linux ~]# lsmod | grep tun
    tun                    12197  2

如果tun模块没有加载，则运行"modprobe tun"命令来加载即可；当然，如果已经将tun编译到内核（可查看内核config文件中是否有"CONFIG_TUN=y"选项），则不需要加载了；而如果内核完全没有配置TUN模块，则需要重新编译内核才行了。

（3）检查/dev/net/tun的权限，需要让当前用户拥有可读可写的权限。

    [root@jay-linux ~]# ll /dev/net/tun

crw-rw-rw- 1 root root 10, 200 Jul 20 16:23 /dev/net/tun

（4）建立一个bridge，并将其绑定到一个可以正常工作的网络接口上，并让bridge成为连接本机与外部网络的接口。主要的配置命令如下面命令行所示。

    [root@jay-linux ~]# brctl addbr br0    #添加br0这个bridge

    [root@jay-linux ~]# brctl addbr br0 eth0    #将br0与eth0绑定起来

    [root@jay-linux ~]# brctl stp br0 on     #将br0加入到STP协议中

    [root@jay-linux ~]# dhclient br0    #将br0的网络配置好

    [root@jay-linux ~]# route   #参看路由表是否正常配置
    Kernel IP routing table
    Destination  Gateway         Genmask     Flags Metric Ref    Use Iface
    192.168.0.0  *               255.255.0.0     U     0      0        0 br0
    default    sqa-gate.tsp.or 0.0.0.0         UG    0      0        0 br0

    [root@jay-linux ~]# ping 192.168.199.99 -c 1  #用ping测试网络通畅
    PING 192.168.199.99 (192.168.199.99) 56(84) bytes of data.
    64 bytes from 192.168.199.99: icmp_seq=1 ttl=64 time=4.16 ms
    - 192.168.199.99 ping statistics -
    1 packets transmitted, 1 received, 0% packet loss, time 4ms
    rtt min/avg/max/mdev = 4.164/4.164/4.164/0.000 ms

    [root@jay-linux ~]# dmesg
    <! - ...  ->
    device eth0 entered promiscuous mode
    br0: port 1(eth0) entered forwarding state

建立bridge后的状态是让网络接口eth0进入混杂模式（promiscuous mode，接收网络中所有数据包），网桥br0进入转发状态（forwarding state），而且br0和eth0有相同的MAC地址，一般也会得到和eth0相同的IP。“brctl stp br0 on”是打开br0的STP协议，STP是生成树协议（Spanning Tree Protocol），它主要是为了避免在建有bridge的以太网LAN中出现桥回路（bridge loop）。如果不打开STP，则可能出现回路从而导致建有bridge的主机网络不畅通。

这里默认是通过DHCP方式动态获得IP；在绑定了bridge之后，也可以使用“ifconfig”和“route”等命令进行设置br0的IP、网关、默认路由等，需要将bridge设置为与其绑定的物理网络接口一样的IP和MAC地址，并让默认路由使用bridge（而不是ethX）来连通。这个步骤可能导致宿主机的网络断掉，之后重新通过bridge建立网络连接，所以建立bridge这个步骤最好不要通过SSH连接远程配置。另外，在RHEL系列系统中最好将NetworkManager这个程序结束掉，因为它并不能管理bridge的网络配置，反而它在后台运行则可能对网络设置有些干扰。

（5）准备qemu-ifup和qemu-ifdown脚本。

在客户机启动网络前，会执行的脚本是“script”选项是由配置的（默认为/etc/qemuif-up），一般在该脚本中去创建一个TAP设备并将其与bridge绑定起来。如下是qemu-ifup脚本的示例，其中“$1”是qemu-kvm命令工具传递给脚本的参数，它是客户机使用的TAP设备名称（如tap0、tap1等，也或者是前面提及的ifname选项的值）。另外，其中的“tunctl”命令这一行是不需要的，因为qemu-bridge-helper程序已经会创建好TAP设备，这里列出来只是为了可能在一些版本较旧的qemu-kvm中没有自动创建TAP设备。

    #!/bin/bash
    #This is a qemu-ifup script for bridging.
    #You can use it when starting a KVM guest with bridge mode network.
    #set your bridge name
    switch=br0
    
    if [ -n "$1" ]; then
    
    #create a TAP interface; qemu will handle it automatically.
    #tunctl -u $(whoami) -t $1
    
    #start up the TAP interface
    ip link set $1 up
    sleep 1
    
    #add TAP interface to the bridge
    brctl addif ${switch} $1
    exit 0
    else
    echo "Error: no interface specified"
    exit 1
    fi

由于qemu-kvm工具在客户机关闭时会去解除TAP设备的bridge绑定，也会自动去删除已不再使用的TAP设备，所以qemu-ifdown这个脚本不是必需的，最好设置为“downscript=no”。如下列出一个qemu-ifdown脚本的示例，是为了说明清理bridge模式网络的环境的步骤，在qemu-kvm没有自动处理时可以使用。

    #!/bin/bash
    #This is a qemu-ifdown script for bridging.
    #You can use it when starting a KVM guest with bridge mode network.
    #Don't use this script in most cases; QEMU will handle it automatically.
    
    #set your bridge name
    switch=br0
    if [ -n "$1" ]; then
    # Delete the specified interfacename
    tunctl -d $1
    #release TAP interface from bridge
    brctl delif ${switch} $1
    #shutdown the TAP interface
    ip link set $1 down
    exit 0
    else
    echo "Error: no interface specified"
    exit 1
    fi

（6）用qemu-kvm命令启动bridge模式的网络。

在宿主机中，用命令行启动客户机和检查bridge的状态，如下：

    [root@jay-linux kvm_demo]# qemu-system-x86_64 rhel6u3.img -smp 2 -m 1024 -net nic -net tap,ifname=tap1,script=/etc/qemu-ifup,downscript=no -vnc :0 -daemonize
    
    [root@jay-linux kvm_demo]# brctl show
    bridge name     bridge id               STP enabled     interfaces
    br0             8000.60eb692129b7       no              eth0
    tap1
    
    [root@jay-linux kvm_demo]# ls /sys/devices/virtual/net/
    lo  br0  tap1

由上面信息可知，创建客户机后，添加了一个名为tap1的TAP虚拟网络设备，它被绑定在br0这个bridge上。查看到的三个虚拟网络设备依次为：网络回路设备lo（就是一般IP为127.0.0.1的设备）、前面建立好的bridge设备br0、给客户机提供网络的TAP设备tap1。

在客户机中，如下的几个命令检查网络是否配置好。

    [root@kvm-guest ~]# ifconfig
    eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56
    inet addr:192.168.63.144  Bcast:192.168.255.255  Mask:255.255.0.0
    inet6 addr: fe80::5054:ff:fe12:3456/64 Scope:Link
    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
    RX packets:2893 errors:0 dropped:0 overruns:0 frame:0
    TX packets:102 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:1000
    RX bytes:465871 (454.9 KiB)  TX bytes:18099 (17.6 KiB)
    Interrupt:11
    
    [root@kvm-guest ~]# ping vt-snb9 -c 1
    PING vt-snb9.tsp.org (192.168.199.99) 56(84) bytes of data.
    64 bytes from 192.168.199.99: icmp_seq=1 ttl=64 time=3.51 ms
    - vt-snb9.tsp.org ping statistics -
    1 packets transmitted, 1 received, 0% packet loss, time 4ms
    rtt min/avg/max/mdev = 3.515/3.515/3.515/0.000 ms
    
    [root@kvm-guest ~]# route
    Kernel IP routing table
    Destination  Gateway       Genmask       Flags Metric Ref    Use Iface
    192.168.0.0  *             255.255.0.0     U     0      0        0 eth0
    default    sqa-gate.tsp.org 0.0.0.0         UG    0      0        0 eth0

而将客户机关机后，在宿主机中再次查看bridge状态和虚拟网络设备的状态，如下所示。

    [root@jay-linux kvm_demo]# brctl show
    bridge name     bridge id               STP enabled     interfaces
    br0             8000.60eb692129b7       no              eth0
    
    [root@jay-linux kvm_demo]# ls /sys/devices/virtual/net/
    lo  br0

由上面输出信息可知，qemu-kvm工具已经将tap1设备删除了。

    #!/bin/bash
    #apt-get install uml-utilities
    echo mmm|sudo -S tunctl -t tap0 -u jin
    echo mmm|sudo -S tunctl -t tap1 -u jin
    echo mmm|sudo -S chmod 0666 /dev/net/tun
    echo mmm|sudo -S ifconfig tap0 192.168.17.1 netmask 255.255.255.0 up
    echo mmm|sudo -S ifconfig tap1 192.168.18.1 netmask 255.255.255.0 up
    echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
    echo mmm|sudo -S iptables -t nat -I POSTROUTING -j MASQUERADE

虚拟主机是这样运行的

    echo mmm|sudo \
    kvm \
    -m 512 \
    -drive file=/home/jin/kvm/xp ,cache=writeback,boot=on \
    -net nic,model=virtio \
    -net tap,ifname=tap1,script=no \
    -usbdevice tablet \
    -nographic \
    -usbdevice host:12d1:1009
    #-usbdevice host:148f:3070 \
