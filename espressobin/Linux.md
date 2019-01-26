# Linux

从U-boot进入Linux内核，随后进入用户态运行Ubuntu。但是这个流程不满足现有需求，
需要添加自己的程序去实现相关功能，例如升级，恢复出厂设置。

实现新增程序的方法，本方案采用了Linux的ramfs功能。ramfs使用ram作为根文件系统，
随后使用chroot工具切换到实际的根文件系统中。ramfs的格式为CPIO格式，而使用
buildroot可以生成CPIO格式的ramfs文档。关于ramfs，可访问Linux文档查阅详细内容。

所有需要增加的内容全部添加在buildroot内，需要对内核级别进行调整的，全部写入在
buildroot项目中，并生成CPIO文档作为Linux内核的ramfs。

## 编译

defconfig文件：espressobin\_defconfig

执行完defconfig之后，需要对.config文件进行修改，请执行make menuconfig进入
Kconfig界面，添加CPIO文件，具体位置在：
```
General setup -->
  Initramfs source file(s)
```

在以上config处填入CPIO文件路径，编译之后该CPIO文件就会与内核一起编译生成Image
文件，在linux/arch/arm64/boot/目录下。

## 引导启动

除了Image文件以外，还需要dtb文件进行引导。由于板子内存及flash贴片情况不一，有
多个dtb文件描述板级，都需要烧录到板子flash中。文件涉及多个dtb文件，但他们都有
相同的前缀：armada-3720-espressobin\*.dtb，位于目录
linux/arch/arm64/boot/dts/marvell/下。

U-boot会将上述Image及dtb文件从根文件系统目录/boot下复制至内存中，然后启动Linux
内核。因此，在rootfs生成后，需要将编译好的Image和dtb复制到rootfs /boot文件夹下
才能正常启动Linux kernel。

通过rootfs-extra项目可以生成一个修改过的Ubuntu根文件系统压缩包rootfs.tar.xz，
但是里面没有可以用于启动的Linux内核及其相关dtb，因此，在生成rootfs.tar.xz压缩
包之后，需要将编译好的Linux Image及所有armada-3720-espressobin\*.dtb文件复制到
该压缩包内，并重新打包压缩，才能形成最后我们需要的整个系统文件包。

