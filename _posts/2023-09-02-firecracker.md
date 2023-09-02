---
layout: post
title: "Firecracker 学习记录"
tagline: ""
description: ""
category: firecracker
tags: [firecracker]
---

早两年就知道firecracker，去年空的时候尝试过基本的启动实例的操作，体感还行，最近重新回顾，并尝试结合设计新的使用模式。

先简单贴一些firecracker的文档使用摘要，详细参考 [官方文档](https://github.com/firecracker-microvm/firecracker/tree/main/docs)

由于我并不打算使用 api进行一次次curl操作，喜好一个配置文件拉起，所以并不会贴curl的东西（除了停止实例等实例操作）

启动我使用如下json

```
{
  "boot-source": {
    "kernel_image_path": "vmlinux-5.10.186",
    "boot_args": "console=ttyS0 reboot=k panic=1 pci=off",
    "initrd_path": null
  },
  "drives": [
    {
      "drive_id": "rootfs",
      "path_on_host": "ubuntu-22.04.ext4",
      "is_root_device": true,
      "partuuid": null,
      "is_read_only": false,
      "cache_type": "Unsafe",
      "io_engine": "Sync",
      "rate_limiter": null
    }
  ],
  "machine-config": {
    "vcpu_count": 2,
    "mem_size_mib": 1024,
    "smt": false,
    "track_dirty_pages": false
  },
  "cpu-config": null,
  "balloon": null,
  "network-interfaces": [{
    "iface_id": "eth0",
    "host_dev_name": "tap0"
  }],
  "vsock": null,
  "logger": null,
  "metrics": null,
  "mmds-config": null,
  "entropy": null
}
```
以上 kernel image 和 rootfs 均是从官方获取。

## 网络配置：
参考 [官方文档](https://github.com/firecracker-microvm/firecracker/blob/main/docs/network-setup.md)

简单贴一些tap
```
sudo ip tuntap add tap0 mode tap
sudo ip addr add 172.16.0.1/24 dev tap0
sudo ip link set tap0 up
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i tap0 -o eth0 -j ACCEPT
```
在实例中需要执行：
```
ip addr add 172.16.0.2/24 dev eth0
ip link set eth0 up
ip route add default via 172.16.0.1 dev eth0
```
目前研究dhcp的方案，反正装个dhcp服务即可

# 一些操作
## 加载网卡
https://github.com/firecracker-microvm/firecracker/blob/main/docs/api_requests/patch-network-interface.md
```
curl --unix-socket /tmp/firecracker.socket -i \
  -X PUT 'http://localhost/network-interfaces/eth0' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
      "iface_id": "eth0",
      "guest_mac": "AA:FC:00:00:00:01",
      "host_dev_name": "tap0"
    }'
```
## 挂盘
https://github.com/firecracker-microvm/firecracker/blob/main/docs/api_requests/patch-block.md

```bash
curl --unix-socket ${socket} -i \
     -X PUT "http://localhost/drives/dummy" \
     -H "accept: application/json" \
     -H "Content-Type: application/json" \
     -d "{
             \"drive_id\": \"dummy\",
             \"path_on_host\": \"${drive_path}\",
             \"is_root_device\": false,
             \"is_read_only\": false,
             \"cache_type\": \"Writeback\"
         }"
```

## 变更实例大小
```
curl --unix-socket /tmp/firecracker.socket -i  \
  -X PUT 'http://localhost/machine-config' \
  -H 'Accept: application/json'            \
  -H 'Content-Type: application/json'      \
  -d '{
      "vcpu_count": 2,
      "mem_size_mib": 1024
  }'
```

## 扩容rootfs大小
```
truncate -s 5G ubuntu.ext4
resize2fs ubuntu.ext4
```

## 制作镜像
可参考 https://github.com/bkleiner/ubuntu-firecracker

https://github.com/firecracker-microvm/firecracker/blob/main/docs/rootfs-and-kernel-setup.md


