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
