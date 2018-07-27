---
title: mac下使用tf卡给树莓派刷系统
categories: 树莓派
tags: 重装系统
---


将tf卡使用mac磁盘管理工具格式化成MS-DOS(FAT) 内存卡名字随便起个英文名

<!-- more -->

<pre>
[#15#zengguang@localhost py]$ df -h        查看内存卡分区
Filesystem      Size   Used  Avail Capacity iused      ifree %iused  Mounted on
/dev/disk1     233Gi  164Gi   68Gi    71% 1051796 4293915483    0%   /
devfs          189Ki  189Ki    0Bi   100%     655          0  100%   /dev
map -hosts       0Bi    0Bi    0Bi   100%       0          0  100%   /net
map auto_home    0Bi    0Bi    0Bi   100%       0          0  100%   /home
/dev/disk2s1    15Gi  2.6Mi   15Gi     1%       0          0  100%   /Volumes/Untitled
[#16#zengguang@localhost py]$ diskutil unmount /dev/disk2s1
Volume (null) on disk2s1 unmounted
[#17#zengguang@localhost py]$ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:          Apple_CoreStorage Macintosh HD            250.1 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3

/dev/disk1 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Macintosh HD           +249.8 GB   disk1
                                 Logical Volume on disk0s2
                                 AAB4B2B1-5C2F-4BAA-B1A3-068EAC889515
                                 Unencrypted

/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *16.1 GB    disk2
   1:                 DOS_FAT_32                         16.1 GB    disk2s1

[#18#zengguang@localhost py]$ sudo dd bs=4m if=2017-08-16-raspbian-stretch.img of=/dev/rdisk2                            ⚠️
Password:
等待几分钟后会出现成功
1170+1 records in
1170+1 records out
4907675648 bytes transferred in 571.325266 secs (8589985 bytes/sec)

diskutil unmountDisk /dev/disk1
得到如下信息，说明卸载成功：

Unmount of all volumes on disk1 was successful
</pre>