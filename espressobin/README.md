Documentation for ESPRESSObin
=============================

About ESPRESSObin
-----------------

ESPRESSObin from GlobalScale equipped with a Marvell Armada 3720 SoC, which
is a dual ARMv8-A A53 cores SoC. More detial please refer to the ESPRESSObin
official site [here][espressobin.net].

Project Requirement Analysis
============================

Software Update
---------------

We design to have software update feature, but due to our heavy workload, we
have to delay this feature to implement it later in the future.

System Reset
------------

Most of the NAS(s) in the market have a small pinhole button in the back of
the machine, which support to erase all user's configurations and reset the
system to manufactory state when push that button with penpoint.

Applications have a lot of configuration files, we can not track each one of
them. The simplest approach is to save two identical system in eMMC flash, we
overlap the current rootfs with reserved rootfs. The overall flash map are
shown in follow:
```
+-------------------------------------------------+
| Boot 1 | Boot 2 | User rootfs | Reserved rootfs |
+-------------------------------------------------+
                         ^               ^
			 |               |
			 '---------------'
		Overlap the User rootfs with Reserved rootfs
```

This approach wasted too much flash space. In router system like OpenWRT, the
ROM space only have 8MB in total, but it still have the ability to support
system reset feature. By reading the documentation of OpenWRT, we have find
that OpenWRT use another different approach by mounting OverlayFS as rootfs to
support system reset.

### OverlayFS

OverlayFS is a filesystem type which merged into Linux kernel mainline before
Linux v4.0. It can merge one RO (Read Only) filesystem like SquashFS (as lower
layer) and one RW (Read Write) filesystem like ext4 (as upper layer) into one
single OverlayFS.

Any write access to an OverlayFS will ONLY effects to upper layer filesystem.
If there is a remove operation to the OverlayFS, the Linux kernel will
generate a dummy file to cover the existed files in lower layer. For more
information about OverlayFS, please refer to Linux kernel documentation,
OverlayFS documentation [here][OverlayFS].

You can mount an overlay using command below.

```
mount -t overlay overlay lower=/lowerdir,upper=/upperdir /overlay_mountpoint
```

Here is some examplesfor OverlayFS.

```
+-----------+--------+--------++-------------+
|   file 1  | file 2 | file 3 ||  OverlayFS  |
+-----------+--------+--------++-------------+
| file 1(b) |        | file 3 || Upper layer |
+-----------+--------+--------++-------------+
| file 1(a) | file 2 |        || Lower layer |
+-----------------------------++-------------+
```

As shown above, lower layer have **file 1** and **file 2**, upper layer have
file **file 1** and **file 3**. After mount the lower layer and upper layer as
an OverlayFS, we can see **file 1**, **file 2** and **file 3** in mounted
OverlayFS.

1. Read **file 1** will read **file 1(b)** in upper layer, **file 1(b)** in
upper layer will hidden **file 1(a)** in lower layer
2. Read **file 2** will read **file 2** in lower layer, since there is no
**file 2** in upper layer
3. Read **file 3** will read **file 3** in upper layer, regardless wether
**file 3** existed in lower layer, the upper layer **file 3** will hidden
the lower layer's **file 3**

4. Write **file 1** will write **file 1(b)** in upper layer, same as read
5. Write **file 2** will copy **file 2** in lower layer to upper layer, and
write to the copied **file 2**, make the original **file 2** in lower layer
untouched
6. Write **file 3** is same as write **file 1**

7. Remove **file 1** will clear **file 1(b)** in upper layer and mark this
file as 'without' file, the OverlayFS will use this empty 'without' file to
hidden **file 1(a)** in lower layer. As result, OverlayFS can not see
**file 1** since the 'without' file is telling the OverlayFS to ignore this
file's existent
8. Remove **file 2** will generate a 'without' file named **file 2** in upper
layer to hidden **file 2** in lower layer, same as remove **file 1**
9. Remove **file 3** will directly remove **file 3** in upper layer

New system reset approach can use this OverlayFS to do our job. The flash map
will like this:
```
+---------------------------------------------+
| Boot 1 | Boot 2 | Lower Layer | Upper Layer |
+---------------------------------------------+
                                       ^
                                       |
                                       |
                Clean all Uper Layer data to support system reset
```

### How mount OverlayFS as rootfs

Normal Linux boot sequence are shown in follow:
```
1. BOOT ROM copy the bootloader(typically U-boot) from boot device to RAM and
switch CPU's PC to point to the entry of bootloader
2. Bootloader is responsable copy the Linux kernel (possible along with .dtb)
from either on board flash or network NFS to RAM and switch CPU's PC to point
to the entry of Linux kernel
3. When Linux kernel start to run, the kernel will mount the rootfs using
bootargs, and entering user space
```

Source code for ESPRESSObin
---------------------------

We have some tested and documented source code for ESPRESSObin 

TODO
====

1. Why we use ESPRESSObin and how we use it.

******

*Copyright (C) 2018, Hunan ChenHan Information Technology Co., Ltd. All rights reserved.*

[espressobin.net]: http://espressobin.net/ "ESPRESSObin"
[OverlayFS]: https://github.com/torvalds/linux/blob/master/Documentation/filesystems/overlayfs.txt "OverlayFS"
