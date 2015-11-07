title: 树莓派装系统
date: 2014-09-03 16:45
tags: [Linux, Raspberry-Pi]
categories: Linux
---

最近手上没什么事，正好公司买了几个树莓派有空闲的，于是乎，决定从零开始折腾一下。

1. 从树莓派的官网<http://www.raspberrypi.org/downloads></http:>上下载系统的镜像。

2. 刻录系统

	$ diskutil list

	$ diskutil unmountDisk /dev/disk1

	$ sudo dd bs=1m if=ArchLinuxARM-2014.06-rpi.img of=/dev/disk1

	$ sudo diskutil eject /dev/disk1

3. 接上电源，就自动开机运行了，然后再接上网线，就可以ssh连过去了。


好吧，就这样就可以在上面装上各种软件跑了。。。
