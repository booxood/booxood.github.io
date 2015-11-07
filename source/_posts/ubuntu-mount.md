title: Ubuntu 挂载硬盘
date: 2014-09-23 14:21
tags: Linux
categories: Linux
---

在 Ubuntu 上挂载新的硬盘

1. 找到新添加的硬盘

    $ sudo fdisk -l

    Disk /dev/sda: 31.5 GB, 31457280000 bytes
    4 heads, 32 sectors/track, 480000 cylinders, total 61440000 sectors
        Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x0004fdcb

       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048    61439999    30718976   83  Linux

    Disk /dev/sdb: 107.4 GB, 107374182400 bytes
    255 heads, 63 sectors/track, 13054 cylinders, total 209715200 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000

    **Disk /dev/sdb doesn't contain a valid partition table**

2. 分区

	$ sudo fdisk /dev/sdb

	选择：n 添加新分区

	选择：p 选择添加主分区

	选择：1  选择主分区编号

	选择：w 保存退出

3. 格式化

	$ sudo mkfs -t ext3 /dev/sdb1  

	用ext3格式对/dev/sdb1 进行格式化

4. 挂载

	$ sudo mount /dev/sdb1 /dir

	将分区挂载在/dir下

	再设置开机自动挂载

	编辑/etc/fstab添加一行

	/dev/sdc1    /dir     ext3    defaults,    0    1

5. 确认

	最后查看一下是否挂载正确

	$ df -h


	Filesystem      Size  Used Avail Use% Mounted on
	/dev/sda1        29G  5.9G   22G  22% /

	udev            827M  8.0K  827M   1% /dev

	tmpfs           168M  392K  168M   1% /run

	/dev/sdb1        69G   52M   66G   1% /mnt

	**/dev/sdc1        99G   60M   94G   1% /dir**
