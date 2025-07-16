---
title: "Firecracker 根镜像制作"
tagline: ""
description: ""
categorys: [firecracker]
tags: [firecracker]
---

## AlmaLinux
准备如下`Dockerfile`，编译 **制作用镜像**
```Dockerfile
FROM almalinux:9 
RUN dnf install -y 'dnf-command(download)' e2fsprogs
## Run those commands in docker run --privileged
# file=AlmaLinux
# mkdir images rootfs downloads -p
# truncate -s 1G images/${file}.ext4 
# mkfs.ext4 images/${file}.ext4 
# mount -t ext4 images/${file}.ext4 rootfs
# dnf download --downloadonly --downloaddir downloads/ almalinux-release almalinux-repos almalinux-gpg-keys
# rpm -ivh --nodeps --root `pwd`/rootfs/ downloads/*.rpm 
# dnf --installroot=`pwd`/rootfs/ group install -y 'Minimal Install' --excludepkgs="*-firmware grub2* grubby openssh* "
# rm rootfs/var/cache/dnf/* -fr
# umount rootfs
```
编译完该镜像后使用 `docker run --privileged` 进行镜像制作

一步步按照上述命令运行，其中 `--excludepkgs` 根据实际情况进行精简

## Debain
与AlmaLinux操作一致
```Dockerfile
FROM debian:stable
RUN apt update && \
    apt install -y debootstrap 
## Run those commands in docker run --privileged
# file=Debain
# mkdir images rootfs downloads -p
# truncate -s 1G images/${file}.ext4 
# mkfs.ext4 images/${file}.ext4 
# mount -t ext4 images/${file}.ext4 rootfs
# debootstrap --include apt,netplan.io,vim stable rootfs http://deb.debian.org/debian
# echo 'debain-stable' > rootfs/etc/hostname
# mkdir rootfs/etc/systemd/system/serial-getty@ttyS0.service.d/
# cat <<EOF > rootfs/etc/systemd/system/serial-getty@ttyS0.service.d/autologin.conf
# [Service]
# ExecStart=
# ExecStart=-/sbin/agetty --autologin root -o '-p -- \\u' --keep-baud 115200,38400,9600 %I $TERM
# EOF
# cat <<EOF > rootfs/etc/netplan/99_config.yaml
# network:
#   version: 2
#   renderer: networkd
#   ethernets:
#     eth0:
#       dhcp4: true
# EOF
# chmod 0400 rootfs/etc/netplan/99_config.yaml
# chroot rootfs netplan generate
# chroot rootfs systemctl disable systemd-resolved.service
# chroot rootfs rm /etc/resolv.conf
# chroot rootfs touch /etc/resolv.conf
# rm rootfs/var/cache/apt/* -rf
# umount rootfs
``` 
