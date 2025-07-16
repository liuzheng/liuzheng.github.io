---
title: "[转]KVM Disk Performance Optimization"
description: ""
categorys: 
tags: [KVM]
---


转: <https://www.pubyun.com/blog/cloud/kvm%E7%A3%81%E7%9B%98%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/>

# KVM磁盘性能优化
* 发表于2013 年 2 月 16 日由refactor *

磁盘性能是虚拟技术中的一个瓶颈，虚拟机由于经过封装以后，磁盘有所下降，尤其要对磁盘性能进行优化。

优化要点：

1、在母机（host）上，设置磁盘调度器为 deadline，有两种方法

– 在启动的时候，加入参数（需要重新启动）：
elevator=deadline

– 或者实时调整参数（不需要重新启动，但是下次启动时丢失）：
for f in /sys/block/sd*/queue/scheduler; do echo “deadline” > $f; done

2、使用 virtio，一定注意，否则导致磁盘性能严重下降

3、在虚拟机（VM）上，设置磁盘调度器为 noop，有两种方法

– 在启动的时候，加入参数（需要重新启动）：
elevator=noop

– 或者实时调整参数（不需要重新启动，但是下次启动时丢失）：
for f in /sys/block/sd*/queue/scheduler; do echo “noop” > $f; done

4、尽量使用 LVM 作为虚拟机的磁盘，qcow2会带来额外的负担，从而导致IO性能下降

5、注意虚拟机内的 virtio驱动程序一定是最新的，特别是windows虚拟机
参考：
https://serverfault.com/questions/360718/kvm-low-io-performance


以下转自：<https://www.hengtianyun.com/download-show-id-11.html>

1.	virtio 

virtio是KVM的半虚拟化机制,用以提高IO性能,使用virtio可以显著提高KVM性能。大部分的linux都已经集成virtio驱动，windows则因没有集成virtio驱动所以需要手动安装。 

2.	使用writeback缓存选项 

针对客户机块设备的缓存,drive有一个子选项cache来设置缓存模式。两个主要的选项为writeback和writethrough,man手册是这样说的 

By default, writethrough caching is used for all block device. This means that the host page cache will be used to read and write data but write notification will be sent to the guest only when the data has been reported as written by the storage subsystem. Writeback caching will report data writes as completed as soon as the data is present in the host page cache. This is safe as long as you trust your host. If your host crashes or loses power, then the guest may experience data corruption. 

writethrough写操作时不使用主机的缓存,只有当主机接受 到存储子系统写操作完成的通知后,主机才通知客户机写操作完成,也就是说这是同步的。而writeback则是异步的,它使用主机的缓存,当客户机写入主机缓存后立刻会被通知写操作完成,而此时主机尚未将数据真正写入存储系统,之后待合适的时机主机会真正的将数据写入存储。显然writeback会更快, 但是可能风险稍大一些,如果主机突然掉电,就会丢失一部分客户机数据。 

这样使用writeback选项 

    -drive file=debian.img,if=virtio,index=0,media=disk,format=qcow2,cache=writeback CDROM设备也可以使用writeback选项 

3. 客户机的磁盘IO调度策略 

磁盘IO要经过调度才可以写入磁盘,这种调度又称作电梯算法。对于客户机对磁盘的IO操作实际上要经过三次IO调度才能真正访问到物理磁盘,客户机对虚拟磁盘执行一次IO调度,KVM主机对所有上层的IO执行一次调度,当KVM主机将IO提交给磁盘阵列时,磁盘阵列也会对IO进行调度,最后才会真正读写物理磁盘。 

客户机看到的磁盘只不过是主机的一个文件,所以其IO调度并无太大意义,反而会影响IO效率,所以可以通过将客户机的IO调度策略设置为NOOP来提高性能。NOOP就是一个FIFO队列,不做IO调度。 

linux客户机使用grub2引导时,可以通过给内核传递一个参数来使用NOOP调度策略 编辑文件/etc/default/grub 

行GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash”后添加elevator=noop,变成为 GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash elevator=noop”

然后 $ sudo update-grub 

4.	打开KSM(Kernel Samepage Merging) 

页共享早已有之,linux中称之为COW(copy on write)。内核2.6.32之后又引入了KSM。KSM特性可以让内核查找内存中完全相同的内存页然后将他们合并,并将合并后的内存页打上COW标记。KSM对KVM环境有很重要的意义,当KVM上运行许多相同系统的客户机时,客户机之间将有许多内存页是完全相同的,特别是只读的内核代码页完全可以 在客户机之间共享,从而减少客户机占用的内存资源,从而可以同时运行更多的客户机。 

Debian系统中KSM默认是关闭的,通过以下命令来开启KSM 

    # echo 1 > /sys/kernel/mm/ksm/run 
    关闭KSM 
    # echo 0 > /sys/kernel/mm/ksm/run 

这样设置后,重新启动系统KSM会恢复到默认状态,尚未找个哪个内核参数可以设置在/etc/sysctl.conf中让KSM持久运行。 

可以在/etc/rc.local中添加 

    echo 1 > /sys/kernel/mm/ksm/run 

让KSM开机自动运行 

通过/sys/kernel/mm/ksm目录下的文件来查看内存页共享的情况,pages_shared文件中记录了KSM已经共享的页面数。 

国人对KSM做了进一步优化,这就是UKSM(Ultra KSM)项目,据说比KSM扫描更全面,页面速度更快,而且CPU占用率更低,希望此项目能尽快进入内核mainline。 

KSM会稍微的影响系统性能,以效率换空间,如果系统的内存很宽裕,则无须开启KSM,如果想尽可能多的并行运行KVM客户机,则可以打开KSM。 

5.	KVM Huge Page Backed Memory 

通过为客户机提供巨页后端内存,减少客户机消耗的内存并提高TLB命中率,从而提升KVM性能。x86 CPU通常使用4K内存页,但也有能力使用更大的内存页,x86_32可以使用4MB内存页，x86_64和x86_32 PAE可以使用2MB内存页。x86使用多级页表结构,一般有三级,页目录表->页表->页,所以通过使用巨页,可以减少页目录表和也表对内存的消耗。当然x86有缺页机制,并不是所有代码、数据页面都会驻留在内存中。 

首先挂装hugetlbfs文件系统 

    #mkdir /hugepages
    #mount -t hugetlbfs hugetlbfs /hugepages 

然后指定巨页需要的内存页面数

    #sysctl vm.nr_hugepages=xxx 

最后指定KVM客户机使用巨页来分配内存 

    kvm -mem-path /hugepages 

也可以让系统开机自动挂载hugetlbfs文件系统,在/etc/fstab中添加 

    hugetlbfs /hugepages hugetlbfs defaults 0 0 

在/etc/sysctl.conf中添加如下参数来持久设定巨页文件系统需要的内存页面数 

    vm.nr_hugepages=xxx 

巨页文件系统需要的页面数可以由客户机需要的内存除以页面大小也就是2M来大体估算。