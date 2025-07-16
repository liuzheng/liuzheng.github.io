---
title: "Android ROM sign"
description: ""
categorys: 
tags: [Android]
---


# Android 手记
参考：<https://stackoverflow.com/questions/16316526/how-to-sign-android-rom-zip-file>

下载[apk-zip-signing-tools.zip](/assets/soft/apk-zip-signing-tools.zip)或者<https://goo.gl/lRNlE8>

解压后在该文件目录下运行如下命令，当然各种指路径都是可以的

    java -Xmx1024m -jar signapk.jar -w testkey.x509.pem testkey.pk8 my_rom.zip my_signed_rom.zip

## 分离驱动和内核（暂时未验证是否有效）
参考：<https://forum.ubuntu.org.cn/viewtopic.php?t=450939>

### 提取驱动
驱动提取很好办，在/system/vendor/modules里面，那一堆.ko文件就是。另外可能还需要固件，在/system/vendor/firmware里面。

### 提取内核:

工具下载，[本地地址](/assets/soft/tools.tar.gz),<https://dl.linux-sunxi.org/users/arete74/tools.tar.gz>

通过命令 `split_bootimg.pl ../boot.img` 来得到一个boot.img-kernel的文件，将它转换成可以从卡上引导的镜像：

    mkimage -A arm -O linux -T kernel -C none -a 0x40008000 -e 0x40008000 -n "Linux 2.6" -d boot.img-kernel uImage

tip:对于 Mac 来说，`mkimage` 命令可以这样安装`brew install u-boot-tools`

注意：
可能一些驱动在ramdisk里面，分离内核之后有个boot.img-ramdisk.gz的文件，两次解压，就可以看到里面有一些安卓启动的脚本，还有一个/system/目录，这个目录下也可能有驱动，如果有就一并提取出来。

    gunzip boot.img-ramdisk.gz
    7z x boot.img-ramdisk


