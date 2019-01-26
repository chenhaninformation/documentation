# 开发流程

## U-boot

### 编译U-boot

rootfs-extra项目已经制作好一键制作U-boot的工具，开发人员可访问U-boot.md及
rootfs-extra项目文档查阅详细内容。本文档简要介绍编译烧录相关流程。

clone上述rootfs-extra，在rootfs-extra文件下下，运行**make u-boot**即可生成所需
要的U-boot二进制镜像，保存在目录rootfs-extra/build/image/u-boot下。

### 烧录U-boot

因为U-boot与硬件相关，不同硬件需要烧录不同的固件，硬件区别区分在以下三点：

1. 内存型号
2. 内存大小（512MB，1GB，2GB）
3. 内存贴片数量（1CS(Chip Select)，2CS(Chip Select)）
4. 板子是否贴有eMMC Flash

例如：一片贴了两片512MB DDR3内存（共1GB），贴有eMMC Flash的板子，应该使用目录
rootfs-extra/build/image/u-boot/ddr3/1g/2cs/mmc/下面的U-boot镜像。具体烧录方法
可阅读U-boot.md文件查阅详细烧录方法。

## Rootfs

当板子U-boot引导的Linux初始化完成之后，将会运行rootfs中的初始化进程，进而完成
系统级别的初始化。

开发及维护人员可以在rootfs-extra项目下，执行**make rootfs**生成一个基于Ubuntu
修改的rootfs。

最后的rootfs为压缩包，在目录rootfs-extra/build/image/文件夹下。具体文件名为
rootfs.tar.xz。

### 临时的rootfs

由于有些特殊的功能直接设计在rootfs内可能实现起来比较复杂，或基于其他的一些因素
考量，在设计之初，在Linux完成初始化准备运行rootfs初始化之前，新增一个临时的
rootfs，等完成相关工作后，再使用chroot命令进入真正的rootfs执行初始化工作。

这种临时的rootfs使用buildroot项目进行编译生成，具体可访问buildroot项目，内部有
完整的说明文档。

正常系统包使用espressobin\_ramfs\_defconifg配置文件，生成CPIO压缩包（受Linux内
核支持），进入编译内核步骤。

## Linux

### Linux编译

1. 下载源代码：Clone Linux源代码，切换到分支：ch\_espressobin\_dev
2. 指定交叉编译器：export CROSS\_COMPILE=aarch64-linux-gnu-
3. 指定目标机器架构：export ARCH=arm64
4. 使用默认配置：make espressobin\_defconfig
5. 修改默认配置：make menuconfig
6. 在menuconfig页面下找到：General setup --> Initramfs source file(s)配置选项
并填入临时rootfs文件路径（一般是buildroot/output/images/rootfs.cpio.xz）
7. 编译：make
8. 将系统rootfs解压至某文件：tar Jvf /path/to/rootfs.tar.xz
9. 安装内核模块：sudo make modules\_install INSTALL\_MOD\_PATH=/path/to/rootfs
10. 复制内核至根文件系统：sudo cp arch/arm64/boot/Image 
/path/to/rootfs/boot
11. 复制dtb至根文件系统：
sudo cp arch/arm64/boot/dts/marvell/armada-3720-espressobin\*.dtb
/path/to/rootfs/boot
12. 重新打包成.tar.xz：
tar cvO -C ${ROOTFS\_DIR}/ . | xz -z9eT0 > ${IMAGE\_DIR}/rootfs.tar.xz

注意：第8，第12步骤为在其他目录下执行，并非Linux源代码目录，注意区分。

## 系统烧录

### 烧录工具编译

按照以上方式编译内核，至第5步，第6步使用espressobin\_usb\_defconfig配置文件
通过buildroot编译出来的CPIO作为临时的rootfs。

编译完成后根据第10、第11步，保存Image和\*.dtb。

在电脑上，使用U盘并格式化为ext4格式，mount起来在该U盘内新建目录boot，将上述文
件复制至U盘。

最终的系统包rootfs.tar.xz复制至U盘根目录，然后umount，拔出U盘

### 烧录

将U盘插入目标板上，短接reset按钮：MMP\_2\_5，并接上电源，目标板会自动从U盘启动
系统，并解压rootfs.tar.xz至指定的Flash（eMMC或者SD卡），通过串口可以看到有状态
输出，根据指示操作即可。

